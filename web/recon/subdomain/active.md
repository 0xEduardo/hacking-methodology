# Active Subdomain Enumeration

> **Summary**: Actively probe DNS infrastructure to discover subdomains through brute-force, permutations, zone transfers, and certificate transparency.
> **Goal**: Find subdomains not exposed by passive sources by directly querying DNS resolvers, generating permutations, and testing for misconfigurations.

## Methodology

### When to Use

- After passive subdomain enumeration to fill gaps in coverage
- When passive sources return limited results for the target domain
- To discover internal or development subdomains not indexed anywhere
- When testing for DNS misconfigurations (zone transfers, wildcard DNS)
- To generate permutation-based subdomains from already known names
- Before probing and screenshotting to have the most complete subdomain list

### Step-by-Step

1. **Gather resolvers** -- obtain a fresh list of public DNS resolvers and validate them
2. **DNS brute-force** -- run subdomain wordlists against the target domain using mass DNS resolution
3. **Resolve and validate** -- filter results through trusted resolvers to eliminate false positives
4. **Detect wildcards** -- identify and handle wildcard DNS records that produce false positives
5. **Generate permutations** -- create variations of known subdomains (dev-, staging-, -api, etc.) and resolve them
6. **Test zone transfers (AXFR)** -- attempt zone transfers against all authoritative nameservers
7. **Search Certificate Transparency** -- query CT logs for certificates issued to the domain
8. **Discover virtual hosts** -- fuzz the Host header to find vhosts not resolvable via DNS
9. **Check for subdomain takeover** -- identify dangling DNS records pointing to unclaimed third-party services
10. **Merge and deduplicate** -- combine results from all methods into a single clean list

## Tools & Commands

### DNS Resolver Setup

```bash
# Fresh resolvers -- always use a validated, up-to-date list
# https://github.com/proabiral/Fresh-Resolvers
# https://github.com/trickest/resolvers

# Download fresh resolvers
wget https://raw.githubusercontent.com/trickest/resolvers/main/resolvers-trusted.txt -O resolvers.txt

# Validate resolvers (remove dead/slow ones)
# https://github.com/blechschmidt/massdns
massdns -r resolvers.txt -t A -o S -q known_domain.com | grep -c "NOERROR"

# dnsvalidator -- actively test resolver quality
# https://github.com/vortexau/dnsvalidator
dnsvalidator -tL https://public-dns.info/nameservers.txt -threads 100 -o validated_resolvers.txt
```

### DNS Brute-Force

| Tool | Command | Purpose |
|------|---------|---------|
| puredns | `puredns bruteforce <WORDLIST> <DOMAIN> -r resolvers.txt` | Fast brute-force with wildcard detection |
| shuffledns | `shuffledns -d <DOMAIN> -w <WORDLIST> -r resolvers.txt` | MassDNS wrapper for brute-force |
| amass | `amass enum -brute -d <DOMAIN> -rf resolvers.txt` | Full-featured enum with brute-force |
| dnsx | `cat <WORDLIST> \| sed "s/$/.DOMAIN/" \| dnsx -silent` | Lightweight DNS resolution |
| gobuster | `gobuster dns -d <DOMAIN> -w <WORDLIST> -t 50` | DNS brute-force mode |

```bash
# puredns -- brute-force with automatic wildcard filtering
# https://github.com/d3mondev/puredns
puredns bruteforce /usr/share/seclists/Discovery/DNS/subdomains-top1million-110000.txt <DOMAIN> \
  -r resolvers.txt --write brute_results.txt

# shuffledns -- high-speed brute-force
# https://github.com/projectdiscovery/shuffledns
shuffledns -d <DOMAIN> -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-110000.txt \
  -r resolvers.txt -o shuffled_results.txt

# amass -- brute-force with intel gathering
amass enum -brute -d <DOMAIN> -rf resolvers.txt -o amass_brute.txt

# dnsenum -- classic DNS enumeration with brute-force
dnsenum --dnsserver <DNS_IP> --enum -p 0 -s 0 -o subdomains.txt \
  -f /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt <DOMAIN>

# dnsrecon -- brute-force with multiple record types
dnsrecon -D /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt -d <DOMAIN> -n <DNS_IP>
```

### DNS Resolution & Validation

```bash
# puredns -- resolve a list of discovered subdomains
puredns resolve subdomains.txt -r resolvers.txt --write resolved.txt

# dnsx -- resolve with multiple record types
cat subdomains.txt | dnsx -silent -a -aaaa -cname -resp -o resolved_details.txt

# dnsx -- filter by response code
cat subdomains.txt | dnsx -silent -rcode noerror -o valid_subdomains.txt

# massdns -- high-speed resolution
massdns -r resolvers.txt -t A -o S subdomains.txt > massdns_results.txt
cat massdns_results.txt | awk '{print $1}' | sed 's/\.$//' | sort -u > resolved.txt
```

### Wildcard Detection & Filtering

```bash
# puredns handles wildcard filtering automatically during brute-force and resolve

# Manual wildcard check -- resolve a random nonexistent subdomain
dig A randomnonexistent12345.<DOMAIN> @8.8.8.8
# If it resolves, the domain has wildcard DNS

# shuffledns with wildcard filtering (built-in)
shuffledns -d <DOMAIN> -w <WORDLIST> -r resolvers.txt -strict-wildcard

# Filter wildcard results by comparing response IPs
# If many subdomains resolve to the same IP, they are likely wildcard matches
cat resolved.txt | dnsx -silent -a -resp | sort -t' ' -k2 | uniq -f1 -D
```

### Permutation & Alteration

| Tool | Command | Purpose |
|------|---------|---------|
| gotator | `gotator -sub known.txt -perm perms.txt -depth 1 -numbers 10 -md` | Subdomain permutation generator |
| altdns | `altdns -i known.txt -o permuted.txt -w words.txt` | Alterations and permutations |
| dnsgen | `cat known.txt \| dnsgen - \| sort -u > permuted.txt` | Generate domain permutations |
| regulator | `python3 regulator.py -t <DOMAIN> -f known.txt` | Smart subdomain permutations |

```bash
# gotator -- generate permutations from known subdomains
# https://github.com/Josue87/gotator
gotator -sub known_subdomains.txt -perm /usr/share/seclists/Discovery/DNS/dns-prefixes.txt \
  -depth 1 -numbers 10 -mindup -adv -md > permuted.txt

# Resolve permuted results
puredns resolve permuted.txt -r resolvers.txt --write new_subdomains.txt

# altdns -- generate and resolve alterations
# https://github.com/infosec-au/altdns
altdns -i known_subdomains.txt -o permuted.txt -w words.txt
puredns resolve permuted.txt -r resolvers.txt --write altdns_resolved.txt

# dnsgen -- pipe-friendly permutation generator
# https://github.com/ProjectAnte/dnsgen
cat known_subdomains.txt | dnsgen - | puredns resolve -r resolvers.txt --write dnsgen_resolved.txt

# Common permutation words to include in wordlists:
# dev, staging, stage, test, qa, uat, prod, api, admin, internal, corp,
# vpn, mail, mx, ns, web, app, portal, dashboard, monitor, grafana,
# jenkins, gitlab, jira, confluence, wiki, docs, cdn, static, media
```

### DNS Zone Transfer (AXFR)

```bash
# Identify authoritative nameservers
dig NS <DOMAIN>

# Attempt zone transfer against each nameserver
dig axfr <DOMAIN> @<NS1>
dig axfr <DOMAIN> @<NS2>

# Automated zone transfer with fierce
fierce --domain <DOMAIN> --dns-servers <DNS_IP>

# dnsrecon zone transfer
dnsrecon -d <DOMAIN> -a -n <DNS_IP>

# nmap NSE script
nmap -p 53 --script dns-zone-transfer --script-args "dns-zone-transfer.domain=<DOMAIN>" <DNS_IP>

# Attempt without specifying domain (may reveal other zones)
dig axfr @<DNS_IP>
```

### Certificate Transparency Logs

```bash
# crt.sh -- query CT logs via API
curl -s "https://crt.sh/?q=%25.<DOMAIN>&output=json" | jq -r '.[].name_value' | sort -u

# Filter and clean results
curl -s "https://crt.sh/?q=%25.<DOMAIN>&output=json" | jq -r '.[].name_value' | \
  sed 's/\*\.//g' | sort -u > ct_subdomains.txt

# ctfr -- dedicated CT log searcher
# https://github.com/UnaPibaGeek/ctfr
python3 ctfr.py -d <DOMAIN> -o ct_results.txt

# Subfinder with CT sources
subfinder -d <DOMAIN> -sources crtsh -o subfinder_ct.txt

# Google Transparency Report
# Manual: https://transparencyreport.google.com/https/certificates

# certspotter API
curl -s "https://api.certspotter.com/v1/issuances?domain=<DOMAIN>&include_subdomains=true" | \
  jq -r '.[].dns_names[]' | sort -u
```

### Virtual Host Discovery

```bash
# ffuf -- vhost brute-force via Host header
ffuf -u http://<TARGET_IP> -H "Host: FUZZ.<DOMAIN>" \
  -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt -ac -c

# gobuster vhost mode
gobuster vhost -u http://<TARGET_IP> -w <WORDLIST> --domain <DOMAIN> --append-domain

# wfuzz vhost discovery
wfuzz -c -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-20000.txt \
  --hc 400,404,403 -H "Host: FUZZ.<DOMAIN>" http://<TARGET_IP>

# Virtual host discovery over HTTPS
ffuf -u https://<TARGET_IP> -H "Host: FUZZ.<DOMAIN>" \
  -w <WORDLIST> -ac -c -k
```

### Subdomain Takeover Checking

```bash
# subjack -- check for subdomain takeover
# https://github.com/haccer/subjack
subjack -w subdomains.txt -t 100 -timeout 30 -ssl -c fingerprints.json -v

# subzy -- faster takeover checker
# https://github.com/PentestPad/subzy
subzy run --targets subdomains.txt

# nuclei -- takeover templates
nuclei -l subdomains.txt -tags takeover

# can-i-take-over-xyz -- reference for takeover fingerprints
# https://github.com/EdOverflow/can-i-take-over-xyz

# Manual check -- look for dangling CNAME records
cat subdomains.txt | dnsx -cname -resp | grep -v "NXDOMAIN"
# If CNAME points to an unclaimed service (S3, GitHub Pages, Heroku, etc.), takeover may be possible
```

See also: [Subdomain Takeover](../../exploitation/vulns/subdomain-takeover.md)

### DNS Record Enumeration

```bash
# Gather all record types for discovered subdomains
dig ANY @<DNS_IP> <DOMAIN>
dig A @<DNS_IP> <SUBDOMAIN>
dig AAAA @<DNS_IP> <SUBDOMAIN>
dig CNAME @<DNS_IP> <SUBDOMAIN>
dig MX @<DNS_IP> <DOMAIN>
dig TXT @<DNS_IP> <DOMAIN>
dig NS @<DNS_IP> <DOMAIN>
dig SOA @<DNS_IP> <DOMAIN>

# Reverse DNS for discovered IPs
dig -x <IP_ADDRESS> @<DNS_IP>

# DNSSEC enumeration (zone walking)
nmap -sSU -p53 --script dns-nsec-enum --script-args "dns-nsec-enum.domains=<DOMAIN>" <DNS_IP>

# SRV records for service discovery
dig SRV _http._tcp.<DOMAIN>
dig SRV _https._tcp.<DOMAIN>
dig SRV _sip._tcp.<DOMAIN>
```

## Recommended Wordlists

| Wordlist | Size | Use Case |
|----------|------|----------|
| `subdomains-top1million-5000.txt` | Small | Quick initial scan |
| `subdomains-top1million-20000.txt` | Medium | Standard brute-force |
| `subdomains-top1million-110000.txt` | Large | Thorough brute-force |
| `dns-prefixes.txt` | Varies | Permutation generation |
| `n0kovo_subdomains_huge.txt` | Huge | Deep enumeration |
| Assetnote `best-dns-wordlist.txt` | Large | High-quality curated list |

All available from [SecLists/Discovery/DNS/](https://github.com/danielmiessler/SecLists/tree/master/Discovery/DNS).

## Tips & Edge Cases

- **Resolver quality matters** -- stale or slow resolvers produce false negatives; always validate your resolver list before large scans
- **Wildcard domains** -- always check for wildcards first; a wildcard `*.domain.com` resolving to a single IP will make all brute-force results appear valid
- **Permutation depth** -- start with depth 1; deeper permutations (dev-api-staging.domain.com) increase results but also increase noise and scan time
- **Recursive brute-force** -- after finding `sub.domain.com`, brute-force `FUZZ.sub.domain.com` for nested subdomains
- **Rate limiting** -- some authoritative DNS servers rate-limit queries; spread queries across multiple resolvers and throttle if needed
- **Internal subdomains** -- subdomains resolving to RFC 1918 addresses (10.x, 172.16-31.x, 192.168.x) may reveal internal network structure
- **Cloud metadata** -- subdomains pointing to cloud IPs (AWS, Azure, GCP) may expose metadata endpoints or misconfigured services
- **Zone transfer success** -- if AXFR works, it reveals the entire zone; this is a significant finding and should be reported
- **Merge passive + active** -- always combine passive enumeration results with active brute-force before running permutations for maximum coverage
- **Continuous monitoring** -- subdomain landscapes change; re-run enumeration periodically during long engagements

## Output Format

Final subdomain list should be deduplicated and formatted for handoff to probing and screenshotting:

```
[SUBDOMAIN]                    [IP]            [CNAME]              [STATUS]
api.target.com                 10.0.1.50       -                    alive
staging.target.com             52.1.2.3        staging.s3.aws.com   alive
old.target.com                 -               old.heroku.com       takeover-candidate
dev-portal.target.com          192.168.1.10    -                    internal
```

## Agent Workflow
> Step-by-step instructions for an AI agent to perform active subdomain enumeration.

### Phase 1: Setup
1. Download and validate a fresh DNS resolver list:
   ```
   wget https://raw.githubusercontent.com/trickest/resolvers/main/resolvers-trusted.txt -O resolvers.txt
   ```
2. Select brute-force wordlist based on desired depth:
   - Quick scan: `subdomains-top1million-5000.txt`
   - Standard: `subdomains-top1million-20000.txt`
   - Thorough: `subdomains-top1million-110000.txt`
3. Gather known subdomains from [passive enumeration](passive.md) as seed for permutations
4. Check target domain for wildcard DNS before brute-forcing:
   ```
   dig A randomnonexistent12345.<DOMAIN> @8.8.8.8
   ```

### Phase 2: Execution
1. Run DNS brute-force with wildcard filtering:
   ```
   puredns bruteforce <WORDLIST> <DOMAIN> -r resolvers.txt --write brute_results.txt
   ```
2. Resolve and validate results through trusted resolvers:
   ```
   puredns resolve brute_results.txt -r resolvers.txt --write resolved.txt
   ```
3. Generate permutations from known subdomains and resolve:
   ```
   gotator -sub known_subdomains.txt -perm /usr/share/seclists/Discovery/DNS/dns-prefixes.txt -depth 1 -numbers 10 -mindup -md > permuted.txt
   puredns resolve permuted.txt -r resolvers.txt --write permutation_results.txt
   ```
4. Attempt DNS zone transfers on all authoritative nameservers:
   ```
   dig NS <DOMAIN>
   dig axfr <DOMAIN> @<NS1>
   dig axfr <DOMAIN> @<NS2>
   ```
5. Query certificate transparency logs:
   ```
   curl -s "https://crt.sh/?q=%25.<DOMAIN>&output=json" | jq -r '.[].name_value' | sed 's/\*\.//g' | sort -u > ct_subdomains.txt
   ```
6. Check for subdomain takeover on all discovered subdomains:
   ```
   subzy run --targets all_subdomains.txt
   nuclei -l all_subdomains.txt -tags takeover
   ```
7. Fuzz virtual hosts on target IPs:
   ```
   ffuf -u http://<TARGET_IP> -H "Host: FUZZ.<DOMAIN>" -w subdomains-top1million-5000.txt -ac
   ```

### Phase 3: Analysis
1. Merge all results and deduplicate:
   ```
   cat brute_results.txt permutation_results.txt ct_subdomains.txt | sort -u > all_subdomains.txt
   ```
2. Resolve all subdomains with multiple record types:
   ```
   cat all_subdomains.txt | dnsx -silent -a -aaaa -cname -resp -o resolved_details.txt
   ```
3. Identify interesting patterns:
   - Subdomains resolving to RFC 1918 addresses (internal network)
   - CNAME records pointing to cloud services (S3, Azure, Heroku)
   - Dangling CNAMEs (subdomain takeover candidates)
4. Flag zone transfer success as a significant finding

### Phase 4: Next Steps
- Feed all discovered subdomains to [probing](../probing.md) for alive detection
- Investigate subdomain takeover candidates from dangling CNAMEs
- Feed internal-resolving subdomains to network mapping
- Feed cloud-pointed subdomains to [cloud testing](../../exploitation/cloud/)
- Run recursive brute-force on interesting subdomains (`FUZZ.sub.domain.com`)

## Decision Tree

```
START: Target domain + known subdomains from passive enum
  |
  +--> Check for wildcard DNS
  |      |
  |      +--> Wildcard detected? --> Use puredns (auto-filters wildcards)
  |
  +--> 1. DNS brute-force (puredns/shuffledns)
  |
  +--> 2. Permutation generation (gotator/altdns/dnsgen) + resolve
  |
  +--> 3. Zone transfer attempts (dig axfr on all nameservers)
  |      |
  |      +--> Zone transfer succeeded? --> Report finding + parse all records
  |
  +--> 4. Certificate transparency log query (crt.sh)
  |
  +--> 5. Virtual host fuzzing (ffuf Host header)
  |
  +--> 6. Subdomain takeover check (subzy/nuclei)
  |      |
  |      +--> Takeover candidate? --> Attempt takeover or report
  |
  +--> Merge + deduplicate --> Feed to probing
```

## Success Criteria

- [ ] DNS resolver list validated and fresh
- [ ] Wildcard DNS checked before brute-forcing
- [ ] DNS brute-force completed with appropriate wordlist
- [ ] Permutation-based enumeration completed on known subdomains
- [ ] Zone transfer attempted on all authoritative nameservers
- [ ] Certificate transparency logs queried
- [ ] Virtual host fuzzing completed
- [ ] Subdomain takeover check completed on all results
- [ ] All results merged, deduplicated, and resolved
- [ ] Findings handed off to probing phase

## References

- [puredns](https://github.com/d3mondev/puredns)
- [shuffledns](https://github.com/projectdiscovery/shuffledns)
- [amass](https://github.com/owasp-amass/amass)
- [dnsx](https://github.com/projectdiscovery/dnsx)
- [massdns](https://github.com/blechschmidt/massdns)
- [gotator](https://github.com/Josue87/gotator)
- [altdns](https://github.com/infosec-au/altdns)
- [dnsgen](https://github.com/ProjectAnte/dnsgen)
- [dnsvalidator](https://github.com/vortexau/dnsvalidator)
- [fierce](https://github.com/mschwager/fierce)
- [dnsrecon](https://github.com/darkoperator/dnsrecon)
- [dnsenum](https://github.com/fwaeytens/dnsenum)
- [subjack](https://github.com/haccer/subjack)
- [subzy](https://github.com/PentestPad/subzy)
- [can-i-take-over-xyz](https://github.com/EdOverflow/can-i-take-over-xyz)
- [Fresh-Resolvers](https://github.com/proabiral/Fresh-Resolvers)
- [trickest resolvers](https://github.com/trickest/resolvers)
- [SecLists DNS](https://github.com/danielmiessler/SecLists/tree/master/Discovery/DNS)

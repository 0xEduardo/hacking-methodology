# Probing

> **Summary**: Validates a list of discovered hosts/URLs to determine which are alive, what technologies they run, and how they respond.
> **Goal**: Filter live targets from large subdomain lists, fingerprint web servers, detect redirects, and identify unique/interesting pages for further testing.

## Methodology

### When to Use

- After subdomain enumeration to filter dead hosts from live ones
- Before fuzzing or vulnerability scanning to build a clean target list
- When you need to fingerprint technologies, detect WAFs, or map redirect chains
- To compare content-length across hosts and spot unique or non-default pages

### Step-by-Step

1. **Prepare input** -- gather all discovered subdomains and URLs into a single file, one per line
   - Success criteria: a clean `hosts.txt` with no duplicates
2. **Run httpx for alive detection and fingerprinting**
   - Feed the list into httpx with technology detection, status codes, and content-length
   - Success criteria: output file with live hosts, their status codes, titles, and tech stack
3. **Analyze redirects** -- identify redirect chains that may reveal internal hostnames or environments
   - Success criteria: list of redirect destinations mapped
4. **Compare content-length** -- hosts with unique content-length values often serve non-default pages worth investigating
   - Success criteria: short list of targets with unusual response sizes
5. **Screenshot interesting targets** -- pipe unique results to a screenshotter for visual triage (see [screenshot.md](screenshot.md))
6. **Filter and prioritize** -- sort results by interesting status codes (200, 301, 302, 401, 403, 500) and technologies

## Tools & Commands

| Tool | Command | Purpose |
|------|---------|---------|
| httpx | `cat hosts.txt \| httpx -random-agent -retries 2 -o alive.txt` | Basic alive check |
| httpx | `cat hosts.txt \| httpx -sc -cl -title -tech-detect -server -o detailed.txt` | Full fingerprinting with status code, content-length, title, tech, server |
| httpx | `cat hosts.txt \| httpx -follow-redirects -location -o redirects.txt` | Map redirect chains and final destinations |
| httpx | `cat hosts.txt \| httpx -sc -cl -fc 404 -o filtered.txt` | Filter out 404s for cleaner output |
| httpx | `cat hosts.txt \| httpx -favicon -jarm -o fingerprint.txt` | Favicon hash and JARM fingerprinting |
| httpx | `cat hosts.txt \| httpx -ss -srd screenshots/` | Take screenshots of alive hosts |
| httpx | `cat hosts.txt \| httpx -json -o results.json` | JSON output for programmatic processing |
| httpx | `cat hosts.txt \| httpx -p 80,443,8080,8443,8000,3000,9090 -o multiport.txt` | Probe multiple ports per host |
| httprobe | `cat hosts.txt \| httprobe -c 50 -t 3000 > alive.txt` | Fast alive check (legacy alternative) |
| httprobe | `cat hosts.txt \| httprobe -p http:8080 -p https:8443` | Probe custom ports |

### httpx Key Flags

```bash
# Full recon probe
cat hosts.txt | httpx \
  -random-agent \
  -retries 2 \
  -sc \                   # Status code
  -cl \                   # Content-length
  -title \                # Page title
  -tech-detect \          # Technology detection (Wappalyzer)
  -server \               # Server header
  -favicon \              # Favicon hash (pivot in Shodan)
  -jarm \                 # JARM TLS fingerprint
  -cdn \                  # Detect CDN
  -follow-redirects \     # Follow redirect chains
  -o recon_results.txt

# Useful filtering flags
-fc 404,502              # Filter out specific status codes
-mc 200,301,302,401,403  # Match only specific status codes
-ms <SIZE>               # Match specific content-length
-ml <LINES>              # Match specific line count
```

### Content-Length Comparison

```bash
# Find unique pages by sorting on content-length
cat hosts.txt | httpx -sc -cl -title | sort -t' ' -k3 -n | uniq -f2

# Extract hosts with unusual content-length (not default pages)
cat hosts.txt | httpx -cl -silent | awk '{print $2, $1}' | sort -n | uniq -c | sort -n
```

### Redirect Chain Analysis

```bash
# Map full redirect chains
cat hosts.txt | httpx -follow-redirects -location -sc -title

# Find hosts redirecting to login pages (potential auth targets)
cat hosts.txt | httpx -follow-redirects -location | grep -i "login\|signin\|auth\|sso"
```

### Multi-Port Probing

```bash
# Probe common web ports
cat hosts.txt | httpx -p 80,443,8080,8443,8000,8888,3000,5000,9090,9443 -sc -title

# Probe all ports from nmap output
nmap -iL hosts.txt -p- --open -oG - | awk '/open/{print $2}' | httpx -sc -title
```

## Tips & Edge Cases

- Always use `-random-agent` to avoid being blocked by WAFs that filter default Go user agents
- Use `-retries 2` or higher for unreliable networks; some hosts may fail on first request
- Pipe httpx output through `anew` to track new discoveries across multiple runs: `cat hosts.txt | httpx -silent | anew alive_master.txt`
- JARM fingerprinting helps group hosts by TLS stack -- same JARM often means same infrastructure
- Favicon hash can be searched in Shodan to find related infrastructure: `http.favicon.hash:<HASH>`
- When dealing with large lists (100k+ hosts), use `-rate-limit 150` and `-threads 50` to avoid overwhelming your network
- Some targets respond differently based on the Host header -- compare results with and without `-follow-host-redirects`
- Content-length of 0 or very small values often indicates empty pages, WAF blocks, or misconfigured hosts
- Status code 403 does not mean "nothing here" -- it often hides content worth fuzzing

## Output Format

```
https://app.target.com [200] [Dashboard - Target Corp] [nginx] [Content-Length: 48392]
https://api.target.com [301] [] [cloudflare] [Content-Length: 0] -> https://api.target.com/v2
https://staging.target.com [401] [401 Authorization Required] [nginx/1.18.0]
https://admin.target.com [403] [403 Forbidden] [Apache/2.4.41]
```

Key fields to capture for each host:
- URL (scheme + hostname + port)
- Status code
- Page title
- Server header
- Content-length
- Technology stack
- Redirect destination (if applicable)

## References

- [httpx - ProjectDiscovery](https://github.com/projectdiscovery/httpx)
- [httprobe - tomnomnom](https://github.com/tomnomnom/httprobe)
- [JARM Fingerprinting](https://github.com/salesforce/jarm)
- [Favicon Hash Hunting with Shodan](https://medium.com/@Asm0d3us/weaponizing-favicon-ico-for-bugbounties-osint-and-what-not-ace3c214e139)

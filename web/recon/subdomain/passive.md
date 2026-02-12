# Passive Resources

A passive resource means that you will grab subdomains that were already discovered by another tools or were found lying around in some place (open source code, legacy scripts, logs, etc).

```bash
# https://github.com/OWASP/Amass
amass enum -passive -d domain.com

# https://github.com/projectdiscovery/subfinder
subfinder -d domain.com -all -silent

# https://github.com/tomnomnom/assetfinder
assetfinder --subs-only example.com

# https://github.com/Findomain/Findomain
findomain -u example.com -q

# https://github.com/lc/gau
# https://github.com/tomnomnom/unfurl
gau --subs example.com | unfurl -u domains
```

## bbot

```
pipx install bbot

bbot -t evilcorp.com -p subdomain-enum -rf passive
```

## All in one script

```bash
amass enum -passive -d $1 -o amass_$1.txt > /dev/null 2>&1
echo "[+] Amass done"

subfinder -silent -d $1 -all -o subfinder_$1.txt > /dev/null 2>&1
echo "[+] Subfinder done"

findomain -t $1 -u findomain_$1.txt > /dev/null 2>&1
echo "[+] Findomain done"

assetfinder -subs-only $1 > assetfinder_$1.txt
echo "[+] Assetfinder done"

cat subfinder_$1.txt findomain_$1.txt assetfinder_$1.txt amass_$1.txt | uniq $1_subdomains
rm subfinder_$1.txt findomain_$1.txt assetfinder_$1.txt amass_$1.txt
```

## Google analytics ID

Extract Google tag manager from webiste, format: `UAXXXXXX`

- [https://builtwith.com/relationships/tag/UAXXXXXX](https://builtwith.com/relationships/tag/UAXXXXXX)
- [https://api.hackertarget.com/analyticslookup/?q=UAXXXXXX](https://api.hackertarget.com/analyticslookup/?q=UAXXXXXX)
- [https://spyonweb.com/UAXXXXXX](https://spyonweb.com/UAXXXXXX)

```
# https://github.com/Josue87/AnalyticsRelationships
cat subdomains.txt | analyticsrelationships
```

## Google Tag Manager

```
https://googletagmanager.com/gtm.js?id=TAGID
```

## Favicon

Search for favicon md5 value in known search engines to find websites related to your search

```
Tip: Use http.favicon.hash:<hash> on Shodan
```

See more shodan dorks here: [https://github.com/dootss/shodan-dorks](https://github.com/dootss/shodan-dorks)

## Agent Workflow
> Step-by-step instructions for an AI agent to perform passive subdomain enumeration.

### Phase 1: Setup
1. Configure API keys for data sources that require them:
   - Shodan, SecurityTrails, Censys, VirusTotal, Chaos, PassiveTotal
   - Store in `~/.config/subfinder/provider-config.yaml` for subfinder
   - Store in `~/.config/amass/config.ini` for amass
2. Identify the target domain(s) and confirm scope boundaries
3. Prepare an output directory for results from each tool

### Phase 2: Execution
1. Run multiple passive enumeration tools in parallel:
   ```
   subfinder -d <DOMAIN> -all -silent -o subfinder_results.txt
   amass enum -passive -d <DOMAIN> -o amass_results.txt
   assetfinder --subs-only <DOMAIN> > assetfinder_results.txt
   findomain -t <DOMAIN> -u findomain_results.txt -q
   ```
2. Collect URLs from archive sources for additional subdomain extraction:
   ```
   gau --subs <DOMAIN> | unfurl -u domains > gau_domains.txt
   ```
3. Run bbot for comprehensive passive enumeration:
   ```
   bbot -t <DOMAIN> -p subdomain-enum -rf passive
   ```
4. Check Google Analytics relationships:
   ```
   cat subdomains.txt | analyticsrelationships
   ```
5. Search favicon hashes in Shodan: `http.favicon.hash:<HASH>`

### Phase 3: Analysis
1. Merge and deduplicate all results:
   ```
   cat subfinder_results.txt amass_results.txt assetfinder_results.txt findomain_results.txt gau_domains.txt | sort -u > all_passive_subs.txt
   ```
2. Identify patterns in subdomain naming (dev-, staging-, test-, api-, internal-)
3. Count total unique subdomains discovered
4. Note which sources contributed the most unique findings

### Phase 4: Next Steps
- Feed deduplicated list to [active subdomain enumeration](active.md) for brute-force and permutations
- Feed deduplicated list to [probing](../probing.md) for alive detection
- Investigate interesting subdomain names for potential internal/staging services
- Use Google Analytics IDs and favicon hashes to discover related infrastructure

## Decision Tree

```
START: Target domain confirmed in scope
  |
  +--> Run subfinder + amass + assetfinder + findomain (in parallel)
  |
  +--> Run gau for archive-based subdomain discovery
  |
  +--> Check Google Analytics relationships
  |
  +--> Search favicon hashes in Shodan
  |
  +--> Merge + deduplicate all results
  |
  +--> Feed to: Active enumeration | Probing
```

## Success Criteria

- [ ] At least 3 passive enumeration tools run with API keys configured
- [ ] Archive sources queried (gau, waymore)
- [ ] Google Analytics and favicon-based pivoting attempted
- [ ] All results merged and deduplicated
- [ ] Findings handed off to active enumeration and probing

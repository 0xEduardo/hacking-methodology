# Spidering / Web Crawling

> **Summary**: Crawls web applications to discover endpoints, links, forms, and JavaScript files by following the application structure.
> **Goal**: Build a comprehensive sitemap of the target including hidden paths, API routes, form actions, and JS-referenced endpoints that fuzzing alone may miss.

## Methodology

### When to Use

- After probing to identify live hosts, before fuzzing and vulnerability testing
- To discover the full application structure, including pages linked only from JavaScript
- When building a target-specific wordlist from real application paths and parameters
- To find forms (login, upload, search) that are prime targets for injection testing
- For JavaScript-heavy SPAs where traditional crawling misses client-side routes

### Step-by-Step

1. **Prepare target list** -- use the alive hosts from [probing](probing.md)
   - Success criteria: list of live URLs to crawl
2. **Run headless crawlers** -- use katana with JavaScript rendering to discover client-side routes
   - Success criteria: comprehensive list of discovered endpoints, JS files, and forms
3. **Run lightweight crawlers** -- use gospider or hakrawler for fast link extraction without rendering
   - Success criteria: additional endpoints not found by headless crawling
4. **Collect DNS records** -- resolve discovered subdomains for additional context
   - Success criteria: DNS record types (A, AAAA, CNAME, MX) mapped for each host
5. **Extract and deduplicate** -- merge results, remove duplicates, and categorize by type
   - Success criteria: clean lists of endpoints, JS files, forms, and API routes
6. **Feed results downstream** -- send discovered URLs to [param-discovery](param-discovery.md), [javascript-analysis](javascript-analysis.md), and [fuzzing](fuzzing.md)

## Tools & Commands

| Tool | Command | Purpose |
|------|---------|---------|
| katana | `cat hosts.txt \| katana -jc -kf all -d 3 -o crawl.txt` | Headless crawl with JS execution |
| katana | `katana -u https://<TARGET> -jc -kf all -ef css,png,jpg,gif,svg,woff,ttf -d 5` | Deep single-target crawl excluding static assets |
| katana | `katana -u https://<TARGET> -jc -headless -o headless_crawl.txt` | Full headless browser crawl for SPAs |
| gospider | `gospider -s https://<TARGET> -c 10 -d 3 --other-source -o output/` | Fast crawl with third-party source discovery |
| gospider | `gospider -S hosts.txt -c 5 -d 2 -t 5 --other-source -o output/` | Batch crawl multiple targets |
| hakrawler | `echo https://<TARGET> \| hakrawler -d 3 -u` | Fast endpoint discovery with unique URLs |
| dnsx | `dnsx -retry 3 -a -aaaa -cname -ns -ptr -mx -soa -resp -silent -l subdomains.txt > dns_info.txt` | Resolve DNS records for discovered hosts |
| gau | `echo "<TARGET_DOMAIN>" \| gau --subs` | Historical URL discovery from archives |
| waymore | `waymore -i <TARGET_DOMAIN> -mode U` | Extended Wayback + Common Crawl URL mining |

### Katana (Recommended Primary Crawler)

```bash
# Standard crawl with JS parsing and known-file discovery
cat hosts.txt | katana \
  -jc \                           # Parse JavaScript for endpoints
  -kf all \                       # Check known files (robots.txt, sitemap.xml, etc.)
  -d 3 \                          # Crawl depth
  -nc \                           # No color (cleaner output for piping)
  -ef png,jpg,jpeg,css,gif,ttf,woff,woff2,svg,eot \  # Exclude static assets
  -o katana_results.txt

# Headless mode for JavaScript-heavy SPAs
katana -u https://<TARGET> \
  -headless \                     # Use headless browser
  -jc \                           # JS crawling
  -d 5 \                          # Deeper crawl
  -rl 100 \                       # Rate limit (requests/sec)
  -timeout 15 \                   # Request timeout
  -o spa_crawl.txt

# Crawl with automatic form filling
katana -u https://<TARGET> \
  -headless \
  -jc \
  -automatic-form-fill \          # Fill and submit forms
  -form-extraction \              # Extract form details
  -o forms_crawl.txt

# Crawl with scope control
katana -u https://<TARGET> \
  -jc \
  -cs <TARGET_DOMAIN> \           # Crawl scope (regex)
  -d 3 \
  -o scoped_crawl.txt
```

### GoSpider

```bash
# Single target with all features
gospider -s https://<TARGET> \
  -c 10 \                        # Concurrent requests
  -d 3 \                         # Depth
  -t 5 \                         # Threads
  --other-source \               # Include Wayback, CommonCrawl, VirusTotal
  --include-subs \               # Include subdomains
  -o gospider_output/

# Batch targets
gospider -S hosts.txt -c 5 -d 2 -t 3 --other-source -o batch_output/
```

### Hakrawler

```bash
# Fast crawl with unique output
echo "https://<TARGET>" | hakrawler -d 3 -u

# Pipe subdomains
cat alive_hosts.txt | hakrawler -d 2 -u -subs
```

### Historical URL Discovery

```bash
# GAU - Get All URLs from archives
echo "<TARGET_DOMAIN>" | gau --subs --o gau_urls.txt

# Waymore - extended archive mining
waymore -i <TARGET_DOMAIN> -mode U -oU waymore_urls.txt

# Combine and deduplicate
cat gau_urls.txt waymore_urls.txt | sort -u > historical_urls.txt

# Filter for interesting extensions
cat historical_urls.txt | grep -iE "\.(php|asp|aspx|jsp|json|xml|conf|bak|old|sql|zip|tar|gz)$"
```

### DNS Record Collection

```bash
# Comprehensive DNS resolution
dnsx -retry 3 -a -aaaa -cname -ns -ptr -mx -soa -resp -silent -l subdomains.txt > dns_records.txt

# Quick A record resolution
dnsx -a -resp -silent -l subdomains.txt
```

### Post-Processing

```bash
# Extract unique endpoints from all crawlers
cat katana_results.txt gospider_output/* | sort -u > all_endpoints.txt

# Extract JS files for further analysis
cat all_endpoints.txt | grep -iE "\.js$" | sort -u > js_files.txt

# Extract API endpoints
cat all_endpoints.txt | grep -iE "/api/|/v[0-9]+/" | sort -u > api_endpoints.txt

# Extract endpoints with parameters
cat all_endpoints.txt | grep "=" | sort -u > parameterized_urls.txt

# Build a custom wordlist from discovered paths
cat all_endpoints.txt | unfurl paths | tr '/' '\n' | sort -u > custom_wordlist.txt
```

## Tips & Edge Cases

- Katana with `-jc` (JavaScript crawling) finds endpoints that traditional crawlers miss entirely
- Use `-headless` for modern SPAs built with React, Angular, or Vue -- non-headless mode will miss routes
- Always exclude static file extensions to reduce noise and speed up crawling
- The `-kf all` flag in katana checks for robots.txt, sitemap.xml, crossdomain.xml, and other known files automatically
- Run GAU/waymore in parallel with active crawling -- historical URLs often reveal deleted but still-accessible endpoints
- Rate limit your crawling (`-rl` in katana) against production targets to avoid triggering WAFs or causing issues
- Use the webpaste Chrome extension ([xnl-h4ck3r/webpaste](https://github.com/xnl-h4ck3r/webpaste)) for manual browsing sessions to capture endpoints from interactive exploration
- CNAME records from dnsx can reveal cloud services (S3, Azure, Heroku) worth testing for misconfigurations
- Compare current crawl results against historical URLs to find endpoints that were removed but may still work

## Output Format

```
# Crawled endpoints
https://target.com/api/v1/users
https://target.com/api/v1/orders?status=pending
https://target.com/admin/dashboard
https://target.com/static/js/app.bundle.js

# DNS records
target.com [A] [93.184.216.34]
api.target.com [CNAME] [api-gateway.amazonaws.com]
mail.target.com [MX] [mail.target.com]
```

## References

- [katana - ProjectDiscovery](https://github.com/projectdiscovery/katana)
- [gospider - jaeles-project](https://github.com/jaeles-project/gospider)
- [hakrawler - hakluke](https://github.com/hakluke/hakrawler)
- [dnsx - ProjectDiscovery](https://github.com/projectdiscovery/dnsx)
- [gau - lc](https://github.com/lc/gau)
- [waymore - xnl-h4ck3r](https://github.com/xnl-h4ck3r/waymore)
- [webpaste - xnl-h4ck3r](https://github.com/xnl-h4ck3r/webpaste)

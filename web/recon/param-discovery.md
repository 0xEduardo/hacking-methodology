# Parameter Discovery

> **Summary**: Discovers hidden, undocumented, or forgotten HTTP parameters on web endpoints that may expose additional attack surface.
> **Goal**: Find parameters accepted by endpoints that are not visible in the UI -- these often lack validation and lead to IDOR, SSRF, SQLi, XSS, or privilege escalation.

## Methodology

### When to Use

- After identifying live endpoints through spidering and fuzzing
- On every interesting endpoint, especially APIs, search forms, login pages, and admin panels
- When testing for hidden functionality like debug modes, verbose errors, or admin features
- Before testing for IDOR, mass assignment, or SSRF -- parameter discovery often reveals the injection points

### Step-by-Step

1. **Collect target URLs** -- gather endpoints from spidering, JavaScript analysis, and fuzzing results
   - Success criteria: clean list of URLs with known endpoints
2. **Mine parameters from existing sources** -- extract known parameters from crawled URLs, JS files, and Wayback Machine
   - Success criteria: list of parameter names already observed in the wild
3. **Brute-force hidden parameters** -- use x8 or Arjun to test for additional accepted parameters
   - Success criteria: new parameters discovered that were not visible in the UI
4. **Test discovered parameters** -- feed them into vulnerability-specific scanners (SQLi, XSS, SSRF)
   - Success criteria: confirmed new attack surface from discovered parameters
5. **Repeat for each HTTP method** -- parameters may differ between GET, POST, JSON, and XML

## Tools & Commands

| Tool | Command | Purpose |
|------|---------|---------|
| x8 | `x8 -u "https://<TARGET>/" -w <WORDLIST>` | Brute-force hidden parameters on a single URL |
| x8 | `x8 -u "https://<TARGET>/" -w <WORDLIST> -m POST` | Test POST parameters |
| x8 | `x8 -u "https://<TARGET>/api/v1/users" -w <WORDLIST> --headers "Authorization: Bearer <TOKEN>"` | Authenticated parameter discovery |
| Arjun | `arjun -u https://<TARGET>/ -w <WORDLIST>` | Discover GET parameters |
| Arjun | `arjun -i urls.txt -oT output/ -m GET` | Batch scan multiple URLs for GET params |
| Arjun | `arjun -i urls.txt -oT output/ -m POST` | Batch scan for POST params |
| Arjun | `arjun -i urls.txt -oT output/ -m POST-JSON` | Batch scan for JSON body params |
| Arjun | `arjun -i urls.txt -oT output/ -m POST-XML` | Batch scan for XML body params |
| ParamSpider | `paramspider -d <TARGET_DOMAIN>` | Mine parameters from web archives |
| Param Miner | Burp Extension | Discover hidden params via Burp Suite |
| unfurl | `cat crawled_urls.txt \| grep "=" \| unfurl format %d%p` | Extract base URLs from parameterized URLs |
| unfurl | `cat crawled_urls.txt \| unfurl keys` | Extract parameter names from URLs |

### Parameter Mining from Crawling Results

Extract parameters already seen in the wild from your crawling output.

```bash
# Extract URLs that have parameters
cat crawled_urls.txt | grep "=" | unfurl format %d%p | sort -u > parameterized_endpoints.txt

# Extract just the parameter names
cat crawled_urls.txt | grep "=" | unfurl keys | sort | uniq -c | sort -rn > known_params.txt

# Extract full key=value pairs
cat crawled_urls.txt | grep "=" | unfurl keypairs | sort -u
```

### Feed Discovered Parameters Back

```bash
# Build a custom wordlist from discovered parameter names
cat crawled_urls.txt | unfurl keys | sort -u > custom_params_wordlist.txt

# Combine with a standard wordlist for brute-forcing
cat custom_params_wordlist.txt /usr/share/seclists/Discovery/Web-Content/burp-parameter-names.txt | sort -u > combined_params.txt
```

### Mining from JavaScript Files

```bash
# Extract parameter names from JS files using grep
cat all_js_files.txt | xargs grep -oP '[\?&](\w+)=' | cut -d= -f1 | sort -u

# Use jsluice to extract URLs with parameters from JS
cat app.js | jsluice urls | jq -r '.url' | unfurl keys | sort -u

# Regex for common parameter patterns in JS
grep -oP '(param|query|key|field|name|id|token|api_key|secret|password|user|admin|debug|test|verbose)\b' *.js | sort -u
```

### Mining from Wayback Machine

```bash
# Get historical URLs with parameters
echo "<TARGET_DOMAIN>" | gau --subs | grep "=" | unfurl keys | sort | uniq -c | sort -rn

# Filter for unique endpoints with params
echo "<TARGET_DOMAIN>" | gau | grep "=" | unfurl format %d%p | sort -u
```

### Brute-Force Workflow

```bash
# Step 1: Run x8 on target endpoints
x8 -u "https://<TARGET>/api/v1/users" -w /usr/share/seclists/Discovery/Web-Content/burp-parameter-names.txt

# Step 2: Run Arjun for a second opinion (different detection logic)
arjun -u "https://<TARGET>/api/v1/users" -w /usr/share/seclists/Discovery/Web-Content/burp-parameter-names.txt

# Step 3: Test all HTTP methods
for METHOD in GET POST PUT PATCH DELETE; do
  arjun -u "https://<TARGET>/api/v1/users" -m $METHOD -oT "params_${METHOD}.txt"
done
```

## Wordlists

| Wordlist | Path / URL | Use Case |
|----------|-----------|----------|
| Burp parameter names | `/usr/share/seclists/Discovery/Web-Content/burp-parameter-names.txt` | General parameter brute-force |
| Arjun default | Built-in (~25k params) | Comprehensive param discovery |
| x8 default | Built-in | Fast param detection |
| Custom from crawl | Generated from `unfurl keys` | Target-specific params |
| Assetnote params | `https://wordlists-cdn.assetnote.io/data/automated/httparchive_parameters_top_1m.txt` | Top 1M parameters from HTTP Archive |

## Tips & Edge Cases

- x8 and Arjun use different comparison algorithms -- running both increases coverage
- Always test multiple HTTP methods (GET, POST, JSON, XML) -- endpoints often accept params they do not document
- Hidden parameters like `debug=true`, `test=1`, `admin=1`, `verbose=1` can unlock debug output or bypass restrictions
- Parameters named `callback`, `redirect`, `url`, `next`, `return` often lead to SSRF or open redirect
- Parameters named `role`, `is_admin`, `group`, `privilege` may be vulnerable to mass assignment
- API endpoints frequently accept undocumented filter/sort parameters (e.g., `sort`, `order`, `fields`, `include`, `expand`)
- Use Burp Suite Param Miner extension for browser-based discovery -- it can find params that automated tools miss
- Some WAFs block rapid parameter testing -- add delays with `--delay` flags
- JSON APIs may accept additional keys in the body that are silently processed by the backend

## Output Format

```
[URL]                                    [Method]  [Discovered Parameters]
https://target.com/api/v1/users          GET       id, role, status, debug, fields
https://target.com/api/v1/search         GET       q, page, limit, sort, order
https://target.com/login                 POST      username, password, remember, redirect, otp
https://target.com/api/v1/profile        PUT       name, email, role, is_admin
```

## Agent Workflow
> Step-by-step instructions for an AI agent to perform parameter discovery reconnaissance.

### Phase 1: Setup
1. Collect target URLs from prior phases:
   - Endpoints from [spidering](spidering.md) results
   - API routes from [javascript-analysis](javascript-analysis.md)
   - Discovered paths from [fuzzing](fuzzing.md)
2. Extract already-known parameter names from crawled URLs:
   ```
   cat crawled_urls.txt | grep "=" | unfurl keys | sort -u > known_params.txt
   ```
3. Build a combined parameter wordlist:
   ```
   cat known_params.txt /usr/share/seclists/Discovery/Web-Content/burp-parameter-names.txt | sort -u > combined_params.txt
   ```
4. Identify priority endpoints for testing (login forms, API endpoints, search, admin panels)

### Phase 2: Execution
1. Mine parameters from JavaScript files:
   ```
   cat all_js_files.txt | xargs grep -oP '[\?&](\w+)=' | cut -d= -f1 | sort -u
   ```
2. Mine parameters from Wayback Machine:
   ```
   echo "<TARGET_DOMAIN>" | gau --subs | grep "=" | unfurl keys | sort | uniq -c | sort -rn
   ```
3. Brute-force hidden GET parameters with x8:
   ```
   x8 -u "https://<TARGET>/<ENDPOINT>" -w combined_params.txt
   ```
4. Brute-force hidden POST parameters with Arjun:
   ```
   arjun -u "https://<TARGET>/<ENDPOINT>" -m POST -w combined_params.txt
   ```
5. Test all HTTP methods on API endpoints:
   ```
   for METHOD in GET POST PUT PATCH DELETE; do
     arjun -u "https://<TARGET>/api/v1/<ENDPOINT>" -m $METHOD -oT "params_${METHOD}.txt"
   done
   ```
6. Test JSON body parameters on API endpoints:
   ```
   arjun -u "https://<TARGET>/api/v1/<ENDPOINT>" -m POST-JSON
   ```

### Phase 3: Analysis
1. Categorize discovered parameters by potential vulnerability type:
   - **Reflected parameters** (value appears in response): potential XSS
   - **Database-backed parameters** (`id`, `user_id`, `order`): potential SQLi, IDOR
   - **URL/redirect parameters** (`url`, `redirect`, `next`, `callback`): potential SSRF, open redirect
   - **Role/privilege parameters** (`role`, `is_admin`, `group`, `privilege`): potential mass assignment
   - **Debug parameters** (`debug`, `test`, `verbose`, `trace`): potential info disclosure
   - **File parameters** (`file`, `path`, `template`, `page`): potential LFI/path traversal
2. Compare parameters found by x8 vs Arjun -- union the results for full coverage
3. Identify parameters that accept different values across HTTP methods

### Phase 4: Next Steps
- Feed reflected parameters to [XSS](../exploitation/vulns/xss.md) testing
- Feed database-backed parameters to [SQLi](../exploitation/vulns/sqli.md) testing
- Feed URL parameters to [SSRF](../exploitation/vulns/ssrf.md) and [open redirect](../exploitation/vulns/open-redirect.md) testing
- Feed role/privilege parameters to mass assignment and [IDOR](../exploitation/vulns/idor.md) testing
- Feed debug parameters to information disclosure analysis
- Feed file parameters to [LFI](../exploitation/vulns/lfi.md) testing

## Decision Tree

```
START: Target endpoints collected from spidering + fuzzing + JS analysis
  |
  +--> 1. Mine known params from crawled URLs (unfurl keys)
  |
  +--> 2. Mine params from JS files (grep patterns)
  |
  +--> 3. Mine params from Wayback Machine (gau + unfurl)
  |
  +--> 4. Brute-force reflected params (x8, GET)
  |      |
  |      +--> Reflected params found? --> XSS testing
  |
  +--> 5. Brute-force hidden params (Arjun, POST/JSON/XML)
  |      |
  |      +--> Hidden params found? --> Categorize by vuln type
  |
  +--> 6. Brute-force header params (x8 with header mode)
  |      |
  |      +--> Header params found? --> Host header injection, cache poisoning
  |
  +--> 7. Brute-force cookie params
         |
         +--> Cookie params found? --> Session manipulation testing
```

## Success Criteria

- [ ] Known parameters mined from crawled URLs and JS files
- [ ] Historical parameters mined from Wayback Machine
- [ ] GET parameter brute-force completed on priority endpoints
- [ ] POST parameter brute-force completed on priority endpoints
- [ ] JSON body parameter testing completed on API endpoints
- [ ] Multiple HTTP methods tested on API endpoints
- [ ] Discovered parameters categorized by vulnerability type
- [ ] Results handed off to vulnerability-specific testing phases

## References

- [x8 - Sh1Yo](https://github.com/Sh1Yo/x8)
- [Arjun - s0md3v](https://github.com/s0md3v/Arjun)
- [ParamSpider - devanshbatham](https://github.com/devanshbatham/ParamSpider)
- [unfurl - tomnomnom](https://github.com/tomnomnom/unfurl)
- [Param Miner - PortSwigger](https://portswigger.net/bappstore/17d2949a985c4b7ca092728dba871943)
- [Burp Parameter Names - SecLists](https://github.com/danielmiessler/SecLists/blob/master/Discovery/Web-Content/burp-parameter-names.txt)
- [Assetnote Wordlists](https://wordlists.assetnote.io/)

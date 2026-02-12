# Fuzzing

> **Summary**: Brute-force discovery of hidden directories, files, parameters, virtual hosts, and API endpoints on web applications.
> **Goal**: Uncover content and functionality not linked or indexed, expanding the attack surface before exploitation.

## Methodology

### When to Use

- After initial spidering/crawling to find content that crawlers miss
- When targeting authenticated or restricted areas
- To discover backup files, configuration files, and debug endpoints
- For virtual host enumeration when multiple apps share an IP
- To find hidden API endpoints and undocumented parameters
- After technology fingerprinting (select extensions and wordlists accordingly)

### Step-by-Step

1. **Identify the technology stack** -- determine the web server, language, and framework to choose appropriate extensions and wordlists
2. **Select a wordlist** -- start general, then refine with technology-specific lists (see Wordlist Strategy below)
3. **Calibrate baseline response** -- send a request with a known non-existent path to identify default 404 behavior (response code, size, word count)
4. **Run directory/file fuzzing** -- use recursive mode; every new directory found should be fuzzed again
5. **Fuzz with extensions** -- append relevant file extensions based on the detected stack
6. **Filter noise** -- remove false positives by filtering on response code, size, words, or lines
7. **Fuzz for backup files** -- target known filenames with backup extensions (.bak, .old, .tmp, ~, .swp)
8. **Fuzz virtual hosts** -- enumerate subdomains or vhosts on the same IP via the Host header
9. **Fuzz parameters** -- discover hidden GET/POST parameters on discovered endpoints
10. **Fuzz API endpoints** -- target /api/v1/FUZZ, /api/v2/FUZZ, and similar patterns
11. **Review and deduplicate results** -- use post-processing tools to remove noise and duplicates

## Tools & Commands

### Directory & File Fuzzing

| Tool | Command | Purpose |
|------|---------|---------|
| ffuf | `ffuf -u https://<TARGET>/FUZZ -w <WORDLIST> -ac -c -se -sf` | Fast general-purpose fuzzer with auto-calibration |
| feroxbuster | `feroxbuster -u https://<TARGET> -w <WORDLIST> --threads 50 --depth 3` | Recursive content discovery |
| gobuster | `gobuster dir -u https://<TARGET> -w <WORDLIST> -t 50 -k` | Directory brute-force (supports self-signed certs) |
| dirsearch | `dirsearch -u https://<TARGET> -w <WORDLIST> -e php,asp,aspx,jsp -r -t 30` | Directory search with recursive mode |
| wfuzz | `wfuzz -c -w <WORDLIST> --hc 404 https://<TARGET>/FUZZ` | Flexible fuzzer with advanced filtering |

### Extension Fuzzing

```bash
# ffuf -- fuzz file extensions on a known path
ffuf -u https://<TARGET>/index.FUZZ -w /usr/share/seclists/Discovery/Web-Content/web-extensions.txt -ac

# Common extensions by technology
# PHP:    -e php,php5,php7,phtml,inc
# ASP:    -e asp,aspx,ashx,asmx,config
# Java:   -e jsp,jspa,do,action,html
# Python: -e py,pyc
# Node:   -e js,json
# Generic:-e txt,bak,old,tmp,swp,zip,tar.gz,sql,log,xml,conf,env

# feroxbuster -- extension list
feroxbuster -u https://<TARGET> -w <WORDLIST> -x php,txt,bak,old,conf -d 2
```

### Recursive Fuzzing

```bash
# ffuf recursive mode
ffuf -u https://<TARGET>/FUZZ -w <WORDLIST> -ac -recursion -recursion-depth 3

# feroxbuster (recursive by default)
feroxbuster -u https://<TARGET> -w <WORDLIST> --depth 4 --dont-filter

# dirsearch recursive
dirsearch -u https://<TARGET> -w <WORDLIST> -r --recursion-depth=3
```

### Virtual Host Fuzzing

```bash
# ffuf -- vhost discovery
ffuf -u http://<TARGET_IP> -H "Host: FUZZ.<DOMAIN>" \
  -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt \
  -ac -c

# gobuster vhost mode
gobuster vhost -u http://<TARGET_IP> -w <WORDLIST> --domain <DOMAIN> --append-domain

# wfuzz vhost
wfuzz -c -w <WORDLIST> -H "Host: FUZZ.<DOMAIN>" --hc 400,404,403 http://<TARGET_IP>
```

### Parameter Fuzzing

```bash
# ffuf -- GET parameter discovery
ffuf -u "https://<TARGET>/endpoint?FUZZ=test" \
  -w /usr/share/seclists/Discovery/Web-Content/burp-parameter-names.txt -ac

# ffuf -- POST parameter discovery
ffuf -u "https://<TARGET>/endpoint" -X POST \
  -d "FUZZ=test" -H "Content-Type: application/x-www-form-urlencoded" \
  -w /usr/share/seclists/Discovery/Web-Content/burp-parameter-names.txt -ac

# wfuzz -- parameter value fuzzing
wfuzz -c -w <WORDLIST> --hc 404 "https://<TARGET>/endpoint?id=FUZZ"

# Arjun -- automated parameter discovery
arjun -u https://<TARGET>/endpoint -m GET
arjun -u https://<TARGET>/endpoint -m POST
```

### API Endpoint Fuzzing

```bash
# REST API path fuzzing
ffuf -u "https://<TARGET>/api/v1/FUZZ" \
  -w /usr/share/seclists/Discovery/Web-Content/api/api-endpoints.txt -ac

# ffuf -- multiple API versions
ffuf -u "https://<TARGET>/api/VER/FUZZ" \
  -w versions.txt:VER -w endpoints.txt:FUZZ -ac

# GraphQL endpoint discovery
ffuf -u "https://<TARGET>/FUZZ" \
  -w /usr/share/seclists/Discovery/Web-Content/graphql.txt -ac
```

### Authenticated Fuzzing

```bash
# ffuf with cookies
ffuf -u https://<TARGET>/FUZZ -w <WORDLIST> -ac \
  -b "session=<SESSION_COOKIE>; token=<TOKEN>"

# ffuf with authorization header
ffuf -u https://<TARGET>/FUZZ -w <WORDLIST> -ac \
  -H "Authorization: Bearer <JWT_TOKEN>"

# ffuf with custom headers
ffuf -u https://<TARGET>/FUZZ -w <WORDLIST> -ac \
  -H "X-Custom-Auth: <VALUE>" -H "Cookie: <COOKIE>"
```

### Response Filtering

```bash
# ffuf filters
-fc 404,403,500         # Filter by status code
-fs 1234                # Filter by response size (bytes)
-fw 50                  # Filter by word count
-fl 10                  # Filter by line count
-fr "not found"         # Filter by regex in response body
-ac                     # Auto-calibrate (recommended first pass)

# Show only specific codes
-mc 200,301,302,403

# Combine filters
ffuf -u https://<TARGET>/FUZZ -w <WORDLIST> -mc all -fc 404 -fs 0
```

### Backup File Discovery

```bash
# bfac -- backup file finder
bfac --no-text --url https://<TARGET>/app.php --level 2

# ffuf -- backup extension fuzzing on known files
ffuf -u "https://<TARGET>/config.FUZZ" \
  -w <(printf "bak\nold\ntmp\nswp\norig\nsave\nbackup\n~\n.bak\n.old\n.1\nzip\ntar.gz\nsql") -mc all -fc 404
```

### Post-Processing

```bash
# ffufPostprocessing -- deduplicate and clean ffuf output
ffuf -u https://<TARGET>/FUZZ -w <WORDLIST> -o /tmp/ffuf/results.json -od /tmp/ffuf/bodies/
ffufPostprocessing -result-file /tmp/ffuf/results.json \
  -bodies-folder /tmp/ffuf/bodies/ -delete-bodies -overwrite-result-file

# uro -- remove duplicate/similar URLs from results
cat urls.txt | uro

# ffuf JSON output for scripting
ffuf -u https://<TARGET>/FUZZ -w <WORDLIST> -o results.json -of json
```

## Wordlist Strategy

| Use Case | Wordlist | Source |
|----------|----------|--------|
| General directories | `raft-large-directories.txt` | SecLists |
| General files | `raft-large-files.txt` | SecLists |
| Common paths | `common.txt`, `big.txt` | dirb |
| Technology-specific | `*-extensions.txt` per framework | SecLists |
| API endpoints | `api-endpoints.txt`, `api/` directory | SecLists |
| Parameters | `burp-parameter-names.txt` | SecLists |
| CMS-specific | per-app wordlists | webapp-wordlists |
| Custom (from target) | Generated with CeWL | CeWL |
| All-in-one | `OneListForAll` | six2dez |
| Assetnote | technology-specific lists | wordlists.assetnote.io |

```bash
# CeWL -- generate custom wordlist from target content
cewl https://<TARGET> -d 3 -m 5 -w custom_wordlist.txt

# Combine wordlists and deduplicate
sort -u wordlist1.txt wordlist2.txt > combined.txt
```

## ffuf Configuration File

Save as `~/.ffufrc` to apply defaults to every run:

```ini
[general]
  colors = true

[http]
  headers = [
    "User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:109.0) Gecko/20100101 Firefox/115.0"
  ]

[input]
  wordlists = [
    "/usr/share/seclists/Discovery/Web-Content/common.txt:FUZZ"
  ]
```

## Tips & Edge Cases

- **Auto-calibrate first** -- always use `ffuf -ac` on a first pass, then fine-tune with explicit filters
- **Rate limiting / WAF** -- if responses become 429 or 403, throttle with `-rate <N>` in ffuf or `-L <N>` in feroxbuster; rotate User-Agent strings; add jitter with `-t 1 -rate 10`
- **403 bypass** -- if fuzzing returns many 403s, try [403 bypass techniques](../exploitation/bypass/403.md) before giving up on those paths
- **Stop on errors** -- use `-se` (stop on errors) and `-sf` (stop on >95% 403s) in ffuf to detect bans early
- **Response comparison** -- compare response body hashes or sizes rather than relying solely on status codes; some apps return 200 for everything
- **Recursive discovery** -- every new directory discovered during fuzzing should be fuzzed again with the same wordlist
- **HTTPS with self-signed certs** -- use `-k` in gobuster and feroxbuster, ffuf handles this automatically
- **Multiple sources** -- combine fuzzing with spidering results; pipe spider output into a new wordlist for targeted fuzzing
- **Recollapse** -- use [recollapse](https://github.com/0xacb/recollapse) to generate breaking-strings for normalization bugs
- **Proxy output** -- send ffuf traffic through Burp with `-x http://127.0.0.1:8080` for manual inspection

## Output Format

Fuzzing results should be saved as JSON for post-processing and piped into the next phase:

```
[STATUS_CODE] [RESPONSE_SIZE] [WORD_COUNT] [URL]
200           4523            312          https://target.com/admin/
301           0               0            https://target.com/api/
200           892             45           https://target.com/config.bak
```

## Agent Workflow
> Step-by-step instructions for an AI agent to perform web fuzzing reconnaissance.

### Phase 1: Setup
1. Identify the target URL and confirm it is alive (check HTTP response)
2. Fingerprint the technology stack from [probing](probing.md) or [web-technologies](../helpers/web-technologies.md) results
3. Select the primary wordlist based on detected technology:
   - PHP target: use `raft-large-files.txt` + extensions `php,php5,phtml,inc,bak`
   - ASP/.NET target: use `raft-large-files.txt` + extensions `asp,aspx,ashx,asmx,config`
   - Java target: use `raft-large-files.txt` + extensions `jsp,jspa,do,action`
   - Generic/unknown: use `common.txt` or `raft-large-directories.txt`
4. If authenticated testing is required, obtain valid session cookies or tokens
5. Verify ffuf is configured with `~/.ffufrc` defaults (custom User-Agent, colors)

### Phase 2: Execution
1. Run initial directory fuzzing with auto-calibration:
   ```
   ffuf -u https://<TARGET>/FUZZ -w <WORDLIST> -ac -c -se -sf -o results_dir.json -of json
   ```
2. Run file fuzzing with technology-specific extensions:
   ```
   ffuf -u https://<TARGET>/FUZZ -w <WORDLIST> -ac -e <EXTENSIONS> -o results_files.json -of json
   ```
3. Run recursive fuzzing on any discovered directories:
   ```
   ffuf -u https://<TARGET>/FUZZ -w <WORDLIST> -ac -recursion -recursion-depth 3
   ```
4. Run backup file discovery on interesting files found:
   ```
   ffuf -u https://<TARGET>/<KNOWN_FILE>.FUZZ -w backup_extensions.txt -mc all -fc 404
   ```
5. Run virtual host fuzzing if multiple apps may share the IP:
   ```
   ffuf -u http://<TARGET_IP> -H "Host: FUZZ.<DOMAIN>" -w subdomains-top1million-5000.txt -ac
   ```
6. Run parameter fuzzing on discovered endpoints:
   ```
   ffuf -u "https://<TARGET>/<ENDPOINT>?FUZZ=test" -w burp-parameter-names.txt -ac
   ```

### Phase 3: Analysis
1. Parse JSON output files and deduplicate results
2. Categorize findings by type:
   - **Admin panels**: paths containing `admin`, `dashboard`, `manager`, `console`
   - **Backup files**: extensions `.bak`, `.old`, `.swp`, `.zip`, `.sql`, `.tar.gz`
   - **Configuration files**: `config`, `.env`, `.ini`, `.xml`, `.yaml`, `.properties`
   - **API endpoints**: paths containing `/api/`, `/v1/`, `/v2/`, `/graphql`
   - **Debug/test endpoints**: `debug`, `test`, `phpinfo`, `trace`, `actuator`
3. Flag 403 responses for [403 bypass testing](../exploitation/bypass/403.md)
4. Flag 401 responses for brute force or default credential testing
5. Compare response sizes to identify unique (non-default) pages

### Phase 4: Next Steps
- Feed discovered endpoints to [spidering](spidering.md) for deeper crawling
- Feed discovered endpoints to [param-discovery](param-discovery.md) for hidden parameter testing
- Feed discovered API endpoints to vulnerability scanners
- Send backup/config files for manual review (credential extraction)
- Test 403 paths with [403 bypass techniques](../exploitation/bypass/403.md)
- Feed admin panels to [brute-force](../helpers/brute-force.md) testing

## Decision Tree

```
START: Target URL confirmed alive
  |
  +--> 1. Directory fuzzing (raft-large-directories.txt + auto-calibrate)
  |      |
  |      +--> New directories found? --> Recursive fuzzing on each
  |
  +--> 2. File fuzzing (technology-specific extensions)
  |      |
  |      +--> Interesting files found? --> Backup extension fuzzing
  |
  +--> 3. Parameter fuzzing (burp-parameter-names.txt on key endpoints)
  |      |
  |      +--> Parameters found? --> Feed to XSS/SQLi/SSRF testing
  |
  +--> 4. Virtual host fuzzing (if IP hosts multiple apps)
  |      |
  |      +--> New vhosts found? --> Add to target list, repeat from step 1
  |
  +--> 5. API endpoint fuzzing (/api/v1/FUZZ, /api/v2/FUZZ)
         |
         +--> API routes found? --> Feed to API vulnerability testing
```

## Success Criteria

- [ ] Directory fuzzing completed with at least one wordlist
- [ ] File fuzzing completed with technology-appropriate extensions
- [ ] All discovered directories recursively fuzzed
- [ ] Backup files checked for all interesting filenames
- [ ] Virtual host fuzzing attempted if applicable
- [ ] Parameter fuzzing run on key endpoints
- [ ] Results deduplicated and categorized by type
- [ ] 403/401 responses flagged for further testing
- [ ] Findings handed off to downstream phases (spidering, param discovery, vuln scanning)

## References

- [ffuf](https://github.com/ffuf/ffuf)
- [feroxbuster](https://github.com/epi052/feroxbuster)
- [gobuster](https://github.com/OJ/gobuster)
- [dirsearch](https://github.com/maurosoria/dirsearch)
- [wfuzz](https://github.com/xmendez/wfuzz)
- [bfac](https://github.com/mazen160/bfac)
- [ffufPostprocessing](https://github.com/Damian89/ffufPostprocessing)
- [Arjun](https://github.com/s0md3v/Arjun)
- [recollapse](https://github.com/0xacb/recollapse)
- [SecLists](https://github.com/danielmiessler/SecLists)
- [OneListForAll](https://github.com/six2dez/OneListForAll)
- [webapp-wordlists](https://github.com/p0dalirius/webapp-wordlists)
- [Assetnote Wordlists](https://wordlists.assetnote.io)
- [CeWL](https://github.com/digininja/CeWL)
- [ffuf Advanced Tricks](https://www.acceis.fr/ffuf-advanced-tricks/)

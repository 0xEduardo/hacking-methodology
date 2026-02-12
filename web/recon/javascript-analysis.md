# JavaScript Analysis

> **Summary**: Discover, collect, and analyze JavaScript files to extract endpoints, secrets, and vulnerabilities from client-side code.
> **Goal**: Reveal hidden API endpoints, hardcoded credentials, API keys, internal paths, and exploitable DOM sinks buried in JavaScript.

## Methodology

### When to Use

- During reconnaissance to expand the list of known endpoints and parameters
- After spidering, to extract paths from JS bundles that crawlers may not follow
- When the application is a SPA (React, Angular, Vue) where most logic lives client-side
- To find leaked secrets (API keys, tokens, credentials) in production JS
- To identify vulnerable third-party libraries
- When source maps are available, allowing full source code recovery

### Step-by-Step

1. **Discover JS files** -- collect all JavaScript URLs from spidering, page source, and network traffic
2. **Download JS files** -- save them locally for offline analysis
3. **Retrieve historical versions** -- pull older JS versions from the Wayback Machine to find secrets removed from current builds
4. **Beautify and deobfuscate** -- format minified code for readability; deobfuscate if needed
5. **Extract endpoints** -- use automated tools to pull URLs, paths, and API routes from JS source
6. **Extract secrets** -- scan for API keys, tokens, passwords, and credentials using pattern matching
7. **Check source maps** -- look for `.js.map` files that expose original unminified source
8. **Analyze webpack/bundles** -- extract individual modules from bundled JS
9. **Find DOM sinks** -- identify dangerous patterns that could lead to DOM XSS
10. **Detect vulnerable libraries** -- check third-party libraries for known CVEs
11. **Monitor for changes** -- track JS file modifications over time for newly introduced vulnerabilities

## Tools & Commands

### JS File Discovery & Collection

| Tool | Command | Purpose |
|------|---------|---------|
| getJS | `getJS --url https://<TARGET> --complete` | Download all JS files from a site |
| subjs | `echo <DOMAIN> \| subjs` | Find JS files across subdomains |
| hakrawler | `echo https://<TARGET> \| hakrawler -js` | Crawl and extract JS file URLs |
| gospider | `gospider -s https://<TARGET> -d 2 --js` | Spider with JS file extraction |
| katana | `katana -u https://<TARGET> -jc -d 3` | JS crawling with headless browser |

```bash
# Collect JS files from page source
curl -s https://<TARGET> | grep -oP 'src="[^"]*\.js[^"]*"' | sed 's/src="//;s/"//'

# Download all discovered JS files
cat js_urls.txt | xargs -I@ sh -c 'wget -q "@" -O "$(echo @ | md5sum | cut -d" " -f1).js"'

# Filter out known libraries from analysis
cat all_js_files.txt | grep -v "jquery\|wp-includes\|wp-content\|bootstrap\|react\.production\|angular\.min" > target_js.txt
```

### Historical JS from Wayback Machine

```bash
# waybackpack -- download historical versions of JS files
# https://github.com/jsvine/waybackpack
cat js_to_analyze.txt | xargs -I@ sh -c "waybackpack @ -d wayback_js/"

# waymore -- discover links from wayback (downloads responses too)
# https://github.com/xnl-h4ck3r/waymore
waymore -i <DOMAIN> -mode U -oU wayback_urls.txt
cat wayback_urls.txt | grep "\.js$" > historical_js.txt

# gau -- get all URLs including JS from archives
gau <DOMAIN> --subs | grep "\.js$" | sort -u > all_js_urls.txt
```

### Beautify & Deobfuscate

```bash
# js-beautify -- format minified JS
js-beautify -f minified.js -o beautified.js

# Bulk beautify all downloaded JS
for f in *.js; do js-beautify -f "$f" -o "beautified_$f"; done
```

| Tool | URL | Purpose |
|------|-----|---------|
| de4js | https://lelinhtinh.github.io/de4js/ | Browser-based deobfuscator and unpacker |
| dcode.fr | https://www.dcode.fr/javascript-unobfuscator | JavaScript unobfuscator |
| JS Beautifier | https://beautifier.io | Online beautifier |
| JS Nice | http://jsnice.org | Statistical deobfuscation (renames variables) |
| JsFuck decoder | https://enkhee-osiris.github.io/Decoder-JSFuck/ | Decode JsFuck encoding |
| 0xdevalias guide | https://gist.github.com/0xdevalias/d8b743efb82c0e9406fc69da0d6c6581 | Comprehensive deobfuscation guide |

### Endpoint Extraction

| Tool | Command | Purpose |
|------|---------|---------|
| xnLinkFinder | `python3 xnLinkFinder.py -i https://<TARGET> -o cli` | Extract endpoints from HTML and JS |
| jsluice | `jsluice urls < file.js` | Extract URLs and paths from JS |
| LinkFinder | `python3 linkfinder.py -i https://<TARGET>/app.js -o cli` | Find endpoints in JS files |
| JSParser | `python2 JSParser.py -u https://<TARGET>/app.js` | Parse relative URLs from JS |
| JSFScan | `bash JSFScan.sh -u https://<TARGET>` | Aggregated JS file scanning |

```bash
# xnLinkFinder -- scan all JS files at once
python3 xnLinkFinder.py -i '*.js' -o cli

# xnLinkFinder -- from a list of URLs
python3 xnLinkFinder.py -i urls.txt -o cli -d 2

# jsluice -- extract URLs from a single file
cat app.js | jsluice urls

# jsluice -- extract from all downloaded JS
for f in *.js; do echo "=== $f ===" && cat "$f" | jsluice urls; done

# Regex -- quick manual endpoint extraction
grep -rhoP "(\/[a-zA-Z0-9_\-\.\/]+)" *.js | sort -u
grep -rhoP "\"(https?://[^\"]+)\"" *.js | sort -u
grep -rhoP "'(\/api\/[^']+)'" *.js | sort -u
```

### Secret Extraction

| Tool | Command | Purpose |
|------|---------|---------|
| jsluice | `jsluice secrets < file.js` | Find secrets using built-in patterns |
| jsluice (custom) | `jsluice secrets --patterns=secrets.json < file.js` | Custom secret patterns |
| semgrep | `semgrep --config "p/secrets" <JS_DIR>` | Rule-based secret detection |
| trufflehog | `trufflehog filesystem --directory <JS_DIR>` | High-signal secret scanner |
| SecretFinder | `python3 SecretFinder.py -i https://<TARGET>/app.js -o cli` | Secrets in JS files |
| Mantra | `mantra -f <JS_FILE>` | API key and secret finder |

```bash
# Common regex patterns for manual secret hunting
# AWS Access Key
grep -rhoP "AKIA[0-9A-Z]{16}" *.js

# Google API Key
grep -rhoP "AIza[0-9A-Za-z\-_]{35}" *.js

# Generic API key patterns
grep -rhoP "(api[_-]?key|apikey|api_secret|token|secret|password|auth)['\"]?\s*[:=]\s*['\"][a-zA-Z0-9_\-]{16,}['\"]" *.js

# JWT tokens
grep -rhoP "eyJ[a-zA-Z0-9_-]*\.eyJ[a-zA-Z0-9_-]*\.[a-zA-Z0-9_-]*" *.js

# Private keys
grep -rl "BEGIN.*PRIVATE KEY" *.js

# Internal URLs / IP addresses
grep -rhoP "https?://[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}[^\s\"']*" *.js
grep -rhoP "https?://[a-z]+\.internal[^\s\"']*" *.js
```

Reference patterns database: [secrets-patterns-db](https://github.com/mazen160/secrets-patterns-db)

### Source Map Exploitation

```bash
# Check for source maps by appending .map to JS URLs
curl -s -o /dev/null -w "%{http_code}" https://<TARGET>/app.js.map

# Bulk check from a list of JS URLs
cat js_urls.txt | while read url; do
  code=$(curl -s -o /dev/null -w "%{http_code}" "${url}.map")
  [ "$code" = "200" ] && echo "[FOUND] ${url}.map"
done

# sourcemapper -- reconstruct original source from .map files
# https://github.com/denandz/sourcemapper
sourcemapper -output recovered_source -url https://<TARGET>/app.js.map

# unwebpack-sourcemap -- extract webpack source maps
# https://github.com/nicedayzhu/unwebpack-sourcemap
python3 unwebpack_sourcemap.py https://<TARGET>/app.js.map -o output_dir
```

### Webpack / Bundle Analysis

```bash
# Look for webpack manifest or chunk loading patterns
grep -rhoP "webpackJsonp\|__webpack_require__\|webpack_modules" *.js

# Extract chunk URLs from webpack runtime
grep -rhoP "\"(static/js/[^\"]+)\"" *.js | sort -u

# After recovering source maps, analyze individual modules in output_dir/
# Look for config files, environment variables, API definitions
grep -rl "process\.env\|API_URL\|BASE_URL\|SECRET\|PASSWORD" recovered_source/
```

### DOM Sink Analysis (XSS)

```bash
# Search for dangerous DOM sinks
grep -rnP "(innerHTML|outerHTML|document\.write|eval\(|setTimeout\(|setInterval\(|Function\(|\.src\s*=|location\s*=|location\.href|window\.open)" *.js

# Search for sources flowing into sinks
grep -rnP "(location\.hash|location\.search|document\.URL|document\.referrer|window\.name|postMessage)" *.js

# Taint flow patterns: source -> sink
grep -rnP "\.innerHTML\s*=.*location\." *.js
grep -rnP "eval\(.*\.(hash|search|href)" *.js
grep -rnP "document\.write\(.*\.(URL|referrer)" *.js
```

### Vulnerable Library Detection

| Tool | Command | Purpose |
|------|---------|---------|
| retire.js | `retire --jspath <JS_DIR>` | Detect known-vulnerable JS libraries |
| retire.js (URL) | `retire --jsuri https://<TARGET>/lib.js` | Check a single remote JS file |
| JSHole | `jshole -u https://<TARGET>` | Find vulnerable JS libraries online |
| Snyk | `npx snyk test` | Dependency vulnerability scanning |

```bash
# retire.js -- scan a directory of JS files
retire --jspath ./downloaded_js/ --outputformat json

# Check package versions in JS comments/headers
grep -rhoP "(jquery|angular|react|vue|lodash|moment|bootstrap)[^\s]*[0-9]+\.[0-9]+\.[0-9]+" *.js | sort -u
```

### JS Monitoring

```bash
# JSMon -- monitor JS files for changes
# https://github.com/robre/jsmon
jsmon -u https://<TARGET>/app.js

# Diff-based monitoring with a cron job
curl -s https://<TARGET>/app.js > /tmp/app_js_$(date +%Y%m%d).js
diff /tmp/app_js_previous.js /tmp/app_js_$(date +%Y%m%d).js
```

## Tips & Edge Cases

- **Filter noise first** -- remove known libraries (jQuery, Bootstrap, React production builds) before analysis to focus on application-specific code
- **Historical versions** -- secrets are often committed and later removed; Wayback Machine preserves the versions that still contain them
- **Source maps in production** -- surprisingly common; always check for `.js.map` files as they expose the entire original source tree
- **Webpack chunks** -- modern SPAs split code into chunks loaded on demand; enumerate all chunks, not just the main bundle
- **Inline JS** -- do not forget `<script>` blocks in HTML pages; they often contain configuration objects with API keys and endpoints
- **Environment leakage** -- look for `process.env`, `__ENV__`, `window.__CONFIG__`, and similar patterns that leak build-time environment variables
- **Obfuscated code** -- if code is obfuscated (not just minified), it may be hiding sensitive logic; invest time in deobfuscation
- **Dynamic loading** -- some endpoints are constructed at runtime by concatenating strings; trace string concatenation patterns manually
- **Service workers** -- check `/sw.js` or service worker registrations for cached API routes and offline functionality
- **API documentation** -- JS files sometimes embed OpenAPI/Swagger schemas or contain full API client definitions

## Output Format

Endpoint and secret extraction results should be structured for handoff to the exploitation phase:

```
[TYPE]    [VALUE]                                    [SOURCE FILE]
endpoint  /api/v2/users/profile                     app.bundle.js
endpoint  /internal/admin/config                    admin.chunk.js
secret    AKIA1234567890ABCDEF                      config.js
secret    Bearer eyJhbGciOiJIUzI1NiIs...            auth-service.js
sink      innerHTML assignment at line 342           dashboard.js
library   jquery 2.1.4 (CVE-2020-11022)            vendor.js
```

## References

- [jsluice](https://github.com/BishopFox/jsluice)
- [xnLinkFinder](https://github.com/xnl-h4ck3r/xnLinkFinder)
- [LinkFinder](https://github.com/GerbenJavado/LinkFinder)
- [getJS](https://github.com/003random/getJS)
- [SecretFinder](https://github.com/m4ll0k/SecretFinder)
- [Mantra](https://github.com/MrEmpy/Mantra)
- [trufflehog](https://github.com/trufflesecurity/truffleHog)
- [semgrep secrets](https://semgrep.dev/p/secrets)
- [secrets-patterns-db](https://github.com/mazen160/secrets-patterns-db)
- [sourcemapper](https://github.com/denandz/sourcemapper)
- [retire.js](https://github.com/retirejs/retire.js)
- [JSMon](https://github.com/robre/jsmon)
- [waybackpack](https://github.com/jsvine/waybackpack)
- [waymore](https://github.com/xnl-h4ck3r/waymore)
- [keyhacks](https://github.com/streaak/keyhacks)
- [JSScanner](https://github.com/dark-warlord14/JSScanner)
- [katana](https://github.com/projectdiscovery/katana)

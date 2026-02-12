# Screenshot / Visual Reconnaissance

> **Summary**: Captures screenshots of live web targets for visual triage, allowing rapid identification of interesting, default, or vulnerable applications.
> **Goal**: Visually inspect hundreds or thousands of live hosts to prioritize targets -- find login pages, admin panels, default installs, error pages, and forgotten applications.

## Methodology

### When to Use

- After probing when you have a large list of alive hosts that cannot be manually visited
- To visually identify interesting targets before deep testing (login pages, dashboards, admin panels)
- To detect default installations (Apache, Nginx, Tomcat, IIS) that may have known vulnerabilities
- When comparing environments (staging vs. production) or tracking changes over time
- To build a visual report of the attack surface for documentation

### Step-by-Step

1. **Prepare input** -- use the alive hosts list from [probing](probing.md)
   - Success criteria: file with valid URLs (scheme included), one per line
2. **Ensure browser is installed** -- screenshotting tools require Chrome or Chromium
   - Success criteria: `which google-chrome` or `which chromium` returns a valid path
3. **Run screenshotter** -- capture screenshots of all targets
   - Success criteria: screenshot directory populated with images for each host
4. **Open gallery view** -- use the tool's built-in report viewer to visually triage
   - Success criteria: HTML report or gallery ready for manual review
5. **Tag and prioritize** -- mark interesting targets for follow-up testing
   - Success criteria: shortlist of high-value targets extracted

## Tools & Commands

| Tool | Command | Purpose |
|------|---------|---------|
| gowitness | `gowitness file -f urls.txt -t <THREADS>` | Screenshot all URLs from file |
| gowitness | `gowitness single https://<TARGET>` | Screenshot a single URL |
| gowitness | `gowitness server` | Start web UI to browse screenshots |
| gowitness | `gowitness report generate` | Generate HTML report |
| httpx | `cat hosts.txt \| httpx -ss -srd screenshots/` | Screenshot with httpx (built-in) |
| httpx | `echo <TARGET> \| httpx -ss` | Quick single-target screenshot |
| EyeWitness | `python3 EyeWitness.py -f urls.txt --web -d output/` | Screenshot + header analysis |
| aquatone | `cat hosts.txt \| aquatone -out aquatone_results/` | Screenshot with clustering |

### GoWitness (Recommended)

```bash
# Basic batch screenshot
gowitness file -f urls.txt -t 10

# With custom resolution and timeout
gowitness file -f urls.txt \
  -t 10 \                           # Threads
  --timeout 15 \                    # Page load timeout (seconds)
  --resolution-x 1920 \             # Screenshot width
  --resolution-y 1080               # Screenshot height

# Screenshot with a specific Chrome path
gowitness file -f urls.txt --chrome-path /usr/bin/chromium

# Start the web server to review results
gowitness server
# Visit http://localhost:7171 to browse the gallery

# Generate static HTML report
gowitness report generate
```

### httpx Screenshots

```bash
# Screenshot all alive hosts
cat hosts.txt | httpx -ss -srd ./screenshots/ -silent

# Combined: probe + screenshot in one pass
cat subdomains.txt | httpx -sc -title -tech-detect -ss -srd ./screenshots/ -o probed.txt
```

### EyeWitness

```bash
# Web screenshot mode
python3 EyeWitness.py -f urls.txt --web -d eyewitness_output/

# With custom timeout and threads
python3 EyeWitness.py -f urls.txt --web -d output/ --timeout 30 --threads 10

# RDP screenshot mode (for internal networks)
python3 EyeWitness.py -f rdp_targets.txt --rdp -d rdp_output/
```

### Aquatone

```bash
# Basic usage (pipe from httpx or other tools)
cat hosts.txt | aquatone -out ./aquatone_results/ -threads 5

# With custom ports
cat hosts.txt | aquatone -out ./aquatone_results/ -ports 80,443,8080,8443

# With proxy
cat hosts.txt | aquatone -out ./aquatone_results/ -proxy http://127.0.0.1:8080
```

## What to Look For

When reviewing screenshots, prioritize these findings:

| Visual Indicator | Significance |
|-----------------|--------------|
| Login pages | Authentication testing, brute-force, default creds |
| Admin panels | Privilege escalation, default credentials |
| Default pages (Apache, Nginx, IIS, Tomcat) | Misconfiguration, known CVEs |
| Error pages with stack traces | Information disclosure, technology fingerprinting |
| API documentation (Swagger, GraphQL Playground) | Full API attack surface revealed |
| Dashboards (Grafana, Kibana, Jenkins) | Unauthenticated access, sensitive data |
| File listing / directory index | Information disclosure, sensitive files |
| Forgotten/staging applications | Weaker security, debug features enabled |
| Custom 403/404 pages | Different backend, potential bypass |
| Blank/empty pages | May hide API endpoints, worth fuzzing |

## Tips & Edge Cases

- Always ensure Chrome/Chromium is installed before running any screenshot tool; headless mode requires it
- httpx `-ss` is the fastest option when you are already running httpx for probing -- no need for a separate tool
- GoWitness provides the best gallery view for manual triage with its built-in web server
- For very large target lists (10k+), run in batches to avoid resource exhaustion
- Some targets load slowly -- increase timeout to at least 15-20 seconds for reliable capture
- Compare screenshots over time to detect changes (new features, removed functionality, environment drift)
- Screenshots of 403 pages are valuable -- different visual layouts suggest different backend handling
- Use aquatone's clustering feature to group similar-looking pages and focus on unique ones
- Save screenshots alongside probing data (status code, title, tech) for faster triage

## Output Format

```
screenshots/
  gowitness.sqlite3             # GoWitness database
  report.html                   # Visual gallery report
  screenshots/
    https-target.com-443.png
    https-admin.target.com-443.png
    http-staging.target.com-8080.png
```

## Agent Workflow
> Step-by-step instructions for an AI agent to perform visual reconnaissance via screenshots.

### Phase 1: Setup
1. Obtain the alive hosts list from [probing](probing.md) results -- ensure all URLs include scheme (`http://` or `https://`)
2. Verify Chrome/Chromium is installed: `which google-chrome || which chromium`
3. Select the screenshotting tool based on needs:
   - **httpx -ss**: fastest option if already running httpx for probing
   - **gowitness**: best gallery view for manual triage
   - **aquatone**: best for clustering similar pages
4. Prepare output directory for screenshots

### Phase 2: Execution
1. Run screenshotting on all alive hosts:
   ```
   gowitness file -f urls.txt -t 10 --timeout 15 --resolution-x 1920 --resolution-y 1080
   ```
   Or combined with probing:
   ```
   cat hosts.txt | httpx -sc -title -tech-detect -ss -srd ./screenshots/ -o probed.txt
   ```
2. Start gallery server for visual review:
   ```
   gowitness server
   ```
   Browse to `http://localhost:7171`

### Phase 3: Analysis
1. Visually triage all screenshots, looking for:
   - **Login pages**: mark for authentication testing and brute force
   - **Admin panels**: mark for default credential testing and privilege escalation
   - **Default server pages** (Apache, Nginx, IIS, Tomcat): mark for known CVEs
   - **API documentation** (Swagger, GraphQL Playground): mark for full API testing
   - **Dashboards** (Grafana, Kibana, Jenkins): check for unauthenticated access
   - **Directory listings**: check for sensitive file exposure
   - **Error pages with stack traces**: extract technology details
   - **Blank/empty pages**: may hide API endpoints, worth fuzzing
   - **Custom 403/404 pages**: different backend handling, potential bypass
2. Group similar-looking pages (use aquatone clustering if available)
3. Compare against known default page templates to identify custom applications

### Phase 4: Next Steps
- Prioritize login pages for [brute-force](../helpers/brute-force.md) and [authentication testing](../exploitation/authentication/)
- Test admin panels for default credentials
- Feed API documentation pages to API-specific vulnerability testing
- Test unauthenticated dashboards for information disclosure
- Feed custom applications to deep manual testing
- Feed default server pages to version-specific CVE research

## Decision Tree

```
START: Alive hosts list from probing
  |
  +--> Run screenshotter (gowitness / httpx -ss / aquatone)
  |
  +--> Open gallery view for visual triage
  |
  +--> Categorize by visual indicator:
         |
         +--> Login page --> Brute force + auth bypass testing
         +--> Admin panel --> Default creds + privilege escalation
         +--> Default page --> CVE research for detected version
         +--> API docs --> Full API attack surface testing
         +--> Dashboard --> Check unauthenticated access
         +--> Directory listing --> Sensitive file review
         +--> Error page --> Technology fingerprinting
         +--> Blank page --> Fuzzing for hidden endpoints
```

## Success Criteria

- [ ] Screenshots captured for all alive hosts
- [ ] Gallery view or HTML report generated for triage
- [ ] All screenshots reviewed and categorized
- [ ] High-value targets identified (login, admin, dashboards, API docs)
- [ ] Shortlist of priority targets created for deep testing
- [ ] Findings handed off to appropriate exploitation phases

## References

- [gowitness - sensepost](https://github.com/sensepost/gowitness)
- [httpx - ProjectDiscovery](https://github.com/projectdiscovery/httpx)
- [EyeWitness - RedSiege](https://github.com/RedSiege/EyeWitness)
- [aquatone - michenriksen](https://github.com/michenriksen/aquatone)

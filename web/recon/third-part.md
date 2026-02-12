# Third-Party Service Enumeration

> **Summary**: Discovers third-party services, exposed dashboards, admin panels, API documentation, and external integrations associated with the target organization.
> **Goal**: Find forgotten or misconfigured services outside the main application -- cloud storage, CI/CD pipelines, monitoring dashboards, and API docs that may leak sensitive data or provide unauthenticated access.

## Methodology

### When to Use

- During the reconnaissance phase to expand the attack surface beyond the main web application
- After subdomain enumeration to investigate what services each subdomain hosts
- When looking for shadow IT, forgotten infrastructure, or developer tools left exposed
- To find API documentation (Swagger, GraphQL Playground) that reveals the full API surface
- When hunting for credentials, tokens, or sensitive data exposed through third-party platforms

### Step-by-Step

1. **Search Wayback Machine and archives** -- find historical URLs, old paths, and forgotten endpoints
   - Success criteria: list of URLs from archive sources including old admin panels, dashboards, and API docs
2. **Query threat intelligence platforms** -- check AlienVault OTX, VirusTotal, and Shodan for associated URLs
   - Success criteria: additional URLs and IPs associated with the target
3. **Scrape cloud infrastructure** -- identify AWS, Azure, and GCP resources tied to the organization
   - Success criteria: cloud storage buckets, functions, and services mapped
4. **Search for exposed dashboards** -- find monitoring, CI/CD, and management panels
   - Success criteria: list of exposed dashboards with access status (authenticated vs. open)
5. **Search for leaked data** -- check paste sites, GitHub, and data breach databases
   - Success criteria: any credentials, API keys, or internal information found

## Tools & Commands

| Tool | Command | Purpose |
|------|---------|---------|
| gau | `echo "<TARGET_DOMAIN>" \| gau --subs` | Historical URLs from Wayback, CommonCrawl, OTX |
| waymore | `waymore -i <TARGET_DOMAIN> -mode U` | Extended archive URL mining |
| Shodan | `shodan search org:"<ORG_NAME>"` | Find all internet-facing assets by organization |
| Shodan | `shodan search ssl:"<TARGET_DOMAIN>"` | Find hosts using target's TLS certificates |
| Censys | `censys search "<TARGET_DOMAIN>"` | Certificate and host search |
| awsScrape | `python3 awsScrape.py -t <TARGET_DOMAIN>` | Find AWS-hosted content via certificate scanning |
| cloud_enum | `python3 cloud_enum.py -k <TARGET_KEYWORD>` | Enumerate cloud resources (AWS, Azure, GCP) |
| S3Scanner | `python3 s3scanner.py --buckets-file buckets.txt` | Scan for open/misconfigured S3 buckets |
| trufflehog | `trufflehog github --org=<TARGET_ORG>` | Scan GitHub org for leaked secrets |
| GitDorker | `python3 GitDorker.py -t <GITHUB_TOKEN> -d dorks.txt -q <TARGET_DOMAIN>` | GitHub dork search for secrets |

### Wayback Machine & Archives

```bash
# Get all known URLs from multiple sources
echo "<TARGET_DOMAIN>" | gau --subs --o gau_results.txt

# Extended archive mining with waymore
waymore -i <TARGET_DOMAIN> -mode U -oU waymore_results.txt

# Filter for interesting file types
cat gau_results.txt | grep -iE "\.(json|xml|yaml|yml|conf|env|bak|sql|log|zip|tar|gz|config|properties|ini)$"

# Filter for admin/dashboard paths
cat gau_results.txt | grep -iE "admin|dashboard|panel|console|manage|monitor|grafana|kibana|jenkins|swagger|graphql|api-doc"
```

### AlienVault OTX

Look under "Associated URLs" for a target domain:

```
https://otx.alienvault.com/indicator/domain/<TARGET_DOMAIN>
```

### Cloud Resource Enumeration

```bash
# AWS - find S3 buckets and services by certificate scanning
# https://github.com/jhaddix/awsScrape/
python3 awsScrape.py -t <TARGET_DOMAIN>

# Enumerate cloud resources across AWS, Azure, GCP
# https://github.com/initstring/cloud_enum
python3 cloud_enum.py -k <TARGET_KEYWORD> -k <TARGET_DOMAIN>

# S3 bucket brute-force
# Generate bucket name permutations
echo "<TARGET_NAME>" | sed 's/$/-dev\n&-staging\n&-prod\n&-backup\n&-assets\n&-data\n&-logs/' > bucket_names.txt
python3 s3scanner.py --buckets-file bucket_names.txt

# Azure blob storage
# https://github.com/NetSPI/MicroBurst
Invoke-EnumerateAzureBlobs -Base <TARGET_NAME>

# GCP bucket enumeration
# https://github.com/RhinoSecurityLabs/GCPBucketBrute
python3 gcpbucketbrute.py -k <TARGET_KEYWORD> -w wordlist.txt
```

### Shodan & Censys Queries

```bash
# Shodan - search by organization name
shodan search org:"<ORG_NAME>" --fields ip_str,port,hostnames

# Shodan - search by SSL certificate
shodan search ssl:"<TARGET_DOMAIN>" --fields ip_str,port,hostnames

# Shodan - search by favicon hash
shodan search http.favicon.hash:<HASH> --fields ip_str,port

# Censys - certificate search
censys search "<TARGET_DOMAIN>" --index certificates

# Censys - host search
censys search "services.tls.certificates.leaf.names: <TARGET_DOMAIN>"
```

### Exposed Dashboard & Panel Discovery

Common dashboards to look for during recon:

| Service | Default Path | What It Exposes |
|---------|-------------|-----------------|
| Swagger/OpenAPI | `/swagger-ui.html`, `/api-docs`, `/swagger.json` | Full API documentation |
| GraphQL Playground | `/graphql`, `/graphiql`, `/__graphql` | Interactive GraphQL explorer |
| Jenkins | `/jenkins`, port 8080 | CI/CD pipelines, build secrets |
| Grafana | `/grafana`, port 3000 | Infrastructure metrics |
| Kibana | `/kibana`, port 5601 | Application logs, sensitive data |
| Prometheus | `/metrics`, port 9090 | Infrastructure metrics |
| phpMyAdmin | `/phpmyadmin`, `/pma` | Database access |
| Adminer | `/adminer.php` | Database access |
| cPanel/WHM | port 2082, 2083, 2086, 2087 | Hosting control panel |
| Tomcat Manager | `/manager/html`, port 8080 | Application deployment |
| Spring Boot Actuator | `/actuator`, `/actuator/env`, `/actuator/health` | Config, env vars, health |
| Webpack Dev Server | port 8080, 3000 | Source code, source maps |
| Jupyter Notebook | port 8888 | Code execution |
| Docker Registry | port 5000, `/v2/_catalog` | Container images |
| Kubernetes Dashboard | port 8443 | Cluster management |
| MinIO Console | port 9001 | Object storage |

### Leak & Secret Search

```bash
# GitHub search for secrets
trufflehog github --org=<TARGET_ORG>
trufflehog github --repo=https://github.com/<TARGET_ORG>/<REPO>

# GitHub dorking
# https://github.com/obheda12/GitDorker
python3 GitDorker.py -t <GITHUB_TOKEN> -d dorks.txt -q "<TARGET_DOMAIN>"

# Manual GitHub dorks
# Search: "<TARGET_DOMAIN>" password
# Search: "<TARGET_DOMAIN>" api_key
# Search: "<TARGET_DOMAIN>" secret
# Search: "<TARGET_DOMAIN>" token

# Paste site monitoring
# https://github.com/d4rckh/pwnbin
# Check: pastebin.com, ghostbin.co, rentry.co

# Data breach search
# https://leakix.net/
# https://search.0t.rocks/
# https://haveibeenpwned.com/
```

### Additional Resources

```
# LeakIX - search for exposed services and data leaks
https://leakix.net/

# Shodan search for target
https://www.shodan.io/search?query=org%3A"<ORG_NAME>"

# BuiltWith - technology profiling
https://builtwith.com/<TARGET_DOMAIN>

# SecurityTrails - historical DNS and subdomain data
https://securitytrails.com/domain/<TARGET_DOMAIN>

# PublicWWW - source code search engine
https://publicwww.com/websites/"<TARGET_DOMAIN>"/
```

## Tips & Edge Cases

- GAU and waymore frequently reveal old admin panels and API endpoints that were removed from the live site but still respond
- AWS resources can be found by scanning certificate transparency logs for the target domain pattern
- Shodan `ssl:` filter catches services running on non-standard ports with the target's TLS certificate
- Spring Boot Actuator endpoints (`/actuator/env`, `/actuator/configprops`) often leak environment variables and secrets
- Grafana, Kibana, and Jenkins instances are frequently left without authentication on internal or staging environments
- Docker registries on port 5000 may allow anonymous pulls of container images containing secrets
- Check for `.env`, `.git/config`, and `robots.txt` on every discovered dashboard URL
- GitHub search is powerful -- search for the org name, domain, internal hostnames, and employee email domains
- BuiltWith and Wappalyzer reveal third-party scripts, analytics, and tracking services that may be hijackable

## Output Format

```
[Source]          [URL / Resource]                                    [Status]
Wayback           https://admin.target.com/old-dashboard              200 - Open
AlienVault        https://api.target.com/swagger.json                 200 - API Docs
Shodan            93.184.216.34:8080 (Jenkins)                        200 - No Auth
AWS Scrape        s3://target-backups.s3.amazonaws.com                Public
GitHub            github.com/target-org/infra (AWS keys in .env)      Leaked
LeakIX            Exposed Elasticsearch on 93.184.216.34:9200         No Auth
```

## Agent Workflow
> Step-by-step instructions for an AI agent to perform third-party service enumeration.

### Phase 1: Setup
1. Gather the target domain, organization name, and known keywords (product names, internal project names)
2. Ensure API keys are configured for:
   - Shodan (`SHODAN_API_KEY`)
   - Censys (`CENSYS_API_ID`, `CENSYS_API_SECRET`)
   - VirusTotal (optional)
3. Install required tools:
   ```
   go install github.com/lc/gau/v2/cmd/gau@latest
   pip install waymore trufflehog
   pip install cloud_enum
   ```
4. Prepare output directory for organized results

### Phase 2: Execution
1. Mine historical URLs from archives:
   ```
   echo "<TARGET_DOMAIN>" | gau --subs --o gau_results.txt
   waymore -i <TARGET_DOMAIN> -mode U -oU waymore_results.txt
   ```
2. Query Shodan for associated infrastructure:
   ```
   shodan search org:"<ORG_NAME>" --fields ip_str,port,hostnames
   shodan search ssl:"<TARGET_DOMAIN>" --fields ip_str,port,hostnames
   ```
3. Enumerate cloud resources:
   ```
   python3 cloud_enum.py -k <TARGET_KEYWORD> -k <TARGET_DOMAIN>
   ```
4. Search for exposed dashboards by filtering archive URLs:
   ```
   cat gau_results.txt | grep -iE "admin|dashboard|panel|console|swagger|graphql|jenkins|grafana|kibana"
   ```
5. Scan for leaked secrets in GitHub repositories:
   ```
   trufflehog github --org=<TARGET_ORG>
   ```
6. Check AlienVault OTX for associated URLs:
   ```
   https://otx.alienvault.com/indicator/domain/<TARGET_DOMAIN>
   ```

### Phase 3: Analysis
1. Categorize discovered services by risk:
   - **Critical**: unauthenticated dashboards (Jenkins, Grafana, Kibana), exposed databases, open S3 buckets
   - **High**: API documentation (Swagger, GraphQL), exposed Actuator endpoints, leaked credentials in GitHub
   - **Medium**: information disclosure in archive URLs, exposed configuration files
   - **Low**: technology fingerprinting data, public documentation
2. Verify unauthenticated access to discovered dashboards and services
3. Cross-reference Shodan/Censys results with subdomain enumeration to identify overlooked hosts
4. Check for `.env`, `.git/config`, and `robots.txt` on every discovered dashboard URL
5. Merge newly discovered hosts into the master subdomain list

### Phase 4: Next Steps
- Feed discovered URLs to [spidering](spidering.md) and [fuzzing](fuzzing.md)
- Feed unauthenticated dashboards to exploitation testing (default credentials, known CVEs)
- Feed cloud storage findings to [AWS](../exploitation/cloud/aws.md), [Azure](../exploitation/cloud/azure.md), or [GCP](../exploitation/cloud/gpc.md) testing
- Feed leaked secrets to credential validation and exploitation
- Feed new hosts/IPs to [probing](probing.md) for alive detection and fingerprinting

## Decision Tree

```
START: Target domain and organization name confirmed
  |
  +--> Mine historical URLs (gau, waymore)
  |      |
  |      +--> Admin/dashboard URLs found? --> Verify access, check for default creds
  |      +--> Config/backup files found? --> Download and analyze for secrets
  |
  +--> Query Shodan/Censys for infrastructure
  |      |
  |      +--> New IPs/hosts found? --> Feed to probing pipeline
  |      +--> Open ports found? --> Service-specific enumeration
  |
  +--> Enumerate cloud resources (cloud_enum, S3Scanner)
  |      |
  |      +--> Public buckets? --> Check for sensitive data
  |      +--> Cloud functions? --> Test for authorization bypass
  |
  +--> Search for leaked secrets (trufflehog, GitHub dorks)
  |      |
  |      +--> Valid credentials? --> Report and exploit
  |      +--> API keys? --> Validate scope and permissions
  |
  +--> Check AlienVault OTX and VirusTotal
         |
         +--> Additional URLs? --> Merge into recon pipeline
```

## Success Criteria

- [ ] Historical URLs mined from Wayback Machine and CommonCrawl
- [ ] Shodan/Censys queried for organization infrastructure
- [ ] Cloud resources enumerated (AWS, Azure, GCP)
- [ ] Exposed dashboards identified and access verified
- [ ] GitHub and paste sites searched for leaked secrets
- [ ] All findings categorized by risk level
- [ ] New hosts merged into master subdomain list
- [ ] Findings handed off to appropriate exploitation phases

## References

- [gau - lc](https://github.com/lc/gau)
- [waymore - xnl-h4ck3r](https://github.com/xnl-h4ck3r/waymore)
- [awsScrape - jhaddix](https://github.com/jhaddix/awsScrape/)
- [cloud_enum - initstring](https://github.com/initstring/cloud_enum)
- [trufflehog - trufflesecurity](https://github.com/trufflesecurity/trufflehog)
- [GitDorker - obheda12](https://github.com/obheda12/GitDorker)
- [LeakIX](https://leakix.net/)
- [AlienVault OTX](https://otx.alienvault.com/)
- [Shodan](https://www.shodan.io/)
- [Censys](https://search.censys.io/)

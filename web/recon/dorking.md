## Google

```
## No longer works, but the wordlists are good
# https://github.com/m3n0sd0n4ld/uDork

# https://dorks.faisalahmed.me/
# Google dork helper, input url and the sites generates dorks
```

```
site:npmjs.com "company"
site:npm.runkit.com "company"
site:pastebin.com "company"
site:bitbucket.org "company"
site:*.atlassian.net "company"
site:trello.com "company"
site:prezi.com "company"
inurl:gitlab.com "company"
```

## Github

```
#https://github.com/obheda12/GitDorker
python3 GitDorker.py -tf github_tokens -q example.com -p -ri -d Dorks/medium_dorks.txt -o gitdorker_out.txt

# https://vsec7.github.io/
# Git dork helper, input url and the site generates dorks
```

```
org:company "firebase"
org:company "password"
org:company "bucket_name"
org:company "aws_access_key"
org:company "aws_secret_key"
org:company "S3_BUCKET"
org:company "S3_ACCESS_KEY_ID"
org:company "S3_SECRET_ACCESS_KEY"
org:company "S3_ENDPOINT"
org:company "AWS_ACCESS_KEY_ID"
org:company "list_aws_accounts"
```

[https://gist.github.com/win3zz/0a1c70589fcbea64dba4588b93095855](https://gist.github.com/win3zz/0a1c70589fcbea64dba4588b93095855)
## Examples:

[](https://gist.github.com/win3zz/0a1c70589fcbea64dba4588b93095855#examples)

**1. OpenAI API keys**

`(path:*.xml OR path:*.json OR path:*.properties OR path:*.sql OR path:*.txt OR path:*.log OR path:*.tmp OR path:*.backup OR path:*.bak OR path:*.enc OR path:*.yml OR path:*.yaml OR path:*.toml OR path:*.ini OR path:*.config OR path:*.conf OR path:*.cfg OR path:*.env OR path:*.envrc OR path:*.prod OR path:*.secret OR path:*.private OR path:*.key) AND (access_key OR secret_key OR access_token OR api_key OR apikey OR api_secret OR apiSecret OR app_secret OR application_key OR app_key OR appkey OR auth_token OR authsecret) AND ("sk-" AND (openai OR gpt))`

**Update:** We can use following refined regular expression to filters out most dummy keys:

`... AND (/sk-[a-zA-Z0-9]{48}/ AND (openai OR gpt))`

Special thanks to [@fkulakov](https://gist.github.com/fkulakov) for the insightful contribution.

## Screeenshot:

[![GithubOpenAIAPIkeysSearch](https://user-images.githubusercontent.com/12781459/246651397-910a2268-3c0f-49ec-9c17-ff435bbabf35.png)](https://user-images.githubusercontent.com/12781459/246651397-910a2268-3c0f-49ec-9c17-ff435bbabf35.png)

**2. Github OAuth/App/Personal/Refresh Access Token**

`(path:*.xml OR path:*.json OR path:*.properties OR path:*.sql OR path:*.txt OR path:*.log OR path:*.tmp OR path:*.backup OR path:*.bak OR path:*.enc OR path:*.yml OR path:*.yaml OR path:*.toml OR path:*.ini OR path:*.config OR path:*.conf OR path:*.cfg OR path:*.env OR path:*.envrc OR path:*.prod OR path:*.secret OR path:*.private OR path:*.key) AND (access_key OR secret_key OR access_token OR api_key OR apikey OR api_secret OR apiSecret OR app_secret OR application_key OR app_key OR appkey OR auth_token OR authsecret) AND (("ghp_" OR "gho_" OR "ghu_" OR "ghs_" OR "ghr_") AND (Github OR OAuth))`

**3. Slack Token**

`(path:*.xml OR path:*.json OR path:*.properties OR path:*.sql OR path:*.txt OR path:*.log OR path:*.tmp OR path:*.backup OR path:*.bak OR path:*.enc OR path:*.yml OR path:*.yaml OR path:*.toml OR path:*.ini OR path:*.config OR path:*.conf OR path:*.cfg OR path:*.env OR path:*.envrc OR path:*.prod OR path:*.secret OR path:*.private OR path:*.key) AND (access_key OR secret_key OR access_token OR api_key OR apikey OR api_secret OR apiSecret OR app_secret OR application_key OR app_key OR appkey OR auth_token OR authsecret) AND (xox AND Slack)`

**4. Google API key**

`(path:*.xml OR path:*.json OR path:*.properties OR path:*.sql OR path:*.txt OR path:*.log OR path:*.tmp OR path:*.backup OR path:*.bak OR path:*.enc OR path:*.yml OR path:*.yaml OR path:*.toml OR path:*.ini OR path:*.config OR path:*.conf OR path:*.cfg OR path:*.env OR path:*.envrc OR path:*.prod OR path:*.secret OR path:*.private OR path:*.key) AND (access_key OR secret_key OR access_token OR api_key OR apikey OR api_secret OR apiSecret OR app_secret OR application_key OR app_key OR appkey OR auth_token OR authsecret) AND (AIza AND Google)`

**5. Square OAuth/access token**

`(path:*.xml OR path:*.json OR path:*.properties OR path:*.sql OR path:*.txt OR path:*.log OR path:*.tmp OR path:*.backup OR path:*.bak OR path:*.enc OR path:*.yml OR path:*.yaml OR path:*.toml OR path:*.ini OR path:*.config OR path:*.conf OR path:*.cfg OR path:*.env OR path:*.envrc OR path:*.prod OR path:*.secret OR path:*.private OR path:*.key) AND (access_key OR secret_key OR access_token OR api_key OR apikey OR api_secret OR apiSecret OR app_secret OR application_key OR app_key OR appkey OR auth_token OR authsecret) AND (("sq0atp-" OR "sq0csp-") AND (square OR OAuth))`

**6. Shopify shared secret, access token, private/custom app access token**

`(path:*.xml OR path:*.json OR path:*.properties OR path:*.sql OR path:*.txt OR path:*.log OR path:*.tmp OR path:*.backup OR path:*.bak OR path:*.enc OR path:*.yml OR path:*.yaml OR path:*.toml OR path:*.ini OR path:*.config OR path:*.conf OR path:*.cfg OR path:*.env OR path:*.envrc OR path:*.prod OR path:*.secret OR path:*.private OR path:*.key) AND (access_key OR secret_key OR access_token OR api_key OR apikey OR api_secret OR apiSecret OR app_secret OR application_key OR app_key OR appkey OR auth_token OR authsecret) AND (("shpss_" OR "shpat_" OR "shpca_" OR "shppa_") AND "Shopify")`

## Parameters Used

[](https://gist.github.com/win3zz/0a1c70589fcbea64dba4588b93095855#parameters-used)

### File Extensions

[](https://gist.github.com/win3zz/0a1c70589fcbea64dba4588b93095855#file-extensions)

|File Extension|Description|
|:--|:--|
|.xml|XML file format|
|.json|JSON (JavaScript Object Notation) file format|
|.properties|Properties file format used for configuration settings|
|.sql|SQL (Structured Query Language) file format used for database queries|
|.txt|Plain text file format|
|.log|Log file format used for recording events or activities|
|.tmp|Temporary file format|
|.backup|Backup file format|
|.bak|Backup file format|
|.enc|Encrypted file format|
|.yml|YAML (YAML Ain't Markup Language) file format used for configuration settings|
|.yaml|YAML (YAML Ain't Markup Language) file format used for configuration settings|
|.toml|TOML (Tom's Obvious, Minimal Language) file format used for configuration settings|
|.ini|INI (Initialization) file format used for configuration settings|
|.config|Configuration file format|
|.conf|Configuration file format|
|.cfg|Configuration file format|
|.env|Environment file format|
|.envrc|Environment file format specific to the Direnv tool|
|.prod|Production file format|
|.secret|Secret file format|
|.private|Private file format|
|.key|Key file format|

### Keynames

[](https://gist.github.com/win3zz/0a1c70589fcbea64dba4588b93095855#keynames)

|Keynames|Description|
|:--|:--|
|access_key|Variable name to store the key used for accessing a resource or service|
|secret_key|Variable name to store the key used for authentication or encryption|
|access_token|Variable name to store the token used for accessing an API or resource|
|api_key|Variable name to store the key used for accessing an API or service|
|apikey|Shortened version of "api_key"|
|api_secret|Variable name to store the secret key used for API authentication|
|apiSecret|An alternate of "api_secret"|
|app_secret|Variable name to store the secret key used for application authentication|
|application_key|Variable name to store the key used for identifying an application|
|app_key|Variable name to store the key used for identifying an application|
|appkey|Shortened version of "app_key"|
|auth_token|Variable name to store the token used for authentication or authorization|
|authsecret|Variable name to store the secret key used for authentication or authorization|

## Other Useful Tools:

[](https://gist.github.com/win3zz/0a1c70589fcbea64dba4588b93095855#other-useful-tools)

- Online IDE Search: [https://redhuntlabs.com/online-ide-search/](https://redhuntlabs.com/online-ide-search/)
- Keyhacks on GitHub: [https://github.com/streaak/keyhacks](https://github.com/streaak/keyhacks)
- Google Hacking Database: [https://www.exploit-db.com/google-hacking-database](https://www.exploit-db.com/google-hacking-database)

## Agent Workflow
> Step-by-step instructions for an AI agent to perform dorking-based reconnaissance.

### Phase 1: Setup
1. Identify the target organization name, domain(s), and any known employee names or email patterns
2. Prepare GitHub tokens for authenticated GitHub search (store in environment variable)
3. Confirm scope boundaries -- which domains, repos, and organizations are in scope
4. Install required tools:
   ```
   pip install GitDorker
   ```

### Phase 2: Execution
1. Run Google dorks for the target domain to find exposed services and data:
   ```
   site:<TARGET_DOMAIN> ext:sql | ext:env | ext:log | ext:bak | ext:conf
   site:<TARGET_DOMAIN> inurl:admin | inurl:dashboard | inurl:login
   site:pastebin.com "<TARGET_DOMAIN>"
   site:trello.com "<COMPANY_NAME>"
   site:*.atlassian.net "<COMPANY_NAME>"
   ```
2. Run GitHub dorking for leaked secrets:
   ```
   python3 GitDorker.py -tf <GITHUB_TOKENS_FILE> -q <TARGET_DOMAIN> -d Dorks/medium_dorks.txt -o gitdorker_out.txt
   ```
3. Manually search GitHub for organization-specific secrets:
   ```
   org:<COMPANY> "aws_access_key"
   org:<COMPANY> "password"
   org:<COMPANY> "api_key"
   org:<COMPANY> "firebase"
   ```
4. Search for exposed API keys using GitHub code search with file extension filters
5. Validate discovered keys using [Keyhacks](https://github.com/streaak/keyhacks)

### Phase 3: Analysis
1. Categorize findings by severity:
   - **Critical**: valid cloud credentials (AWS keys, GCP service accounts), database passwords
   - **High**: API keys with privileged access, OAuth tokens, internal URLs
   - **Medium**: expired or rotated keys, internal documentation, employee information
   - **Low**: public information, non-sensitive configuration files
2. Validate discovered credentials against their respective services
3. Cross-reference exposed domains and URLs with [subdomain enumeration](subdomain/passive.md) results
4. Document unique findings not discovered through other recon methods

### Phase 4: Next Steps
- Feed discovered URLs to [spidering](spidering.md) and [fuzzing](fuzzing.md)
- Feed discovered credentials to targeted authentication testing
- Feed discovered cloud resources to [AWS](../exploitation/cloud/aws.md), [Azure](../exploitation/cloud/azure.md), or [GCP](../exploitation/cloud/gpc.md) testing
- Feed discovered API keys to API-specific vulnerability testing
- Report any valid leaked credentials as immediate findings

## Decision Tree

```
START: Target domain and organization name confirmed
  |
  +--> Run Google dorks (site:, inurl:, ext:)
  |      |
  |      +--> Exposed files found? --> Download and analyze
  |      +--> Admin panels found? --> Feed to auth testing
  |
  +--> Run GitHub dorks (org:, keyword search)
  |      |
  |      +--> Secrets found? --> Validate with Keyhacks
  |      |      |
  |      |      +--> Valid? --> Report as critical finding
  |      |      +--> Invalid/rotated? --> Note for context
  |      |
  |      +--> Internal URLs found? --> Feed to recon pipeline
  |
  +--> Search paste sites and third-party platforms
  |      |
  |      +--> Credentials found? --> Test against target services
  |
  +--> Cross-reference all findings with other recon phases
```

## Success Criteria

- [ ] Google dorks executed for target domain (site:, inurl:, ext: operators)
- [ ] GitHub dorking completed for organization and domain
- [ ] Discovered API keys validated using Keyhacks
- [ ] Paste sites and third-party platforms searched
- [ ] Findings categorized by severity
- [ ] Valid credentials reported and handed off for exploitation
- [ ] Discovered URLs fed into the recon pipeline

# Web Technologies

## Technology Fingerprinting

### Tools

```bash
# Wappalyzer CLI
wappalyzer https://TARGET

# WhatWeb
whatweb https://TARGET -a 3

# httpx with tech detection
echo TARGET | httpx -tech-detect -status-code -title

# Webanalyze (Go-based Wappalyzer)
webanalyze -host https://TARGET

# BuiltWith (online)
# https://builtwith.com/
```

### Manual Indicators

| Header / Pattern | Technology |
|---|---|
| `X-Powered-By: Express` | Node.js / Express |
| `X-Powered-By: PHP/x.x` | PHP |
| `X-AspNet-Version` | ASP.NET |
| `Server: Apache` | Apache HTTPD |
| `Server: nginx` | Nginx |
| `X-Drupal-Cache` | Drupal CMS |
| `X-Generator: WordPress` | WordPress |
| `__viewstate` in forms | ASP.NET Web Forms |
| `csrfmiddlewaretoken` | Django |
| `_rails` cookie prefix | Ruby on Rails |
| `JSESSIONID` cookie | Java (Tomcat/JBoss) |
| `laravel_session` cookie | Laravel (PHP) |
| `ci_session` cookie | CodeIgniter (PHP) |
| `connect.sid` cookie | Express/Node.js |

---

## Adobe AEM

- [aem-hacker](https://github.com/0ang3el/aem-hacker)
- [aemscan](https://github.com/Raz0r/aemscan)
- [aem-paths.txt](https://github.com/emadshanab/Adobe-Experience-Manager/blob/main/aem-paths.txt)
- [Writeup: AEM exploitation](https://medium.com/@SecTech/adobe-experience-manager-exploitation-24bd9eb75ed9)

## Adobe ColdFusion

- [Basic Recon to RCE](https://www.jomar.fr/posts/2021/basic_recon_to_rce/)

## API

- [API-Security-Checklist](https://github.com/shieldfy/API-Security-Checklist)
- [api-testing-checklist](https://hackanythingfor.blogspot.com/2020/07/api-testing-checklist.html)
- [31-days-of-API-Security-Tips](https://github.com/inonshk/31-days-of-API-Security-Tips)
- [awesome-api-security](https://github.com/arainho/awesome-api-security)
- [OpenAPI Scanner](https://github.com/ngalongc/openapi_security_scanner)
- [MindAPI](https://dsopas.github.io/MindAPI/play/)

## Apache

- [Exploit default Apache servlet](https://infosecwriteups.com/apache-example-servlet-leads-to-61a2720cac20)

## Apache Struts2

Endpoints with `.action`, `.do`, `.go` are using Struts2.

```
Content-Type: %{#context['com.opensymphony.xwork2.dispatcher.HttpServletResponse'].addHeader('Added Header',4*4)}.multipart/form-data
```

## Artifactory

- [Artifactory Hacking Guide](https://book.hacktricks.xyz/pentesting/pentesting-web/artifactory-hacking-guide)

## ASP.NET / IIS

- ASPX and ASP.NET have `__VIEWSTATE` in cookie values; default ASP does not
- [Exploiting ViewState Deserialization](https://notsosecure.com/exploiting-viewstate-deserialization-using-blacklist3r-and-ysoserial-net#PoC)
- [viewgen](https://github.com/0xacb/viewgen)
- [IIS shortname scanner](https://github.com/sw33tLie/sns)
- [ASP.NET XSS](https://blog.isec.pl/all-is-xss-that-comes-to-the-net/)
- [.NET Remoting Exploitation](https://code-white.com/blog/leaking-objrefs-to-exploit-http-dotnet-remoting/)

```
# Interesting paths
trace.axd
any.aspx/trace.axd
WEB-INF/web.xml
con/
aux/
con.aspx
aux.aspx
```

## Auth0

- [Auth0 Misconfiguration Bypass](https://amjadali110.medium.com/how-i-exploited-an-auth0-misconfiguration-to-bypass-login-restrictions-c5d8c20d5505)

## Axis2

- [Hidden RCE on Axis2 Instance](https://medium.com/@domenicoveneziano/hidden-in-plain-sight-uncovering-rce-on-a-forgotten-axis2-instance-86ddc91f1415)

## Cloudflare

Techniques to uncover origin servers behind Cloudflare:

- Search domain in [leaked.site](https://leaked.site/index.php?resolver%2Fcloudflare.0%2F=)
- [CloudFlair](https://github.com/christophetd/CloudFlair) -- find origin servers using certificate transparency
- Historical DNS records (SecurityTrails, ViewDNS)
- Search Shodan/Censys for the site title or unique response body

## Cockpit CMS

- [RCE in Cockpit CMS](https://swarm.ptsecurity.com/rce-cockpit-cms/)

## Django

- Try to `POST` to `/admin` -- data may leak in error pages
- [SQLi via date parameter (CVE-2022-34265)](https://github.com/vulhub/vulhub/tree/master/django/CVE-2022-34265)
- Shodan dork for DEBUG mode: `http.title:"DisallowedHost at /"`

## Flask

- [Flask-Unsign](https://github.com/Paradoxis/Flask-Unsign) -- decode, brute-force, and craft Flask session cookies

## GraphQL

- [GraphQL exploitation guide](https://blog.yeswehack.com/yeswerhackers/how-exploit-graphql-endpoint-bug-bounty/)
- [Voyager](https://ivangoncharov.github.io/graphql-voyager/) -- visual introspection
- [clairvoyance](https://github.com/nikitastupin/clairvoyance) -- brute-force introspection when disabled
- [graphw00f](https://github.com/dolevf/graphw00f) -- fingerprint GraphQL engines
- [GraphiQL Explorer](https://cloud.hasura.io/public/graphiql)
- [Awesome GraphQL Security](https://github.com/Escape-Technologies/awesome-graphql-security)

See also: [GraphQL Attacks](../exploitation/vulns/graphql.md)

## Intercom

Interact with an email, log off, then run the command below with the same email. If "Enforce Identity Validation" is not set up, you can view conversation history.

```javascript
Intercom('boot', { email: '<TARGET_EMAIL>' });
```

- [Writeup](http://dday.us/2021/11/03/h1vendorATO.html)

## Java RMI

- [BaRMIe](https://github.com/NickstaDB/BaRMIe)

## JBoss

- [jexboss](https://github.com/joaomatosf/jexboss)

## Jenkins

- [pwn_jenkins](https://github.com/gquere/pwn_jenkins)

```
JENKINS_IP/PROJECT/securityRealm/user/admin
JENKINS_IP/jenkins/script
```

## Jetty

RCE by hotdeploy is enabled by default.

- [Jetty features for hacking](https://swarm.ptsecurity.com/jetty-features-for-hacking-web-apps/)

## Jira

```
# Check unauthenticated access -- users should not have privileges
/rest/api/2/mypermissions
/rest/api/3/mypermissions
```

- [jirascan](https://github.com/bcoles/jira_scan)
- [jiraffe](https://github.com/0x48piraj/Jiraffe)

## JSON Web Tokens

- [jwt_tool](https://github.com/ticarpi/jwt_tool)
- [jwt-pwn](https://github.com/mazen160/jwt-pwn)
- Hashcat brute force: mode 16500

See also: [JWT Attacks](../exploitation/authentication/jwt.md)

## Meteor

- [Wekan Authentication Bypass](https://www.offsec.com/offsec/wekan-authentication-bypass/)

## MongoDB

[Mongo IDs can lead to IDOR](https://www.mickaelwalter.fr/idor-with-mongodb-understanding-objectid/)

```
username[$ne]=toto&password[$ne]=toto

{"username": {"$ne": null}, "password": {"$ne": null}}
{"username": {"$gt":""}, "password": {"$gt":""}}
```

See also: [NoSQL Injection](../exploitation/vulns/nosql.md)

## Next.js

Look for `_buildManifest.js` in source code -- it exposes routes inside `sortedPages`.

```javascript
console.log(__BUILD_MANIFEST.sortedPages)
```

## Node.js / Express

1. If `X-Powered-By: Express` is present and HTML is in responses, server-side templating is likely in use
2. Add `layout` to parameter discovery wordlists (GET query or POST body)
3. If an arbitrary value for `layout` returns 500 with `ENOENT: no such file or directory`, you have Local File Read

- [Secret Parameter LFR in NodeJS Apps](https://blog.shoebpatel.com/2021/01/23/The-Secret-Parameter-LFR-and-Potential-RCE-in-NodeJS-Apps/)

## Pentaho

- [Pentah0wnage](https://research.aurainfosec.io/pentest/pentah0wnage/)

## Ruby on Rails

- Add `.json` to the end of endpoints
- Ruby server-side JS rendering returns `application/javascript`; this can be embedded to leak info
- Add `_method=VERB` in body to override HTTP method
- [String Interpolation RCE](https://buer.haus/2017/03/13/airbnb-ruby-on-rails-string-interpolation-led-to-remote-code-execution/)

## Salesforce

- [Salesforce Exploitation](https://web.archive.org/web/20210812053905/enumerated.de/index/salesforce)
- [sret](https://github.com/reconstation/sret)
- [cirrusgo](https://github.com/Ph33rr/cirrusgo)

## SAP

- [SAP_RECON](https://github.com/chipik/SAP_RECON)
- [SAP-wordlist](https://github.com/emadshanab/SAP-wordlist/blob/main/SAP-wordlist.txt)
- [mySapAdventures](https://github.com/shipcod3/mySapAdventures)

## ServiceNow

```
# Unauthenticated endpoint that sometimes returns data
kb_view_customer.do?sysparm_article=KB00XXXXX
```

## SharePoint

- [SharePoint API Misconfigurations](https://medium.com/@ujmalhotra95/tales-of-sharepoint-api-misconfigurations-11073ad384fd)

## Spring Boot

- [Spring Boot Security Introduction](https://tutorialboy24.blogspot.com/2022/02/introduction-to-spring-boot-related.html)
- [Spring Boot Actuators](https://0xn3va.gitbook.io/cheat-sheets/framework/spring/spring-boot-actuators)
- [Spring RCE](https://f002.backblazeb2.com/file/sec-news-backup/files/writeup/deadpool.sh/_2017_RCE_Springs_/index.html)
- [Exploiting APLS from Spring Data REST](https://niemand.com.ar/2021/01/08/exploiting-application-level-profile-semantics-apls-from-spring-data-rest/)

Common actuator endpoints to check:

```
/actuator
/actuator/env
/actuator/health
/actuator/heapdump
/actuator/mappings
/actuator/configprops
/actuator/beans
```

## Swagger / OpenAPI

- [Hacking Swagger UI (XSS to ATO)](https://www.vidocsecurity.com/blog/hacking-swagger-ui-from-xss-to-account-takeovers/)
- [springfox-swagger-xss](https://github.com/seanmarpo/springfox-swagger-xss)
- [swagroutes](https://github.com/amalmurali47/swagroutes) -- extract routes from Swagger docs
- [sj](https://github.com/BishopFox/sj) -- Swagger/OpenAPI auditing

## Symfony

- [Symfony Pentesting](https://book.hacktricks.xyz/pentesting/pentesting-web/symphony)

## Telerik Web UI

- [Pwning with Telerik](https://captmeelo.com/pentest/2018/08/03/pwning-with-telerik.html)

## Tomcat

- Check for `WEB-INF/web.xml` exposure
- Default manager creds: `tomcat:tomcat`, `admin:admin`
- Manager path: `/manager/html`, `/host-manager/html`

## Traccar 5

- [Traccar RCE Vulnerabilities](https://www.horizon3.ai/attack-research/disclosures/traccar-5-remote-code-execution-vulnerabilities/)

## WebDAV

- Methods `PROPPATCH`, `PROPFIND`, and `LOCK` accept XML input (XXE potential)
- [Exploiting OOB XXE via WebDAV](https://dhiyaneshgeek.github.io/web/security/2021/02/19/exploiting-out-of-band-xxe/)

## WebLogic

- [WebLogic SSRF with Deserialized JDBC](https://pyn3rd.github.io/2022/06/18/Weblogic-SSRF-Involving-Deserialized-JDBC-Connection/)

## WordPress

See also: [wp-cve-boilerplate.md](wp-cve-boilerplate.md)

```bash
# WPScan
wpscan --url https://TARGET --enumerate u,p,t --api-token TOKEN

# Check for XML-RPC
curl -X POST https://TARGET/xmlrpc.php -d '<methodCall><methodName>system.listMethods</methodName></methodCall>'
```

## Agent Workflow
> Step-by-step instructions for an AI agent to perform technology fingerprinting and identify technology-specific attack vectors.

### Phase 1: Setup
1. Obtain the list of alive hosts from [probing](../recon/probing.md) results
2. Ensure technology detection tools are available:
   - httpx with `-tech-detect` flag
   - WhatWeb (`whatweb`)
   - Wappalyzer CLI or browser extension
3. Prepare a checklist of technologies with known attack vectors (this file)

### Phase 2: Execution
1. Run automated technology detection on all alive hosts:
   ```
   cat alive_hosts.txt | httpx -tech-detect -sc -title -server -o tech_results.txt
   whatweb -i alive_hosts.txt -a 3 --log-brief whatweb_results.txt
   ```
2. Manually inspect response headers for technology indicators:
   - `X-Powered-By`, `Server`, `X-AspNet-Version`, `X-Generator`
   - Cookie names: `JSESSIONID` (Java), `laravel_session` (Laravel), `connect.sid` (Node.js), `ci_session` (CodeIgniter)
   - HTML source: `__VIEWSTATE` (ASP.NET), `csrfmiddlewaretoken` (Django), `_rails` (Rails)
3. Check for technology-specific paths:
   ```
   # Spring Boot Actuator
   curl -s -o /dev/null -w "%{http_code}" <TARGET>/actuator/health
   # Drupal
   curl -s -o /dev/null -w "%{http_code}" <TARGET>/CHANGELOG.txt
   # WordPress
   curl -s -o /dev/null -w "%{http_code}" <TARGET>/wp-login.php
   # Joomla
   curl -s -o /dev/null -w "%{http_code}" <TARGET>/administrator/
   ```
4. Check for API documentation endpoints:
   ```
   curl -s -o /dev/null -w "%{http_code}" <TARGET>/swagger-ui.html
   curl -s -o /dev/null -w "%{http_code}" <TARGET>/api-docs
   curl -s -o /dev/null -w "%{http_code}" <TARGET>/graphql
   ```

### Phase 3: Analysis
1. Map each target to its technology stack (web server, framework, CMS, language)
2. For each identified technology, look up known attack vectors:
   - **WordPress**: see [WordPress attacks](../exploitation/cms/wordpress.md)
   - **Drupal/Joomla**: see [Other CMS attacks](../exploitation/cms/others.md)
   - **Spring Boot**: check Actuator endpoints for info disclosure
   - **ASP.NET**: test ViewState deserialization, IIS shortname scanning
   - **Node.js/Express**: test for `layout` parameter LFI
   - **Django**: test admin page, debug mode information disclosure
   - **GraphQL**: test introspection, see [GraphQL attacks](../exploitation/vulns/graphql.md)
3. Check for known CVEs matching the exact version detected
4. Group targets by technology for batch testing

### Phase 4: Next Steps
- Feed CMS targets to CMS-specific scanners (WPScan, droopescan, joomscan)
- Feed API documentation to API-specific vulnerability testing
- Feed Actuator/debug endpoints to information disclosure analysis
- Feed version-specific CVE matches to exploitation
- Feed technology groupings to targeted fuzzing with appropriate wordlists

## Decision Tree

```
START: Alive hosts list from probing
  |
  +--> Run technology detection (httpx -tech-detect, whatweb)
  |
  +--> Inspect headers, cookies, HTML source manually
  |
  +--> Categorize by technology:
         |
         +--> CMS detected (WordPress, Drupal, Joomla)?
         |      --> CMS-specific scanner + known CVEs
         |
         +--> Framework detected (Spring, Django, Rails, Express)?
         |      --> Framework-specific paths + debug endpoints
         |
         +--> API docs found (Swagger, GraphQL)?
         |      --> Full API attack surface testing
         |
         +--> Java/ASP.NET detected?
         |      --> Deserialization testing + technology-specific vulns
         |
         +--> Unknown/custom application?
                --> Deep manual testing + fuzzing
```

## Success Criteria

- [ ] Automated technology detection run on all alive hosts
- [ ] Response headers and cookies manually inspected for additional indicators
- [ ] Technology-specific paths probed (Actuator, CHANGELOG.txt, wp-login.php, etc.)
- [ ] API documentation endpoints checked (Swagger, GraphQL)
- [ ] Each target mapped to its technology stack
- [ ] Known CVEs researched for detected versions
- [ ] Targets grouped by technology for batch testing
- [ ] Findings handed off to technology-specific exploitation phases

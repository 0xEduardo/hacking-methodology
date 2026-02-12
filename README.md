# Hacking Methodology

Bug bounty and web application security testing methodology.

All the information here and on the following pages was gathered through reading articles and books, as well as watching videos and talks. None of this content is exclusive, so feel free to copy and share it freely.

It is highly recommended to open this document with [Obsidian](https://obsidian.md/).

---

## Infrastructure

### Recon

- [ASN / CIDR](infra/recon/asn-cidr.md)
- [Whois](infra/recon/whois.md)
- [Company Information](infra/recon/company-information.md)
- [Port Scan](infra/recon/port-scan.md)
- [Enumerating Services](infra/recon/services.md)

### Exploitation

- [Attacking Services](infra/exploitation/attacking-services.md)
- [Password Attacks](infra/exploitation/password-attacks.md)

### Helpers

- [Pivoting, Tunneling, and Port Forwarding](infra/helpers/pivoting.md)
- [Linux Privilege Escalation](infra/helpers/linux-privesc.md)
- [Windows Privilege Escalation](infra/helpers/windows-privesc.md)

---

## Web Recon

### Subdomain Enumeration

- [Passive](web/recon/subdomain/passive.md)
- [Active](web/recon/subdomain/active.md)

### Discovery

- [Probing](web/recon/probing.md)
- [Screenshot](web/recon/screenshot.md)
- [Dorking](web/recon/dorking.md)
- [Spidering](web/recon/spidering.md)
- [Fuzzing](web/recon/fuzzing.md)
- [Param Discovery](web/recon/param-discovery.md)
- [JavaScript Analysis](web/recon/javascript-analysis.md)
- [Third Party](web/recon/third-part.md)

---

## Web Exploitation

### Authentication

- [JWT](web/exploitation/authentication/jwt.md)
- [SAML](web/exploitation/authentication/saml.md)
- [OAuth](web/exploitation/authentication/oauth.md)
- [2FA Bypass](web/exploitation/authentication/2fa.md)
- [Registration Vulnerabilities](web/exploitation/authentication/registration-vulns.md)

### Bypass

- [403 Bypass](web/exploitation/bypass/403.md)
- [WAF Bypass](web/exploitation/bypass/waf.md)

### Cloud

- [AWS](web/exploitation/cloud/aws.md)
- [Azure](web/exploitation/cloud/azure.md)
- [GPC](web/exploitation/cloud/gpc.md)
- [Others](web/exploitation/cloud/others.md)

### CMS

- [WordPress](web/exploitation/cms/wordpress.md)
- [Others](web/exploitation/cms/others.md)

### Injection

- [XSS](web/exploitation/vulns/xss.md)
- [SQLi](web/exploitation/vulns/sqli.md)
- [NoSQL Injection](web/exploitation/vulns/nosql.md)
- [Command Injection](web/exploitation/vulns/command-injection.md)
- [SSTI](web/exploitation/vulns/ssti.md)
- [CSTI](web/exploitation/vulns/csti.md)
- [SSI](web/exploitation/vulns/ssi.md)
- [LDAP Injection](web/exploitation/vulns/ldap-injection.md)
- [X-Path Injection](web/exploitation/vulns/x-path-injection.md)
- [XSLT Injection](web/exploitation/vulns/xslt-injection.md)
- [Email Injection](web/exploitation/vulns/email-injection.md)
- [Formula Injection](web/exploitation/vulns/formula-injection.md)
- [Unicode Injection](web/exploitation/vulns/unicode-injection.md)

### Client-Side

- [Clickjacking](web/exploitation/vulns/clickjacking.md)
- [CSRF](web/exploitation/vulns/csrf.md)
- [CORS Misconfiguration](web/exploitation/vulns/cors.md)
- [Prototype Pollution](web/exploitation/vulns/prototype-pollution.md)
- [PostMessage Vulnerabilities](web/exploitation/vulns/postmessage-vulns.md)
- [XS-Leaks](web/exploitation/vulns/xs-leaks.md)
- [Cookie-Based Attacks](web/exploitation/vulns/cookie-based-attacks.md)
- [CSP Bypass](web/exploitation/vulns/csp-bypass.md)

### Server-Side

- [SSRF](web/exploitation/vulns/ssrf.md)
- [XXE](web/exploitation/vulns/xxe.md)
- [LFI](web/exploitation/vulns/lfi.md)
- [Deserialization](web/exploitation/vulns/deserialization.md)
- [Request Smuggling](web/exploitation/vulns/request-smuggling.md)
- [Host Header Injection](web/exploitation/vulns/host-header-injection.md)
- [CRLF Injection](web/exploitation/vulns/crlf.md)
- [PDF Generation Vulnerabilities](web/exploitation/vulns/pdf-generation.md)
- [Dependency Confusion](web/exploitation/vulns/dependency-confusion.md)

### Access Control

- [IDOR](web/exploitation/vulns/idor.md)
- [Mass Assignment](web/exploitation/vulns/mass-assignment.md)
- [Parameter Pollution](web/exploitation/vulns/parameter-pollution.md)
- [Open Redirect](web/exploitation/vulns/open-redirect.md)

### Account Security

- [Account Takeover](web/exploitation/vulns/account-takeover.md)
- [Reset Password](web/exploitation/vulns/reset-password.md)
- [Captcha Bypass](web/exploitation/vulns/captcha-bypass.md)
- [Rate Limit Bypass](web/exploitation/vulns/rate-limit-bypass.md)
- [Payment Bypass](web/exploitation/vulns/payment-bypass.md)

### Caching

- [Web Cache Poisoning](web/exploitation/vulns/cache-poisoning.md)
- [Web Cache Deception](web/exploitation/vulns/cache-deception.md)

### Protocol and Infrastructure

- [HTTP/TLS Attacks](web/exploitation/vulns/http-tls-attacks.md)
- [WebSocket Attacks](web/exploitation/vulns/websocket-attacks.md)
- [Race Conditions](web/exploitation/vulns/race-conditions.md)
- [Subdomain Takeover](web/exploitation/vulns/subdomain-takeover.md)
- [IIS](web/exploitation/vulns/iis.md)

### API

- [GraphQL](web/exploitation/vulns/graphql.md)

### Other

- [File Upload](web/exploitation/vulns/file-upload.md)
- [Session Puzzling](web/exploitation/vulns/session-puzzling.md)
- [Whitebox Pentesting](web/exploitation/vulns/whitebox-pentesting.md)

---

## Web Helpers

- [Brute Force](web/helpers/brute-force.md)
- [Web Technologies](web/helpers/web-technologies.md)
- [SMS Verification](web/helpers/sms-verification.md)
- [WordPress CVE Boilerplates](web/helpers/wp-cve-boilerplate.md)
- [Burp Suite](web/helpers/burp-suite.md)
- [File Transfer](web/helpers/file-transfer.md)
- [Shells and Payloads](web/helpers/shells-and-payloads.md)

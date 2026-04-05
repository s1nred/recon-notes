# 05 — OSINT (Passive Reconnaissance)

> Goal: gather intelligence without touching the target directly.
> No active connections — zero traces on target systems.

---

## 5.1 Google Dorks

```
# Find subdomains
site:target.com -www

# Specific file types
site:target.com filetype:pdf
site:target.com filetype:xls
site:target.com filetype:sql
site:target.com filetype:log

# Login pages
site:target.com inurl:login
site:target.com inurl:admin
site:target.com intitle:"login"
site:target.com intitle:"admin panel"

# Exposed config/sensitive files
site:target.com inurl:.env
site:target.com inurl:config
site:target.com ext:conf
site:target.com ext:bak

# Error pages revealing tech stack
site:target.com intext:"Warning: mysql_"
site:target.com intext:"PHP Parse error"
site:target.com intext:"Fatal error"

# Exposed cameras/devices (general)
intitle:"webcam" inurl:view.shtml
intitle:"Network Camera" inurl:ViewerFrame

# Find related sites
related:target.com

# Cache
cache:target.com
```

**Reference:** [exploit-db.com/google-hacking-database](https://www.exploit-db.com/google-hacking-database)

---

## 5.2 theHarvester — Email & Domain OSINT

```bash
# Basic email/subdomain harvesting
theHarvester -d target.com -b all

# Specific sources
theHarvester -d target.com -b google,bing,duckduckgo

# LinkedIn (people enumeration)
theHarvester -d target.com -b linkedin

# Save results
theHarvester -d target.com -b all -f results.html

# Limit results
theHarvester -d target.com -b google -l 100
```

**Sources available:** google, bing, duckduckgo, github, linkedin, shodan, hunter, anubis, certspotter, etc.

---

## 5.3 Shodan

```bash
# CLI usage
shodan init YOUR_API_KEY
shodan search "target.com"
shodan host 10.10.10.5

# Find specific tech
shodan search "apache 2.4.49"
shodan search 'http.title:"target.com"'
shodan search 'ssl.cert.subject.CN:"*.target.com"'
shodan search 'org:"Target Company Name"'

# Count results
shodan count 'org:"Target Company"'
```

### Useful Shodan Filters

```
hostname:target.com
org:"Company Name"
net:10.10.10.0/24
port:3389
os:"Windows Server 2019"
product:"Apache httpd"
http.html:"Powered by WordPress"
ssl:"target.com"
before:2023-01-01
country:US
```

**Web interface:** [shodan.io](https://shodan.io) — free account gives limited results.

---

## 5.4 Censys

```bash
# Web interface: search.censys.io
# More powerful than Shodan for certificate data

# Useful searches on censys.io:
# services.tls.certificates.leaf_data.subject.common_name: "*.target.com"
# ip: 10.10.10.5
# services.http.response.html_title: "Target App"
```

---

## 5.5 WHOIS & Domain Info

```bash
# WHOIS lookup
whois target.com
whois 10.10.10.5

# ASN lookup
whois -h whois.radb.net -- '-i origin AS12345'

# Reverse WHOIS (find all domains by same registrant)
# Use: viewdns.info/reversewhois/

# Domain history
# Use: whoishistory.com or domaintools.com
```

---

## 5.6 Wayback Machine / Archive

```bash
# gau — get all URLs from archives
gau target.com
gau target.com --providers wayback,commoncrawl,otx,urlscan

# Filter interesting URLs
gau target.com | grep -E "\.(php|asp|aspx|jsp)(\?|$)"
gau target.com | grep "="   # URLs with parameters

# Fetch old versions of a page
curl "https://web.archive.org/web/*/target.com/*" | grep -oP 'http[^"]*' | sort -u

# waybackurls
echo "target.com" | waybackurls
```

---

## 5.7 Certificate Transparency (crt.sh)

```bash
# Find all certificates issued for a domain
curl -s "https://crt.sh/?q=%.target.com&output=json" | \
  jq -r '.[].name_value' | \
  sed 's/\*\.//g' | \
  sort -u

# Save to file
curl -s "https://crt.sh/?q=%.target.com&output=json" | \
  jq -r '.[].name_value' | sort -u > crt-subdomains.txt
```

---

## 5.8 Social & GitHub Recon

```bash
# GitHub — search for exposed secrets
# On github.com, search:
# "target.com" password
# "target.com" api_key
# "target.com" secret
# org:targetorg filename:.env
# org:targetorg filename:config.php

# CLI with gh tool
gh search repos "target.com password" --limit 20

# Gitleaks — scan repos for secrets
gitleaks detect --source . -v
```

---

## 5.9 Email Verification

```bash
# hunter.io — find emails by domain (web)
# https://hunter.io/search/target.com

# phonebook.cz — email/domain search
# https://phonebook.cz

# Validate email format (basic)
echo "user@target.com" | grep -E "^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$"

# Common email patterns to try
# firstname.lastname@target.com
# flastname@target.com
# firstname@target.com
```

---

## 📋 Checklist

- [ ] Google Dorks (site:, filetype:, inurl:, intext:)
- [ ] theHarvester — emails, subdomains, hosts
- [ ] Shodan — open ports, tech, certificates
- [ ] crt.sh — subdomain discovery via certificates
- [ ] WHOIS — registrant info, ASN
- [ ] Wayback Machine / gau — historical URLs
- [ ] GitHub — exposed secrets or source code
- [ ] Censys — additional infrastructure data

---

## ⚠️ Legal & Ethical Notes

- OSINT is **passive** — no direct connection to the target
- All sources used here are **publicly available**
- Always confirm **scope** before any engagement
- In real engagements: document all sources used
- For personal practice: use your own domains or bug bounty programs

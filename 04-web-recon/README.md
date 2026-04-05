# 04 — Web Reconnaissance

> Goal: map the web attack surface — directories, endpoints, tech stack, parameters.
> Usually the richest phase for CTFs and bug bounty.

---

## 4.1 Technology Fingerprinting

```bash
# whatweb — identify CMS, frameworks, server
whatweb http://target.com
whatweb -v http://target.com   # verbose
whatweb -a 3 http://target.com # aggression level 1-4

# wafw00f — detect WAF
wafw00f http://target.com
wafw00f -a http://target.com   # try all WAF detections

# curl — manual header inspection
curl -I http://target.com
curl -I -L http://target.com   # follow redirects
curl -sv http://target.com 2>&1 | head -50

# Wappalyzer — browser extension (manual)
# Useful for: CMS version, JS frameworks, CDN, analytics
```

**Look for:** Server header, X-Powered-By, Set-Cookie (session tech), Content-Security-Policy.

---

## 4.2 Directory & File Fuzzing

### ffuf

```bash
# Basic directory fuzzing
ffuf -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt \
     -u http://target.com/FUZZ

# File extensions
ffuf -w /usr/share/seclists/Discovery/Web-Content/raft-medium-words.txt \
     -u http://target.com/FUZZ \
     -e .php,.html,.txt,.bak,.old,.zip,.conf

# Filter by status code
ffuf -w wordlist.txt -u http://target.com/FUZZ -fc 404

# Filter by response size
ffuf -w wordlist.txt -u http://target.com/FUZZ -fs 0,1234

# Recursive fuzzing
ffuf -w wordlist.txt -u http://target.com/FUZZ -recursion -recursion-depth 2

# Save output
ffuf -w wordlist.txt -u http://target.com/FUZZ -o ffuf-results.json -of json

# Rate limiting
ffuf -w wordlist.txt -u http://target.com/FUZZ -rate 50
```

### gobuster

```bash
# Directory scan
gobuster dir -u http://target.com \
             -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt

# With extensions
gobuster dir -u http://target.com \
             -w wordlist.txt \
             -x php,html,txt,bak

# With cookies (authenticated)
gobuster dir -u http://target.com \
             -w wordlist.txt \
             -c "session=abc123"

# Follow redirects
gobuster dir -u http://target.com -w wordlist.txt -r
```

### feroxbuster

```bash
# Recursive by default
feroxbuster -u http://target.com -w wordlist.txt

# With extensions
feroxbuster -u http://target.com -w wordlist.txt -x php,txt,html

# Limit recursion depth
feroxbuster -u http://target.com -w wordlist.txt -d 2

# Filter responses
feroxbuster -u http://target.com -w wordlist.txt --filter-status 404,403
```

---

## 4.3 Parameter Fuzzing

```bash
# GET parameter discovery
ffuf -w /usr/share/seclists/Discovery/Web-Content/burp-parameter-names.txt \
     -u "http://target.com/page?FUZZ=test" \
     -fs 0  # filter default response size

# POST parameter fuzzing
ffuf -w params.txt \
     -u http://target.com/login \
     -X POST \
     -d "FUZZ=test" \
     -H "Content-Type: application/x-www-form-urlencoded"

# Value fuzzing (known parameter)
ffuf -w /usr/share/seclists/Fuzzing/LFI/LFI-LFISuite-pathtotest.txt \
     -u "http://target.com/page?file=FUZZ"
```

---

## 4.4 Web Crawling & JS Analysis

```bash
# gospider
gospider -s http://target.com -d 3   # depth 3
gospider -s http://target.com -o crawl-output/

# hakrawler
echo "http://target.com" | hakrawler
echo "http://target.com" | hakrawler -depth 3 -plain

# gau — fetch known URLs from archives
gau target.com
gau target.com | grep "\.js$"     # JS files only
gau target.com | grep "\.php$"    # PHP files

# Combine gau + ffuf to find forgotten endpoints
gau target.com | grep "=" | qsreplace "FUZZ" | ffuf -w - -u FUZZ
```

### JS Analysis

```bash
# Download and grep for secrets in JS
curl http://target.com/app.js | grep -iE "(api_key|secret|token|password|aws)"

# linkfinder — extract endpoints from JS
python3 linkfinder.py -i http://target.com/app.js -o cli

# Beautify minified JS
curl http://target.com/app.min.js | js-beautify > app-readable.js
```

---

## 4.5 SSL/TLS Certificate Inspection

```bash
# Check cert for extra domains (SAN)
openssl s_client -connect target.com:443 2>/dev/null | openssl x509 -noout -text | grep DNS

# crt.sh — passive cert transparency search
curl -s "https://crt.sh/?q=%.target.com&output=json" | jq '.[].name_value' | sort -u
```

**Why:** TLS certificates often reveal internal subdomains, staging environments, etc.

---

## 4.6 Useful Wordlists (SecLists)

```
# Directories
/usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt
/usr/share/seclists/Discovery/Web-Content/raft-medium-directories.txt

# Files with extensions
/usr/share/seclists/Discovery/Web-Content/raft-medium-files.txt

# Parameters
/usr/share/seclists/Discovery/Web-Content/burp-parameter-names.txt

# API endpoints
/usr/share/seclists/Discovery/Web-Content/api/objects.txt
```

---

## 📋 Checklist

- [ ] Identify tech stack (whatweb, curl headers)
- [ ] Check for WAF (wafw00f)
- [ ] Directory fuzzing with common wordlist
- [ ] File fuzzing with relevant extensions (.php, .bak, .txt)
- [ ] Check robots.txt, sitemap.xml, .git/, /.env
- [ ] JavaScript files — look for API endpoints and secrets
- [ ] Parameter fuzzing on discovered endpoints
- [ ] SSL cert inspection (crt.sh, openssl)
- [ ] Crawl for hidden links

---

## 🎯 Quick Wins to Always Check

```bash
# Files that are often forgotten
curl http://target.com/robots.txt
curl http://target.com/sitemap.xml
curl http://target.com/.git/HEAD
curl http://target.com/.env
curl http://target.com/backup.zip
curl http://target.com/config.php.bak
curl http://target.com/admin/
curl http://target.com/api/
```

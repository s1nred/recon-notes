# 03 — DNS Enumeration

> Goal: extract DNS records, discover subdomains, attempt zone transfers.
> DNS leaks a LOT of infrastructure information if misconfigured.

---

## 3.1 Basic DNS Queries (dig)

```bash
# A record (IPv4)
dig A target.com

# All records
dig ANY target.com

# MX records (mail servers)
dig MX target.com

# NS records (nameservers)
dig NS target.com

# TXT records (SPF, DKIM, secrets)
dig TXT target.com

# AAAA record (IPv6)
dig AAAA target.com

# CNAME
dig CNAME www.target.com

# Specific DNS server
dig @8.8.8.8 A target.com

# Short output
dig +short A target.com
```

---

## 3.2 Zone Transfer (AXFR)

```bash
# Find nameservers first
dig NS target.com

# Attempt zone transfer against each NS
dig AXFR target.com @ns1.target.com
dig AXFR target.com @ns2.target.com

# Using host
host -t axfr target.com ns1.target.com

# Using nslookup
nslookup
> server ns1.target.com
> set type=any
> ls -d target.com
```

**Why it matters:** misconfigured DNS servers expose ALL DNS records — instant subdomain map.

---

## 3.3 Reverse DNS Lookup

```bash
# Single IP
dig -x 10.10.10.5
host 10.10.10.5

# Nmap reverse DNS
nmap -sn --dns-servers 8.8.8.8 10.10.10.0/24

# Bulk reverse lookup
for ip in $(cat ips.txt); do host $ip; done
```

---

## 3.4 Subdomain Enumeration — Passive

```bash
# subfinder (passive — queries public sources)
subfinder -d target.com
subfinder -d target.com -o subdomains.txt
subfinder -d target.com -v   # verbose

# assetfinder
assetfinder target.com
assetfinder --subs-only target.com

# amass (passive mode)
amass enum -passive -d target.com
amass enum -passive -d target.com -o amass-passive.txt
```

---

## 3.5 Subdomain Enumeration — Active (Brute Force)

```bash
# dnsx with wordlist
dnsx -d target.com -w /usr/share/wordlists/dns/subdomains-top1million-5000.txt

# fierce
fierce --domain target.com

# dnsrecon
dnsrecon -d target.com -t brt -D /usr/share/wordlists/dns/subdomains-top1million-5000.txt

# gobuster DNS mode
gobuster dns -d target.com -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt

# ffuf DNS bruteforce
ffuf -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt \
     -u http://FUZZ.target.com -v
```

---

## 3.6 dnsrecon — All-in-one

```bash
# Standard enum (A, AAAA, NS, SOA, MX, TXT)
dnsrecon -d target.com

# Zone transfer attempt
dnsrecon -d target.com -t axfr

# Reverse lookup on a range
dnsrecon -r 10.10.10.0/24

# Google enumeration (passive)
dnsrecon -d target.com -t goo

# Cache snooping
dnsrecon -d target.com -t snoop -D /usr/share/wordlists/dns/...
```

---

## 3.7 Amass — Full Active Enumeration

```bash
# Active + passive combined
amass enum -d target.com

# Active with brute force
amass enum -active -brute -d target.com

# Multiple domains
amass enum -d target.com -d target2.com

# Show IP addresses
amass enum -d target.com -ip

# Output to file
amass enum -d target.com -o amass-results.txt

# Database dump (if using amass db)
amass db -show
```

---

## 3.8 Virtual Hosts (vhost enumeration)

```bash
# ffuf vhost fuzzing
ffuf -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt \
     -u http://10.10.10.5 \
     -H "Host: FUZZ.target.com" \
     -fs 0   # filter by size to remove false positives

# gobuster vhost
gobuster vhost -u http://10.10.10.5 \
               -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt \
               --append-domain
```

**Important for HTB:** many machines host multiple services on different virtual hosts.

---

## 📋 Checklist

- [ ] dig ANY — grab all public records
- [ ] Attempt zone transfer (AXFR) on all nameservers
- [ ] Passive subdomain enum (subfinder, assetfinder)
- [ ] Active subdomain brute force (dnsx, gobuster dns)
- [ ] Reverse lookups on discovered IPs
- [ ] Virtual host fuzzing if web server found
- [ ] Check TXT records for sensitive info (API keys, configs)

---

## 📝 Interesting TXT Record Patterns

```
# SPF record — reveals mail infrastructure
v=spf1 include:... ip4:...

# Google verification
google-site-verification=...

# AWS keys accidentally exposed (rare but happens)
# Internal notes left in DNS
```

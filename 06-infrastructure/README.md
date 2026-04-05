# 06 — Infrastructure Reconnaissance

> Goal: map the network infrastructure — ASN, IP ranges, routing, cloud services.
> Useful for understanding the full scope of a target organization.

---

## 6.1 ASN & IP Range Discovery

```bash
# WHOIS for IP info
whois 10.10.10.5

# Find ASN for a domain
whois -h whois.cymru.com " -v 10.10.10.5"

# bgp.he.net (web) — comprehensive ASN data
# https://bgp.he.net/ip/10.10.10.5

# amass — ASN enumeration
amass intel -asn 12345
amass intel -cidr 10.10.10.0/24

# Find all IP ranges owned by an org
amass intel -org "Target Company"

# Shodan ASN search
shodan search 'asn:AS12345'
```

---

## 6.2 Traceroute & Network Path

```bash
# Basic traceroute
traceroute target.com
traceroute -n target.com   # no DNS resolution (faster)

# mtr — real-time traceroute
mtr target.com
mtr -n target.com          # numeric only
mtr --report target.com    # generate report

# Nmap traceroute
nmap --traceroute target.com
nmap -sn --traceroute 10.10.10.5

# TCP traceroute (bypass ICMP blocks)
sudo tcptraceroute target.com 80
```

---

## 6.3 Cloud Infrastructure Detection

```bash
# Check if IP belongs to cloud provider
curl -s "https://ip-api.com/json/TARGET_IP" | jq .

# AWS IP ranges
curl -s "https://ip-ranges.amazonaws.com/ip-ranges.json" | \
  jq '.prefixes[] | select(.ip_prefix | startswith("10.10.10"))'

# Common cloud indicators in DNS
dig A target.com    # Look for:
# *.amazonaws.com → AWS
# *.azurewebsites.net → Azure
# *.cloudfront.net → AWS CloudFront
# *.herokuapp.com → Heroku
# storage.googleapis.com → GCP

# S3 bucket discovery
# Pattern: target-backup.s3.amazonaws.com
curl -s http://target-backup.s3.amazonaws.com/
aws s3 ls s3://target-backup --no-sign-request
```

---

## 6.4 SSL/TLS Infrastructure Mapping

```bash
# Full SSL info
openssl s_client -connect target.com:443 -showcerts

# Certificate details
echo | openssl s_client -connect target.com:443 2>/dev/null | \
  openssl x509 -noout -text

# Find all SANs (Subject Alternative Names)
echo | openssl s_client -connect target.com:443 2>/dev/null | \
  openssl x509 -noout -text | grep DNS:

# SSL Labs rating check (web)
# https://www.ssllabs.com/ssltest/analyze.html?d=target.com

# testssl.sh — comprehensive SSL audit
./testssl.sh target.com
```

---

## 6.5 CDN & Real IP Discovery

```bash
# Check if behind CDN
dig A target.com   # Cloudflare IPs: 104.x.x.x, 172.64.x.x

# Try to find real IP behind CDN
# 1. Check old DNS records (historical)
curl -s "https://viewdns.info/iphistory/?domain=target.com"

# 2. Check MX/mail server IPs (often not behind CDN)
dig MX target.com
dig A mail.target.com

# 3. Subdomains that might bypass CDN
dig A direct.target.com
dig A origin.target.com
dig A dev.target.com

# 4. SSL cert SAN IPs
# 5. Shodan: "ssl.cert.subject.CN:target.com"
```

---

## 6.6 Email Infrastructure

```bash
# MX records
dig MX target.com

# SPF record (reveals authorized mail servers)
dig TXT target.com | grep spf

# DMARC policy
dig TXT _dmarc.target.com

# DKIM (if key name known)
dig TXT selector1._domainkey.target.com

# Reverse lookup on MX IPs
dig A $(dig MX target.com +short | awk '{print $2}')
```

---

## 6.7 Service-specific Recon

### SNMP (UDP 161)
```bash
# Community string brute force
onesixtyone -c /usr/share/seclists/Discovery/SNMP/snmp.txt 10.10.10.5

# SNMP walk (if community string known)
snmpwalk -c public -v1 10.10.10.5
snmpwalk -c public -v2c 10.10.10.5 1.3.6.1.2.1.1   # system info
```

### NetBIOS (UDP 137)
```bash
# NetBIOS scan
nbtscan 10.10.10.0/24
nmap -sU --script nbstat.nse -p137 10.10.10.5
```

### LDAP (TCP 389)
```bash
# Anonymous LDAP query
ldapsearch -x -H ldap://10.10.10.5 -b "dc=target,dc=com"
nmap -p 389 --script ldap-search 10.10.10.5
```

---

## 📋 Checklist

- [ ] WHOIS on target domain and IPs
- [ ] ASN lookup — find owned IP ranges
- [ ] Traceroute — map network path
- [ ] Detect CDN / real IP discovery
- [ ] SSL cert inspection — SANs, issuer
- [ ] Cloud infrastructure identification
- [ ] Email infrastructure (MX, SPF, DMARC)
- [ ] Check SNMP if UDP 161 is open

# 02 — Port Scanning

> Goal: identify open ports, running services, versions, and OS.
> This is the most important phase — it defines your attack surface.

---

## 2.1 Nmap — Basic Scans

```bash
# Top 1000 ports (default)
nmap 10.10.10.5

# All 65535 ports
nmap -p- 10.10.10.5

# Specific ports
nmap -p 22,80,443,8080 10.10.10.5

# Port range
nmap -p 1-10000 10.10.10.5

# Fast scan — top 100 ports
nmap -F 10.10.10.5
```

---

## 2.2 Nmap — Service & Version Detection

```bash
# Service version detection
nmap -sV 10.10.10.5

# Default scripts + version detection (most common combo)
nmap -sC -sV 10.10.10.5

# Aggressive scan (OS + version + scripts + traceroute)
nmap -A 10.10.10.5

# Adjust version intensity (0-9)
nmap -sV --version-intensity 5 10.10.10.5
```

---

## 2.3 Nmap — OS Detection

```bash
# OS fingerprinting (requires root)
sudo nmap -O 10.10.10.5

# OS + version
sudo nmap -O -sV 10.10.10.5

# OS detection with guessing if uncertain
sudo nmap -O --osscan-guess 10.10.10.5
```

---

## 2.4 Nmap — UDP Scanning

```bash
# UDP scan (slow — top 1000 UDP ports)
sudo nmap -sU 10.10.10.5

# UDP + top 20 ports only (faster)
sudo nmap -sU --top-ports 20 10.10.10.5

# Common UDP ports to always check:
# 53 (DNS), 161 (SNMP), 123 (NTP), 137 (NetBIOS), 500 (IKE)
sudo nmap -sU -p 53,123,161,137,500 10.10.10.5
```

---

## 2.5 Nmap — Output Formats

```bash
# Normal output to file
nmap -sC -sV -oN scan.txt 10.10.10.5

# Grepable output
nmap -sC -sV -oG scan.gnmap 10.10.10.5

# XML output
nmap -sC -sV -oX scan.xml 10.10.10.5

# All formats at once
nmap -sC -sV -oA scan 10.10.10.5
# creates: scan.nmap, scan.gnmap, scan.xml
```

---

## 2.6 RustScan — Fast Port Discovery

```bash
# Scan all ports super fast, then pass to nmap
rustscan -a 10.10.10.5 -- -sC -sV

# Limit batch size (lower = stealthier)
rustscan -a 10.10.10.5 -b 500 -- -sC -sV

# Full port range
rustscan -a 10.10.10.5 -r 1-65535 -- -sV
```

**Workflow:** use RustScan to find open ports fast → feed to Nmap for service details.

---

## 2.7 Masscan — High-speed TCP Scan

```bash
# All ports at 10k packets/sec
sudo masscan 10.10.10.5 -p0-65535 --rate=10000

# Save output
sudo masscan 10.10.10.5 -p0-65535 --rate=10000 -oG masscan.txt

# Multiple targets
sudo masscan 10.10.10.0/24 -p22,80,443 --rate=5000
```

---

## 2.8 Banner Grabbing (Manual)

```bash
# Netcat
nc -nv 10.10.10.5 22
nc -nv 10.10.10.5 80

# Telnet
telnet 10.10.10.5 25

# curl (HTTP headers)
curl -I http://10.10.10.5
curl -I -k https://10.10.10.5   # ignore SSL errors

# openssl (HTTPS banner + cert info)
openssl s_client -connect 10.10.10.5:443
```

---

## 2.9 Recommended HTB/CTF Workflow

```bash
# Step 1 — quick all-ports scan
sudo nmap -p- --min-rate 5000 -T4 10.10.10.5 -oN ports.txt

# Step 2 — extract open ports
ports=$(grep "open" ports.txt | awk -F/ '{print $1}' | tr '\n' ',' | sed 's/,$//')

# Step 3 — detailed scan on open ports only
nmap -sC -sV -p$ports 10.10.10.5 -oN detailed.txt
```

---

## 📋 Checklist

- [ ] Quick all-ports scan (-p-)
- [ ] Service + version detection (-sV -sC)
- [ ] UDP on common ports (53, 161, 137, 500)
- [ ] Banner grab on interesting services
- [ ] OS fingerprinting (-O)
- [ ] Save all output to files

---

## 🚩 Interesting Ports to Always Investigate

| Port | Service | Notes |
|------|---------|-------|
| 21 | FTP | Anonymous login? |
| 22 | SSH | Version vulns, weak creds |
| 23 | Telnet | Cleartext — grab traffic |
| 25 | SMTP | User enumeration |
| 53 | DNS | Zone transfer attempt |
| 80/443 | HTTP/S | Full web recon |
| 139/445 | SMB | EternalBlue, null sessions |
| 1433 | MSSQL | SA account, xp_cmdshell |
| 3306 | MySQL | Remote login |
| 3389 | RDP | BlueKeep, brute force |
| 5985/5986 | WinRM | Evil-WinRM |
| 6379 | Redis | Unauthenticated access |
| 8080/8443 | Alt HTTP | Dev environments |

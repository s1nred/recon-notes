# 🔍 recon-notes

> Personal notes, commands, and methodology for reconnaissance — both passive and active.
> Covering network, web, DNS, OSINT, and infrastructure recon.

---

## 📁 Structure

```
recon-notes/
├── 01-host-discovery/       # Ping sweeps, ARP scans, live host detection
├── 02-port-scanning/        # TCP/UDP scanning, service detection, OS fingerprinting
├── 03-dns-enumeration/      # DNS records, zone transfers, subdomain enum
├── 04-web-recon/            # Subdomain enum, directory fuzzing, tech fingerprinting
├── 05-osint/                # Google dorks, email harvesting, passive recon
├── 06-infrastructure/       # WHOIS, ASN, traceroute, Shodan
└── templates/               # Recon checklists and report templates
```

---

## ⚡ Quick Reference

| Phase | Goal | Key Tools |
|-------|------|-----------|
| Host Discovery | Find live hosts | `nmap -sn`, `arp-scan`, `netdiscover` |
| Port Scanning | Open ports & services | `nmap`, `masscan`, `rustscan` |
| DNS Enum | DNS records, subdomains | `dig`, `dnsx`, `fierce`, `dnsrecon` |
| Web Recon | Dirs, subdomains, stack | `ffuf`, `subfinder`, `whatweb` |
| OSINT | Emails, leaks, dorks | `theHarvester`, Google Dorks, Shodan |
| Infrastructure | ASN, IP ranges, routing | `whois`, `amass`, `mtr` |

---

## 🔁 General Recon Methodology

```
Target scope defined
        │
        ▼
   Passive OSINT ──────────────────────────────────┐
   (no direct interaction)                          │
   - WHOIS / ASN                                    │
   - Google Dorks                                   │
   - Shodan / Censys                                │
   - theHarvester                                   │
        │                                           │
        ▼                                           │
   Active Recon ◄───────────────────────────────────┘
   (direct interaction)
   - Host discovery
   - Port scanning
   - Service fingerprinting
   - DNS enum
   - Web fuzzing
        │
        ▼
   Document findings → move to exploitation
```

---

## 📌 Notes

- Always verify scope before running active scans
- Passive recon first — leave no traces when possible
- Document every finding with tool + command + output
- These notes are for **CTF / lab environments** and authorized engagements

---

## 🧰 Core Toolset

```bash
# Install essentials (Debian/Ubuntu)
sudo apt install nmap netdiscover arp-scan dnsutils fierce dnsrecon \
                 gobuster ffuf whatweb wafw00f whois traceroute mtr -y

# Go-based tools
go install github.com/projectdiscovery/subfinder/v2/cmd/subfinder@latest
go install github.com/projectdiscovery/dnsx/cmd/dnsx@latest
go install github.com/tomnomnom/assetfinder@latest
go install github.com/lc/gau/v2/cmd/gau@latest
go install github.com/hakluke/hakrawler@latest

# Python tools
pip install theHarvester
```

---

*Updated regularly as I progress through HTB CPTS / CDSA paths.*

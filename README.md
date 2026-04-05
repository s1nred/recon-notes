# recon-notes

Personal reconnaissance notes focused on repeatable workflow, not just command dumps.

This repo is where I document how I move from target scope to usable findings across host discovery, ports, DNS, web, OSINT, and infrastructure.

## Purpose

Most recon repos look the same: a long cheat sheet with tools and syntax.

This one aims to be more useful in practice:

- phase-based notes with a clear order
- quick decision points for what to do next
- checklists and templates for documenting real work
- commands that are useful for labs, CTFs, and authorized assessments

## Structure

```text
recon-notes/
├── 01-host-discovery/
├── 02-port-scanning/
├── 03-dns-enumeration/
├── 04-web-recon/
├── 05-osint/
├── 06-infrastructure/
├── examples/
└── templates/
```

## How I Use This Repo

1. Confirm scope before doing anything active.
2. Start with passive recon when possible.
3. Move into host discovery and port scanning.
4. Branch based on what the target exposes.
5. Write down findings immediately with command, output, and next step.

## Recon Flow

```text
Scope confirmed
    |
    +--> Passive recon
    |    - WHOIS / ASN
    |    - certificates
    |    - Shodan / Censys
    |    - historical URLs / public data
    |
    +--> Active discovery
         - live hosts
         - open ports
         - exposed services
         - DNS data
         - web surface
              |
              +--> document findings
              +--> decide next branch
```

## Quick Reference

| Phase | Main goal | Typical next move |
|------|------|------|
| Host Discovery | Find live systems | Scan only confirmed live hosts |
| Port Scanning | Map attack surface | Run service detection on interesting ports |
| DNS Enumeration | Discover records and subdomains | Pivot into web or infrastructure recon |
| Web Recon | Map endpoints, tech, and content | Fuzz, crawl, inspect JS, and note parameters |
| OSINT | Gather public intelligence | Enrich scope without touching target |
| Infrastructure | Map ownership and routing | Identify ranges, providers, and shared assets |

## What Makes It Better Than A Basic Cheat Sheet

- it keeps notes by phase instead of mixing everything together
- it includes templates so findings can be documented consistently
- it favors small decision rules over random command lists
- it stays grounded in actual workflow from beginner labs and HTB-style targets

## Templates

The `templates/` directory contains practical note formats for:

- target tracking
- recon checklists
- findings writeups
- scan result notes

## Examples

The `examples/` directory contains short, realistic walkthroughs that show how I turn scattered recon output into actual notes and next steps.

## Core Tools

```bash
sudo apt install nmap netdiscover arp-scan dnsutils fierce dnsrecon \
                 gobuster ffuf whatweb wafw00f whois traceroute mtr -y

go install github.com/projectdiscovery/subfinder/v2/cmd/subfinder@latest
go install github.com/projectdiscovery/dnsx/cmd/dnsx@latest
go install github.com/tomnomnom/assetfinder@latest
go install github.com/lc/gau/v2/cmd/gau@latest
go install github.com/hakluke/hakrawler@latest

pip install theHarvester
```

## Scope And Ethics

- active scans only against systems you own or are authorized to test
- passive recon still needs scope discipline
- always record what you ran and why

Updated as I improve my recon workflow through labs, HTB, and structured practice.

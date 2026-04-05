# Example — HTB-Style Web Target Recon

This is a simple example of how I would document a beginner-friendly lab target.

## Target Summary

- Target IP: `10.10.10.25`
- Goal: identify useful attack surface before exploitation
- Context: lab / HTB-style machine

## Phase 1 — Host Discovery

The host is already known, so discovery is minimal.

```bash
ping -c 3 10.10.10.25
```

Observation:

- host responds
- move directly to full TCP scanning

## Phase 2 — Port Scanning

```bash
sudo nmap -p- --min-rate 5000 -T4 10.10.10.25 -oN ports.txt
```

Result:

- `22/tcp` open: SSH
- `80/tcp` open: HTTP

Follow-up:

```bash
nmap -sC -sV -p22,80 10.10.10.25 -oN detailed.txt
```

Notes:

- SSH looks standard, likely lower priority at this point
- HTTP becomes the main branch for recon

## Phase 3 — Web Recon

```bash
whatweb http://10.10.10.25
curl -I http://10.10.10.25
ffuf -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -u http://10.10.10.25/FUZZ -fc 404
```

Interesting findings:

- `/admin` exists
- `/assets/app.js` exposes API paths
- response headers suggest a PHP application behind Apache

## Phase 4 — DNS / Vhost Check

The landing page references `intranet.target.htb`, which suggests hostname-based routing.

Add to hosts file locally, then test:

```bash
ffuf -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt \
     -u http://10.10.10.25 \
     -H "Host: FUZZ.target.htb" \
     -fs 1234
```

Finding:

- `dev.target.htb` responds differently from the default site

## Phase 5 — Notes That Matter

Instead of dumping every command, I would preserve the signals that change direction:

- open ports: `22`, `80`
- main web app on `80`
- `/admin` discovered
- JavaScript reveals internal API naming
- vhost discovery finds `dev.target.htb`

## Why This Is Useful

This is the difference between a cheat sheet and a working methodology:

- each phase produces a smaller, better next step
- weak signals are turned into pivots
- the notes record decisions, not just commands

## Minimal Final Note Template

```text
Target: 10.10.10.25
Open ports: 22, 80
Primary focus: web
Interesting paths: /admin
Interesting hostnames: dev.target.htb
Next step: inspect dev site and enumerate exposed functionality
```

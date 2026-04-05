# 01 — Host Discovery

> Goal: identify live hosts in a network before deeper scanning.
> Always start here — no point scanning ports on dead hosts.

## Quick Decision Flow

```text
On the same local network?
    |
    +--> Yes: use ARP-based discovery first
    |
    +--> No: start with ICMP or TCP-based discovery
             |
             +--> If results look too empty, test with -Pn
```

## What This Phase Should Produce

- a reliable list of live hosts
- a note on which discovery method actually worked
- a shortlist of targets worth deeper scanning

---

## 1.1 ICMP Ping Sweep (Nmap)

```bash
# Basic ping sweep — no port scan
nmap -sn 10.10.10.0/24

# Save results to file
nmap -sn 10.10.10.0/24 -oG hosts-up.txt

# Extract only live IPs from grepable output
grep "Up" hosts-up.txt | awk '{print $2}'

# Ping sweep + disable DNS resolution (faster)
nmap -sn -n 10.10.10.0/24
```

**When to use:** first sweep on a /24. Quick, works over most networks.
**Limitation:** hosts with ICMP blocked will appear as down.

**Practical note:** if a range looks empty too quickly, treat that as possible filtering instead of proof that no hosts exist.

---

## 1.2 ARP Scan (Local Network Only)

```bash
# ARP scan on local subnet (requires root)
sudo arp-scan --localnet

# Specific range
sudo arp-scan 192.168.1.0/24

# Specify interface
sudo arp-scan -I eth0 192.168.1.0/24
```

**When to use:** local LAN — ARP cannot be blocked like ICMP.
**Advantage:** much more reliable on local networks than ping.

---

## 1.3 Netdiscover (Passive + Active)

```bash
# Active scan
sudo netdiscover -r 192.168.1.0/24

# Passive mode (just listen — leaves no trace)
sudo netdiscover -p

# Specify interface
sudo netdiscover -i eth0 -r 10.10.10.0/24
```

**Passive mode:** great for stealth — just listens for ARP broadcasts.

---

## 1.4 Nmap — No-Ping Scan (bypass ICMP blocks)

```bash
# Treat all hosts as up (skip host discovery)
nmap -Pn 10.10.10.5

# Combined: no ping + version detection
nmap -Pn -sV 10.10.10.5

# Useful when firewall blocks ICMP but ports are open
```

**When to use:** after normal discovery looks incomplete. `-Pn` is how you keep moving when host discovery is being filtered.

---

## 1.5 Masscan — Ultra-fast Sweep

```bash
# Ping sweep equivalent (send SYN to port 80)
sudo masscan 10.10.10.0/24 -p80 --rate=1000

# Multiple ports to detect live hosts
sudo masscan 10.10.10.0/24 -p22,80,443 --rate=5000
```

**Note:** masscan is very loud — not for stealth engagements.

**Use carefully:** this is useful for speed, not subtlety. Rate and scope should match the environment.

---

## 1.6 Ping (manual / quick check)

```bash
# Single host
ping -c 3 10.10.10.5

# Quick one-liner sweep (bash)
for i in $(seq 1 254); do ping -c1 -W1 192.168.1.$i &>/dev/null && echo "192.168.1.$i is up"; done
```

---

## 📋 Checklist

- [ ] Run ARP scan if on local network
- [ ] Run nmap -sn ping sweep
- [ ] Try -Pn if hosts appear down but ports are open
- [ ] Compare results from more than one method if output looks incomplete
- [ ] Save all live IPs to a file for next phase

---

## Common Mistakes

- Assuming one empty ping sweep means nothing is alive
- Forgetting that ARP is usually better on local networks
- Starting full port scans before validating which hosts are real
- Not recording which method found each host

---

## 📝 Output Template

```
Date: 
Target range: 
Method used: 
Live hosts found:
  - 10.10.10.x  (hostname if known)
  - 10.10.10.y
Notes:
```

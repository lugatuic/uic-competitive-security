# Vulnerability Hunters Role — Technical Prep Guide

**Last updated:** September 2026  
**Owner:** AJ  
**For:** Vulnerability Hunter competitors and team captains

---

## Role Overview

Vulnerability Hunters (3 people per team) own two distinct jobs:
solving anomaly challenges and conducting red team recon across all
VMs. They are the largest sub-team because their work spans two
scoring buckets — Anomaly Scoring (15%) and the investigation side
of Red Team scoring (25%).

Hunters are the most offensively-minded role on the team. While M&H
locks systems down and Green Team keeps the application running,
Hunters are actively investigating, problem-solving, and tracking
adversary behavior across the entire network.

---

## What You Own

**Anomaly challenges** — CTF-style problems released by the White Team
throughout competition day. Time-limited — close whether you submit
or not.

**Red team recon** — investigating what the red team is doing across
all VMs including HMI and PLC. Identifying indicators of compromise
on Traditional VMs. Supporting incident report drafting.

---

## How CyberForce Access Works

CyberForce gives your team a **~3 week contract period** (Oct 26 –
Nov 14) of SSH access to all Traditional VMs before and during
competition day. This is not just for M&H — Hunters use this window
actively too.

**What Hunters do during the contract period:**
- Enumerate your own systems to identify vulnerabilities before the
  red team does
- Get familiar with the ICS environment — understand what normal looks
  like on HMI and PLC before you need to spot anomalies on competition
  day
- Practice anomaly-style challenges
- Study the scenario — the more you understand Vulcana Dynamics'
  operations, the faster you recognize anomalies themed around them

**Competition day (Nov 14):** the red team goes live. Hunters
shift from prep mode to active anomaly solving and red team tracking.
Systems are already hardened by M&H — your job is investigation and
documentation, not setup.

---

## Contract Period Timeline — Vulnerability Hunters

**Week 1 (Oct 26 – Nov 1): Enumerate and familiarize**
- Run nmap against your own Traditional VMs — see what the red team
  sees from outside
- Review open ports and services on each VM alongside M&H
- Get familiar with the ICS environment — review OpenPLC monitoring
  dashboard, understand normal register and coil values
- Study the Vulcana Dynamics scenario — what does the company do,
  what systems are critical, what would an attacker target

**Week 2 (Nov 2 – Nov 8): Deepen recon knowledge**
- Practice CTF challenges in your weakest categories
- Understand Modbus traffic on port 502 — what normal looks like
- Review Windows Event IDs and Linux auth logs on Traditional VMs
  so you can read them quickly on competition day

**Week 3 (Nov 9 – Nov 13): Final prep**
- Run final enumeration sweep — verify you know the state of every
  Traditional VM going into competition day
- Confirm ICS monitoring workflow with M&H — how anomalous behavior
  gets surfaced and documented
- Review the ICS anomaly (cargo ship at 12:30 PM and 2:45 PM) —
  understand the crane controls on the HMI Home page

---

## Technical Prep — Priority Order

### 1. CTF Fundamentals by Category

Anomalies are CTF-style challenges themed around the year's scenario.
The 2026 scenario is Vulcana Dynamics Ltd. — an energy utility with
power grid and radar network controls. Anomalies will likely touch
industrial control concepts, network protocols, and forensics themes
relevant to the scenario.

A team of three Hunters should collectively cover all CTF
categories — you do not all need to be strong in everything, but
every category should have at least one Hunter who can attempt it.

**Web Exploitation**
What to know:
- SQL injection — identifying and exploiting vulnerable input fields
- Cross-site scripting (XSS) — reflected and stored
- Directory traversal — accessing files outside the web root
- Common HTTP methods and status codes
- Burp Suite or browser dev tools for intercepting and modifying
  requests
- Reading and manipulating cookies and session tokens

Where to learn:
- [PortSwigger Web Security Academy](https://portswigger.net/web-security)
  — free, hands-on, the best resource for web exploitation

**Cryptography**
What to know:
- Caesar cipher, Vigenère, and basic substitution ciphers
- Base64, hex, and binary encoding/decoding
- XOR operations and why they appear in CTF crypto
- RSA basics — public/private keys, why small exponents are vulnerable
- Hash functions — MD5, SHA1, SHA256 — and hash cracking with
  hashcat or john the ripper
- CyberChef for rapid encoding/decoding operations

Where to learn:
- [CryptoPals](https://cryptopals.com) — hands-on crypto challenges
- [CyberChef](https://gchq.github.io/CyberChef) — bookmark this now

**Forensics**
What to know:
- File format identification — `file` command, magic bytes
- Steganography — hidden data in images and audio:
  `steghide`, `binwalk`, `strings`
- pcap analysis — reading network captures in Wireshark
- Memory forensics basics — Volatility framework
- Log file analysis — reading and searching large log files
- Metadata extraction — `exiftool` for images and documents

Where to learn:
- [Wireshark sample captures](https://wiki.wireshark.org/SampleCaptures)
- CTFtime.org writeups for forensics challenges

**Reverse Engineering**
What to know:
- Reading disassembled x86/x64 assembly — identifying functions,
  loops, conditionals
- Static analysis with Ghidra (free) or Binary Ninja
- Dynamic analysis with GDB — setting breakpoints, reading registers
- Identifying common patterns — string comparisons, license checks,
  flag validation routines
- Python scripting for automating RE tasks

Where to learn:
- [Ghidra](https://ghidra-sre.org) — install and complete the
  built-in tutorial
- [pwn.college](https://pwn.college) — reverse engineering modules

**Binary Exploitation (pwn)**
What to know:
- Stack buffer overflows — overwriting return addresses
- Format string vulnerabilities
- Python pwntools library for exploit scripting
- GDB with pwndbg or peda for dynamic analysis
- Basic ROP concepts
- Understanding protections: NX, ASLR, stack canaries

Where to learn:
- [pwn.college](https://pwn.college) — the best free resource for pwn
- [LiveOverflow YouTube](https://youtube.com/@LiveOverflow)

**OSINT**
What to know:
- Google dorking — advanced search operators
- Reverse image search — Google Images, TinEye, Yandex
- WHOIS lookups and domain history
- Wayback Machine for historical website content
- Geolocation from image metadata and visual clues

Where to learn:
- [OSINT Framework](https://osintframework.com)
- Trace Labs CTF events — OSINT-specific competitions

**Networking**
What to know:
- TCP/IP model — how packets flow between hosts
- Reading pcap files in Wireshark — filtering by protocol, IP, port
- Common protocols: HTTP, DNS, FTP, SMTP, Modbus — what normal
  traffic looks like for each
- nmap — port scanning, service detection, OS fingerprinting
- netcat for basic network connections and testing
- Identifying anomalous traffic patterns in captures

Where to learn:
- [Wireshark](https://wireshark.org) — practice on sample captures
- nmap official documentation

**General Skills / Miscellaneous**
What to know:
- Linux command line fluency — file navigation, grep, find, pipes
- Python scripting — automating tasks, parsing output, quick scripts
- Reading and writing basic bash scripts
- Recognizing and decoding common encodings quickly
- Git basics — cloning repos, reading commit history

---

### 2. Self-Enumeration During the Contract Period

One of the most valuable things Hunters can do during the contract
period is enumerate your own Traditional VMs the way the red team
will. This tells you what is exposed and gives M&H actionable
findings to harden before competition day.

**nmap against your own systems:**
```bash
# Service and version detection
nmap -sV [target-ip]

# OS detection
nmap -O [target-ip]

# Full port scan
nmap -p- [target-ip]

# Script scan for common vulnerabilities
nmap --script vuln [target-ip]

# Scan all Traditional VMs
nmap -sV 10.x.x.[DB] 10.x.x.[AD] 10.x.x.[taskbox] 10.x.x.[webserver]
```

Share findings with M&H immediately — every open port that should not
be open, every service running an outdated version, every unnecessary
service is something they can harden before competition day.

**What to look for:**
- Unnecessary open ports — services that should not be exposed
- Default credentials still in place
- Outdated software versions with known CVEs
- Misconfigured services

---

### 3. Recon and Network Monitoring on Competition Day

These are the tools and techniques to know.

**Wireshark / tcpdump:**
```bash
# Capture traffic on your network
tcpdump -i eth0 -w capture.pcap

# Filter by source IP in Wireshark
ip.src == [suspicious-ip]

# Filter Modbus traffic
tcp.port == 502
```

**Log analysis — Windows Event IDs:**

| Event ID | What It Means |
|---|---|
| 4624 | Successful logon |
| 4625 | Failed logon — watch for spikes |
| 4720 | New user account created |
| 4728/4732 | User added to security group |
| 4672 | Special privileges assigned |

**Linux auth logs:**
```bash
# Failed SSH attempts
grep "Failed password" /var/log/auth.log

# Successful logins
grep "Accepted" /var/log/auth.log

# New user creation
grep "useradd\|adduser" /var/log/auth.log
```

---

### 4. ICS / OT Awareness

**Normal PLC register values — know these before competition day:**

| Register | Normal Range | Alert If... |
|---|---|---|
| WellPressure | Below 5000.0 | Above 5000 → BOP activates |
| WellTemp | Below 95.0 | Above 95 → BOP activates |
| WellFlowRate | 0.17 – 10.42 | Outside range → oil gen stops |
| SeparatorTemp | 20.0 – 90.0 | Outside range → separator off |

**Red team indicators on ICS:**
- SafeToOperate flips FALSE without fire or hurricane event
- BOP activates without pressure/temp threshold being reached
- ManualOverride activates unexpectedly
- Modbus write commands from unexpected source IPs
- PLC program stops unexpectedly

**The ICS anomaly:**
At 12:30 PM and 2:45 PM a cargo ship arrives for resupply. Person 1
handles crane interactions via the HMI Home page within the scheduled
time window. Prepare for this before competition day — review the
crane controls in the HMI during the contract period so it is not
the first time you see them on competition day.

---

### 5. Anomaly Triage Under Time Pressure

**When an anomaly drops:**
1. Read the full challenge description before doing anything
2. Estimate difficulty and point value
3. Assign to the Hunter best suited for the category
4. Set a mental time limit — flag and move on if no progress in 20 min
5. Never let one hard anomaly consume all three Hunters while easier
   ones go unsolved

**Time limits are real:** submit partial progress if a deadline is
approaching. Some anomalies award partial credit. A zero for a closed
anomaly is worse than partial credit for an incomplete one.

---

## What a Bad Day Looks Like

Three anomalies drop in the first hour. Two close before submission
because everyone was heads-down on the hardest one. The red team
pivots from HMI to AD and nobody notices until a backdoor account
has been live for 45 minutes. The incident report is vague because
nobody documented the timeline as it happened. The 12:30 PM cargo
ship anomaly window opens and nobody is ready because the HMI crane
controls were never reviewed during the contract period.

---

## See Also

- [cyberforce101.md](../../docs/cyberforce101.md)
- [shared-foundations.md](../../docs/shared-foundations.md)
- [Monitoring & Hardening Guide](monitoring+hardening_guide.md)
- [Green Team Guide](green_team_guide.md)
- [CyberForce Technical Prep](../cyberforce-technical-prep.md)

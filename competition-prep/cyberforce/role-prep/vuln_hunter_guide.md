# Vulnerability Hunters Role — Technical Prep Guide

**Last updated:** September 2026  
**Owner:** AJ  
**For:** Vulnerability Hunter competitors and team captains

---

## Role Overview

Vulnerability Hunters (3 people per team) own two distinct jobs during
competition: solving anomaly challenges and conducting red team recon
across all VMs. They are the largest sub-team because their work spans
two scoring buckets — Anomaly Scoring (15%) and the investigation side
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
- Practice anomaly-style challenges using past CyberForce scenarios
  in the archival folder in the repo
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
- Practice past anomaly challenges from the archival folder

**Week 2 (Nov 2 – Nov 8): Deepen recon knowledge**
- Study past CyberForce anomaly solutions — what categories appeared,
  what techniques were used
- Practice CTF challenges in your weakest categories
- Understand Modbus traffic on port 502 — what normal looks like
- Review Windows Event IDs and Linux auth logs on Traditional VMs
  so you can read them quickly on competition day

**Week 3 (Nov 9 – Nov 13): Final prep**
- Run final enumeration sweep — verify you know the state of every
  Traditional VM going into competition day
- Confirm ICS monitoring workflow with M&H — how anomalous behavior
  gets surfaced and documented

---

## Technical Prep — In Order

Work through these in sequence. Each item builds on the one before it.
The CTF category work in items 4 and 5 will not land well without the
Linux, Windows, and enumeration foundation from items 1–3.

---

### 1. Linux Fundamentals

The majority of competition VMs run Linux. You need to be comfortable
at the Linux command line before anything else.

**What to know:**
- File system navigation: `cd`, `ls`, `pwd`, `find`, `locate`
- File permissions: `chmod`, `chown`, `umask` — reading `rwxr-xr-x`
  and knowing what it means
- User and group management: `cat /etc/passwd`, `cat /etc/group`,
  `id`, `who`, `w`
- Process management: `ps aux`, `top`, `kill`, `systemctl status`
- File operations: `cat`, `less`, `grep`, `head`, `tail`, `wc`
- Searching: `grep -r "string" /path/`, `find / -name "file"`
- Package management: `apt` (Ubuntu/Debian), `dnf` (Fedora/RHEL),
  `zypper` (OpenSUSE)
- Networking basics: `ip a`, `ss -tulnp`, `ping`, `curl`, `wget`
- Log files: `/var/log/auth.log`, `/var/log/syslog`, `journalctl`
- SSH: connecting to a remote machine, understanding key-based auth

**Why it matters for Hunters specifically:**
Red team recon and anomaly solving both happen on Linux systems.
If you cannot navigate the file system, read logs, or check what
services are running, you cannot do either job effectively.

**Where to learn:**
- [OverTheWire: Bandit](https://overthewire.org/wargames/bandit/) —
  the best beginner Linux CLI wargame, start here
- [Linux Journey](https://linuxjourney.com) — structured beginner
  course
- Practice: spin up a free Ubuntu VM on your own machine

---

### 2. Windows Fundamentals

The AD/DNS and DB VMs run Windows Server. The HMI runs Windows Server
2019. You need enough Windows knowledge to investigate activity on
these machines.

**What to know:**
- Command Prompt vs. PowerShell — when to use each
- File system navigation in both CMD and PowerShell
- User and group management:
  ```powershell
  Get-LocalUser
  Get-LocalGroupMember "Administrators"
  net user
  net localgroup
  ```
- Process management: Task Manager, `Get-Process`, `tasklist`
- Services: `Get-Service`, `services.msc`, `sc query`
- Event Viewer: where it is, how to filter by Event ID
- Registry basics: `regedit`, common registry locations
- PowerShell for searching:
  ```powershell
  Get-ChildItem -Recurse | Select-String "password"
  ```
- Windows Firewall: viewing rules, understanding inbound vs. outbound

**Why it matters for Hunters specifically:**
Red team activity on Windows systems shows up in Event Viewer. New
accounts, privilege escalation, and lateral movement all leave Windows
Event logs. If you cannot read Event Viewer, you cannot write incident
reports on Windows-based compromises.

**Where to learn:**
- [TryHackMe Windows Fundamentals](https://tryhackme.com/module/windows-fundamentals)
  — free, hands-on, covers exactly what you need
- [Microsoft PowerShell documentation](https://learn.microsoft.com/en-us/powershell/)
- Practice: use a Windows VM or the TryHackMe browser-based machines

---

### 3. Enumeration Tools

With Linux and Windows fundamentals in place, now learn the tools
used to map and investigate a network.

**nmap — network mapping and service detection:**
```bash
# Find what is alive on a subnet
nmap -sn 10.x.x.0/24

# Service and version detection
nmap -sV [target-ip]

# OS detection
nmap -O [target-ip]

# Full port scan
nmap -p- [target-ip]

# Script scan for common vulnerabilities
nmap --script vuln [target-ip]
```

**Wireshark — network traffic analysis:**
- Capturing traffic on a network interface
- Filtering by protocol, IP, and port
  - `ip.src == 10.x.x.x` — traffic from a specific IP
  - `tcp.port == 502` — Modbus traffic
  - `http` — all HTTP traffic
- Identifying port scans — SYN packets to many ports rapidly
- Identifying unexpected connections between VMs

**tcpdump — command-line packet capture:**
```bash
# Capture all traffic on eth0
tcpdump -i eth0

# Write to file for later analysis in Wireshark
tcpdump -i eth0 -w capture.pcap

# Filter by host
tcpdump -i eth0 host [target-ip]

# Filter by port
tcpdump -i eth0 port 502
```

**grep for log hunting:**
```bash
# Search for "password" recursively through /etc
grep -r "password" /etc/

# Search case-insensitively
grep -ri "password" /etc/

# Show line numbers
grep -rn "failed" /var/log/auth.log
```

**Where to learn:**
- [nmap official documentation](https://nmap.org/docs.html)
- [Wireshark sample captures](https://wiki.wireshark.org/SampleCaptures)
  — practice reading real traffic
- TryHackMe has nmap and Wireshark rooms

---

### 4. CTF Fundamentals + Category Self-Assessment

With the technical foundation in place, now assess which CTF
categories you are strongest and weakest in. Anomalies in CyberForce
are CTF-style challenges themed around the Vulcana Dynamics scenario
— energy utility, power grid, radar controls.

**Rate yourself 1–5 in each category:**

| Category | What It Tests | Key Tools |
|---|---|---|
| Web Exploitation | Finding and exploiting web vulnerabilities | Burp Suite, browser dev tools |
| Cryptography | Encoding, ciphers, hash cracking | CyberChef, hashcat, john |
| Forensics | Hidden data, file analysis, pcap analysis | Wireshark, steghide, binwalk, exiftool |
| Reverse Engineering | Understanding compiled code | Ghidra, GDB |
| Binary Exploitation | Memory corruption, buffer overflows | pwntools, GDB |
| OSINT | Finding information from public sources | Google dorking, Wayback Machine |
| Networking | Packet analysis, protocol identification | Wireshark, nmap |
| General Skills | Encoding/decoding, scripting, misc | CyberChef, Python |

**After rating yourself:**
- Identify your two weakest categories
- Focus practice time there before the CyberForce CTF on Oct 7
- As a team of three, ensure every category has at least one Hunter
  who can attempt it

**Where to learn:**
- [CTFtime.org](https://ctftime.org) — archive of past CTF writeups
  by category
- [PortSwigger Web Security Academy](https://portswigger.net/web-security)
  — free web exploitation course
- [CyberChef](https://gchq.github.io/CyberChef) — bookmark this,
  you will use it constantly

---

### 5. Anomaly Tools

Learn the specific tools that come up most in CyberForce anomaly
challenges. These build on the category knowledge from item 4.

**Steghide — extracting hidden data from images:**
```bash
# Check if an image has hidden data
steghide info suspicious.jpg

# Extract hidden data
steghide extract -sf suspicious.jpg
```

**binwalk — analyzing binary files for embedded content:**
```bash
# Scan a file for embedded content
binwalk suspicious.file

# Extract embedded content
binwalk -e suspicious.file
```

**exiftool — reading file metadata:**
```bash
# View all metadata
exiftool image.jpg

# View specific field
exiftool -GPS:all image.jpg
```

**strings — extracting readable text from binary files:**
```bash
# Extract all strings longer than 4 characters
strings binary.file

# Search strings output for something specific
strings binary.file | grep "flag"
```

**CyberChef — encoding/decoding operations:**
Accessible at [gchq.github.io/CyberChef](https://gchq.github.io/CyberChef).
Use it for: Base64, hex, ROT13, XOR, URL encoding, hash identification,
and chaining multiple operations together. Bookmark it — it is the
fastest tool for quick encoding/decoding on competition day.

**Hashcat — password hash cracking:**
```bash
# Crack MD5 hashes with a wordlist
hashcat -m 0 hashes.txt /usr/share/wordlists/rockyou.txt

# Show cracked results
hashcat -m 0 hashes.txt --show
```

**John the Ripper — alternative hash cracker:**
```bash
# Crack a hash file
john --wordlist=/usr/share/wordlists/rockyou.txt hashes.txt

# Show cracked results
john --show hashes.txt
```

**Hashcat vs. John:** Hashcat is faster with GPU support. John is
simpler to use from the command line without GPU. For competition day,
use whichever you are more comfortable with.

**Ghidra — reverse engineering:**
Free NSA-developed tool for analyzing compiled code. Download at
[ghidra-sre.org](https://ghidra-sre.org). Complete the built-in
tutorial before competition. Useful for reverse engineering anomaly
challenges that involve compiled binaries.

---

### 6. Hands-On CTF Practice

Put the tools and categories together by solving real challenges.
The CyberForce CTF on Oct 7 is your primary practice event. Hack-O-Ween
(Oct 29, WiCyS collab) is a second opportunity.

**Before the CyberForce CTF (Oct 7):**
- Practice triage — open three challenges at once and decide which
  order to attempt them in

**During the CyberForce CTF:**
- Triage by point value and difficulty — do not all work the same
  challenge simultaneously
- Set a 20-minute time limit per challenge — if no meaningful progress,
  move on and return later
- Document every challenge attempted: what you tried, what worked,
  what did not

**After the CyberForce CTF — debrief these questions:**
- Which challenges were solved? Which were not?
- Which categories appeared? Were they covered in your prep?
- What tool or technique did you not know that would have helped?
- How did triage feel — did the team coordinate or did people step
  on each other?

This debrief directly shapes what you focus on in Week 7 (ICS/OT)
and during the contract period.

---

### 7. ICS/OT Awareness

CyberForce is distinctly different from standard CTF competitions
because of the HMI and PLC — two Assume Breach VMs that represent
real industrial control systems. Understanding how they work and what
red team activity looks like is a Hunter-specific skill that most CTF
prep resources do not cover.

**The ICS architecture:**
```
Physical Equipment (simulated)
        ↑↓ Modbus protocol (port 502)
PLC — Ubuntu 22.04, OpenPLC
        ↑↓ OPC UA / Modbus TCP
HMI — Windows Server 2019, Ignition Gateway (port 8088)
        ↑↓
Engineer/Operator views Ignition screens
```

The PLC runs the control logic. The HMI shows operators what is
happening and lets them intervene. They communicate via Modbus TCP
on port 502.

**Normal PLC register values — know these:**

| Register | Normal Range | What Happens Outside Range |
|---|---|---|
| WellPressure | Below 5000.0 | BOP activates above 5000 |
| WellTemp | Below 95.0 | BOP activates above 95 |
| WellFlowRate | 0.17 – 10.42 | Oil generation stops outside range |
| SeparatorTemp | 20.0 – 90.0 | Separator disables outside range |
| ExportPumpVibration | Below 5.0 | Alert above threshold |
| ExportPumpTemp | Below 95.0 | Alert above threshold |
| ExportPressure | Below 500.0 | Alert above threshold |

**Key coils to monitor:**

| Coil | Normal State | Red Team Indicator If... |
|---|---|---|
| SafeToOperate | TRUE | Flips FALSE without fire or hurricane |
| ESDActive | FALSE | Activates without threshold trigger |
| BOP | FALSE | Activates without pressure or temp cause |
| ManualOverride | FALSE | Activates unexpectedly |
| FireDetected | FALSE | TRUE without a scenario fire event |

**Red team activity on ICS systems looks like:**
- Coil state changes that do not match system logic thresholds
- SafeToOperate flipping FALSE without a scenario event
- ManualOverride activating — disables automatic control logic
- Unexpected Modbus write commands from non-HMI source IPs
- PLC program status changing from Running to Stopped in the
  OpenPLC dashboard

**The ICS anomaly:**
At 12:30 PM and 2:45 PM on competition day, a cargo ship arrives for
resupply. Person 1 (anomaly lead) handles crane interactions via the
HMI Home page within the scheduled time window. Review the crane
controls during the contract period — do not see them for the first
time on competition day.

---

### 8. Self-Enumeration During the Contract Period

Once SSH access opens Oct 26, one of the most valuable things Hunters
can do is enumerate your own Traditional VMs the way the red team
will. This surfaces findings for M&H to harden before competition day.

**nmap against your own systems:**
```bash
# Service and version detection
nmap -sV [target-ip]

# Full port scan
nmap -p- [target-ip]

# Script scan
nmap --script vuln [target-ip]

# Scan all Traditional VMs at once
nmap -sV [DB-ip] [AD-ip] [taskbox-ip] [webserver-ip]
```

**What to look for and report to M&H:**
- Open ports that should not be exposed
- Services running outdated versions with known CVEs
- Default credentials still in place
- Misconfigured services

Share findings with M&H immediately — every vulnerability you surface
is something they can harden before the red team finds it.

---

## Anomaly Triage Under Time Pressure

This is a meta-skill that applies across all categories. Internalize
these principles before competition day.

**When an anomaly drops:**
1. Read the full challenge description before doing anything
2. Estimate difficulty and point value
3. Assign to the Hunter best suited for the category
4. Set a 20-minute time limit — flag and move on if no progress
5. Never let one hard anomaly consume all three Hunters while easier
   ones go unsolved
   
**Time limits are real:** submit partial progress if a deadline is
approaching. Some anomalies award partial credit. Zero for a closed
anomaly is worse than partial credit for an incomplete one.

---

## What a Bad Day Looks Like

Three anomalies drop in the first hour. Two close before submission
because everyone was heads-down on the hardest one and nobody was
watching the platform for new drops. The red team pivots from HMI
to AD and nobody notices until a backdoor account has been live for
45 minutes because nobody was watching the logs. 

---

## See Also

- [cyberforce101.md](../../docs/cyberforce101.md)
- [shared-foundations.md](../../docs/shared-foundations.md)
- [monitoring-hardening.md](monitoring-hardening.md)
- [green-team.md](green-team.md)
- [competition-prep/cyberforce/week-by-week.md](../week-by-week.md)

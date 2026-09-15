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

Hunters are the most offensively-minded role on the team. While
Monitoring and Hardening locks systems down and Green Team keeps the
application running, Hunters are actively investigating, problem-solving,
and tracking adversary behavior across the entire network.

---

## What You Own

**Anomaly challenges** — CTF-style problems released by the White Team
throughout competition day. Time-limited — close whether you submit or
not.

**Red team recon** — investigating what the red team is doing across
all VMs including HMI and PLC. Identifying indicators of compromise
on Traditional VMs. Supporting incident report drafting.

---

## Technical Prep — Priority Order

### 1. CTF Fundamentals by Category

Anomalies are CTF-style challenges themed around the year's scenario.
The 2026 scenario is Vulcana Dynamics Ltd. — an energy utility with
power grid and radar network controls. Anomalies will likely touch
industrial control concepts, network protocols, and forensics themes
relevant to the scenario.

You do not all need to be strong in everything, but
every category should have at least one Hunter who can attempt it.

**Web Exploitation**
What to know:
- SQL injection — identifying and exploiting vulnerable input fields
- Cross-site scripting (XSS) — reflected and stored
- Directory traversal — accessing files outside the web root
- Common HTTP methods and status codes
- Burp Suite or browser dev tools for intercepting and modifying requests
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
- Hash functions — MD5, SHA1, SHA256 — and hash cracking with hashcat
  or john the ripper
- CyberChef for rapid encoding/decoding operations

Where to learn:
- [CryptoPals](https://cryptopals.com) — hands-on crypto challenges
- [CyberChef](https://gchq.github.io/CyberChef) — bookmark this now

**Forensics**
What to know:
- File format identification — `file` command, magic bytes
- Steganography — hidden data in images, audio files
  (`steghide`, `binwalk`, `strings`)
- pcap analysis — reading network captures in Wireshark
- Memory forensics basics — Volatility framework
- Log file analysis — reading and searching large log files
- Metadata extraction — `exiftool` for images and documents

Where to learn:
- [Wireshark sample captures](https://wiki.wireshark.org/SampleCaptures)
- CTFtime.org writeups for forensics challenges from past competitions

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
- Basic ROP (Return Oriented Programming) concepts
- Understanding checksums: NX, ASLR, stack canaries — and when each
  can be bypassed

Where to learn:
- [pwn.college](https://pwn.college) — the best free resource for pwn
- [LiveOverflow YouTube](https://youtube.com/@LiveOverflow) — binary
  exploitation series

**OSINT (Open Source Intelligence)**
What to know:
- Google dorking — advanced search operators (`site:`, `filetype:`,
  `inurl:`, `intitle:`)
- Reverse image search — Google Images, TinEye, Yandex
- WHOIS lookups and domain history
- Social media investigation techniques
- Wayback Machine for historical website content
- Geolocation from image metadata and visual clues

Where to learn:
- [OSINT Framework](https://osintframework.com)
- Trace Labs CTF events — OSINT-specific competitions

**Networking**
What to know:
- TCP/IP model — how packets flow between hosts
- Reading pcap files in Wireshark — filtering by protocol, IP, port
- Common protocols: HTTP, DNS, FTP, SMTP, Modbus — what normal traffic
  looks like for each
- nmap — port scanning, service detection, OS fingerprinting
- netcat for basic network connections and testing
- Identifying anomalous traffic patterns in captures

Where to learn:
- [Wireshark](https://wireshark.org) — download and practice on
  sample captures
- nmap official documentation and the nmap book (free online)

**General Skills / Miscellaneous**
What to know:
- Linux command line fluency — file navigation, grep, find, pipes
- Python scripting — automating repetitive tasks, parsing output,
  writing quick exploit scripts
- Reading and writing basic scripts in bash
- Recognizing and decoding common encodings quickly
- Git basics — cloning repos, reading commit history (sometimes
  relevant in forensics challenges)

---

### 2. Recon and Network Monitoring Tools

**nmap**
- Service enumeration: `nmap -sV [target]`
- OS detection: `nmap -O [target]`
- Running against your own network to see what the red team sees
- Script scanning for common vulnerabilities: `nmap --script vuln [target]`
- Do not run offensive scans against other teams — rules violation and
  disqualification risk

**Wireshark / tcpdump**
- Capturing traffic on your network interfaces
- Filtering by protocol, IP address, and port
- Identifying red team reconnaissance — port scans appear as SYN
  packets to many ports in rapid succession
- Identifying lateral movement — unexpected connections between VMs
- Identifying data exfiltration — large outbound transfers to
  unexpected destinations
- tcpdump for quick command-line captures:
  `tcpdump -i eth0 -w capture.pcap`

**Log analysis**
- Windows Event Viewer — key Event IDs to watch:
  - 4624: successful logon
  - 4625: failed logon (brute force indicator)
  - 4720: new user account created (red team persistence)
  - 4728/4732: user added to security/local group (privilege escalation)
  - 4672: special privileges assigned (admin logon)
- Linux auth logs: `/var/log/auth.log` — watch for failed SSH attempts
  and successful logins from unexpected sources
- Feeding log findings to Splunk (Incident Response owns this) —
  communicate what you find, do not try to manage Splunk yourself

---

### 3. ICS / OT Awareness (CyberForce-Specific)

The HMI and PLC are Assume Breach VMs — Hunters help Monitoring and
Hardening investigate red team activity on them. You do not need to be
an industrial control systems expert, but you need enough context to
recognize anomalous behavior.

**What normal looks like on the PLC:**
- WellPressure below 5000.0 (above triggers Blowout Prevention)
- WellTemp below 95.0 (above triggers Blowout Prevention)
- WellFlowRate between 0.17 and 10.42 (normal oil generation range)
- SeparatorTemp between 20.0 and 90.0 (Separator enabled in this range)
- SafeToOperate = TRUE (FALSE triggers Emergency Shutdown)
- Modbus traffic on port 502 — the PLC communicates via Modbus TCP

**Red team indicators on ICS systems:**
- SafeToOperate flipped to FALSE unexpectedly
- BOP (Blowout Prevention) activating without WellPressure or WellTemp
  threshold being reached
- Unexpected coil state changes — valves opening or closing without
  system logic triggering them
- Modbus write commands from unexpected source IPs
- ExportPump or WaterInjectPump activating outside normal parameters

**The ICS anomaly (competition day):**
At 12:30 PM and 2:45 PM a cargo ship arrives for resupply. Crane
interactions via the HMI Home page must be completed within the
scheduled time slot. It is a timed anomaly that requires HMI interaction, not a traditional
CTF challenge.

---

### 4. Anomaly Triage Under Time Pressure

**When an anomaly drops:**
1. Read the full challenge description before doing anything
2. Estimate difficulty and point value — is this worth the time?
3. Assign it to the Hunter best suited for the category
4. Set a mental time limit — if you have not made meaningful progress
   in 20 minutes, flag it and move on
5. Never let one hard anomaly consume all three Hunters while easier
   ones go unsolved

**Time limits are real:**
Anomalies close on a schedule. A challenge that closes before you
submit earns zero regardless of how close you were. Submit partial
progress if the deadline is approaching — some anomalies award partial
credit.

---

## What a Bad Day Looks Like

Three anomalies drop in the first hour. Two close before the team
submits because everyone was heads-down on the hardest one. The red
team pivots from HMI to AD and nobody notices until they have had a
backdoor account live for 45 minutes. The incident report is vague
because nobody documented the timeline as it happened.

---

## What a Good Day Looks Like

Someone is monitoring the platform continuously — the moment an
anomaly drops they read it, assign it, and set a deadline. Another person
catches unusual Modbus traffic on the PLC at 11 AM and immediately
calls it out — the team documents source IP, timestamp, and affected
coils, and submits a detailed incident report that scores well even
though the red team got in. The cargo ship anomaly at 12:30 PM is
completed within the time window.

---

## See Also

- [cyberforce101.md](../../docs/cyberforce101.md)
- [shared-foundations.md](../../docs/shared-foundations.md)
- [monitoring-hardening.md](monitoring-hardening.md)
- [competition-prep/cyberforce/week-by-week.md](../week-by-week.md)

# Monitoring & Hardening Role — Technical Prep Guide

**Last updated:** September 2026  
**Owner:** AJ  
**For:** Monitoring & Hardening competitors and team captains

---

## Role Overview

Monitoring and Hardening (2 people per team) is the defensive core of
the team. It is also the most technically demanding role because it
spans two completely different sets of responsibilities — fully
hardening the Traditional VMs and monitoring-only on the Assume Breach
VMs — with completely different rules governing each.

Understanding the distinction between Traditional and Assume Breach is
not optional. Confusing them during competition is a rules violation
that can cost points and flag a compliance issue with the White Team.

---

## What You Own

**Traditional VMs — full control:**
- DB: Windows Server 2022, MariaDB + phpMyAdmin
- AD: Windows Server 2019, Active Directory + DNS
- Task box: Ubuntu 22.04, SSH/SMTP/IMAP/SMB

**Assume Breach VMs — monitor only, no configuration changes:**
- HMI: Windows Server 2019, Ignition Gateway (port 8088)
- PLC: Ubuntu 22.04, OpenPLC (port 8080, Modbus TCP port 502)

**Shared responsibility with Green Team:**
- Webserver firewall rules that affect HTTP/HTTPS traffic — coordinate
  any changes with Green Team before implementing

---

## The Rule That Cannot Be Broken

**Do not touch the configuration on HMI or PLC. Ever.**

No password changes. No patches. No firewall rules. No service
modifications. No restarting services. The configuration on both
Assume Breach VMs is locked — your only job on these machines is to
watch what happens and document it.

If you accidentally modify an Assume Breach VM, report it to the
White Team via Support Ticket immediately.

---

## How CyberForce Access Works

CyberForce gives your team a **~3 week contract period** (Oct 26 –
Nov 14) of SSH access to all Traditional VMs before and during
competition day. This is the window where the majority of hardening
happens — not on competition day itself.

**Competition day (Nov 13–14)** is about active defense and incident
response, not setup. Systems should be hardened, monitored, and stable
by the time the red team goes live.

Use every day of the contract period. Teams that wait until the final
week to harden consistently run out of time and leave scored services
exposed.

---

## Contract Period Timeline — Monitoring & Hardening

**Week 1 (Oct 26 – Nov 1): Credential changes and initial hardening**
- Change all default credentials on Traditional VMs immediately on
  Oct 26 — report every scored service password change to the White
  Team via Support Ticket before moving on
- AD: audit all accounts and privileged groups, enable audit logging,
  enforce password and lockout policy via GPO
- Task box: harden SSH, audit users and services, configure ufw
- DB: change MariaDB root password, harden phpMyAdmin access,
  restrict remote database connections
- Coordinate with Hunters on enumeration findings — they will scan
  Traditional VMs and surface vulnerabilities for you to fix
- Get familiar with the ICS environment — review the OpenPLC
  monitoring dashboard and Ignition views, understand normal values
  before you need to spot anomalies

**Week 2 (Nov 2 – Nov 8): Deep hardening and monitoring setup**
- Continue hardening based on Hunter enumeration findings
- Set up log forwarding to Splunk if available
- Configure monitoring on Traditional VMs — Windows Event logging,
  Linux auth logs
- Verify all scored services are still running after hardening
  changes — test each one explicitly after every change
- Document everything changed for Green Team's Security Documentation

**Week 3 (Nov 9 – Nov 13): Final checks and competition prep**
- Final enumeration sweep — verify the state of every Traditional VM
- Confirm ICS monitoring workflow with Hunters
- Verify Ignition Gateway OPC UA connection to PLC is stable
- Confirm all scored services are responding correctly
- Competition Day (Nov 13): active defense — monitor for red team
  activity, respond to intrusions, keep scored services running

---

## Technical Prep — In Order

Work through these in sequence. Each item builds on the one before
it. The Active Directory and ICS work in items 3 and 8 will not land
well without the Linux, Windows, logging, and firewall foundation
from items 1–2 and 4–5.

The prep items map directly to your weekly captain meeting assignments
— one item per week, with the contract period applied work in Week 8+.

---

### 1. Linux Hardening Fundamentals

The Task box (Ubuntu 22.04) runs SSH, SMTP, IMAP, and SMB — all
scored services. Multiple scored services running on one machine means
any hardening mistake that breaks one of them costs Blue Team (20%)
uptime points immediately.

**VM this applies to:** Task box (Ubuntu 22.04)

**What to know:**

SSH hardening:
```bash
sudo nano /etc/ssh/sshd_config

# Key settings:
PermitRootLogin no
PasswordAuthentication no   # only after key-based auth confirmed
MaxAuthTries 3
Protocol 2
AllowUsers sysadmin [your-users]

# Restart and verify — do not close current session until confirmed
sudo systemctl restart sshd
ssh sysadmin@localhost
```

User and group audit:
```bash
# All users with login shells
grep -v '/nologin\|/false' /etc/passwd

# Sudo access
sudo -l
cat /etc/sudoers

# SUID binaries (privilege escalation risk)
find / -perm -4000 -type f 2>/dev/null

# World-writable files
find / -perm -o+w -type f 2>/dev/null
```

Service audit — disable what is not needed:
```bash
# List running services
systemctl list-units --type=service --state=running

# List open ports
ss -tulnp

# Disable unnecessary services
sudo systemctl stop [service]
sudo systemctl disable [service]
```

Firewall (ufw):
```bash
sudo ufw enable
sudo ufw allow 22/tcp    # SSH
sudo ufw allow 25/tcp    # SMTP
sudo ufw allow 143/tcp   # IMAP
sudo ufw allow 445/tcp   # SMB
sudo ufw status verbose
```

**Critical rule — verify scored services after every firewall change:**
```bash
# Test SMTP
telnet localhost 25

# Test IMAP
telnet localhost 143

# Test SSH
ssh sysadmin@localhost
```

Never add a firewall rule without immediately testing the services it
could affect. A rule that blocks SMTP silently kills uptime scoring
until someone notices the service is down.

**File permissions:**
- `chmod` — changes what users can do with a file (read/write/execute)
- `chown` — changes who owns a file
- `umask` — sets default permissions for newly created files
- Know how to read `rwxr-xr-x` and what each section means

**Where to learn:**
- [OverTheWire: Bandit](https://overthewire.org/wargames/bandit/) —
  Linux fundamentals through hands-on challenges
- [Linux Journey](https://linuxjourney.com) — structured beginner
  course covering permissions, users, and services

---

### 2. Windows Hardening Fundamentals

The DB VM (Windows Server 2022) runs MariaDB and phpMyAdmin. These
are high-value targets — MariaDB stores the application data and
phpMyAdmin is a web-accessible database management interface that the
red team will target if it is left exposed.

**VM this applies to:** DB (Windows Server 2022)

**What to know:**

Change Administrator password immediately and report to White Team:
```powershell
net user Administrator [new-strong-password]
```

Disable unnecessary startup services:
- Open `services.msc`
- Right-click any unnecessary service → Properties → Startup type:
  Disabled

Audit local users:
```powershell
# List all local users
Get-LocalUser

# List Administrators group members
Get-LocalGroupMember "Administrators"

# Disable an account
Disable-LocalUser -Name "[username]"
```

Windows Firewall:
- Open "Windows Firewall with Advanced Security"
- Block external access to port 3306 (MariaDB) — only the webserver
  VM IP should be able to connect
- phpMyAdmin runs on HTTP (port 80 or 8080) — restrict access to
  only what is needed for the application

**MariaDB hardening:**
```sql
-- Connect to MariaDB
mysql -u root -p

-- Change root password
ALTER USER 'root'@'localhost' IDENTIFIED BY '[new-password]';

-- Remove anonymous users
DELETE FROM mysql.user WHERE User='';

-- Remove remote root login
DELETE FROM mysql.user WHERE User='root'
  AND Host NOT IN ('localhost', '127.0.0.1');

-- Remove test database
DROP DATABASE IF EXISTS test;

-- Apply changes
FLUSH PRIVILEGES;
```

**Coordinate with Green Team on MariaDB credential changes:**
The Laravel application on the webserver connects to MariaDB using
credentials stored in the `.env` file. If you change the MariaDB
password without telling Green Team first, the application will throw
database connection errors and stop serving pages — which costs Green
Team uptime points. Always notify Green Team before changing DB
credentials.

**phpMyAdmin hardening:**
- Restrict phpMyAdmin access to localhost only if possible
- Change login credentials
- Consider disabling phpMyAdmin entirely during competition — it is
  a high-value red team target and the application may not need it

**Where to learn:**
- [TryHackMe Windows Fundamentals](https://tryhackme.com/module/windows-fundamentals)
- MySQL/MariaDB official documentation for user management
- Microsoft documentation for Windows Server hardening

---

### 3. Active Directory

AD is the red team's highest-value target. If they own AD, they own
authentication across the entire network — every user account,
permission, and service that authenticates against the domain is at
risk. This is why AD gets its own week and why it comes before
logging, firewall, and monitoring in the prep sequence.

**VM this applies to:** AD/DNS (Windows Server 2019)

**Why AD hardening matters more than everything else:**
- POP3 (scored service) authenticates via AD accounts — if AD breaks,
  POP3 scoring stops
- DNS is integrated with AD — if DNS breaks, AD authentication breaks,
  which cascades into POP3
- The red team will target AD first because compromising it gives
  them access to everything that authenticates against the domain

**At contract start — do these immediately:**

Change the built-in Administrator password:
```powershell
net user Administrator [new-strong-password]
# Report to White Team via Support Ticket immediately
```

Rename the built-in Administrator account via Group Policy:
```
Computer Config → Windows Settings → Security Settings →
Local Policies → Security Options →
Accounts: Rename administrator account
```

Audit all domain accounts:
```powershell
# List all users
Get-ADUser -Filter * | Select Name, Enabled, PasswordNeverExpires

# Find accounts with password never expires
Get-ADUser -Filter {PasswordNeverExpires -eq $true} | Select Name

# Find accounts with no password required
Get-ADUser -Filter {PasswordNotRequired -eq $true} | Select Name

# Check last logon for all users
Get-ADUser -Filter * -Properties LastLogonDate |
  Select Name, LastLogonDate | Sort LastLogonDate
```

Audit privileged group memberships — document the baseline:
```powershell
# Domain Admins
Get-ADGroupMember "Domain Admins" | Select Name, SamAccountName

# Enterprise Admins
Get-ADGroupMember "Enterprise Admins" | Select Name, SamAccountName

# Schema Admins
Get-ADGroupMember "Schema Admins" | Select Name, SamAccountName
```

Write down this baseline. Check it throughout competition day. Any
new member in Domain Admins that you did not add is a red team
indicator.

**Group Policy settings to enforce:**

Account lockout policy:
- Lockout threshold: 5 invalid attempts
- Lockout duration: 30 minutes
- Observation window: 30 minutes

Password policy:
- Minimum length: 12 characters
- Complexity required: yes
- Password history: 10 passwords

Audit policy (set to Success and Failure for all):
- Logon events
- Account management
- Privilege use
- Object access

Restricted groups:
- Enforce Domain Admins membership via GPO so any unauthorized
  addition gets overwritten on next GPO refresh

**DNS — keep it healthy:**
```powershell
# Verify DNS is resolving correctly
nslookup [domain-name]
Resolve-DnsName [domain-name]

# Check DNS zones
Get-DnsServerZone
```

Never change DNS server settings without verifying AD still
authenticates correctly afterward. Test POP3 connectivity after
any DNS change — POP3 authenticates via AD accounts and DNS
breakage will silently kill POP3 scoring.

**Key Windows Event IDs to monitor throughout competition:**

| Event ID | What It Means | Action If Unexpected |
|---|---|---|
| 4624 | Successful logon | Note source IP |
| 4625 | Failed logon | Spike = brute force |
| 4720 | New user account created | Red team persistence |
| 4722 | User account enabled | Check if expected |
| 4728 | User added to global group | Check if Domain Admins |
| 4732 | User added to local group | Check if Administrators |
| 4672 | Special privileges assigned | Admin logon — expected? |
| 4776 | Credential validation | Failed = brute force |

**Where to learn:**
- [TryHackMe Active Directory Basics](https://tryhackme.com/room/activedirectorybasics)
- Microsoft documentation for Group Policy and Active Directory
- [Microsoft Learn: AD DS](https://learn.microsoft.com/en-us/windows-server/identity/ad-ds/get-started/virtual-dc/active-directory-domain-services-overview)

---

### 4. Logging and Log Analysis

Logs are how you know something is wrong before it becomes
unrecoverable. This item covers where logs live on each Traditional
VM and how to read them quickly under competition pressure.

**VMs this applies to:** All Traditional VMs

**Linux log locations (Task box — Ubuntu 22.04):**

| Log | Location | What It Shows |
|---|---|---|
| Auth log | `/var/log/auth.log` | SSH logins, sudo usage, su |
| System log | `/var/log/syslog` | General system events |
| Mail log | `/var/log/mail.log` | SMTP activity |
| Kernel log | `/var/log/kern.log` | Kernel-level events |

```bash
# Watch auth log live
tail -f /var/log/auth.log

# Find failed login attempts
grep "Failed password" /var/log/auth.log

# Find successful logins
grep "Accepted" /var/log/auth.log

# Find sudo usage
grep "sudo" /var/log/auth.log

# Find new user creation
grep "useradd\|adduser" /var/log/auth.log
```

**Windows log locations (AD VM and DB VM):**

Event Viewer → Windows Logs:
- **Security** — logons, account management, privilege use (most
  important for competition)
- **System** — system-level events, service starts and stops
- **Application** — application-level events

```powershell
# Filter Event Viewer for a specific Event ID via PowerShell
Get-EventLog -LogName Security -InstanceId 4720 -Newest 20

# Find all failed logons
Get-EventLog -LogName Security -InstanceId 4625 -Newest 50

# Find new accounts created
Get-EventLog -LogName Security -InstanceId 4720
```

**What a brute force attack looks like vs. a single failed login:**
- Single failed login: one Event ID 4625 from one source IP
- Brute force: many Event ID 4625 entries from the same source IP
  in rapid succession — dozens or hundreds in a short window
- Password spray: many Event ID 4625 entries across many accounts
  from the same source IP — attacker trying one password against
  many accounts

**Where to learn:**
- [TryHackMe: Investigating Windows](https://tryhackme.com/room/investigatingwindows)
- Linux man pages for `grep`, `tail`, `journalctl`

---

### 5. Firewall Configuration

Firewall rules are one of the most important hardening tools and one
of the most common ways teams accidentally break scored services.
Every firewall change must be verified immediately by testing the
service it could affect.

**VMs this applies to:**
- Task box → ufw (Ubuntu)
- DB VM → Windows Firewall (Windows Server 2022)
- Webserver → firewalld (OpenSUSE Leap) — coordinate with Green Team

**The principle of least privilege for firewalls:**
Only allow traffic that is explicitly needed. Deny everything else.
Do not leave ports open "just in case" — every open port is an
attack surface.

**ufw on Task box:**
```bash
# Reset to default (deny all incoming, allow all outgoing)
sudo ufw default deny incoming
sudo ufw default allow outgoing

# Allow only what is needed
sudo ufw allow 22/tcp    # SSH
sudo ufw allow 25/tcp    # SMTP
sudo ufw allow 143/tcp   # IMAP
sudo ufw allow 445/tcp   # SMB

# Enable and check
sudo ufw enable
sudo ufw status verbose
```

**After every ufw rule change — test the affected service:**
```bash
telnet localhost 25    # SMTP
telnet localhost 143   # IMAP
ssh sysadmin@localhost # SSH
```

**Windows Firewall on DB VM:**
- Block port 3306 (MariaDB) from all external sources
- Only the webserver VM's IP should be able to connect to MariaDB
- phpMyAdmin port (80 or 8080) — restrict to necessary access only

**firewalld on Webserver (coordinate with Green Team):**
```bash
# Check current rules
sudo firewall-cmd --list-all

# Allow HTTP and HTTPS
sudo firewall-cmd --add-service=http --permanent
sudo firewall-cmd --add-service=https --permanent
sudo firewall-cmd --reload
```

**What to do if a scored service goes down after a firewall change:**
1. Check the service status: `systemctl status [service]`
2. Check ufw rules: `sudo ufw status verbose`
3. Test the port directly: `telnet localhost [port]`
4. If the firewall is blocking it, temporarily allow the port to
   confirm: `sudo ufw allow [port]`
5. Fix the rule and re-test before moving on

**Where to learn:**
- ufw official documentation
- [DigitalOcean ufw guide](https://www.digitalocean.com/community/tutorials/how-to-set-up-a-firewall-with-ufw-on-ubuntu)
- Microsoft documentation for Windows Firewall with Advanced Security

---

### 6. Monitoring Tools

These tools give you visibility across your systems during the
contract period and competition day. You cannot respond to red team
activity you cannot see.

**VMs this applies to:** All Traditional VMs

**Wazuh — host-based intrusion detection:**
Wazuh is an open-source SIEM and intrusion detection platform. It
collects logs from agents installed on each VM, correlates them, and
generates alerts for suspicious activity.

What to configure it to alert on:
- New user accounts created
- Changes to privileged groups
- SUID binary changes
- Failed authentication spikes
- File modifications in sensitive directories (`/etc/passwd`,
  `/etc/shadow`, `/etc/sudoers`)

**Zeek — network traffic analysis:**
Zeek (formerly Bro) analyzes network traffic and generates structured
logs for connections, DNS queries, HTTP requests, and more. More
powerful than reading pcap files manually because it automatically
categorizes traffic into meaningful logs.

Why Zeek is useful over manual log checking:
- Automatically identifies scanning behavior (many connections to
  many ports from one source)
- Logs all DNS queries — useful for detecting unusual lookups
- Logs all HTTP requests — useful for detecting web application attacks
- Runs continuously — you do not have to be watching to capture events

**Splunk — log aggregation and search:**
Splunk collects logs from all your VMs into one searchable interface.
Free account available — sign up before the contract period opens.

Basic SPL (Splunk Processing Language) queries:
```
# All failed logons
index=* EventID=4625

# New user accounts created
index=* EventID=4720

# Failed SSH attempts on Task box
index=* source="/var/log/auth.log" "Failed password"

# All events from a specific source IP
index=* src_ip=[suspicious-ip]
```

**Where to learn:**
- [Splunk free training](https://education.splunk.com) — Intro to
  Splunk and Search Under the Hood are the most relevant
- [Wazuh documentation](https://documentation.wazuh.com)
- [Zeek documentation](https://docs.zeek.org)

---

### 7. Audit Logs and System Monitoring

Audit logs are more granular than system logs — they record specific
actions taken on specific files and objects, not just events. This
item covers setting up audit rules so you are notified when something
sensitive changes.

**VMs this applies to:** All Traditional VMs

**Linux — auditctl:**
```bash
# Monitor changes to /etc/passwd
sudo auditctl -w /etc/passwd -p wa -k passwd_changes

# Monitor changes to /etc/shadow
sudo auditctl -w /etc/shadow -p wa -k shadow_changes

# Monitor changes to /etc/sudoers
sudo auditctl -w /etc/sudoers -p wa -k sudoers_changes

# View audit log
sudo ausearch -k passwd_changes

# Make rules persistent across reboots
sudo nano /etc/audit/rules.d/audit.rules
# Add the same -w rules there
```

**Windows — audit policy:**

Enable via Group Policy:
```
Computer Config → Windows Settings → Security Settings →
Advanced Audit Policy Configuration
```

Key categories to enable (Success and Failure):
- Account Logon — credential validation
- Account Management — user/group changes
- Logon/Logoff — interactive and network logons
- Object Access — file and registry access
- Privilege Use — use of sensitive privileges
- Policy Change — audit policy changes themselves

**What good incident report evidence looks like:**
When the red team does something, you need:
- Timestamp of when it happened
- Source IP of where it came from
- What they did (new account, file modified, privilege escalated)
- What was affected
- What you did in response

Audit logs provide the timestamp and the what. Network logs (Zeek,
Wireshark) provide the source IP. Putting them together is what makes
an incident report specific and scoreable.

**Where to learn:**
- Linux `auditd` man page
- Microsoft documentation for Advanced Audit Policy Configuration

---

### 8. ICS Monitoring — Contract Period Applied Work

**This section requires access to the actual HMI and PLC VMs.**
Do not try to practice this before the contract period opens — you
cannot meaningfully simulate monitoring real industrial control
systems without the actual machines.

Once SSH access opens Oct 26, use Week 1 of the contract period to
get familiar with both VMs before competition day. Understanding what
normal looks like is the only way to recognize anomalies when they
appear.

**The Assume Breach Rule — repeat because it matters:**
No password changes, no patches, no firewall rules, no service
modifications on HMI or PLC. Monitor only. If you accidentally
modify either VM, report it to the White Team immediately.

---

**The ICS Architecture — Vulcana Dynamics Ltd.:**

VDL is a geothermal power utility — the sole provider of electricity
across the Kaldera Archipelago. The island defense radar network
(two installations: northern atoll and southern reef) feeds the Air
Defense Operations Centre on Kaldera Main. Both radar sites depend
entirely on uninterrupted power from the VDL grid. No backup power
exists at the radar sites.

The HMI and PLC represent the controls for this grid:

```
Physical Equipment (simulated — geothermal grid)
        ↑↓ Modbus protocol (port 502)
PLC — Ubuntu 22.04, OpenPLC
        ↑↓ OPC UA / Modbus TCP
HMI — Windows Server 2019, Ignition Gateway (port 8088)
        ↑↓
Engineer/Operator views Ignition screens
```

---

**Monitoring the PLC (Ubuntu 22.04, OpenPLC):**

Access: `<plc-ip>:8080` — credentials: `openplc:openplc`
Do not change. Assume Breach rule.

Normal register values:

| Register | Normal Range | Alert If... |
|---|---|---|
| WellPressure | Below 5000.0 | Above 5000 → BOP activates |
| WellTemp | Below 95.0 | Above 95 → BOP activates |
| WellFlowRate | 0.17 – 10.42 | Outside range → oil gen stops |
| SeparatorTemp | 20.0 – 90.0 | Outside → separator disables |
| ExportPumpVibration | Below 5.0 | Above → export pump alert |
| ExportPumpTemp | Below 95.0 | Above → export pump alert |
| ExportPressure | Below 500.0 | Above → export pump alert |

Key coils to monitor:

| Coil | Normal | Red Team Indicator If... |
|---|---|---|
| SafeToOperate | TRUE | Flips FALSE without fire/hurricane |
| ESDActive | FALSE | Activates without threshold trigger |
| BOP | FALSE | Activates without pressure/temp cause |
| ManualOverride | FALSE | Activates unexpectedly |
| FireDetected | FALSE | TRUE without scenario event |

Monitoring workflow during competition:
1. Check OpenPLC dashboard status (Running vs. Stopped) every 15–20
   minutes
2. Review the Monitoring tab — compare current values to normal ranges
3. Document anomalies immediately: timestamp, coil/register name,
   value before and after, source IP if visible in network logs
4. Alert Vulnerability Hunters — they investigate and draft the
   incident report

---

**Monitoring the HMI (Windows Server 2019, Ignition):**

Access: Ignition Gateway at `localhost:8088`
Credentials: `blueteam:BlueTeam2025!` — do not change. Assume Breach.

The four Ignition views:

**Home page** — overall system status:
- Weather (Sunny, Hurricane — weather affects system behavior)
- Current and Total Oil Export in BBL
- Fire Detection status
- Blowout Prevention System status
- Flare Valve and Flare Pilot status
- Crane controls (bottom right — used for the ICS anomaly)

**Systems page** — individual sensors and direct controls:
- Well Pressure/Valve/Temp sensors
- Gas/Water/Oil Valve, Export Pump status
- Fire Suppression Pump, Flare Valve
- Start System / Stop System / Manual Override buttons
- Watch for state changes you did not initiate — this is where
  red team interference is most visible

**Alarms page** — threshold-triggered alerts:
- All system alarms with priority levels (Critical, High, Medium)
- A Critical alarm you did not trigger is a red team indicator
- Check this page frequently throughout competition day

**Charts page** — historical production data:
- Oil output over time
- Sudden drops or spikes correspond to events — cross-reference
  with timestamps from PLC monitoring log

Red team activity on the HMI looks like:
- Critical alarms appearing without corresponding system events
- Systems page valve state changes you did not initiate
- Manual Override activating on the Systems page
- Stop System triggered unexpectedly
- Ignition Gateway OPC UA connection to PLC dropping

**The ICS Anomaly — your role:**
At 12:30 PM and 2:45 PM on competition day, a cargo ship arrives for
resupply. Vulnerability Hunters execute the crane interactions via
the HMI Home page. Your job is to ensure the HMI and PLC are
operational when the anomaly windows open.

Pre-anomaly checklist (before 12:15 PM and 2:30 PM):
- Confirm PLC is Running in the OpenPLC dashboard
- Confirm Ignition Gateway OPC UA connection to PLC shows Connected
- Confirm HMI Home page is loading and showing current system status
- Alert Vulnerability Hunters that the anomaly window is approaching

---

## Communication Protocols

**→ Vulnerability Hunters:**
Surface anomalous ICS behavior immediately. Do not wait until you
are certain. A callout like "PLC showing BOP coil TRUE at 11:23 AM,
WellPressure is 1800 — below threshold, investigating" gives Hunters
what they need to start an incident report.

**→ Green Team:**
Any firewall change that could affect HTTP/HTTPS traffic must be
communicated to Green Team before implementing. A rule that breaks
scoring engine access to the webserver costs both of you uptime points.

**→ Hunters (enumeration findings during contract period):**
When Hunters run nmap against Traditional VMs and find open ports or
vulnerable services, they flag them to you. Address these in Week 1
of the contract period — do not let them sit.

**→ White Team (password changes):**
Every scored service password change must be reported via Support
Ticket immediately. Business & Writing (CCDC) or the designated
ticket filer on your team handles this for CyberForce — but the
responsibility is yours to flag it the moment the change is made.

---

## What a Bad Day Looks Like

The red team creates a new domain admin account on AD in the first
hour and nobody notices because audit logging was never enabled during
the contract period. The Task box mail services go down after a Week
3 firewall rule change and stay down for 45 minutes because nobody
tested SMTP after the change. The PLC shows anomalous Modbus commands
at 2 PM but there are no timestamps or source IPs because the
monitoring workflow was never established during the contract period.

---

## What a Good Day Looks Like

By competition day AD is hardened, audit logging is running, and
privileged group memberships are baselined from Week 1. Every Task
box firewall change was verified immediately by testing SMTP and
IMAP. The PLC monitoring dashboard is checked every 20 minutes and
the HMI Alarms page shows no unexpected Critical alerts. When the
red team attempts lateral movement from HMI at 1 PM, it is caught
within 10 minutes, documented with source IP and timestamp, and
handed to Hunters for an incident report that scores well even though
the red team got in.

---

## See Also

- [cyberforce101.md](../../docs/cyberforce101.md)
- [shared-foundations.md](../../docs/shared-foundations.md)
- [green-team.md](green-team.md)
- [vuln-hunters.md](vuln-hunters.md)
- [competition-prep/cyberforce/week-by-week.md](../week-by-week.md)

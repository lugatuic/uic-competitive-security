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

**Competition day (Nov 14)** is about active defense and incident
response, not setup. Systems should be hardened, monitored, and stable
by the time the red team goes live.

Use every day of the contract period. Teams that wait until the final
week to harden consistently run out of time.

---

## Contract Period Timeline — Monitoring & Hardening

**Week 1 (Oct 26 – Nov 1): Credential changes and initial hardening**
- Change all default credentials on Traditional VMs immediately on
  Oct 26 — report every scored service password change to White Team
  via Support Ticket before moving on
- AD: audit all accounts and privileged groups, enable audit logging,
  enforce password and lockout policy via GPO
- Task box: harden SSH, audit users and services, configure ufw
- DB: change MariaDB root password, harden phpMyAdmin access,
  restrict remote database connections
- Coordinate with Hunters on enumeration findings — they will scan
  your Traditional VMs and surface vulnerabilities for you to fix
- Get familiar with the ICS environment — review OpenPLC monitoring
  dashboard and Ignition views, understand normal values before you
  need to spot anomalies

**Week 2 (Nov 2 – Nov 8): Deep hardening and monitoring setup**
- Continue hardening based on Hunter enumeration findings
- Set up log forwarding to Splunk if available
- Configure monitoring on Traditional VMs — Windows Event logging,
  Linux auth logs
- Verify all scored services are still running after hardening changes
  (use curl and telnet to test each one)
- Document everything changed for Green Team's Security Documentation

**Week 3 (Nov 9 – Nov 13): Final checks and competition prep**
- Final enumeration sweep — verify the state of every Traditional VM
- Confirm ICS monitoring workflow with Hunters — how anomalous
  behavior gets surfaced and documented
- Verify Ignition Gateway OPC UA connection to PLC is stable
- Confirm all scored services are responding correctly
- Competition Day (Nov 14): active defense — monitor for red team
  activity, respond to intrusions, keep scored services running

---

## Technical Prep — Traditional VMs

### Priority 1: Active Directory (AD VM)

AD is the red team's highest-value target on the Traditional side.
Harden AD first. Everything else waits.

**Week 1 — do these immediately at contract start:**

Change the built-in Administrator password:
```powershell
net user Administrator [new-strong-password]
```

Rename the built-in Administrator account:
```powershell
# Via Group Policy: Computer Config → Windows Settings →
# Security Settings → Local Policies → Security Options →
# Accounts: Rename administrator account
```

Audit all domain accounts:
```powershell
# List all users
Get-ADUser -Filter * | Select Name, Enabled, PasswordNeverExpires

# Find accounts with password never expires
Get-ADUser -Filter {PasswordNeverExpires -eq $true} | Select Name

# Find disabled accounts
Get-ADUser -Filter {Enabled -eq $false} | Select Name

# Check last logon
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

**Group Policy settings to enforce:**
- Account lockout: threshold 5 attempts, duration 30 min,
  observation window 30 min
- Password policy: minimum 12 characters, complexity required,
  history 10 passwords
- Audit policy: logon events, account management, privilege use,
  object access — all set to Success and Failure
- Restricted groups: enforce Domain Admins membership

**DNS — keep it healthy:**
AD authentication depends on DNS. If DNS breaks, POP3 scoring stops
because POP3 authenticates via AD accounts.

```powershell
# Verify DNS is resolving correctly
nslookup [domain-name]
Resolve-DnsName [domain-name]

# Check DNS zones
Get-DnsServerZone
```

Never change DNS server settings without verifying AD still
authenticates correctly afterward.

**What to watch for throughout competition day:**

| Event ID | What It Means | Action |
|---|---|---|
| 4624 | Successful logon | Note source IP if unexpected |
| 4625 | Failed logon | Spike = brute force attempt |
| 4720 | New user created | Red team persistence — investigate |
| 4728/4732 | User added to group | Check if it was Domain Admins |
| 4672 | Special privileges assigned | Admin logon — expected or not? |
| 4776 | Credential validation | Failed = brute force |

---

### Priority 2: Task Box (Ubuntu 22.04)

Multiple scored services run here — SSH, SMTP, IMAP, SMB. Every
service you accidentally break costs Blue Team (20%) uptime points.
Verify each service after every change.

**Week 1 — immediate actions:**

Change default credentials first:
```bash
# Change password
passwd sysadmin
# Report to White Team via Support Ticket immediately
```

SSH hardening:
```bash
sudo nano /etc/ssh/sshd_config

# Set these values:
PermitRootLogin no
MaxAuthTries 3
Protocol 2
AllowUsers sysadmin [your-team-users]

# Restart and verify
sudo systemctl restart sshd
# Verify SSH still works before closing current session
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

**After every firewall change — verify scored services:**
```bash
# Test SMTP
telnet localhost 25

# Test IMAP
telnet localhost 143

# Test SSH
ssh sysadmin@localhost
```

---

### Priority 3: DB VM (Windows Server 2022)

The Laravel application on the webserver connects to MariaDB here.
If the database goes down or credentials change without coordinating
with Green Team, the application breaks.

**Week 1 — immediate actions:**

Change Administrator password immediately. Report to White Team.

MariaDB hardening:
```sql
-- Change root password
ALTER USER 'root'@'localhost' IDENTIFIED BY '[new-password]';

-- Remove anonymous users
DELETE FROM mysql.user WHERE User='';

-- Remove remote root login
DELETE FROM mysql.user WHERE User='root'
  AND Host NOT IN ('localhost', '127.0.0.1');

-- Remove test database
DROP DATABASE IF EXISTS test;

-- Flush privileges
FLUSH PRIVILEGES;
```

phpMyAdmin hardening:
- Restrict access to localhost only if possible
- Change login credentials
- Consider disabling phpMyAdmin if the application does not need it
  during competition — it is a high-value red team target

**Coordinate with Green Team:** if you change MariaDB credentials,
Green Team must update the Laravel `.env` file immediately or the
application will throw database connection errors and stop serving
pages. Never change DB credentials without telling Green Team first.

Windows Firewall:
- Block external access to port 3306 (MariaDB)
- Only the webserver VM IP should be able to connect to MariaDB

---

## Technical Prep — Assume Breach VMs

### The ICS Environment — How It Works

The HMI and PLC are an interconnected industrial control system. The
PLC runs the physical control logic. The HMI is the operator interface.
They communicate via Modbus TCP on port 502.

```
Physical Equipment (simulated)
        ↑↓ Modbus protocol (port 502)
PLC — Ubuntu 22.04, OpenPLC
        ↑↓ OPC UA / Modbus TCP
HMI — Windows Server 2019, Ignition Gateway (port 8088)
        ↑↓
Engineer/Operator views Ignition screens
```

**Use the contract period to get familiar with both VMs** — the
Ignition views, the normal register values, the OpenPLC monitoring
dashboard. You cannot harden them but you can learn what normal looks
like so anomalies are obvious on competition day.

---

### Monitoring the PLC (Ubuntu 22.04, OpenPLC)

**Access:** `<plc-ip>:8080` — credentials: `openplc:openplc`
Do not change these. Assume Breach rule.

**Normal register values — know these before competition day:**

| Register | Normal Range | Alert If... |
|---|---|---|
| WellPressure | Below 5000.0 | Above 5000 → BOP fires |
| WellTemp | Below 95.0 | Above 95 → BOP fires |
| WellFlowRate | 0.17 – 10.42 | Outside range → oil gen stops |
| SeparatorTemp | 20.0 – 90.0 | Outside → separator disables |
| ExportPumpVibration | Below 5.0 | Above → export pump alert |
| ExportPumpTemp | Below 95.0 | Above → export pump alert |
| ExportPressure | Below 500.0 | Above → export pump alert |

**Key coils to monitor:**

| Coil | Normal State | Red Team Indicator If... |
|---|---|---|
| SafeToOperate | TRUE | Flips FALSE without fire/hurricane |
| ESDActive | FALSE | Activates without threshold trigger |
| BOP | FALSE | Activates without pressure/temp cause |
| ManualOverride | FALSE | Activates unexpectedly |
| FireDetected | FALSE | TRUE without scenario fire event |
| WellValve | Varies | Changes without system logic cause |

**Red team activity on the PLC looks like:**
- Coil state changes that do not match system logic thresholds
- SafeToOperate flipping FALSE without a scenario event
- ManualOverride activating — disables automatic control logic
- Unexpected Modbus write commands from non-HMI source IPs
- PLC program status changing from Running to Stopped

**Monitoring workflow during competition:**
1. Check OpenPLC dashboard status (Running vs. Stopped) every 15–20
   minutes
2. Review the Monitoring tab — compare current coil and register
   values against the normal ranges above
3. Document anomalies immediately: timestamp, coil/register name,
   value before and after, source IP if visible
4. Alert Vulnerability Hunters immediately — they investigate and
   draft the incident report

---

### Monitoring the HMI (Windows Server 2019, Ignition)

**Access:** Ignition Gateway at `localhost:8088`
Credentials: `blueteam:BlueTeam2025!` — do not change. Assume Breach.

**The four Ignition views:**

**Home page** — overall system status:
- Weather (Sunny, Hurricane — affects system behavior)
- Current and Total Oil Export in BBL
- Fire Detection, Blowout Prevention System status
- Flare Valve, Flare Pilot status
- Crane controls bottom right — used for the ICS anomaly

**Systems page** — individual sensors and direct controls:
- Well Pressure/Valve/Temp sensors
- Gas/Water/Oil Valve, Export Pump status
- Fire Suppression Pump, Flare Valve
- Start System / Stop System / Manual Override buttons
- This is where a red team would interact with controls directly —
  watch for state changes you did not initiate

**Alarms page** — threshold-triggered alerts:
- All system alarms with priority levels (Critical, High, Medium)
- Active and cleared alarm history
- A Critical alarm you did not trigger is a red team indicator

**Charts page** — historical production data:
- Oil output over time — sudden drops or spikes correspond to events
- Use during the contract period to understand normal output patterns

**Red team activity on the HMI looks like:**
- New Critical alarms without corresponding system events
- Systems page valve state changes you did not initiate
- Manual Override activating on the Systems page
- Stop System triggered unexpectedly
- Ignition Gateway OPC UA connection to PLC dropping

**Data Historian (MySQL, port 3306):**
Read-only monitoring only — do not modify the database.
```bash
# View recent production data
mysql -u blueteam -p -h localhost obsidianpearl
SELECT * FROM production ORDER BY t_stamp DESC LIMIT 25;

# View recent alarm events
SELECT * FROM alarm_events ORDER BY t_stamp DESC LIMIT 25;
```

---

### The ICS Anomaly — Your Role

At 12:30 PM and 2:45 PM on competition day, a cargo ship arrives for
resupply. Vulnerability Hunters execute the crane interactions via
the HMI Home page — that is not your job.

**Your job:** ensure HMI and PLC are operational when the anomaly
windows open.

**Pre-anomaly checklist (before 12:15 PM and 2:30 PM):**
- Confirm PLC is Running in the OpenPLC dashboard
- Confirm Ignition Gateway OPC UA connection to PLC shows Connected
- Confirm HMI Home page is loading and showing current system status
- Alert Vulnerability Hunters that the anomaly window is approaching

If the OPC UA connection is down, the crane controls will not
function. Catch this early — do not discover it at 12:29 PM.

---

## Communication Protocols

**→ Vulnerability Hunters:**
Surface anomalous ICS behavior immediately. Do not wait until you
are certain — flag it early. A callout like "PLC showing BOP coil
TRUE at 11:23 AM, WellPressure is 1800 — below threshold,
investigating" gives Hunters what they need to start an incident
report.

**→ Green Team:**
Any firewall change that could affect HTTP/HTTPS traffic must be
communicated to Green Team before implementing. A rule that breaks
scoring engine access to the webserver costs both of you uptime points.

**→ Hunters (enumeration findings):**
When Hunters run nmap against your Traditional VMs during the contract
period and find open ports or vulnerable services, they will flag them
to you. Address these findings in Week 1 — do not let them sit.

---

## What a Bad Day Looks Like

The red team creates a new domain admin account on AD in the first
hour and nobody notices until the debrief because audit logging was
never enabled during the contract period. The Task box mail services
go down after a Week 3 firewall rule change and stay down for 45
minutes because nobody tested SMTP after the change. The PLC shows
anomalous Modbus commands at 2 PM but there are no timestamps or
source IPs in the incident report because the monitoring workflow was
never established.

---

## What a Good Day Looks Like

By competition day AD is hardened, audit logging is running, and
privileged group memberships are baselined from Week 1. Every Task
box firewall change was verified immediately by testing SMTP and IMAP.
The PLC monitoring dashboard is checked every 20 minutes and the HMI
Alarms page shows no unexpected Critical alerts. When the red team
attempts lateral movement from HMI at 1 PM it is caught within 10
minutes, documented with source IP and timestamp, and handed to
Hunters for an incident report that scores well.

---

## See Also

- [cyberforce101.md](../../docs/cyberforce101.md)
- [shared-foundations.md](../../docs/shared-foundations.md)
- [green-team.md](green-team.md)
- [vuln-hunters.md](vuln-hunters.md)
- [competition-prep/cyberforce/week-by-week.md](../week-by-week.md)

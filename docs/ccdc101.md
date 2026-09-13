# CCDC 101 — What Is It?

**Last updated:** September 2026  
**Owner:** AJ  
**For:** Everyone on the CyberForce roster considering CCDC

---

> **2026 Note:** NCCDC is transitioning from UTSA/CIAS to the
> [National Cyber Readiness Foundation (NCRF)](https://ccdc.io) starting
> with the 2027 season. The competition structure is expected to stay the
> same. Monitor [ccdc.io](https://ccdc.io) and
> [caeepnc.org/mwccdc](https://www.caeepnc.org/mwccdc/) for official 2027
> updates. Registration opens October 1, 2026.

---

## Quick Answer

**CCDC is a live collegiate cyber defense competition where your team defends
a business network against a professional red team — in real time, for two
full days.**

Unlike CyberForce, there is no prep period where you quietly harden systems
before competition day. The red team is active from the moment the drop flag
fires. You defend, respond to business tasks, and write incident reports all
at the same time.

---

## Who Is Eligible

**CCDC at UIC is not separately recruited.** The CCDC roster is drawn
exclusively from people competing in CyberForce. If you are on the
CyberForce roster and want to compete in CCDC, you are the target audience
for this document.

Team size is 4–8 members (minimum 4 to compete). UIC targets 6.

---

## How It Works

### The Scenario

Each year CCDC creates a fictional company scenario. Your team plays the role
of IT professionals newly hired to manage that company's network. You inherit
systems that are not properly secured, and your job is to lock them down,
keep services running for the business, and respond to management requests —
all while a professional red team is actively trying to break in and kick
you out.

### The Environment (Based on 2026 MWCCDC)

The 2026 competition used 11 virtual machines across two network segments.
The 2027 topology has not been published yet — treat this as the training
baseline.

| VM | OS | Role |
|---|---|---|
| Ubuntu Ecom | Ubuntu Server 24.04 | E-commerce server (HTTP/HTTPS) |
| Fedora Webmail | Fedora 42 | Webmail (SMTP, POP3) |
| Splunk | Oracle Linux 9.2 / Splunk 10.0.2 | Log monitoring |
| Ubuntu Workstation | Ubuntu Desktop 24.04 | User workstation |
| Windows Server AD/DNS | Server 2019 | Active Directory + DNS |
| Windows Server Web | Server 2019 | Web server |
| Windows Server FTP | Server 2022 | FTP server |
| Windows Workstation | Windows 11 | User workstation |
| Firewall 1 | Palo Alto PA 11.0.2 | Perimeter firewall |
| Firewall 2 | Palo Alto PA 11.0.2 | Internal firewall |
| VyOS Router | VyOS 1.4.3 | Network router |

### Scored Services

The scoring engine checks these services continuously. SLA penalties accrue
when services are down too long:

- **HTTP / HTTPS** — web page content must match expected output exactly
- **SMTP** — email send/receive through valid accounts
- **DNS** — lookups must resolve correctly
- **POP3** — mail retrieval via Active Directory accounts
- **FTP** — file access and presence checks
- **TFTP** — file pull with integrity check
- **NTP** — time server accuracy

Service uptime is 35–50% of your total score.

### Injects (Business Tasks)

Throughout competition, the White Team drops "injects" — business tasks your
team must complete within a time window. Every inject response is submitted
as a **business memo in PDF format** via the NISE platform.

Injects are worth 35–50% of your total score.

### Incident Response

When the red team successfully compromises something, you detect it, document
it, and submit an incident report. Worth 10–30% of your score. Good incident
reports on successful compromises can outscore a team that had no compromises
but documented nothing.

---

## The 2027 Season Structure

```
Oct 24          — Invitational 1 ✅ ($100/team)
Nov 7           — Invitational 2 ✅ ($100/team)
[Nov 14 skipped — conflicts with CyberForce Competition]
        ↓
Feb 13, 2027    — IL/MO State Qualifier ✅ ($500/team)
        ↓
Feb 20, 2027    — Midwest Wildcard ✅ (no added fee)
        ↓
Mar 19–20, 2027 — MWCCDC Regionals ✅ (no added fee)
        ↓
Apr 2027        — National CCDC ✅ (exact dates TBD)
```

**Wildcard eligibility:** 2nd and 3rd place from each state qualifier advance
to the Wildcard on Feb 20. Top Wildcard finishers advance to Regionals. There
are more paths to Regionals than just winning the state qualifier.

**Over 40 Midwest teams** are anticipated for the 2027 season — a record
number. Qualifier competition will be stronger than prior years.

### Invitationals — What They Are

The October and November Invitationals are optional dry runs. They mirror the
real competition format — same NISE platform, same inject structure, live red
team — but results do not affect qualification.

**UIC is targeting Invitationals 1 and 2 only.** Invitational 3 (Nov 14)
conflicts with the CyberForce Competition.

**Cost:** $100 per team per Invitational. National CCDC registration is NOT
required to compete in Invitationals.

### Registration

National registration opens October 1, 2026 and historically stays open
into mid-to-late January. The real deadline is before the first Invitational
(Oct 24).

**Qualifier fee:** $500 per team. No additional charge if the team advances
to Wildcard, Regionals, or Nationals.

---

## How CCDC Differs from CyberForce

| | CyberForce | CCDC |
|---|---|---|
| Red team timing | Competition day only | Active from drop flag |
| Prep access | ~3 weeks of SSH access | None — inherit machines live |
| Business tasks | Injects during competition | Injects throughout, memo format |
| OT/ICS systems | Yes (HMI, PLC) | Generally no |
| AD/DNS | Sometimes | Always — central to everything |
| Palo Alto firewalls | No | Yes — two of them |
| Scoring weight | Anomalies heaviest | Services + injects roughly equal |
| Duration | One full day | Two full days |

The biggest gaps for CyberForce veterans: Active Directory and Palo Alto
firewalls. Both appear in every CCDC environment, neither appears in
CyberForce. Start there.

---

## Scoring Breakdown

| Category | Weight |
|---|---|
| Functional service uptime | 35–50% |
| Inject completion | 35–50% |
| Incident response | 10–30% |

There is no single thing to optimize. A team that ignores injects to focus
on hardening will lose. A team that writes perfect memos but lets services
go down will also lose.

---

## What the Competition Days Look Like

**Day 1 (Friday)**
- Check-in, receive credentials
- Log into NISE, respond to Welcome inject
- Drop flag: 2:30 PM — scoring runs until 8:30 PM
- Red team active the entire time

**Day 2 (Saturday)**
- Drop flag: 9:00 AM — scoring runs until 6:00 PM
- Injects throughout the day
- Team presentations: 2:00–4:00 PM
- Awards: 7:00 PM

Six hours on Day 1, nine hours on Day 2.

---

## Common Questions

**Which Invitationals are we doing?**
Oct 24 and Nov 7. Nov 14 is skipped — it conflicts with CyberForce.

**Do we need national registration before Invitationals?**
No. National registration (ccdc.io) is not required to compete in
Invitationals. Register before the Qualifier deadline instead.

**What if we finish 2nd or 3rd at the Qualifier?**
You advance to the Wildcard on Feb 20. Losing the Qualifier does not
end the season.

**What does the $500 fee cover?**
Entry into the IL/MO State Qualifier only. No added charge for Wildcard,
Regionals, or Nationals if you advance.

---

## Next Steps

**Want to compete?**
1. Confirm you are on the CyberForce roster
2. Attend CCDC 101 (Sept 24) and CCDC Deep Dive (Sept 29)
3. Sign up on the roster sheet at the Sept 29 session

**Want to prepare now?**
- Study Active Directory basics
- Palo Alto PAN-OS free labs: paloaltonetworks.com/services/education
- Read [inject-response-template.md](../drills/inject-response-template.md)
- Review [2026-mwccdc-topology.md](../topology/2026-mwccdc-topology.md)

**Questions?**
Ask in #ccdc on Discord or DM @AJ.

---

## Useful Links

| Resource | URL |
|---|---|
| MWCCDC main site | https://www.caeepnc.org/mwccdc/ |
| National CCDC (NCRF) | https://ccdc.io |
| NISE platform | ccdcadmin1.morainevalley.edu |
| NETLAB access | ccdc.cit.morainevalley.edu |

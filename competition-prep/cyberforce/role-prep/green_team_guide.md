# Green Team Role — Technical Prep Guide

**Last updated:** September 2026  
**Owner:** AJ  
**For:** Green Team competitors and team captains

---

## Role Overview

Green Team (1 person per team) owns the webserver. It is
the most cross-functional role on the team: part sysadmin, part
application engineer.

Green Team's technical job is keeping the public-facing web application
running and accessible to Green Team scorers throughout competition day.

---

## What You Own

**VM:** Webserver — OpenSUSE Leap running nginx/HTTP  
**Application stack:** Laravel/PHP (2025 stack — confirm when 2026
rulebook releases)  
**Scoring bucket:** Green Team (15%) — website usability as experienced
by end user volunteers

---

## Technical Prep — Priority Order

Learn these in order. Each layer builds on the one below it. Start with
Laravel/PHP regardless of your existing background — it is where
application-level problems surface during competition and where Green
Team spends most of their time.

### 1. Laravel + PHP
The actual application stack running on the webserver. This is your
primary technical domain.

What to know:
- PHP syntax basics — variables, functions, arrays, conditionals
- Laravel project structure — where routes, controllers, views, and
  config files live
- Running and restarting a Laravel application
- Reading Laravel error logs (`storage/logs/laravel.log`)
- Common Laravel errors and what they mean — 500 errors, missing
  `.env` config, database connection failures
- Artisan CLI basics — `php artisan serve`, `php artisan cache:clear`,
  `php artisan config:clear`
- How Laravel connects to the database (`.env` DB credentials)

Where to learn:
- [Laravel official docs](https://laravel.com/docs) — read the
  Getting Started and Directory Structure sections
- [Laracasts](https://laracasts.com) — free Laravel beginner series
- PHP.net — reference for PHP syntax

**Note:** The exact application stack may change year to year. The 2025
stack was Laravel/PHP. Confirm when the 2026 technical rulebook releases
and adjust prep accordingly.

### 2. nginx
The web server software serving the Laravel application over HTTP/HTTPS.
If nginx goes down or misconfigures, the application becomes
unreachable — Green Team scorers cannot access the site and you lose
uptime points in both the Green (15%) and Blue (20%) scoring buckets.

What to know:
- Starting, stopping, and restarting nginx (`systemctl restart nginx`)
- Reading nginx error logs (`/var/log/nginx/error.log`)
- Basic nginx config structure — server blocks, root directory,
  index files
- How nginx connects to PHP-FPM to serve Laravel
- Checking nginx status (`systemctl status nginx`)
- Understanding what a 502 Bad Gateway error means and how to fix it

Where to learn:
- [nginx beginner's guide](https://nginx.org/en/docs/beginners_guide.html)
- DigitalOcean nginx tutorials — practical and well-written

### 3. Linux Basics (OpenSUSE Leap)
The webserver runs on OpenSUSE Leap — an RPM-based Linux distribution.
Most Linux fundamentals transfer from Ubuntu, but the package manager
and some tooling differ.

What to know:
- OpenSUSE-specific: `zypper` package manager (equivalent of `apt`)
- File system navigation — finding config files, log files,
  application directories
- File permissions — reading and fixing permission errors that break
  web applications
- Process management — `ps`, `top`, `systemctl`
- SSH access — connecting to and working within the VM
- Firewall basics — `firewalld` on OpenSUSE (not ufw)

Where to learn:
- OpenSUSE documentation at doc.opensuse.org
- Any general Linux command line tutorial transfers — the commands
  are the same, only the package manager differs

### 4. MySQL / MariaDB Basics
The webserver's Laravel application connects to the database VM (DB)
to read and write application data. Green Team does not own the DB VM
— that belongs to Monitoring and Hardening — but you need to understand
the connection so you can diagnose application errors caused by database
connectivity issues.

What to know:
- How Laravel's `.env` file specifies database connection settings
  (`DB_HOST`, `DB_PORT`, `DB_DATABASE`, `DB_USERNAME`, `DB_PASSWORD`)
- What a database connection failure looks like in Laravel error logs
- Basic MySQL commands for verifying connectivity:
  `mysql -u [user] -p -h [host]`
- How to tell whether an application error is a code problem vs. a
  database connectivity problem

Where to learn:
- Laravel docs on database configuration
- MySQL official getting started guide

### 5. HTML / CSS / JavaScript
Useful context for understanding the front end of the web application
but not a priority for competition prep. Green Team scorers interact
with the application as end users — they click buttons and submit forms.
If something is broken at this layer it is usually a symptom of a
Laravel or nginx problem, not an HTML/CSS problem.

What to know:
- Enough to read page source and understand what the application is
  serving
- Basic browser developer tools — inspecting network requests,
  reading console errors
- Recognizing whether an error is front-end (JavaScript console error)
  vs. back-end (Laravel 500 error in logs)

### 6. Basic Networking
Understanding how traffic flows to your webserver and why a service
might appear down to the scoring engine even when nginx is running.

What to know:
- HTTP vs. HTTPS — ports 80 and 443, what each serves
- How the scoring engine checks your webserver (it makes HTTP requests
  and compares returned content to an expected file)
- What it means when a service is "up" but not "scoring" — the content
  returned does not match what the engine expects
- Basic `curl` usage for testing whether your webserver is serving
  expected content: `curl http://[your-ip]/[expected-path]`
- How firewall rules can accidentally block scoring engine traffic

## What a Bad Day Looks Like

The webserver goes down mid-competition and stays down for 30 minutes
while you diagnose whether it is an nginx issue, a Laravel issue, or
a database connectivity issue. That is lost Green Team uptime points
and lost Blue Team uptime points simultaneously. 

---

## What a Good Day Looks Like

nginx is running, Laravel is serving the expected pages, Green Team
scorers can interact with the application throughout competition day.

---

## See Also

- [cyberforce101.md](../../docs/cyberforce101.md)
- [shared-foundations.md](../../docs/shared-foundations.md)
- [competition-prep/cyberforce/week-by-week.md](../week-by-week.md)

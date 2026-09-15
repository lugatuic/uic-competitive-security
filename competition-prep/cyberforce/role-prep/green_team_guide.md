# Green Team Role — Technical Prep Guide

**Last updated:** September 2026  
**Owner:** AJ  
**For:** Green Team competitors and team captains

---

## Role Overview

Green Team (1 person per team) owns the webserver. It is
the most cross-functional role on the team: part sysadmin, part
application person.

Green Team's job is keeping the public-facing web application
running and accessible to Green Team scorers throughout competition day.

---

## What You Own

**VM:** Webserver — OpenSUSE Leap running nginx/HTTP  
**Application stack:** Laravel/PHP (2025 stack — confirm when 2026
rulebook releases)  
**Scoring bucket:** Green Team (15%) — website usability as experienced
by end user volunteers

---

## How CyberForce Access Works

Unlike CCDC where you inherit live machines the moment the drop flag
fires, CyberForce gives your team a **~3 week contract period** (Oct 26
– Nov 14) of SSH access to all Traditional VMs before and during
competition day.

This means:
- You have time to get familiar with the actual application stack,
  test it, fix problems, and document findings
- Competition day (Nov 14) is about active defense, keeping things
  running, and responding to the red team — not starting from scratch

Use the contract period well.

---

## Contract Period Timeline — Green Team

**Week 1 (Oct 26 – Nov 1): Get familiar and harden**
- SSH into the webserver immediately after contract period opens
- Change default credentials — report scored service password changes
  to White Team via Support Ticket
- Explore the Laravel application — understand what it does, what
  routes exist, what database tables it connects to
- Verify the application is serving expected content
- Harden nginx and the OS (see technical prep below)
- Identify any vulnerabilities in the application or web server config

**Week 2 (Nov 2 – Nov 8): Document and record**
- C-Suite video is due approximately one week after scenario drop
  (est. early November — confirm from official rulebook)
- Security Documentation is produced based on actual findings from
  Week 1
- Continue monitoring the webserver — verify scored services remain
  up after any changes from other roles (M&H firewall rules can
  affect web traffic)

**Week 3 (Nov 9 – Nov 14): Final prep and competition**
- Final checks — verify application is stable and serving expected
  content
- Coordinate with M&H on any last hardening changes that could affect
  web traffic
- Competition Day (Nov 14): active defense — keep the application
  running, respond to any outages immediately

---

## Technical Prep — Priority Order

Learn these in order. Each layer builds on the one below it. Start
with Laravel/PHP regardless of your existing background — it is where
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
- Artisan CLI basics:
  ```bash
  php artisan serve
  php artisan cache:clear
  php artisan config:clear
  php artisan route:list
  ```
- How Laravel connects to the database (`.env` DB credentials)
- Checking and fixing file permissions on Laravel directories:
  ```bash
  sudo chown -R www-data:www-data /var/www/[app]
  sudo chmod -R 755 /var/www/[app]/storage
  ```

**During the contract period:** SSH into the webserver and actually
run the Laravel application. Read the routes. Understand what pages
exist and what the scoring engine will be checking. Do not wait until
competition day to see the application for the first time.

**Note:** The exact application stack may change year to year. The
2025 stack was Laravel/PHP. Confirm when the 2026 technical rulebook
releases and adjust prep accordingly.

Where to learn:
- [Laravel official docs](https://laravel.com/docs)
- [Laracasts](https://laracasts.com) — free Laravel beginner series
- PHP.net — reference for PHP syntax

---

### 2. nginx

The web server software serving the Laravel application over HTTP/HTTPS.
If nginx goes down or misconfigures, the application becomes
unreachable — Green Team scorers cannot access the site and you lose
uptime points in both the Green (15%) and Blue (20%) scoring buckets.

What to know:
- Starting, stopping, and restarting nginx:
  ```bash
  sudo systemctl restart nginx
  sudo systemctl status nginx
  ```
- Reading nginx error logs: `/var/log/nginx/error.log`
- Basic nginx config structure — server blocks, root directory,
  index files
- How nginx connects to PHP-FPM to serve Laravel
- Understanding what a 502 Bad Gateway error means and how to fix it
- Testing nginx config before reloading:
  ```bash
  sudo nginx -t
  sudo systemctl reload nginx
  ```

**During the contract period:** make any nginx configuration changes
early and verify the application still serves correctly after each
change. Never make nginx changes on competition day without testing
first.

Where to learn:
- [nginx beginner's guide](https://nginx.org/en/docs/beginners_guide.html)
- DigitalOcean nginx tutorials

---

### 3. Linux Basics (OpenSUSE Leap)

The webserver runs on OpenSUSE Leap — an RPM-based Linux distribution.
Most Linux fundamentals transfer from Ubuntu but the package manager
and some tooling differ.

What to know:
- OpenSUSE-specific: `zypper` package manager (equivalent of `apt`)
  ```bash
  sudo zypper update
  sudo zypper install [package]
  sudo zypper remove [package]
  ```
- File system navigation — finding config files, log files,
  application directories
- File permissions — reading and fixing permission errors that break
  web applications
- Process management — `ps`, `top`, `systemctl`
- SSH access — connecting to and working within the VM
- Firewall basics — `firewalld` on OpenSUSE (not ufw):
  ```bash
  sudo firewall-cmd --list-all
  sudo firewall-cmd --add-service=http --permanent
  sudo firewall-cmd --reload
  ```

**During the contract period:** harden the OS during Week 1. Audit
users, disable unnecessary services, configure the firewall. Verify
the webserver application still works after each change.

---

### 4. MySQL / MariaDB Basics

The webserver's Laravel application connects to the DB VM (Windows
Server 2022, MariaDB) to read and write application data. Green Team
does not own the DB VM — that belongs to M&H — but you need to
understand the connection to diagnose application errors caused by
database connectivity issues.

What to know:
- How Laravel's `.env` file specifies database connection settings:
  ```
  DB_HOST=
  DB_PORT=3306
  DB_DATABASE=
  DB_USERNAME=
  DB_PASSWORD=
  ```
- What a database connection failure looks like in Laravel error logs
- Basic connectivity test:
  ```bash
  mysql -u [user] -p -h [DB-VM-IP]
  ```
- How to tell whether an error is a code problem vs. a database
  connectivity problem

**During the contract period:** verify the Laravel application
connects to the DB VM successfully early in Week 1. If M&H changes
the MariaDB credentials, they must coordinate with you immediately
so the `.env` file can be updated.

---

### 5. HTML / CSS / JavaScript

Useful context for understanding the front end of the web application
but not a priority for competition prep. If something is broken at
this layer it is usually a symptom of a Laravel or nginx problem.

What to know:
- Enough to read page source and understand what the application is
  serving
- Basic browser developer tools — inspecting network requests,
  reading console errors
- Recognizing whether an error is front-end (JavaScript console error)
  vs. back-end (Laravel 500 error in logs)

---

### 6. Basic Networking

Understanding how traffic flows to your webserver and why a service
might appear down to the scoring engine even when nginx is running.

What to know:
- HTTP vs. HTTPS — ports 80 and 443
- How the scoring engine checks your webserver — it makes HTTP
  requests and compares returned content to an expected file exactly
- What it means when a service is "up" but not "scoring" — the
  content returned does not match what the engine expects
- Basic curl usage for testing:
  ```bash
  curl http://[your-ip]/[expected-path]
  ```
- How firewall rules can accidentally block scoring engine traffic

**During the contract period:** use curl to test what the scoring
engine will see before competition day. If the returned content does
not match what is expected, find out why now — not during scored hours.

---

## What a Bad Day Looks Like

The webserver goes down mid-competition and it takes 30 minutes to
diagnose whether it is an nginx issue, a Laravel issue, or a database
connectivity issue — because Green Team never got fully familiar with
the application stack during the contract period.

---

## What a Good Day Looks Like

By competition day the webserver is hardened, the Laravel application
is serving expected content, and Green Team knows exactly how to
restart nginx, clear Laravel cache, and check error logs from memory. 
On competition day, Green Team's job is
mostly monitoring — keeping the application running while the red
team tests everything else.

---

## See Also

- [cyberforce101.md](../../docs/cyberforce101.md)
- [shared-foundations.md](../../docs/shared-foundations.md)
- [Vulnerability Hunter Guide](vuln_hunter_guide.md)
- [Monitoring & Hardening Guide](monitoring+hardening_guide.md)
- [CyberForce Technical Prep](../cyberforce-technical-prep.md)

# Bug Bounty Master Checklist

> Day 1: Sept 29, 2026. Started with NahamSec's no-BS roadmap. First mission: Linux.

## How to read this file

Every topic has a target level. The level says how deep to go, so nothing is vague.

- **L1 Know it:** I can explain what it is in one or two sentences
- **L2 Use it:** I can do it by following a guide or looking things up
- **L3 Own it:** I can do it alone, without a guide or a cheat sheet

Rules:
1. Never skip ticking. The tick IS my motivation system
2. Practice first, notes second. A day note is short: the level, the command that worked, where I got stuck
3. Month ranges are targets, not deadlines. Moving on when a level is reached matters more than the date

---

## PHASE 1: Foundations (Months 1 to 2)

### 1A. Linux (target: L3 for the core commands, L2 for the extras)
- [x] Install Ubuntu (WSL) on Windows (L3)
- [x] Navigation: `pwd`, `cd`, `ls`, `mkdir`, `touch` (L3)
- [x] Reading files: `cat`, `less`, `head`, `tail` (L3)
- [x] File inspection: `file`, flags of `ls` (`-l`, `-a`, `-h`) (L3)
- [x] Permissions: `chmod`, `chown`, reading `ls -l` output (L3)
- [x] Finding things: `find`, `grep`, `which` (L3)
- [x] Getting help: `man`, `--help` (L3)
- [x] Text editing with `nano` (L3)
- [ ] Text editing with `vim`, basics only: open, edit, save, quit (L1, only needed to survive on servers)
- [x] Downloading: `wget`, `curl` (L2)
- [x] Process control: `ps`, `top`, `kill`, `Ctrl+C` (L2)
- [x] Text Fu: pipes, redirection, `sort`, `uniq`, `wc`, `nl`, `cut`, `paste`, `tr`, `join`, `split`, `grep` (L3)
- [x] `sudo`, `whoami`, `id`, `su` (L2)
- [x] `apt`: installing tools (L2)
- [x] Environment variables: `$PATH`, `env`, `export` (L2)
- [x] Archives: `tar`, `gzip` (L2)

### 1B. Bandit (this is where Linux becomes L3)
Rule: try each level for 10 to 15 minutes without the cheat sheet, then look things up.
- [x] Levels 0 to 2: readme, dash file, spaces file (L3)
- [x] Level 3: hidden file with `ls -a` (L3)
- [x] Level 4: `file` command to find the data file (L3)
- [ ] Level 5: `find` with the size flag (L3)
- [ ] Levels 6 to 10: `find` with permissions, `grep`, `sort` and `uniq`, `base64`, `tr` and rot13, `gzip` (L3)
- [ ] Levels 11 to 15: cron, git, passwords (L2)
- [ ] Levels 16 to 20: `nc`, `openssl`, port knocking (L2)
- [ ] Levels 21 to 34: finish the game (optional but powerful) (L2)

### 1C. Networking basics (start after Bandit 10, run alongside Bandit 11 to 20)
- [ ] IP address, ports, DNS: what they are and how a name becomes an address (L2)
- [ ] TCP vs UDP at a basic level (L1)
- [ ] HTTP and HTTPS: requests, responses, methods (GET and POST), status codes, headers, cookies (L3, the most important item in Phase 1)
- [ ] Browser DevTools: the Network tab, reading a request and a response (L3)
- [ ] `curl -v`: see a full request and response in the terminal (L3)
- [ ] `ping` and `dig` (or `nslookup`): check if a host is up and look up a DNS name (L2)
- [ ] SSH: keys, ports, config basics (L2)

**Resources:** NetworkChuck networking videos, Linux Journey "Networking Nomad" section

### 1D. Programming (only enough to read and write small tools)
- [ ] Python: add the `requests` library to what I already know. Send a GET and a POST, read headers and cookies (L2)
- [ ] Bash scripting: variables, `if`, `for` loops, running a command on every line of a file (L2)
- [ ] Git: clone, add, commit, push, log, diff (L2)
- [ ] JavaScript: read a small script, understand the DOM, events and `fetch` (L2, needed for XSS in Phase 2)

**Resource:** freeCodeCamp Python and JS (do not over-invest here)

---

## PHASE 2: Web and Core Security (Months 2 to 4)

### 2A. How the web works (deep understanding)
- [ ] HTML, CSS and JS basics: how a browser turns a page into what I see (L2)
- [ ] Forms, logins, sessions and cookies: what happens from the login click to the logged-in page (L3)
- [ ] Same-Origin Policy and CORS (L2, concept level)
- [ ] Authentication flows: password reset, JWT, OAuth (L2)

**Resource:** PortSwigger topics (free)

### 2B. Burp Suite
- [ ] Install and proxy setup (Firefox and FoxyProxy) (L3)
- [ ] Intercepting requests (L3)
- [ ] Repeater: modify and resend (L3)
- [ ] Scope and target mapping (L3)
- [ ] Intruder: fuzzing basics (L2)
- [ ] Decoder and Comparer (L2)

### 2C. Core vulnerabilities (OWASP Top 10, 2025 edition)
For each one: **read, lab, writeup**. Check the exact names on owasp.org before starting.
The three marked L3 are the most useful ones to know deeply as a beginner. SQL is already a strength, so Injection should go quickly.
- [ ] A01 Broken Access Control: IDOR, privilege escalation (L3)
- [ ] A05 Injection: SQLi, XSS basics, command injection (L3)
- [ ] A07 Authentication Failures (L3)
- [ ] A02 Security Misconfiguration (L2)
- [ ] A04 Cryptographic Failures (L2)
- [ ] A06 Insecure Design (L2)
- [ ] SSRF: no longer its own OWASP entry, but still a PortSwigger topic (L2)
- [ ] A03 Software Supply Chain Failures (L1)
- [ ] A08 Software or Data Integrity Failures (L1)
- [ ] A09 Logging and Alerting Failures (L1)
- [ ] A10 Mishandling of Exceptional Conditions (L1)

**Resources:** PortSwigger Web Security Academy (free, the core of everything), Rana Khalil's YouTube series

### 2D. Supporting practice
- [ ] TryHackMe free rooms: Web Fundamentals path (L2)
- [ ] OWASP Juice Shop and WebGoat, the vulnerable practice apps (L2)
- [ ] Read 50 or more real disclosed reports on HackerOne Hacktivity (L2)

---

## PHASE 3: Methodology and Recon (Months 4 to 6)

- [ ] How a pentest engagement flows: scope, recon, testing, report (L2)
- [ ] Reading scope and program rules, and what is legally safe to test (L3)
- [ ] Recon tools: `subfinder`, `amass`, `httpx`, `nuclei` (L2)
- [ ] Content discovery: `ffuf`, `dirsearch`, `gobuster` (L2)
- [ ] API basics: REST, JSON, auth headers, testing with Burp or Postman (L2)
- [ ] Note-taking template with evidence (L3)
- [ ] Bugcrowd University (free), the bounty mindset (L2)
- [ ] Write 10 practice reports from labs (L3)

---

## PHASE 4: First Live Hunting (Months 6 to 9)
Goals, not deadlines. Real bug bounty results take time and are never guaranteed.

- [ ] Set up my HackerOne account and profile
- [ ] Read program policies and scope carefully, legal safety first (L3)
- [ ] Start with VDPs (non-paying programs), lower competition (L2)
- [ ] Report writing: title, severity, steps, impact, PoC (L3)
- [ ] Land my first valid report, even informational or low
- [ ] Get my first duplicate
- [ ] Earn my first bounty payout
- [ ] Build an Upwork profile with sample reports and land my first freelance audit client

---

## PHASE 5: Leveling Up (Months 9 to 14+)

- [ ] Specialize: pick 2 to 3 bug classes to go deep, for example IDOR, auth bugs, business logic (L3 in each)
- [ ] Master business logic vulnerabilities, my data analysis brain helps here (L3)
- [ ] Write my own Python scripts for recon and exploits (L3)
- [ ] API hacking, increasingly where the money is (L3)
- [ ] Mobile basics, optional branch (L1)
- [ ] Compete in CTFs: HackerOne CTF, NahamCon CTF (L2)
- [ ] Earn consistent monthly bounty income
- [ ] Grow my public presence: blog writeups, Twitter/X, GitHub tools

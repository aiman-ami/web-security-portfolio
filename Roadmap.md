# Bug Bounty Master Checklist 

&gt; Day 1: Sept 29, 2026. Started with NahamSec's no-BS roadmap. First mission: Linux.

## PHASE 1: Foundations (Months 1–2)

### Linux 
- [x] Install Ubuntu (WSL) on Windows
- [ ] Learn basic navigation: `pwd`, `cd`, `ls`, `mkdir`, `touch`
- [ ] Learn to read files: `cat`, `less`, `head`, `tail`
- [ ] Learn file inspection: `file`, flags of `ls` (`-l`, `-a`, `-h`)
- [ ] Master permissions: `chmod`, `chown`, reading `ls -l` output
- [ ] Learn finding things: `find`, `grep`, `which`
- [ ] Learn to get help: `man`, `--help`
- [ ] Learn text editing: `nano` (and later `vim` basics)
- [ ] Learn downloading: `wget`, `curl`
- [ ] Learn process control: `ps`, `top`, `kill`, `Ctrl+C`

### Networking basics
- [ ] Understand what an IP address, ports, and DNS are
- [ ] Understand TCP vs UDP (basic level)
- [ ] Master HTTP/HTTPS: requests, responses, methods (GET/POST), status codes, headers, cookies
- [ ] Deepen my SSH understanding (I've already used it! ✅)

**Resources:** NetworkChuck networking videos / Linux Journey "Networking Nomad" section

### Programming (basics only enough to read code)
- [ ] Python: variables, loops, functions, `requests` library
- [ ] JavaScript: enough to read web page code
- [ ] Bash scripting basics

**Resource:** freeCodeCamp Python + JS (I won't over-invest here — I'll return later)

### Bandit  
- [ ] Levels 0–2 (readme, dash file, spaces file)
- [ ] Level 3 (hidden file → `ls -a`)
- [ ] Level 4 (`file` command to find the data file)
- [ ] Level 5 (find + size flag)
- [ ] Levels 6–10 (find with permissions, grep, base64, tr/rot13, gzip)
- [ ] Levels 11–15 (cron, git, passwords)
- [ ] Levels 16–20 (nc, openssl, port knocking)
- [ ] Levels 21–34 (finish the game — optional but powerful)

---

## PHASE 2: Web & Core Security (Months 2–4)

### The Web ( deep understanding )
- [ ] HTML/CSS/JS basics → how browsers render pages
- [ ] How forms, logins, sessions, and cookies work
- [ ] Same-Origin Policy, CORS (concept level)

**Resource:** PortSwigger topics (free)

### Burp Suite 
- [ ] Install + proxy setup (Firefox + FoxyProxy)
- [ ] Intercepting requests
- [ ] Repeater (modify & resend)
- [ ] Intruder (fuzzing basics)
- [ ] Decoder, Comparer
- [ ] Scope + target mapping

### OWASP Top 10 
For each one: **read → lab → writeup**
- [ ] Broken Access Control (IDOR, privilege escalation)
- [ ] Cryptographic Failures
- [ ] Injection (SQLi, XSS basics, command injection)
- [ ] Insecure Design
- [ ] Security Misconfiguration
- [ ] Vulnerable Components
- [ ] Authentication Failures
- [ ] Software/Data Integrity Failures
- [ ] Logging/Monitoring Failures
- [ ] SSRF

**Resources:** PortSwigger Web Security Academy (free the core of everything) + Rana Khalil's YouTube series

### Supporting practice
- [ ] TryHackMe free rooms: Web Fundamentals path
- [ ] OWASP Juice Shop / WebGoat (vulnerable practice apps)
- [ ] Read 50+ real disclosed reports on HackerOne Hacktivity

---

## PHASE 3: Methodology & Recon (Months 4–6)

- [ ] Learn how a pentest engagement flows: scope → recon → testing → report
- [ ] Recon tools: subfinder, amass (subdomains), httpx, nuclei
- [ ] Content discovery: ffuf, dirsearch, gobuster
- [ ] Take notes like a pro: template + evidence
- [ ] Complete Bugcrowd University (free) — bounty-specific mindset
- [ ] Write 10 practice reports from labs (template reports)

---

## PHASE 4: First Live Hunting (Months 6–9)

- [ ] Set up my HackerOne account + profile
- [ ] Read program policies & scope carefully (legal safety!)
- [ ] Start with VDPs (non-paying programs) lower competition
- [ ] Learn report writing: title, severity, steps, impact, PoC
- [ ] Land my first valid report (even informational/low) 
- [ ] Get my first duplicate 
- [ ] Earn my first bounty payout 
- [ ] Build an Upwork profile with sample reports → land my first freelance audit client

---

## PHASE 5: Leveling Up (Months 9–14+)

- [ ] Specialize: pick 2–3 bug classes to go deep (e.g., IDOR, auth bugs, business logic)
- [ ] Master business logic vulnerabilities (my data-analysis brain helps here!)
- [ ] Write my own Python scripts for recon/exploits
- [ ] Learn API hacking (increasingly where the money is)
- [ ] Explore mobile basics (optional branch)
- [ ] Compete in CTFs: HackerOne CTF, NahamCon CTF
- [ ] Earn consistent monthly bounty income
- [ ] Grow my public presence: blog writeups, Twitter/X, GitHub tools

---

## 📋 How I use this

1. This file lives in my GitHub repo as `roadmap.md`
2. **Rule:** never skip ticking the tick IS my motivation system
3. **Pace:** ~2–4 ticks per day = on schedule for first bounties around month 8–10

---

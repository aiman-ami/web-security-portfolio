# Day 14: Bandit 19 and 20, a Python script with error handling

- Bandit levels 19 and 20
- Wrote `check.py` alone with Python `requests`, then added `try` and `except`
- Fixed Ubuntu (WSL) after it stopped starting

Roadmap: 1B Bandit levels 19 and 20 done. 1D Python `requests`: wrote a small script alone that sends a GET and prints the status code and content-type (L3 for that task).

## What I learned

### Bandit level 19: a setuid binary

Idea: the home folder has a program, `bandit20-do`, that runs any command I give it as the user bandit20. bandit20 can read its own password file, so I use the program to read it.

```bash
$ ls -l bandit20-do
-rwsr-x--- 1 bandit20 bandit19 ...
$ ./bandit20-do id
euid=11020(bandit20) ...
$ ./bandit20-do cat /etc/bandit_pass/bandit20
```

How it works: the `s` in the permissions (`rws`) is the setuid bit. It makes the program run as its owner, whoever starts it. The owner here is bandit20, so the command runs with bandit20's permissions. The group is bandit19, so I am allowed to run it. `id` shows `euid=` (the effective user), which confirmed the program acts as bandit20. The password file is named after the user it belongs to, so the file for this level is `bandit20`.

My notes: `cat` run by bandit19 on that file would be refused. Through `bandit20-do` it works because of the `s` instead of `x`.

Extra notes:

- A setuid program that lets the user run any command hands over the owner's access. That is why setuid programs matter in security.

### Bandit level 20: be the server

Idea: the program `suconnect` connects to a port on localhost, reads a line, compares it with the current password, and if it matches, sends back the next password. This time I am the server, so I have to listen on a port.

```bash
# Terminal 1
$ nc -lv 9999
# Terminal 2
$ ./suconnect 9999
```

How it works: in terminal 1, `nc -l` listens on a port I choose (`-v` prints a message when something connects). In terminal 2, `./suconnect 9999` connects to that port. When "connection received" shows in terminal 1, I type the current password there and press Enter. `suconnect` checks it and sends the next password back to the listener.

My notes: I needed two terminals, both logged in as bandit20.

Extra notes:

- In levels 14 and 15 I was the client connecting to a port. In this level the roles are flipped.
- `nc -lvp` is a common form, but some versions of `nc` do not allow `-l` together with `-p`. `nc -lv 4444` avoids that.
- Pick a port between 1025 and 65535 that is not in use.

### Python: check.py with error handling

First version, written alone:

```python
import requests
r = requests.get('https://github.com')
print("Status code:",r.status_code)
print("Content-Type:",r.headers.get('content-type'))
```

I changed the URL to a site that does not exist to see the crash. The last line of the traceback was:

```
requests.exceptions.ConnectionError: ... Failed to resolve ... Name or service not known
```

Final version:

```python
import requests
try:
    r = requests.get('https://github.com')
    print("Status code:",r.status_code)
    print("Content-Type:",r.headers.get('content-type'))
except requests.exceptions.ConnectionError:
    print("Got a DNS error, no such site, try again")
```

How it works: `try` runs the code, and if it fails with the named error, Python jumps to `except` and runs that instead of crashing. I tested both paths: a real site printed `Status code: 200` and `Content-Type: text/html; charset=utf-8`, and a site that does not exist printed my message.

My notes: a traceback is read from the bottom up. The last line names the error and the lines above show the path to it. If the print lines were outside the `try`, a failed `requests.get` left `r` undefined, which gave `NameError: name 'r' is not defined`. Moving the prints inside the `try` fixed it.

Extra notes:

- `r.headers.get('content-type')` returns `None` if the header is missing. `r.headers['content-type']` would raise an error.
- `Failed to resolve` means the DNS step failed (name in, no IP address out), so it never reached the connect step.
- This is not a finished script yet. The URL is fixed in the code, there is no `timeout`, and only the connection error is caught.

### Ubuntu (WSL) stopped starting

Ubuntu closed on its own, and the window showed `WslRegisterDistribution failed with error: 0x8007019e` and said the Windows Subsystem for Linux is not installed. The files were not lost, because they live in Ubuntu's own disk image.

Fix, in PowerShell opened as administrator:

```powershell
wsl --status
dism.exe /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux /all /norestart
dism.exe /online /enable-feature /featurename:VirtualMachinePlatform /all /norestart
shutdown /r /t 0
```

Extra notes:

- `wsl --status` still listed `Ubuntu` as the default distribution, which showed it was still registered.
- Do not run `wsl --unregister`, because it deletes the distribution and all its files.
- A likely cause is a Windows update that switched off optional features, but I could not confirm it.
- A harmless warning about the systemd user session appeared after the restart.

## Key takeaways

- A setuid program (`s` in the permissions) runs as its owner, so it can reach files the caller cannot.
- `id` shows the effective user (`euid`), which is who a program is really acting as.
- `nc -l` listens on a port, so I can be the server while another program is the client.
- Port-based levels need two terminals: one to listen and one to connect.
- Read a traceback from the last line upward.
- `try` and `except` stop a crash and print a clean message, and the code that depends on the result belongs inside the `try`.
- Test both paths of a script: the normal one and the failing one.

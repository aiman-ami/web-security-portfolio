# Day 13: Bandit 16 to 18 and Python requests

- Bandit levels 16, 17 and 18
- Python `requests`: GET, headers, body and POST in the Python shell

Roadmap: 1B Bandit levels 16 to 18 done. 1D started: Python `requests` used with a guide (L2).

## What I learned

### Bandit level 16: find the port that speaks TLS

Idea: the service is on one of the ports in the range 31000 to 32000, but only some of them are open and only some of those speak TLS. First find the open ports, then find which one uses TLS, then send the password to that one.

```bash
$ nmap -p 31000-32000 127.0.0.1
$ nmap -sV -p PORT1,PORT2,PORT3,PORT4,PORT5 localhost
$ openssl s_client -connect localhost:PORT -quiet
```

How it works: `nmap` knocks on each port in the range and lists the open ones. `-sV` detects which service runs on each open port, and the ones showing `ssl` speak TLS. `openssl s_client -connect` talks to that port, and I type the current password and press Enter. The level answers with a private key for the next user.

My notes: I first tried `-A`, which works but runs many extra checks. `-sV` is lighter and enough here.

Extra notes:

- Without `-quiet`, `openssl s_client` treats a line that starts with certain letters as one of its own commands. My password started with one of them, so it was read as a key update request (`KEYUPDATE`) and not sent as text. `-quiet` turns that off and also hides the long certificate text.
- The warnings `self-signed certificate` and `Can't use SSL_get_servername` are harmless here.
- I saved the key in a file on my own machine with `vim`, then ran `chmod 600` on it. The key never goes into the repo.

```bash
$ chmod 600 sshkey17.private
$ ssh -i sshkey17.private bandit17@bandit
```

### Bandit level 17: find the changed line

Idea: two files are almost the same, and exactly one line differs. The line that changed is the answer.

```bash
$ diff passwords.old passwords.new
```

How it works: `diff` compares two files and prints only the lines that differ. Lines marked `<` are from the first file and lines marked `>` are from the second.

### Bandit level 18: run a command through ssh

Idea: this level logs me out as soon as I log in, so a normal login closes at once. If I add a command after `ssh`, it runs on the server and prints the result before the logout happens.

```bash
$ ssh bandit18@bandit cat readme
```

How it works: `ssh user@host command` runs that one command on the server instead of opening a login session.

My notes: I found this in the `ssh` help and the level goal text, not from a guide.

### Python requests: curl in code

```bash
$ python3 --version
Python 3.14.4
$ python3 -c "import requests; print(requests.__version__)"
```

Then in the Python shell (`python3`, prompt `>>>`):

```python
>>> import requests
>>> r = requests.get("https://example.com")
>>> r.status_code
200
>>> r.headers["content-type"]
>>> r.text[:200]
>>> r = requests.post("https://httpbin.org/post", data={"name": "aiman"})
>>> r.json()["form"]
{'name': 'aiman'}
```

How it works: `requests.get` sends a GET request, like `curl https://example.com`. `r.status_code` is the `< HTTP` line, `r.headers` holds the response headers, and `r.text` is the body. `requests.post` with `data=` is like `curl -d`, and `r.json()` turns a JSON reply into a Python dictionary.

Extra notes:

- Same results as `curl -v` and DevTools: status 200, content-type `text/html`, and the POST body came back under `"form"`.
- This was done with a guide, so it is L2. For L3 I need to write a small `.py` file alone that sends a GET to a site and prints the status code and content-type.

## Key takeaways

- `nmap` finds open ports, and `-sV` shows which service runs on each.
- `openssl s_client -quiet` stops the tool from treating my text as commands.
- A private key copied to my machine needs `chmod 600` and stays out of the repo.
- `diff` shows only the lines that differ between two files.
- `ssh user@host command` runs one command on the server without a login session.
- Reading the help text and the level goal can solve a level without outside help.
- `requests` in Python gives the same status code, headers and body as `curl -v`.

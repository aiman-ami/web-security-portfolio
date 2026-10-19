# Networking Cheat Sheet

Concepts from roadmap 1C. Linux commands are in `linux-cheatsheet.md`.

## HTTP methods

| Method | What it does | Example |
|---|---|---|
| `GET` | Asks the server for a page or resource. Data goes in the URL | Opening example.com in the browser |
| `POST` | Sends data to the server in the request body | Submitting a login form |

## HTTP status codes

| Code | Meaning | Example |
|---|---|---|
| `200 OK` | The request worked and the content was sent | Normal page load |
| `301 Moved Permanently` | The page moved. The `location` header gives the new address | `http://github.com` to `https://github.com/` |
| `304 Not Modified` | The browser can use its saved copy | Normal reload in DevTools |
| `403 Forbidden` | The server understood but refuses access | A page you are not allowed to open |
| `404 Not Found` | The server works but has no page at that address | `github.com/this-page-does-not-exist-12345` |
| `500 Internal Server Error` | The server failed while handling the request | A broken page on the server side |

## Headers

| Header | What it does | Example |
|---|---|---|
| `content-type` | Says what kind of data is in the response | `text/html; charset=utf-8` |
| `user-agent` | Request header that tells the server which browser or tool is asking | Shown in DevTools |
| `host` | Request header with the site name. In HTTP/2 it appears as `:authority` | `:authority: example.com` |
| `location` | Response header with the new address for a redirect | `location: https://github.com/` |
| `set-cookie` | Response header that asks the browser to store a cookie | Seen on google.com in DevTools |
| `cookie` | Request header that sends stored cookies back to the server | Sent on later requests |

## Cookie attributes

| Attribute | What it does | Example |
|---|---|---|
| `Expires` | When the cookie is deleted | A date and time |
| `Domain` | Which site receives the cookie | `.google.com` |
| `Path` | Which paths on the site receive the cookie | `/` |
| `Secure` | Cookie is only sent over HTTPS | Protects it on plain HTTP |
| `HttpOnly` | JavaScript cannot read the cookie | Helps against cookie theft through XSS |
| `SameSite` | Limits sending the cookie on requests from other sites | `Lax` or `Strict` |

## TCP vs UDP

| Protocol | What it does | Example |
|---|---|---|
| `TCP` | Makes a connection first, confirms delivery, resends lost data | Web pages, logins |
| `UDP` | No connection, no confirmation, some data may be lost, but fast | Video calls, online games |

## DNS

| Term | What it does | Example |
|---|---|---|
| IP address | The real address of a machine | `104.20.23.154` |
| Domain name | The easy name for people | `example.com` |
| DNS | Turns a name into IP addresses | `dig example.com` |
| `A` record | Maps a name to an IPv4 address | `example.com. 109 IN A 104.20.23.154` |
| TTL | Seconds an answer may be reused before asking again | `109` |

## Ports and SSH config

| Item | What it does | Example |
|---|---|---|
| Port | A numbered door on a machine, with a program listening behind it | `30000` |
| Port 22 | Default SSH port | `ssh user@host` |
| Port 443 | Default HTTPS port | `https://example.com` |
| `~/.ssh/known_hosts` | SSH's memory of servers it has connected to | Filled when you answer `yes` |
| `~/.ssh/config` | Shortcuts for long SSH commands | See below |

### DNS and SSH notes

- DNS only returns an address. After that, the browser connects to the IP on a port and the server sends the page.
- One name can have several `A` records.
- Example config block, then `ssh bandit` replaces the long command:

```
Host bandit
    HostName bandit.labs.overthewire.org
    Port 2220
    User bandit15
```

- Without a `Port` line, SSH uses port 22.

### HTTP notes

- Status code families: 2xx success, 3xx redirect, 4xx client problem, 5xx server problem.
- GET puts data in the URL. POST puts it in the body. Neither is encrypted. HTTPS is what encrypts the traffic.
- A password in a GET URL would be saved in browser history, bookmarks and server logs, so logins use POST.
- Hard reload in the browser: Ctrl+Shift+R. It ignores the saved copy, so the status becomes 200 again.
- TCP analogy: a phone call. UDP analogy: postcards.
- HTTP/3 runs on QUIC, which uses UDP and adds its own reliability.

## Python requests

| Code | What it does | Example |
|---|---|---|
| `requests.get(url)` | Sends a GET request | `r = requests.get("https://example.com")` |
| `r.status_code` | The status code of the response | `200` |
| `r.headers["name"]` | One response header | `r.headers["content-type"]` |
| `r.text` | The response body as text | `r.text[:200]` |
| `requests.post(url, data=...)` | Sends a POST request with data in the body | `requests.post("https://httpbin.org/post", data={"name": "aiman"})` |
| `r.json()` | Turns a JSON response into a Python dictionary | `r.json()["form"]` |

### Python requests notes

- Same results as `curl -v`: status code, headers and body.
- `data={...}` is the same as `curl -d`.
- Check that the library is installed with `python3 -c "import requests; print(requests.__version__)"`.

## Python error handling

| Code | What it does | Example |
|---|---|---|
| `try:` | Runs the code under it | `try:` |
| `except ErrorName:` | Runs only if the named error happens, instead of crashing | `except requests.exceptions.ConnectionError:` |
| `r.headers.get("name")` | Reads a header and returns `None` if it is missing | `r.headers.get("content-type")` |

### Python error handling notes

- Read a traceback from the last line upward. The last line names the error.
- `ConnectionError` with `Failed to resolve` means the DNS step failed, so the name gave no IP address.
- Put code that uses `r` inside the `try`. If `requests.get` fails, `r` does not exist and the later lines raise `NameError`.
- Indent the lines under `try` and `except` with 4 spaces.
- Test both paths: a real site and a site that does not exist.

```python
import requests
try:
    r = requests.get("https://github.com")
    print("Status code:", r.status_code)
    print("Content-Type:", r.headers.get("content-type"))
except requests.exceptions.ConnectionError:
    print("Got a DNS error, no such site, try again")
```

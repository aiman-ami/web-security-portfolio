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

### HTTP notes

- Status code families: 2xx success, 3xx redirect, 4xx client problem, 5xx server problem.
- GET puts data in the URL. POST puts it in the body. Neither is encrypted. HTTPS is what encrypts the traffic.
- A password in a GET URL would be saved in browser history, bookmarks and server logs, so logins use POST.
- Hard reload in the browser: Ctrl+Shift+R. It ignores the saved copy, so the status becomes 200 again.
- TCP analogy: a phone call. UDP analogy: postcards.
- HTTP/3 runs on QUIC, which uses UDP and adds its own reliability.

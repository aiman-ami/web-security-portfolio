
# Day 11: Bandit 11, HTTP in DevTools and curl

- Bandit level 11 (tr with rot13 ranges)
- Watched a video for Bandit level 12 (not solved yet)
- Browser DevTools, Network tab, on example.com, google.com and linkedin.com
- `curl -v` on example.com and linkedin.com
- `curl -I` to see a redirect (301) and a not found (404)
- `curl -X POST` against httpbin.org
- Revised TCP vs UDP with examples

Roadmap: 1C Networking. Ticked Browser DevTools, `curl -v`, and HTTP and HTTPS (a borderline L3, to be tested again in Phase 2). 

## What I learned

### Bandit level 11: tr with rot13

Level 11 used `tr` with rot13 ranges. The text in the data file is rotated by 13 letters, so it has to be rotated back.

Extra notes:

- The command below is from general knowledge, not tested here.

```
$ cat data.txt | tr 'A-Za-z' 'N-ZA-Mn-za-m'
```

- How it works: `tr` replaces each character in the first set with the character at the same position in the second set. `N-ZA-M` starts at N, so A becomes N, B becomes O, and so on. The lowercase range does the same for lowercase letters.
- Rot13 moves each letter 13 places. The alphabet has 26 letters, so running it twice returns the original text.

### Browser DevTools: the Network tab

- Open with F12, click the Network tab, then reload the page.
- Request URL: the full address that was requested.
- Request Method: `GET` for a normal page load.
- Status code: `200 OK` means the full page was sent. `304 Not Modified` means the browser used its saved copy. A hard reload with Ctrl+Shift+R ignores the saved copy, so the status becomes 200 again.
- content-type: `text/html; charset=utf-8`, an HTML page encoded as UTF-8.
- user-agent: a request header that tells the server which browser is asking.
- `:authority`: in HTTP/2 the headers show `:authority` instead of `host`.
- Set-Cookie attributes: expires, domain, path, Secure, HttpOnly, SameSite.
- Same result on linkedin.com without a guide: method `GET`, status 200, content-type `text/html`.

Extra notes:

- Expires: when the cookie is deleted.
- Domain and path: which site and which paths receive the cookie.
- Secure: the cookie is only sent over HTTPS.
- HttpOnly: JavaScript cannot read the cookie.
- SameSite: limits sending the cookie on requests that come from other sites.

### curl -v: a full request and response in the terminal

```
$ curl -v https://example.com
> GET / HTTP/2
< HTTP/2 200
< content-type: text/html; charset=utf-8
```

(Key lines only. The full output is much longer.)

- Lines starting with `>` are what my terminal sent. Lines starting with `<` are what the server sent back.
- Found the request line, the status line and the content-type line.
- Repeated on linkedin.com without a guide: status 200, content-type `text/html`. DevTools and curl show the same request and response in two different tools.

### curl -I: redirect and not found

```
$ curl -I http://github.com
301 Moved Permanently
location: https://github.com/
```

```
$ curl -I https://github.com/this-page-does-not-exist-12345
404 Not Found
```

(Key lines only.)

- A 301 means the page moved permanently. The `location` header gives the new address. Here the plain `http://` request was sent to the `https://` version.
- A 404 means the server works but has no page at that address.

### curl POST: sending data

```
$ curl -v -X POST -d "name=aiman" https://httpbin.org/post
> POST /post HTTP/2
```

- `-X POST` sets the method and `-d` sends data in the request body.
- httpbin.org sends back what it received. In the response body, the `"form"` section contained `"name": "aiman"`, so the server received the data in the body of the POST.

### Why logins use POST

- My answer: with GET the data is shown in the URL. POST does not show it in the URL.

Extra notes:

- POST is not encrypted. The body can be read by anyone who can see plain `http://` traffic, for example someone on the same Wi-Fi. HTTPS is what encrypts it.
- A password in a GET URL would be saved in browser history, bookmarks and server logs. That is the real reason logins use POST.

### TCP vs UDP (revision)

- Video call, UDP: sometimes the video freezes or the voice cuts off, but it is fast. When a person speaks, the other person hears it immediately.
- Login page and web pages, TCP: every piece is necessary. If anything is lost the login fails, and half the data is useless.

Extra notes:

- TCP is like a phone call. A connection is made first, and lost data is sent again. It is reliable but a little slower.
- UDP is like posting postcards. There is no connection and no confirmation, so some may be lost. It is faster.
- HTTP/3 runs on QUIC, which uses UDP and adds its own reliability on top.

## Key takeaways

- Read a request and response with both DevTools (Network tab) and `curl -v`. Both show the same thing.
- In `curl -v`, `>` lines are sent and `<` lines are received.
- Status code families: 2xx success, 3xx redirect, 4xx client problem, 5xx server problem.
- A 301 sends the browser to a new address given in the `location` header.
- GET puts data in the URL. POST puts it in the body. Neither is encrypted. HTTPS is what protects the data.
- TCP when everything must arrive (web, login). UDP when speed matters more (video calls).
- `tr 'A-Za-z' 'N-ZA-Mn-za-m'` undoes rot13.

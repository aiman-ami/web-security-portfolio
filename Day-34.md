# Day 34: Five Access Control Labs, Four Vertical Privilege Escalations and One Horizontal (IDOR)

- Lab: Unprotected admin functionality, solved (Apprentice)
- Lab: Unprotected admin functionality with unpredictable URL, solved (Apprentice)
- Lab: User role controlled by request parameter, solved (Apprentice)
- Lab: User role can be modified in user profile, solved (Apprentice)
- Lab: User ID controlled by request parameter, solved (Apprentice)

Roadmap: A01 Broken Access Control, 5 of 13 Access control labs solved. 8 Apprentice and Practitioner labs remain.

## What I learned

### Unprotected admin functionality

Visited `/administrator-panel` directly and found a working admin interface with no login check at all.

- The admin panel exists at a predictable, guessable URL, and the server never checks whether the current user is actually an admin before showing it
- One sentence: if a sensitive page has no access check at all, simply knowing or guessing its URL is enough to use it

### Unprotected admin functionality with unpredictable URL

Checked the page's source code and found the admin panel's real path, `/admin-uqh7ma`, referenced somewhere in it (usually in a comment or a linked script), rather than being a plain guessable word.

- Same root cause as lab 1, no access check on the page itself, but this time the developers tried to rely on the URL being hard to guess instead of actually checking permissions
- A URL being obscure is not the same as it being protected. Anyone who finds the URL by any means, source code, a leaked link, a robots.txt entry, gets the same unrestricted access as before
- One sentence: hiding a sensitive endpoint behind an unguessable name is not access control, it just changes how the URL gets discovered

### User role controlled by request parameter

Logged in as wiener, intercepted the login response in Burp, and found the server sets a cookie, `Admin=false`, to track role. Changed it to `Admin=true` before letting the response reach the browser.

- The server trusts a value the client's own browser is simply storing and sending back on every request, with nothing to stop that value being edited
- Browser cookies are just text the client holds and submits, they carry no inherent trust or integrity unless the server adds something like a signature to detect tampering, which this server does not
- One sentence: if the server's idea of your role lives in a cookie it never verifies, editing that cookie directly changes your role

### User role can be modified in user profile

Used the account's email-update feature, which echoed back a JSON response showing the current roleid. Sent that same request to Repeater, added `"roleid":2` into the request body, and resent it, the server accepted it and actually elevated the account's real role on the backend.

- Different mechanism to lab 3: here, the role field was never something the normal UI sent at all, it only appeared in the server's response. Adding a field the client was never meant to supply, but the server still reads and trusts if present, is a form of parameter injection sometimes called mass assignment
- One sentence: a field only visible in the server's output can sometimes still be accepted as input, if the server trusts whatever shape of data it is given without checking whether that field is one the client should be allowed to set

### User ID controlled by request parameter

Noticed the account page's URL includes an `id` parameter holding the username, `wiener`. Changed it to `carlos` directly in Repeater, and the server returned carlos's account data, including his API key, without checking whether the logged-in session actually belonged to carlos.

- This is horizontal privilege escalation, not vertical like the first four labs. Vertical means a regular user reaching admin-only functionality. Horizontal means one ordinary user reaching another ordinary user's own data, same privilege level, wrong owner
- The server identifies whose data to return based on a parameter the client supplies and controls directly, with no check that the parameter matches the actual logged-in session
- This exact pattern, an ID in a URL or parameter that is never cross-checked against the session, is the classic definition of IDOR (Insecure Direct Object Reference)
- One sentence: if the server decides whose data to return based only on an ID you supply, rather than who you actually are, swapping that ID reaches someone else's data

## Key takeaways

- Vertical privilege escalation (regular user to admin, labs 1 to 4) and horizontal privilege escalation (user to a different, same-level user, lab 5) are the two main shapes access control bugs take
- Four different root causes produced the same kind of failure today: no check at all, security by obscurity instead of a real check, trusting a client-held cookie without verifying it, and trusting a client-supplied field the UI never normally sends
- A URL or parameter being hard to guess, or not shown anywhere in the normal UI, is never a substitute for the server actually checking session identity and permissions on every request
- The same investigative instinct keeps paying off: check the source code for hidden paths, intercept and edit cookies, and try adding fields to a request body that the response reveals but the UI never sends

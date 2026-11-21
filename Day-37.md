# Day 37: Four Authentication Labs, Username Enumeration, 2FA Bypass, and Broken Password Reset

- Lab: Username enumeration via different responses, solved (Apprentice)
- Lab: 2FA simple bypass, solved (Apprentice)
- Lab: Password reset broken logic, solved (Apprentice)
- Lab: Username enumeration via subtly different responses, solved (Practitioner)

Roadmap: A07 Authentication Failures, 4 of 14 Authentication labs solved. All 3 Apprentice labs done, 1 of 9 Practitioner labs done.

## What I learned

### Username enumeration via different responses

Found the valid username and password using Burp Intruder, same Sniper attack and payload-list workflow used throughout SQLi and earlier labs. Username: root, password: trustno1.

- A properly built login should give the exact same response, same message, same length, whether the username exists or not, so an attacker cannot tell which part of their guess was wrong
- Here, an invalid username returned "Invalid username", while a valid username with a wrong password returned a different message, "Incorrect password". That difference in response text, and resulting response length, is enough to confirm a username exists before any password is tried at all
- Once the valid username is known, the same Intruder attack is repeated against the password list instead, this time watching for a 302 response (a redirect, meaning login succeeded) rather than the usual 200 (meaning the login page was shown again after failure)
- One sentence: if the server's wording changes based on whether the username alone was right, that difference leaks which usernames are real, turning a two-unknown password guess into a one-unknown problem

### 2FA simple bypass

Logged in as carlos, and instead of entering a 2FA verification code at the prompt, manually changed the browser's URL directly to /my-account.

- The server appears to treat successful completion of the first login step, password accepted, as enough to consider the session fully authenticated for navigation purposes, without actually enforcing that the second factor, the 2FA code, must also be completed before account pages become accessible
- No code was ever entered or guessed. The vulnerability is not in the 2FA mechanism's cryptography or guessability at all, it is that the application simply forgets to block access to protected pages while a login is still mid-way through its second step
- One sentence: having a 2FA prompt on screen means nothing if the server does not actually check, on every subsequent page request, that 2FA was truly completed before granting access

### Password reset broken logic

The password reset flow sends an email containing a reset link with a token in the URL, used to authorize setting a new password. Testing in Burp Repeater showed the reset request still succeeds even when the token value is deleted entirely from both the URL and the request body.

- The token's presence in the URL looks like proper security, but the server never actually verifies that the submitted token matches the one that was generated and emailed, it simply ignores the token field when processing the new password
- Since the token is not being checked at all, the only thing actually determining whose password gets reset is the username field submitted alongside it, which is not meant to be user-editable but was sent as a hidden form field anyway
- Changing that hidden username field to carlos, while leaving the token blank as already confirmed safe, reset carlos's password directly with no proof of access to his email ever required
- One sentence: a security token that exists in the request but is never actually validated by the server provides no protection at all, whatever field the server does trust becomes the real point of attack

### Username enumeration via subtly different responses

Same core idea as lab 1, but the difference between a valid and invalid username is nearly invisible, both show the same message, "Invalid username or password.", with only a single trailing space differing in one of them, a typo in the server's error-generation code.

- Burp Intruder's Grep - Extract setting was used to automatically pull out the exact error message text from every response in the attack, placing it into its own column so the full set of responses could be compared directly rather than inspected one at a time
- Sorting by that extracted column surfaced the one response whose message text differed from all the others by a single trailing space, revealing the valid username despite the message appearing identical at a glance
- One sentence: even a difference too subtle to notice by eye becomes obvious once every response's relevant text is extracted and sorted side by side, which is exactly what Grep - Extract is for

## Key takeaways

- Username enumeration always comes down to the same root problem, something in the response (wording, length, timing, or even a single stray character) differs based on whether a username is valid, regardless of how subtle that difference is made
- Burp Intruder's Grep - Extract feature turns a response comparison that would be impractical to do by eye into a simple sort, by pulling the one piece of text that actually matters out of every response in an attack
- Multi-factor authentication and token-based resets only add security if the server actually enforces them on every relevant request. A 2FA prompt that does not block direct navigation, and a reset token that is never checked, both provide zero real protection despite looking secure on the surface
- A field that is not meant to be user-controlled, like a hidden username field in a password reset form, is still exploitable if the server trusts it without re-verifying it against an actual authenticated identity

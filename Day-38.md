# Day 38: Username Enumeration via Response Timing

- Lab: Username enumeration via response timing, solved (Practitioner)

Roadmap: A07 Authentication Failures, 5 of 14 Authentication labs solved.

## What I learned

### Username enumeration via response timing

Unlike the two earlier username enumeration labs, the response text here gives nothing away, valid and invalid usernames can return identical-looking error messages. The leak this time is how long the server takes to respond, not what it says.

- A common, flawed login implementation looks up the username first, and only hashes and compares the submitted password if that username actually exists. Password hashing is deliberately slow by design, so an invalid username gets rejected almost instantly with no hashing step at all, while a valid username takes measurably longer, since the server goes on to hash the submitted password before comparing it
- Submitting a very long password, around 100 characters, exaggerates this timing gap, since hashing a long string takes noticeably longer than hashing a short one, making the valid-username delay easier to detect
- The login endpoint blocks an IP after too many failed attempts, which would normally prevent testing many usernames in a row. The X-Forwarded-For header, meant to tell a server the original client IP when traffic passes through a proxy, is trusted blindly here, so sending a different spoofed value on every request makes each attempt appear to come from a different IP, and the block-after-N-attempts logic never triggers for any single IP
- A Pitchfork attack in Burp Intruder, rather than Sniper, was needed because two things had to change together on every single request: a new spoofed IP and the next candidate username, in lockstep. Pitchfork advances multiple payload lists in parallel, position 1 and position 2 moving together on each request, instead of testing every combination of both lists like Sniper does
- Keeping the password fixed as the same long string across every attempt isolates the one variable that matters, since any timing difference observed can then only be explained by the username being valid or invalid, not by password length changing between requests
- The Response received and Response completed columns in Burp's results table show the actual time each response took, letting the slowest response be identified and compared the same way earlier labs sorted by an extracted error message instead of timing
- One sentence: when a response's content reveals nothing, how long it takes to arrive can still give away whether a username is real, since checking a password is inherently slower than rejecting an unknown one outright, and spoofing a trusted IP header defeats the brute-force protection that would otherwise stop enough attempts to measure it

## Key takeaways

- Timing-based username enumeration works on the same principle as the content-based versions from Day 37, something about the response differs based on username validity, timing is just a subtler channel than wording or length
- A header like X-Forwarded-For, meant for legitimate proxy use, becomes a bypass for IP-based rate limiting whenever the application trusts its value without any verification that it reflects the real connecting client
- Burp Intruder's Pitchfork attack type exists specifically for cases where multiple payload positions must change together in lockstep on every request, rather than being tested in every possible combination against each other

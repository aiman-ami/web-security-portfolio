# Day 38: Timing-Based and Logic-Flawed Brute-Force Protection

- Lab: Username enumeration via response timing, solved (Practitioner)
- Lab: Broken brute-force protection, IP block, solved (Practitioner)
- Lab: Username enumeration via account lock, in progress (Practitioner)

Roadmap: A07 Authentication Failures, 6 of 14 Authentication labs solved.

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

### Broken brute-force protection, IP block

The lab blocks an IP after 3 failed logins in a row, but logging in successfully resets that failure counter back to zero. Password: freedom.

- A Pitchfork attack was built where position 1 (username) and position 2 (password) alternate together on every request: wiener paired with the correct password peter, then carlos paired with the next candidate password, repeating down both lists in lockstep
- Every other request, the wiener:peter login, is a guaranteed success, which resets the failure counter to zero immediately after it. The very next request, a guess at carlos's password, only ever counts as a single failure before being reset again by the following success, so the 3-in-a-row block threshold is never reached, no matter how many total guesses are made
- Resource pool concurrency had to be set to 1 (Maximum concurrent requests), otherwise Burp could fire several requests in parallel and the carefully ordered alternation of success/guess could arrive at the server out of sequence, breaking the reset timing the whole attack depends on
- After the attack, filtering out 200 responses and sorting by username left exactly one 302 response under carlos, with his real password in the adjacent payload column
- One sentence: if a successful login resets the failed-attempt counter, interleaving one guaranteed-success login between every single guess keeps the counter from ever reaching the block threshold, as long as requests are forced to fire in that exact order

## In progress

### Username enumeration via account lock

A Cluster bomb attack was run with username as position 1 and a blank Null payload position (5 repeats) as position 2, intended to send each candidate username 5 times in a row to trigger that username's account lock, which should then reveal itself through a longer, different error message. All 505 responses came back with an identical length of 3344 bytes, meaning the lock never actually triggered for any username. Suspected cause: Resource pool concurrency was not set to 1, the same setting needed in the IP block lab above, so requests to different usernames likely interleaved instead of each username's 5 attempts running back to back, preventing any single username's count from reaching the lock threshold uninterrupted. Plan for tomorrow: rerun with Resource pool Maximum concurrent requests set to 1 and check whether the response lengths finally diverge.

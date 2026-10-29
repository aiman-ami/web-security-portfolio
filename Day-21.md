# Day 21: Blind SQL Injection with Conditional Responses

- Lab: Blind SQL injection with conditional responses, solved
- Confirmed the TrackingId cookie was the injection point
- Confirmed the users table and username column exist with boolean true/false checks
- Found password length (20) with a manual binary search using length(password) > N
- Tried a Python requests script for the character-by-character brute force, hit a 504 timeout, switched to Burp Intruder instead
- Set up Intruder with two payload positions (substring position and character), Cluster bomb attack type, Brute forcer payload for the character set, and a Grep-Match on "Welcome back"
- Logged in as administrator with the recovered password to solve the lab

Roadmap: 2C blind SQL injection, first lab done. 2B Intruder fuzzing basics done at L2.

## Writeup

```
Bug: TrackingId cookie value injectable, no input sanitization
Technique: blind SQL injection, no error messages or data shown directly, had to infer
true or false from whether the page showed "Welcome back"
Payload pattern: ' and (select substring(password,POSITION,1) from users where
username='administrator')='CHAR'--
Why it worked: tested each of the 20 password positions against every possible
character using Burp Intruder's Cluster bomb attack, the response either showed
"Welcome back" (correct character) or didn't (wrong character), one true result per
position builds the full password
```

## Key takeaways
- Blind SQL injection has no visible data or error messages, the only signal is a
  true/false difference in the page's behavior
- LIMIT 1 can matter even in a boolean subquery, a subquery returning more than one
  row can break the check entirely on some databases
- Binary search (checking length > N) finds a number fast; brute force (trying every
  character) is better suited to automation than to doing by hand
- Burp Intruder's Cluster bomb attack type tests every combination of two or more
  payload positions, useful when two unknowns (position and character) both need
  testing together
- Grep-Match in Intruder flags which results contain a chosen string, turning a wall
  of responses into an easy to scan table
- A quick restart of Burp fixed two separate "communication error" issues today,
  worth trying before deeper debugging

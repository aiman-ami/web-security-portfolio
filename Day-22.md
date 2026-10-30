# Day 22: Blind SQL Injection with Conditional Errors

- Lab: Blind SQL injection with conditional errors, solved

Roadmap: 2C blind SQL injection, second lab done.

## Writeup

```
Bug: TrackingId cookie value injectable, no input sanitization, Oracle database
Technique: blind SQL injection, no visible data and no behavior difference in the
page itself. The only signal is whether the query throws an error (500) or not (200)
Payload pattern: '||(select case when CONDITION then to_char(1/0) else '' end from
users where username='administrator')||'
Why it worked: a true condition runs to_char(1/0), a divide by zero that always
errors. A false condition returns an empty string instead, no error. Used this to
confirm the users table exists, confirm administrator exists, find the password
length (20), then find each character with Burp Intruder, Sniper attack, stopping at
the first 500 per position instead of testing all 36 characters every time
```

## Key takeaways
- Conditional errors and conditional responses are the same underlying idea, only
  the signal changes: a response difference (Welcome back) vs an error code (500)
- to_char(1/0) is a reliable way to force an Oracle error only when a condition is
  true, since dividing by zero always fails
- Oracle needs an explicit FROM clause, testing with FROM DUAL first for a database
  type check is a quick way to confirm Oracle before building the real payload
- Sniper attack with a single payload position, stopping as soon as the target
  status code appears, is faster than Cluster bomb when you only need the first
  match, not every combination

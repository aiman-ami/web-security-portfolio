# Day 23: Visible Error-Based and Blind Time-Delay SQL Injection

- Lab: Visible error-based SQL injection, solved
- Lab: Blind SQL injection with time delays, solved
- Lab: Blind SQL injection with time delays and information retrieval, solved

Roadmap: 2C blind SQL injection, time-delay labs done. Out-of-band labs remain.

## Writeup: Visible error-based SQL injection

```
Bug: TrackingId cookie value injectable, no input sanitization
Technique: error-based SQL injection, the database's own error message leaks data
directly, no brute forcing or true/false testing needed
Payload: ' AND 1=CAST((SELECT password FROM users LIMIT 1) AS int)--
Why it worked: casting text to an integer fails when the text isn't numeric, and the
database includes the actual offending value in its error message. The password
appeared directly in the error text: invalid input syntax for type integer
```

## Writeup: Blind SQL injection with time delays

```
Bug: TrackingId cookie value injectable, likely inserted into a log table rather
than used in a WHERE clause
Payload: ' || (SELECT pg_sleep(10)) --
Why it worked: AND did not work here, since there was no existing boolean condition
to extend. || (string concatenation) does work on a plain inserted value, and since
pg_sleep(10) is a function call, PostgreSQL has to evaluate it to build the final
string, forcing the 10 second delay regardless of any condition
```

## Writeup: Blind SQL injection with time delays and information retrieval

```
Bug: TrackingId cookie value injectable, same concatenation-based injection point
Payload pattern: '||(select case when (username='administrator' and
substring(password,POSITION,1)='CHAR') then pg_sleep(10) else pg_sleep(0) end from
users)--
Why it worked: no visible output and no error difference, only a true condition
causes a 10 second delay. Used Burp Intruder Sniper, one payload position at a time,
watching response time instead of status code, stopping each position as soon as a
slow response appeared
```

## Key takeaways
- AND extends an existing WHERE condition; || concatenates onto a plain string
  being inserted or logged, use whichever matches how the input is actually used
- CAST to an incompatible type can leak data straight into the error message,
  no blind inference needed when this works
- pg_sleep(N) forces a real delay only when PostgreSQL has to evaluate it, making
  it reliable for both unconditional and CASE-conditional delays
- A reversed true/false mapping in a timing attack makes the common case (wrong
  guesses) slow instead of the rare case (right guess), turning a 10 minute attack
  into an hour long one
- Burp Intruder Sniper, stopping at the first matching signal per position, works
  the same way whether that signal is a status code or a response time

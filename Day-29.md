# Day 29: More JS String Context and Introducing Exploitation Labs

- Lab: Reflected XSS into a JavaScript string with single quote and backslash escaped, solved
- Lab: Reflected XSS into a JavaScript string with angle brackets and double quotes HTML-encoded and single quotes escaped, solved
- Lab: Stored XSS into onclick event with angle brackets, double quotes HTML-encoded, single quotes and backslash escaped, solved
- Lab: Reflected XSS into a template literal with angle brackets, quotes, backslash and backticks Unicode-escaped, solved
- Lab: Exploiting cross-site scripting to steal cookies, blocked (requires Burp Suite Professional / Collaborator)
- Lab: Exploiting cross-site scripting to capture passwords, blocked (requires Burp Suite Professional / Collaborator)

Roadmap: 2C XSS, filter-bypass Practitioner group continuing. Two exploitation
labs blocked, to revisit using the CSRF-based alternative once the
"Exploiting XSS to bypass CSRF defenses" lab is done.

## What I learned

### Reflected XSS into a JavaScript string with single quote and backslash escaped

```
</script><script>alert(1)</script>
```

Why it worked: both the single quote and backslash were escaped this time, so
breaking out of just the string wasn't possible. Instead, the payload breaks
out of the entire surrounding script block with `</script>`, then opens a
brand new one with its own clean `<script>alert(1)</script>`, sidestepping
the string escaping altogether.

### Reflected XSS into a JavaScript string with angle brackets and double quotes HTML-encoded and single quotes escaped

```
\'-alert(1)//
```

Same double-backslash-cancels-the-escape trick as the Day 26/27 eval() lab.
Quotes were escaped but the backslash itself was not, so sending a raw `\`
combines with the server's own auto-added backslash to cancel out, letting
the next real quote close the string early.

### Stored XSS into onclick event with angle brackets, double quotes HTML-encoded, single quotes and backslash escaped

```
http://foo?&apos;-alert(1)-&apos;
```

Why it worked: this lab's filter escapes any literal `'` character you type.
`&apos;` is an HTML entity, a way of writing a single quote using plain text
(`&`, `a`, `p`, `o`, `s`, `;`) instead of the character itself. At the moment
the filter checks your input, there is no `'` character present for it to
catch, the browser only converts `&apos;` into a real `'` afterward, when it
renders the page, after the filter has already finished running. `foo` is
just a placeholder word, like "example", the actual domain doesn't matter.

### Reflected XSS into a template literal with angle brackets, quotes, backslash and backticks Unicode-escaped

```
${alert(1)}
```

Why it worked: a template literal (text wrapped in backticks in JavaScript)
treats anything inside `${ }` as a real JavaScript expression to evaluate and
insert, not plain text. Since backticks themselves were escaped, breaking out
of the template literal wasn't needed, staying inside it and using its own
`${ }` syntax was enough to run code directly.

### Exploiting XSS to steal cookies / capture passwords, blocked

Both labs' official solution requires Burp Suite Professional specifically
for Collaborator, the same wall hit with the SQLi out-of-band labs on Day 24.
Both labs mention an alternative solution using XSS combined with CSRF, which
makes the victim post their own cookie or credentials publicly as a comment
instead of exfiltrating them to an external server. That alternative depends
on the "Exploiting XSS to bypass CSRF defenses" lab, not yet done, so both
are queued to revisit once that lab is complete.

## Key takeaways
- When both the quote and the backslash used to escape it are blocked,
  breaking out of the entire script block with </script><script> can work
  where breaking out of just the string cannot
- HTML entities (&apos; for ', &amp; for &, &lt; for <, &gt; for >,
  &quot; for ") let a character travel through a filter as plain text, since
  the browser only decodes them into real characters after the filter runs
- A template literal's ${ } syntax runs real JavaScript directly, no need to
  escape the surrounding backticks if the ${ } itself isn't blocked
- Not every lab is solvable in Burp Community Edition, some have an
  alternative path using earlier or later labs' techniques instead

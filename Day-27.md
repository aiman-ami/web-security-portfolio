# Day 27: Understanding Reflected DOM XSS, Stored DOM XSS

- Revisited the Reflected DOM XSS lab from Day 26, traced eval() in searchResults.js through the browser's Debugger tab, and worked through why the payload actually works
- Lab: Stored DOM XSS, solved

Roadmap: 2C XSS, DOM XSS Practitioner group complete, all 4 labs done.

## What I learned

### Revisiting Reflected DOM XSS, the real mechanism

Found the vulnerable line directly in the browser's Debugger tab (F12, Debugger,
resources/js/searchResults.js):

```javascript
eval('var searchResultsObj = ' + this.responseText);
```

- `this.responseText` is the raw JSON text that came back from the search
  request, just text at this point, nothing has run yet
- `+` glues that text onto the string `'var searchResultsObj = '`, building one
  long line of text
- `eval()` takes that final text and actually runs it as real JavaScript code,
  not just stores it as data. That's the whole bug: anything the server sends
  back becomes a real command the moment eval() touches it

Why the payload `\"-alert(1)}//` works, traced step by step:

- The server auto-escapes any quote mark the user types, turning `"` into `\"`
  to keep its JSON valid
- The server does NOT escape backslashes the user types themselves
- Sending a raw `\` right before a `"` means the final text has two
  backslashes in a row before that quote: the user's own, plus the server's
  auto-added one
- In JavaScript, `\\` (two backslashes) collapses into a single real
  backslash character, not a double-escape. That uses up the escaping, so the
  quote mark immediately after it is no longer protected, it actually closes
  the string early
- Everything after that point, `-alert(1)`, is no longer text inside a
  string, it's real JavaScript sitting in a spot where a value is expected
- `"\\" - alert(1)` is a valid subtraction expression. To compute it,
  JavaScript must run both sides, so it calls alert(1) to get a value back,
  that's the popup, a side effect of doing math, not a broken rule
- `}` closes the JSON object normally, `//` comments out the leftover `"}`
  that the server still glues onto the end, so nothing errors out afterward

### Stored DOM XSS

Payload, posted as a blog comment:

```
<><img src=1 onerror=alert(1)>
```

Why it worked: the site uses JavaScript's `replace()` function to encode angle
brackets and prevent XSS. When replace() is given a plain string (not a regex
with the /g flag), it only replaces the first match it finds, not every one.
The leading, empty `<>` pair gets caught by that first, one-time replace,
"using it up". Every angle bracket after that, including the real
`<img src=1 onerror=alert(1)>` tag, passes through completely unfiltered.

## Key takeaways
- eval() turns text into running code, the moment any attacker-controlled text
  reaches it, that text can become a real command
- An inconsistency in what gets escaped (quotes escaped, but not the
  backslashes used to escape them) can be chained to cancel out the escaping
  entirely
- A subtraction or other operator placed where a value is expected forces
  JavaScript to evaluate both sides first, which can be used to trigger a
  function call as a side effect
- `.replace()` on a plain string only replaces the first match, not every
  occurrence, a filter built this way can be defeated just by adding one extra
  harmless match before the real payload
- Tracing a bug through the browser's Debugger tab, not just the Network tab,
  matters when the vulnerability lives in the site's own JavaScript logic
  rather than in how data is reflected into HTML

# Day 26: DOM XSS Practitioner Labs

- Lab: DOM XSS in document.write sink using source location.search inside a select element, solved
- Lab: DOM XSS in AngularJS expression with angle brackets and double quotes HTML-encoded, solved
- Lab: Reflected DOM XSS, solved
- Started xss-cheatsheet.md, a "find the house, match the trick" reference table

Roadmap: 2C XSS, DOM Practitioner group, 3 of 4 labs done. Stored DOM XSS remains.

## What I learned

### How to pick a payload (the method)

1. Type a random word into the input, submit it
2. Right-click where it shows up, Inspect, find the word in the actual code
3. Look at what's immediately around it, that's the "house" it's sitting in
4. Match the house to the trick (see xss-cheatsheet.md for the full table)

### DOM XSS in document.write sink using source location.search inside a select element

Payload, as the storeId parameter:

```
"></select><img src=1 onerror=alert(1)>
```

Why it worked: the storeId value gets placed inside a select dropdown as an
option. A script tag wouldn't run inside a select element. The payload closes
the broken attribute and the select tag itself first, then adds a brand new,
independent img element right after it, where onerror can fire normally.

### DOM XSS in AngularJS expression with angle brackets and double quotes HTML-encoded

Payload:

```
{{$on.constructor('alert(1)')()}}
```

Why it worked: angle brackets and quotes were both encoded, so no HTML
injection was possible. The page uses AngularJS, which evaluates anything
inside double curly braces as its own expression syntax on any element with
ng-app. This payload uses AngularJS's own syntax to reach JavaScript's
Function constructor and run code, no angle brackets needed anywhere.

### Reflected DOM XSS

Payload, as the search term:

```
\"-alert(1)}//
```

Still being worked through, not fully understood yet. What's confirmed so far:
- The search box sends a request to a search-results endpoint, visible under
  the Network tab's XHR filter, which returns JSON like
  {"results":[],"searchTerm":"XXS"}
- A file called searchResults.js (visible in Burp's Site map, or in the
  browser's DevTools Debugger tab) takes that JSON and runs it through eval(),
  which executes text as real code instead of just reading it as data
- Testing a quote character showed it passes through unescaped, confirming the
  string could be broken out of
- The payload ends up making part of the text look like a subtraction
  operation (-alert(1)), and JavaScript has to actually run alert(1) to
  perform that subtraction
- Still unclear: exactly how to independently find a eval() call in a JS file
  before knowing it's there, and the full character-by-character reasoning for
  why \" and the trailing }// are needed. To revisit.

## Key takeaways
- A dropdown or other structural HTML element needs its whole container closed
  before a payload can run, not just the attribute
- A framework using {{ }} expression syntax can be abused with its own syntax,
  with no HTML involved at all
- JSON responses used with eval() are a DOM XSS source worth checking, found
  by watching the Network tab's XHR requests, not just the page's HTML
- Reflected DOM XSS still needs step 1 and 2 of the method (random word in,
  inspect where it lands), but the "where it lands" can be inside a JS file's
  logic, not just the page HTML, which makes it harder to spot

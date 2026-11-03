# Day 25: Cross-Site Scripting Basics

- Lab: Reflected XSS into HTML context with nothing encoded, solved
- Lab: Stored XSS into HTML context with nothing encoded, solved
- Lab: DOM XSS in document.write sink using source location.search, solved
- Lab: DOM XSS in innerHTML sink using source location.search, solved
- Lab: DOM XSS in jQuery anchor href attribute sink using location.search source, solved
- Lab: DOM XSS in jQuery selector sink using a hashchange event, solved
- Lab: Reflected XSS into attribute with angle brackets HTML-encoded, solved
- Lab: Stored XSS into anchor href attribute with double quotes HTML-encoded, solved
- Lab: Reflected XSS into a JavaScript string with angle brackets HTML encoded, solved
- Closed Burp's Scope and target mapping item: added the lab domain to scope, removed an accidental extra, filtered the site map to in-scope only (L2)

Roadmap: 2C XSS basics, all 9 Apprentice labs done. 2B Scope and target mapping done at L2.

## What I learned

### XSS, the core idea

XSS means getting a website to run your own JavaScript inside someone else's browser,
as if it were the site's own script. Once it runs, it has that person's session,
cookies, and login, so it can steal data, change the page, or act as them.

### Reflected XSS into HTML context with nothing encoded

```
<script>alert(1)</script>
```

Typed into the search bar. The search term gets inserted straight into the page's
HTML with no filtering at all, so the script tag runs as written. "Reflected" means
it only exists for that one request, it has to be sent again each time.

### Stored XSS into HTML context with nothing encoded

```
<script>alert(1)</script>
```

Same payload, but submitted as a blog comment instead of a search. "Stored" means it
gets saved in the database, so it runs for every single visitor who views that blog
post afterward, not just once for the person who submitted it. A bigger real risk
than reflected, since the attacker only has to post once.

### DOM XSS in document.write sink using source location.search

Payload, entered into the search box:

```
"><svg onload=alert(1)>
```

Extra notes:
- The search term gets written into an `img src="..."` attribute by
  `document.write`, which copies text from the URL straight into the page without
  checking it
- `">` closes the broken img tag early, breaking out into the surrounding HTML
- `<svg>` is a normal element for drawing graphics, not malicious by itself
- `onload` is an event handler, a small piece of JavaScript many HTML elements can
  run automatically. `onload` fires the instant that element finishes loading, no
  click needed
- `<svg>` was used here instead of `<script>` because some filters specifically
  block the word "script". An event handler on any element does the same job

### DOM XSS in innerHTML sink using source location.search

Payload:

```
<img src=1 onerror=alert(1)>
```

Extra notes:
- `innerHTML` replaces the contents of a page element with whatever text is given
  it, including real HTML tags, if that text isn't checked first
- `src=1` is not a valid image location, so the image fails to load
- `onerror` fires automatically when that failure happens, same idea as `onload`,
  just triggered by an error instead of a success

### DOM XSS in jQuery anchor href attribute sink using location.search source

Payload, as the `returnPath` value:

```
javascript:alert(document.cookie)
```

Extra notes:
- The page builds a "back" link's `href` straight from the URL, with jQuery's `$`
  selector finding the link and setting it
- A `javascript:` URL is a real, valid link type, clicking it runs the JavaScript
  after the colon instead of navigating anywhere
- `document.cookie` reads the browser's stored cookies for that site, which is
  exactly what a real attacker would want to steal

### DOM XSS in jQuery selector sink using a hashchange event

This lab needed a separate victim, delivered through the exploit server, not just
typing into the address bar.

```html
<iframe src="https://YOUR-LAB-ID.web-security-academy.net/#" onload="this.src+='<img src=x onerror=print()>'"></iframe>
```

Extra notes, explained simply:
- An `iframe` is a webpage embedded inside another page, like a window showing a
  different site
- The vulnerable code only runs when the URL's hash (the part after `#`) changes
  while the page is already open, just loading it once with a hash isn't enough
- `onload="..."` runs once the embedded page finishes loading the first time
- `this` refers to the iframe itself, `this.src` is its current address
- `this.src += '...'` adds text onto the end of that address after the first load,
  which changes the hash, triggering the "hashchange" the vulnerable code listens
  for
- The appended text, `<img src=x onerror=print()>`, is the real payload, broken
  image, onerror fires, print() runs
- So the trick is two steps disguised as one: load the real page first with an
  empty hash, then immediately change the hash to the real payload, since the bug
  only fires on a change, not on the initial load

### Reflected XSS into attribute with angle brackets HTML-encoded

```
"onmouseover="alert(1)
```

Why it worked: angle brackets `<` `>` were HTML-encoded, so a script or svg tag
wouldn't survive, but quotes were not encoded. This payload closes the current
attribute early with `"`, then opens a brand new attribute, `onmouseover`, with its
own event handler. No angle brackets needed anywhere, so the filter never triggers.

### Stored XSS into anchor href attribute with double quotes HTML-encoded

```
javascript:alert(1)
```

Submitted as the comment author's "website" value. Why it worked: the website field
becomes the href of the author name link. Double quotes were encoded, so breaking
out of the attribute wasn't an option, but replacing the whole field with a
javascript: URL didn't need to break out of anything, it just became the entire
href value, and runs when the author name link is clicked.

### Reflected XSS into a JavaScript string with angle brackets HTML encoded

```
'-alert(1)-'
```

Why it worked: the search term is reflected inside an existing JavaScript string,
something like `var x = 'INPUT';`. Angle brackets were encoded so a tag-based
payload wouldn't work, but the single quote wasn't encoded. The leading `'` closes
the string early, `alert(1)` runs as its own statement, and the trailing `-'` keeps
the rest of the original line from breaking, using subtraction as a harmless way to
glue the leftover syntax together.

## Key takeaways
- Reflected XSS needs the victim to send the malicious request themselves (click a
  link); stored XSS runs for anyone who later views the saved content
- document.write and innerHTML both insert raw text into the page as real HTML if
  it isn't filtered first, that's the actual vulnerability in DOM-based XSS
- Event handler attributes (onload, onerror, onclick, and others) can run
  JavaScript on many HTML elements, not just inside a script tag, useful when
  "script" itself is filtered
- A javascript: URL runs code when clicked or navigated to, instead of going to a
  real address, and can replace an entire attribute value rather than needing to
  break out of it
- Some DOM-based bugs only trigger on a change (like the URL hash changing) after
  the page has already loaded, not on the first load, which is why the exploit
  server and iframe trick was needed here
- When angle brackets are encoded but quotes are not, breaking out of an attribute
  or a JavaScript string with just a quote is often enough, no tags required at all

# Day 28: XSS Filter Bypass with Burp Intruder

- Lab: Reflected XSS into HTML context with most tags and attributes blocked, solved
- Lab: Reflected XSS into HTML context with all tags blocked except custom ones, solved
- Lab: Reflected XSS with some SVG markup allowed, solved
- Lab: Reflected XSS in canonical link tag, solved

Roadmap: 2C XSS, filter-bypass Practitioner group, 4 labs done.

## What I learned

### Finding what a WAF blocks, with Burp Intruder

Method used across several labs today:
1. Try an obvious payload, confirm it gets blocked (usually a 400 response)
2. Send the request to Intruder, replace the search term with `<>`, mark a
   payload position between the brackets
3. Paste PortSwigger's own list of HTML tags (from their XSS cheat sheet)
   as the payload list, run the attack
4. Look for the rare 200 responses among the 400s, those are the tags the
   WAF doesn't recognize or block
5. Repeat the same process for event handlers (onload, onerror, onfocus, and
   so on) once a working tag is found, to find which event survives too

### Reflected XSS into HTML context with most tags and attributes blocked

Found via the method above: body tag and onresize event both pass the filter.
Delivered through the exploit server, since onresize needs the page to
actually resize after loading, not just load once:

```html
<iframe src="https://YOUR-LAB-ID.web-security-academy.net/?search=%22%3E%3Cbody%20onresize=print()%3E" onload=this.style.width='100px'>
```

Decoded payload: `"><body onresize=print()>`

### Reflected XSS into HTML context with all tags blocked except custom ones

```html
<script> location = 'https://YOUR-LAB-ID.web-security-academy.net/?search=%3Cxss+id%3Dx+onfocus%3Dalert%28document.cookie%29%20tabindex=1%3E#x'; </script>
```

Decoded payload: `<xss id=x onfocus=alert(document.cookie) tabindex=1>#x`

Why it worked:
- `<xss>` is not a real HTML tag, so it's not on the WAF's blocklist of known
  dangerous tags, but the browser still renders it as an element that can
  carry attributes and events
- `tabindex=1` makes this meaningless tag focusable
- a URL hash matching an element's id (`#x` matching `id=x`) tells the
  browser to scroll to and focus that element automatically on page load,
  no click needed
- `onfocus` fires the instant that happens
- the outer `<script>location = '...'</script>` on the exploit server is just
  the delivery step, it redirects the victim's browser to the real crafted
  URL, it isn't part of the XSS itself

### Reflected XSS with some SVG markup allowed

Found via the Intruder method: svg, animatetransform, and title/image tags
survived the tag filter; onbegin survived the event filter.

```
"><svg><animatetransform onbegin=alert(1)>
```

Why it worked:
- `<svg>` here is just a container, some special tags only mean anything
  inside an svg wrapper
- `<animatetransform>` is a real SVG tag meant for animating elements over
  time, so it has its own lifecycle events
- `onbegin` fires the instant the animation starts, automatically, as soon
  as the element loads
- filters often miss these SVG animation tags and their events because
  they're obscure, not because they're hard to find, this is exactly what
  the Intruder method is for

### Reflected XSS in canonical link tag

```
'accesskey='x'onclick='alert(1)
```

Why it worked: reflected inside a `<link rel="canonical" href='...'>` tag,
inside single quotes. `accesskey` is a real HTML attribute that assigns a
keyboard shortcut to any element. `onclick` fires when that shortcut is
pressed, even though no real mouse click happened. The lab's simulated
victim presses the shortcut automatically, so no exploit server was needed
even though a "second step" (the keypress) is technically involved. This is
an edge case to the normal URL-vs-exploit-server rule.

## Key takeaways
- When manual guessing fails against a WAF, Burp Intruder with PortSwigger's
  own cheat sheet tag/event lists turns guessing into a systematic search
- Not every blocked filter blocks everything, obscure tags (custom ones,
  SVG animation tags) and obscure events are commonly missed
- A made-up tag name still renders as a real element in the browser, and can
  still carry working attributes and event handlers
- accesskey plus onclick fires from a keyboard shortcut, not just a real
  mouse click, a second way to trigger an event handler without needing
  onload, onerror, or similar
- The URL-hash-matches-id trick (#x matching id=x) auto-focuses an element
  on page load, useful for triggering onfocus without any interaction

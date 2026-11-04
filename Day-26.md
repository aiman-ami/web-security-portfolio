### How to pick a payload (the method, not memorized answers)

1. Put a random alphanumeric string into the input, submit it
2. Find where it landed: right-click, Inspect (or check the response in Repeater),
   and read the actual HTML or JavaScript surrounding the string
3. Pick the smallest change that either escapes that surrounding structure, or
   abuses what that structure is built to do, rather than reaching for a
   memorized "XSS payload"

The container changes every time (an HTML tag's content, an attribute, a
JavaScript string, a framework's own expression syntax), which is why XSS
payloads look so different from each other. SQLi feels more unified because
it's always SQL. XSS payloads differ because HTML, JavaScript, and frameworks
like AngularJS are different languages with different escape rules.

### DOM XSS in document.write sink using source location.search inside a select element

Payload, as the storeId parameter:

```
"></select><img src=1 onerror=alert(1)>
```

Why it worked: the storeId value gets placed inside a `<select>` dropdown as an
option. A script tag wouldn't run inside a select element even if injected, so
the payload first closes the broken attribute and the select tag itself with
`"></select>`, then adds a brand new, independent img element right after it,
outside the dropdown, where onerror can fire normally.

### DOM XSS in AngularJS expression with angle brackets and double quotes HTML-encoded

Payload:

```
{{$on.constructor('alert(1)')()}}
```

Why it worked: no HTML injection here at all, angle brackets and quotes were
both encoded, so breaking out of a tag or attribute wasn't an option. The page
uses AngularJS, which scans any element with the ng-app directive and evaluates
anything inside double curly braces as its own expression syntax, not plain
text. This payload uses legitimate AngularJS syntax to reach JavaScript's
Function constructor and run arbitrary code, entirely within AngularJS's own
rules, no angle brackets needed anywhere.

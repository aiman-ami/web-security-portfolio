# Day 31: Three Expert XSS Labs (CSP and AngularJS Sandbox, SVG Animate, onerror)

- Lab: Reflected XSS with AngularJS sandbox escape and CSP, solved (Expert)
- Lab: Reflected XSS with event handlers and href attributes blocked, solved (Expert)
- Lab: Reflected XSS in a JavaScript URL with some characters blocked, solved (Expert)

Roadmap: 2C XSS, three Expert labs done.

## What I learned

### Reflected XSS with AngularJS sandbox escape and CSP

Delivered through the exploit server, which redirects the victim to a search URL carrying the payload. The exploit server body:

```
<script>
location = 'https://YOUR-LAB-ID.web-security-academy.net/?search=%3Cinput%20id=x%20ng-focus=$event.composedPath()|orderBy:%27(z=alert)(document.cookie)%27%3E#x';
</script>
```

URL encoding used in the search value:

```
%3C → <     %20 → space     %3E → >     %27 → '
```

The injected HTML, decoded for readability:

```
<input id=x ng-focus=$event.composedPath()|orderBy:'(z=alert)(document.cookie)'>#x
```

How it works, step by step:

- CSP (Content Security Policy) is a response header that tells the browser which script sources are allowed. It blocks injected inline script tags and typical XSS payloads even when the injection itself succeeds
- The bypass idea: the page already loads and trusts the AngularJS library. ng-focus is not raw JavaScript. It is an AngularJS attribute, and the AngularJS library evaluates its expression. Because that library is already allowed, CSP never sees a new script, only AngularJS doing its normal work
- The `#x` hash matches id=x, so the input receives focus the moment the page loads. That is why ng-focus is used instead of ng-click: focus happens automatically and no click is needed
- A second defence sits inside AngularJS: the sandbox. Its job is to stop expressions from reaching the window object directly
- Trick 1, reaching window without typing it: `$event.composedPath()` returns the list of objects the focus event passed through, from the element outward. The last item in that list is the window object. The sandbox never sees the word window in the expression
- `|orderBy:...` is AngularJS filter syntax, normally used to sort a list. Here it evaluates the quoted expression against each item of the composedPath list. When the walk reaches the window item, the expression runs with window as its scope
- Trick 2, avoiding the sandbox's check on direct calls: if the expression called alert(document.cookie) directly, the sandbox would detect a function being called with window as its context and block it. `(z=alert)(document.cookie)` assigns alert to the name z and then calls the result of that assignment, so the call no longer looks like a direct call on window and the check is avoided
- One sentence: use the AngularJS machinery that CSP already trusts to walk up to the window object, then call alert(document.cookie) in a form the sandbox does not recognise as a direct window call

My notes:

- This was the hardest lab so far on first read. The story version made it clear: a trusted employee (AngularJS) walks past the first guard (CSP), and two small tricks defeat the second guard (the sandbox)
- The lab's own hint described the shape of the solution: ng-focus bypasses CSP, the path array ends at window, and assigning to z avoids the sandbox check

### Reflected XSS with event handlers and href attributes blocked

Delivered by visiting the crafted URL directly. No exploit server is needed.

URL encoding used in the search value:

```
%3C → <     %3E → >     %2F → /     %3A → :
```

The injected HTML, decoded for readability:

```
<svg><a><animate attributeName=href values=javascript:alert(1) /><text x=20 y=20>Click me</text></a>
```

How it works, step by step:

- The filter blocks every event handler (onclick and similar) and any anchor href written directly. Writing href="javascript:alert(1)" by hand is caught immediately
- The trick: do not write the dangerous href at all. The link has no href attribute in the injected source, and SVG's animate tag sets one after the page has loaded, once the filter has already finished checking
- `<svg><a>...</a></svg>`: svg is a container, and an a element inside it is a real clickable link, drawn with SVG instead of regular HTML
- `<animate attributeName=href values=javascript:alert(1) />`: animate normally changes an element's property over time. attributeName=href names the property being animated, and values=javascript:alert(1) is the value it is set to. The dangerous value is never written as the link's own href in the source, so the filter does not see it. It is applied live, as an animation effect
- `<text x=20 y=20>Click me</text>` draws the clickable words at position 20,20. The lab requires the word Click, because the simulated victim only clicks something labelled that way
- Once animate has set the href, clicking the link behaves like the javascript: URL in the Day 25 lab "DOM XSS in jQuery anchor href attribute sink using location.search source": a javascript: href runs its code on click, so no event handler is needed
- One sentence: instead of writing the dangerous destination directly, which gets blocked, use SVG's animation feature to set it after the filter has checked the page

My notes:

- Much simpler than the first lab once decoded. The core idea is timing: filters check the initial value, and animation changes it later
- Same pattern as lab 1: animate is a real SVG feature doing its real job, aimed at a property the filter did not expect

### Reflected XSS in a JavaScript URL with some characters blocked

Delivered by appending the payload to the end of a blog post URL, after the postId parameter, and visiting it. The "Back to blog" link on that page then carries the payload.

URL encoding and substitutions used:

```
%27 → '     %3D → =     %3E → >     %2B → +     /**/ → an empty comment standing in for a blocked space
```

The payload, decoded for readability:

```
'},x=x=>{throw/**/onerror=alert,1337},toString=x,window+'',{x:'
```

How it works, step by step:

- My input lands inside the options object of a fetch() call that builds the "Back to blog" javascript: link. Spaces and some other characters are blocked. I must escape that object, make something fire when the link is clicked, and get the string 1337 into the alert message
- The plan: I cannot call alert directly, so I set the browser's built-in error handler (window.onerror, which fires on any unhandled error) to be alert. Then I rig window so that reading it as text causes an error that contains 1337. When the javascript: link runs on click, it triggers the second part, which sets off the first
- `'}` closes my string, then closes the object I was sitting inside. This is a clean breakout
- `x=x=>{throw/**/onerror=alert,1337}`: defines a function called x. throw is a statement, and a statement cannot sit inside other code as an expression, so it must be wrapped in its own function. That is the purpose of the arrow function x=>{...}. Inside it, onerror=alert sets the error handler, and the comma means the value thrown is the number 1337
- `/**/` sits where a space would normally go. Spaces are blocked in this lab, and an empty comment is ignored by the browser but still separates the words
- `toString=x`: every JavaScript object has a built-in toString that runs automatically whenever the object must become text. This line overwrites it with my function x
- `window+''`: adding an empty string forces JavaScript to convert window to text. Because toString is overwritten, the conversion runs my function, which throws 1337. window.onerror catches it, and onerror is alert
- `,{x:'` opens a fresh, valid object literal so the original code after my input still parses and does not crash
- It only fires when someone clicks "Back to blog", because that click runs the javascript: URL and forces window to be converted to text
- One sentence: set the error handler to be alert, then rig window so that reading it as text throws 1337 and triggers the handler

My notes:

- Every unusual piece has a reason: the arrow function because throw needs a block, /**/ because spaces are blocked, toString because that is how objects become text, and window+'' because that forces the conversion

## Key takeaways

- Filters, CSP, and sandboxes do not make XSS impossible. They force indirection: use the framework's own features, change values after the check happens, or fire code through built-in machinery such as onerror
- The same pattern repeated in all three labs: ng-focus is an AngularJS feature, animate is an SVG feature, and onerror is a browser feature. Real features aimed at unexpected targets pass checks that block raw scripts
- Reaching window without typing the word: $event.composedPath() ends at window, and orderBy evaluates the expression with window as scope. Avoiding the sandbox's direct-call check: assign alert to another name and call the result of the assignment, as in (z=alert)(...)
- SVG animate changes an attribute after the filter has checked the initial value. Timing is a bypass, not only encoding
- throw is a statement and needs an arrow-function block to exist inside other code. An empty comment /**/ can replace a blocked space
- The reading method that worked today: decode the URL first, rewrite the payload as a short story, then map the story back to the lab's hint text. Understand before memorising

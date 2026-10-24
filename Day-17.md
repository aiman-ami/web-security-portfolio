# Day 17: git diff, Bash Arguments and Status Scripts, JavaScript Basics, DOM, Events and fetch

What I did:
- Cloned my portfolio repo into WSL and practised `git status`, `git diff` and `git restore`
- Learned Bash arguments (`$1`), the empty check (`-z`) and `exit 1`
- Wrote `status.sh` solo, then `check2.sh` solo (kept as local practice, not in the repo)
- Learned JavaScript in the browser DevTools console: print, variables, joining text, if/else, lists, loops
- Used the DOM (select and change page elements), events (click), `fetch`, and read a small script
- Hit a CORS error on a cross-origin `fetch`

Roadmap: Git (git diff) done at L2. Bash scripting done at L3. JavaScript (read a small script, DOM, events, fetch) done at L2 (guided, not yet written alone). Phase 1 complete.

## What I learned

### git diff

`git diff` shows exactly what changed in tracked files that are not yet staged.

```bash
$ cd web-security-portfolio
$ echo "test line" >> README.md
$ git status
modified:   README.md
$ git diff
--- a/README.md
+++ b/README.md
@@ -1 +1 @@
-# web-security-portfolio
\ No newline at end of file
+# web-security-portfoliotest line
$ git restore README.md
```

How to read it:
- `---` is the old version and `+++` is the new version
- `@@ -1 +1 @@` means line 1 changed
- A line starting with `-` was removed and a line starting with `+` was added
- `git restore FILE` throws away unstaged changes and brings back the last committed version
- Editing a cloned copy only changes my local files. Ownership is checked when I push to GitHub, not when I edit.

Extra notes (general knowledge, not tested):
- `git diff --staged` shows changes that are already staged with `git add`
- `git diff` shows nothing for brand new untracked files until they are added

### Bash: arguments, empty check and exit

```bash
#!/bin/bash
if [ -z "$1" ]; then
    echo "Usage: ./arg.sh WORD"
    exit 1
fi
echo "You gave me: $1"
```

```bash
$ ./arg.sh
Usage: ./arg.sh WORD
$ ./arg.sh hello
You gave me: hello
$ ./arg.sh hello world
You gave me: hello
```

- `$1` holds the first word after the script name. `world` went into `$2`.
- `[ -z "$1" ]` is true when the argument is empty
- `exit 1` stops the whole script, not just the `if` block. Without it the script printed the usage message and then carried on to the last line.

Extra notes (general knowledge, not tested):
- `exit 0` means success and `exit 1` means failure
- `$?` holds the exit code of the last command, for example `ls` then `echo $?`

### Bash: status code scripts with curl

`status.sh` asks for a URL with `read -p`. `check2.sh` takes the URL as an argument and prints a message for each status code.

```bash
#!/bin/bash
if [ -z "$1" ]; then
        echo "Usage: ./check2.sh url"
        exit 1
fi
code=$(curl -s -o /dev/null -w "%{http_code}" "$1")
if [ "$code" -eq 200 ]; then
        echo "200: $1 is up"
elif [ "$code" -eq 301 ] || [ "$code" -eq 302 ]; then
        echo "$code: $1 is redirect"
elif [ "$code" -eq 404 ]; then
        echo "404: $1 not found"
elif [ "$code" -eq 000 ]; then
        echo "000: $1 unreachable"
else
        echo "Status code: $code"
fi
```

Tests I ran:

```bash
$ ./check2.sh https://github.com
200: https://github.com is up
$ ./check2.sh https://httpbin.org/status/404
404: https://httpbin.org/status/404 not found
$ ./check2.sh https://thisdoesnotexist12345.com
000: https://thisdoesnotexist12345.com unreachable
$ ./check2.sh http://github.com
301: http://github.com is redirect
$ ./check2.sh
Usage: ./check2.sh url
```

How the curl line works:
- `-s` hides the progress output
- `-o /dev/null` throws away the page body
- `-w "%{http_code}"` prints only the status code
- `$(...)` stores the printed result in a variable

Extra notes (general knowledge, not tested):
- `-eq`, `-ne`, `-gt`, `-lt` compare numbers. `=` and `!=` compare text. Using `-eq` on text gives an error.
- Spaces inside `[ ]` are required: `[ "$1" ]` works and `["$1"]` does not
- Always quote variables like `"$1"` so empty or spaced values do not break the test
- `curl -L` follows redirects, so `http://github.com` would end at the final 200 instead of showing 301
- `curl -I` fetches only the headers
- A `case` statement is a cleaner way to write a long `if/elif` chain on one value:

```bash
case "$code" in
    200) echo "up" ;;
    301|302) echo "redirect" ;;
    404) echo "not found" ;;
    000) echo "unreachable" ;;
    *) echo "Status code: $code" ;;
esac
```

### JavaScript basics (DevTools console)

Open DevTools with `F12`, click the Console tab, click the input row next to `>`, type a line and press Enter.

```javascript
> console.log("hello")
hello
> let name = "Aiman"
> console.log(name)
Aiman
> name = "Ali"
> console.log("Hello " + name)
Hello Ali
> let code = 404
> if (code === 200) { console.log("up") } else { console.log("not up") }
not up
> let sites = ["github.com", "linkedin.com"]
> console.log(sites[0])
github.com
> for (let s of sites) { console.log(s) }
github.com
linkedin.com
```

- `let` makes a variable. No `$` is used to read it.
- `+` joins text exactly as written, so the space has to be inside the quotes
- `===` means equal to. It is the JavaScript version of `-eq` in Bash and `==` in Python.
- Lists start at index 0, the same as Python
- `for (let s of sites)` reads as "for each s in sites". Here `s` is the site name itself, not the index.

Same check in three languages:

| Language | Check if code is 200 |
|---|---|
| Bash | `if [ "$code" -eq 200 ]; then ... fi` |
| Python | `if code == 200:` |
| JavaScript | `if (code === 200) { ... }` |

Extra notes (general knowledge, not tested):
- JavaScript and Java are different languages
- Semicolons at the end of lines are optional in JavaScript because the language adds them automatically

### JavaScript: the DOM (changing the page)

`document` is the page I am on. I tested these on `https://example.com`.

```javascript
> console.log(document.title)
> document.title = "I changed this"
> document.body.style.background = "pink"
> document.querySelector("h1")
null
> document.body.innerHTML
(the page's HTML: p tags and one a tag, no h1)
> document.querySelector("p").textContent = "Changed by Aiman"
```

- `document.title` is the tab title. Changing it changed the tab, only in my browser.
- `document.body.style.background = "pink"` changed the page colour. Any colour name works.
- `querySelector("tag")` picks the first matching element on the page
- `querySelector("h1")` returned `null` because this version of example.com has no h1. Setting text on `null` gives "Cannot set properties of null".
- `document.body.innerHTML` shows what the page is really made of, so I check it before assuming which tags exist
- `.textContent` is the text inside an element
- Reloading the page restores the original. Changes in the console never touch the real site.

Extra notes (general knowledge, not tested):
- JavaScript that can change the page is the basis of attacks like XSS, which comes in Phase 2

### JavaScript: events

An event is something the user does, such as a click, that JavaScript can react to.

```javascript
> document.querySelector("p").addEventListener("click", () => { console.log("you clicked the paragraph") })
```

Clicking the paragraph printed `you clicked the paragraph` once per click.

- `addEventListener("click", ...)` runs the code after `=>` every time the element is clicked
- `() => { ... }` is a small function. It is the code that runs on each click.

### JavaScript: fetch

`fetch` makes HTTP requests from JavaScript. It does the same job as `curl` in Bash and `requests` in Python.

```javascript
> fetch("/").then(r => console.log(r.status))
200
```

- `fetch("/")` makes a GET request to the same site
- `.then(r => ...)` waits for the reply and then runs the code. `r` is the reply.
- `r.status` is the status code, the same number `check2.sh` printed

### Reading a small script and the CORS error

Script I read first and explained in my own words:

```javascript
let sites = ["github.com", "example.com"]
for (let s of sites) {
  fetch("https://" + s).then(r => console.log(s + " returned " + r.status))
}
```

I explained the loop, the `"https://" + s` join, `r.status` and the spaces around `returned` correctly. Correction: `s` is the site name, not the index.

Result, run on the example.com tab:

```
example.com returned 200
Access to fetch at 'https://github.com/' from origin 'https://example.com' has been blocked by CORS policy: No 'Access-Control-Allow-Origin' header is present on the requested resource.
Uncaught (in promise) TypeError: Failed to fetch
```

- `example.com` is the page I was on, so it is the same origin and the request worked
- `github.com` is a different origin. The browser blocked my script from reading the reply because GitHub sent no permission header. This is CORS. It does not mean GitHub was down.
- Only browsers enforce this. `curl` and Python `requests` have no such limit.

Extra notes (general knowledge, not tested):
- The rule behind CORS is the same-origin policy. It stops a page on one site from reading pages of another site, for example another site reading my GitHub or bank pages.
- Many web attacks target this rule or sites that configure it badly. It comes up again in Phase 2.

## Key takeaways
- `git diff` shows unstaged changes line by line, and `git restore FILE` undoes them
- `$1` is the first argument, `-z` tests for empty, and `exit 1` stops the whole script
- `curl -s -o /dev/null -w "%{http_code}"` returns only the status code, which is useful in scripts
- Use `-eq` for numbers and `=` for text in Bash tests, and quote variables
- JavaScript uses `let`, `===` and `{ }`, but the ideas match Bash and Python
- `document.querySelector` selects page elements, and `innerHTML` shows what a page really contains
- `fetch` sends requests from the browser, but a cross-origin reply is blocked by CORS unless the other site allows it

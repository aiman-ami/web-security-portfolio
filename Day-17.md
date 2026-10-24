# Day 17: git diff, Bash Arguments and Status Scripts, JavaScript Basics

What I did:
- Cloned my portfolio repo into WSL and practised `git status`, `git diff` and `git restore`
- Learned Bash arguments (`$1`), the empty check (`-z`) and `exit 1`
- Wrote `status.sh` solo, then `check2.sh` solo (kept as local practice, not in the repo)
- Learned JavaScript basics in the browser DevTools console: print, variables, joining text, if/else, lists, loops, and the `document` object

Roadmap: Git (git diff) done at L2. Bash scripting done at L3. JavaScript basics done at L2 (guided, not yet solo).

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
> console.log(document.title)
> document.title = "I changed this"
```

- `let` makes a variable. No `$` is used to read it.
- `+` joins text exactly as written, so the space has to be inside the quotes
- `===` means equal to. It is the JavaScript version of `-eq` in Bash and `==` in Python.
- Lists start at index 0, the same as Python
- `for (let s of sites)` reads as "for each s in sites"
- `document` is the page I am on. Changing `document.title` changed the tab title, but only in my browser. Reloading the page restores it.

Same check in three languages:

| Language | Check if code is 200 |
|---|---|
| Bash | `if [ "$code" -eq 200 ]; then ... fi` |
| Python | `if code == 200:` |
| JavaScript | `if (code === 200) { ... }` |

Extra notes (general knowledge, not tested):
- JavaScript and Java are different languages
- Semicolons at the end of lines are optional in JavaScript because the language adds them automatically
- JavaScript that can change the page is the basis of attacks like XSS, which comes in Phase 2

## Key takeaways
- `git diff` shows unstaged changes line by line, and `git restore FILE` undoes them
- `$1` is the first argument, `-z` tests for empty, and `exit 1` stops the whole script
- `curl -s -o /dev/null -w "%{http_code}"` returns only the status code, which is useful in scripts
- Use `-eq` for numbers and `=` for text in Bash tests, and quote variables
- JavaScript uses `let`, `===` and `{ }`, but the ideas match Bash and Python
- `document` lets JavaScript read and change the page, which is why it matters for web security

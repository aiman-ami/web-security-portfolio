# Day 36: Final Three Access Control Labs, Section Complete (13 of 13)

- Lab: Method-based access control can be circumvented, solved (Practitioner)
- Lab: Multi-step process with no access control on one step, solved (Practitioner)
- Lab: Referer-based access control, solved (Practitioner)

Roadmap: A01 Broken Access Control, Access control section fully complete, 13 of 13 labs solved, none blocked.

## What I learned

### Method-based access control can be circumvented

The admin panel's promote-user action is checked for admin privilege, but only when the request uses the POST method. Captured the real promote request as admin in Burp Repeater, then swapped in a non-admin session cookie and replayed it.

- Changing the method to a deliberately invalid one, POSTX, changed the response from "Unauthorized" to "missing parameter", a useful signal that the server had stopped evaluating the role check entirely for a method it did not specifically expect, before committing to a real method swap
- Converting the request to GET and resending with the non-admin cookie and the target username succeeded, because the server's access check exists only for POST requests to this action, and simply is not applied at all when the same action arrives as GET
- One sentence: if an access check is written to guard one specific HTTP method, performing the exact same action through a different method the check never considered walks straight past it

### Multi-step process with no access control on one step

The admin panel's promote-user flow has two parts: selecting a user and role, which is checked for admin privilege, and a separate confirmation request that actually performs the promotion, which is not checked at all.

- The server appears to assume that if a confirmation request arrives, the earlier, properly-checked step must have already happened, rather than verifying permission again at the point where the action is actually carried out
- Captured the real confirmation request as admin, swapped in a non-admin session cookie, changed the target username to the non-admin account itself, and replayed it directly, skipping the first step entirely
- One sentence: an access check performed on an earlier step in a multi-step process means nothing if the step that actually performs the sensitive action is never checked itself

### Referer-based access control

The promote-user action checks the Referer header and only allows the request through if it claims to have come from the admin panel page.

- Browsing directly to the admin action's URL as a non-admin user fails, since a direct navigation carries no Referer header at all, and the check rejects an absent or wrong value
- The Referer header is set entirely by the client's browser, describing where it says it came from, and carries no actual proof of identity or authorization
- Reusing the already-captured Repeater request from the genuine admin promotion, which still carries the real Referer value from that original legitimate navigation, and only swapping the session cookie and target username, keeps the header's value intact and valid while changing who the action is performed as
- One sentence: a header the client controls and simply reports is not a safe way to verify where a request actually originated, since a captured request's own header value can be reused directly

## Key takeaways

- The Access control section is now fully complete, 13 of 13 labs, with every failure in the section ultimately coming down to the same question never being properly enforced: does this specific request, right now, actually have permission to do what it is asking
- Three different "final step" shortcuts appeared today: trusting only one HTTP method, trusting an earlier step instead of the action itself, and trusting a self-reported header instead of real verification. All three are the access control equivalent of checking someone's ID at the door but never checking it again once they are inside and asking for something more
- A consistent practical move across most of this section's harder labs: capture a legitimate, successful privileged request in Burp Repeater first, then replay it with only the session cookie and target value swapped, rather than trying to build a new request from scratch

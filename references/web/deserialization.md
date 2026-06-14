# Insecure deserialization (A08:2025)

Deserializing attacker-controlled data lets an attacker influence the **objects the app
instantiates** — and in many languages, the side effects of object construction/destruction lead
straight to **RCE**. It's high-severity and easy to miss because the dangerous call looks innocent.
CWE-502.

## Where it hides

Any place the app turns bytes from an untrusted source back into objects:
- session cookies / tokens that embed serialized state, hidden form fields, caches,
- message queues, RPC, inter-service payloads, websockets,
- file imports, API bodies, `ViewState` (.NET), Java RMI/JMX, and "remember me" blobs.

Language-specific sinks to grep:
```bash
# Java
rg -n "ObjectInputStream|readObject|XMLDecoder|readUnshared|XStream|Kryo|SnakeYAML" .
# Python
rg -n "pickle\.loads|cPickle|yaml\.load\b|jsonpickle|shelve|dill\.loads|marshal\.loads" .
# PHP
rg -n "unserialize\(" .
# .NET
rg -n "BinaryFormatter|LosFormatter|NetDataContractSerializer|TypeNameHandling|JavaScriptSerializer" .
# Node / Ruby
rg -n "node-serialize|funcster|Marshal\.load|YAML\.load\b" .
```

## How to find it

1. Identify serialized blobs in transit: Java `rO0AB...` (base64 of `\xac\xed`), PHP
   `O:8:"User":...`, Python pickle opcodes, .NET `AAEAAAD...`, signed/unsigned cookies.
2. Determine the format and whether the app **deserializes before authenticating/validating**.
3. **Detect first** with a safe probe: a malformed blob causing a deserialization error (vs a generic
   error) signals the code path. For Java, an OOB/DNS gadget (e.g. a URLDNS-style payload that triggers
   a DNS lookup) confirms *reachability without code execution* — the preferred non-destructive PoC.
4. Identify available **gadget chains** (libraries on the classpath/installed packages). `ysoserial`
   (Java), `ysoserial.net`, PHPGGC (PHP), and known Python `pickle.__reduce__` payloads generate them.

> ⚠️ On a target you don't own: stop at a **DNS/OOB callback** proving the sink deserializes your data.
> Do **not** run an RCE gadget against third-party infra. On your own app/lab, demonstrate the full
> chain so you understand the impact.

## Confirm (PoC)

- Non-destructive: an out-of-band DNS/HTTP callback fired by deserializing your crafted blob → proves
  the vulnerable sink and reachability.
- Full (own/lab): a gadget that runs `id`/sleeps/pings your collaborator → demonstrates RCE.

## How to fix

- **Don't deserialize untrusted data with native/dangerous deserializers.** This is the only robust
  fix. Replace with a **data-only format** — JSON/Protobuf parsed into known schemas (no polymorphic
  type resolution).
- If a native serializer is unavoidable:
  - **Integrity-protect** the blob (HMAC/signature) so it can't be tampered with — but signing a cookie
    doesn't help if the key leaks; prefer eliminating deserialization of user input.
  - **Restrict types** with an allowlist (Java `ObjectInputFilter`/`JEP 290`; SnakeYAML `SafeConstructor`;
    .NET avoid `BinaryFormatter` entirely — it's deprecated/removed; never enable
    `TypeNameHandling.All` in Json.NET; Python use `json` not `pickle`, or `yaml.safe_load`).
  - Run the deserializer with least privilege / in a sandbox.
- **Keep gadget-bearing libraries patched** and minimal (fewer libs on the classpath = fewer chains).
- Treat **`ViewState`/session blobs** as security-sensitive: encrypt+MAC, protect the keys (a leaked
  .NET machine key → ViewState RCE).

```python
# ❌ data = pickle.loads(request.body)            # arbitrary code execution
# ✅ data = json.loads(request.body)              # data only
#    + validate against a schema (pydantic/marshmallow)
```

## CTF angle

Deserialization challenges hand you a serialized cookie/param and a set of installed libraries; you
build a gadget chain (PHPGGC/ysoserial) to pop a shell or read the flag, or abuse a PHP `__wakeup`/
`__destruct` magic method, or a Python `__reduce__`. Identify the language + available gadgets first.

## Real-world cases

- **FoxGlove Security (2015)** — "What do WebLogic, WebSphere, JBoss, Jenkins, OpenNMS… have in common?"
  launched the Java-deserialization RCE era using the Apache Commons-Collections gadget chain (now
  bundled in `ysoserial`). Any endpoint that `readObject()`s attacker-controlled bytes is potential RCE.
  [Writeup](https://foxglovesecurity.com/2015/11/06/what-do-weblogic-websphere-jboss-jenkins-opennms-and-your-application-have-in-common-this-vulnerability/).
- **Disclosed reports:** deserialization usually surfaces as
  [TOP RCE on HackerOne](https://github.com/reddelexc/hackerone-reports/blob/master/docs/tops_by_bug_type/TOPRCE.md).

## References

[OWASP A08:2025](https://owasp.org/Top10/2025/) · WSTG-INPV-17 · [CWE-502](https://cwe.mitre.org/data/definitions/502.html) ·
[OWASP Deserialization Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Deserialization_Cheat_Sheet.html) ·
[PortSwigger: Deserialization](https://portswigger.net/web-security/deserialization) · gadgets: [ysoserial](https://github.com/frohoff/ysoserial) / [ysoserial.net](https://github.com/pwntester/ysoserial.net) / [PHPGGC](https://github.com/ambionics/phpggc).
Full bibliography: [research/web.md](../../research/web.md). Back to [web/](README.md).

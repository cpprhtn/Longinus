# 🔗 Account-takeover (ATO) chains

The bug-bounty bread-and-butter: a pile of "Lows" that compose into **someone else's account**. ATO is
almost never one bug — it's a chain across the auth boundary. Each chain below is offense-driven; the
**🛡️ seal** at the end is the single control that breaks it. Parent: [../chaining-and-impact.md](../chaining-and-impact.md).

## Chain 1 — open-redirect + OAuth `redirect_uri`

1. Find an **open redirect** (a `?next=`/`?url=` that 302s anywhere) → [../web/access-control.md](../web/access-control.md) · [../web/misconfiguration.md](../web/misconfiguration.md).
2. Find an **OAuth flow whose `redirect_uri` is loosely matched** (`startsWith`/regex without `$`, or a wildcard) → [../identity/README.md](../identity/README.md).
3. Craft an authorize URL whose `redirect_uri` passes the loose check but bounces through the open
   redirect to **your** host → the `code`/token lands on the attacker → exchange → **full ATO**.
- Boundary crossed: *unauthenticated → victim's session*. Often a "won't fix" open-redirect becomes Critical here.

## Chain 2 — self-XSS + CSRF (or cache poisoning) → delivered XSS

1. You have a **self-XSS** (only fires in your own session — usually rated Low) → [../web/xss.md](../web/xss.md).
2. **Deliver it to the victim:** a **CSRF** that writes your payload into *their* stored field
   ([../web/csrf.md](../web/csrf.md)), or **web cache poisoning** that serves it to every user
   ([../web/request-smuggling-and-desync.md](../web/request-smuggling-and-desync.md)).
3. Payload steals the session cookie / makes an authenticated state-change → **ATO**.

## Chain 3 — user enumeration + weak reset token + no rate limit → *mass* ATO

1. **Enumerate** valid accounts (login/reset/signup oracle) → [../web/auth-and-session.md](../web/auth-and-session.md).
2. The **password-reset token** is predictable (sequential, timestamp-seeded, short) or reusable, and the
   endpoint isn't rate-limited → brute/predict it.
3. Reset at scale → **mass account takeover**.

## Chain 4 — subdomain takeover + loosely-scoped cookie/OAuth

1. Find a **dangling DNS** record you can claim (dead CNAME to an unclaimed SaaS) → [../recon/active-recon.md](../recon/active-recon.md).
2. The app sets cookies on `*.example.com` or trusts `*.example.com` as an OAuth/redirect origin →
   your claimed subdomain receives the victim's cookie/token → **ATO** → [../identity/README.md](../identity/README.md).

## Real-world cases

Disclosed ATO chains (PoCs): [TOP ACCOUNT-TAKEOVER](https://github.com/reddelexc/hackerone-reports/blob/master/docs/tops_by_bug_type/TOPACCOUNTTAKEOVER.md)
· [TOP OAUTH](https://github.com/reddelexc/hackerone-reports/blob/master/docs/tops_by_bug_type/TOPOAUTH.md)
— the majority are exactly Chains 1–4. See also [../identity/README.md](../identity/README.md) "Real-world cases."

## Prove each link

Per-link PoC ([../severity-and-triage.md](../severity-and-triage.md)). Demonstrate the *token/cookie
crossing accounts* on **your own** test accounts only — never capture a real user's token
([../authorization-and-scope.md](../authorization-and-scope.md)).

## 🛡️ Defensive seal (what breaks the chain)

- **Exact-match `redirect_uri` allowlist + mandatory `state`/PKCE** (kills Chain 1 even with an open redirect).
- **`SameSite` cookies + anti-CSRF tokens** (kills the delivery step of Chain 2).
- **Single-use, time-limited, high-entropy reset tokens + rate limit + no user-enumeration oracle** (Chain 3).
- **Scope cookies to the exact host; exact-match trusted origins; monitor dangling DNS** (Chain 4).

→ enforce these per [../enforce-forward.md](../enforce-forward.md) (Identity/Web rows) · [../templates/validation-layer.md](../templates/validation-layer.md).

Back to [chain catalog](../chaining-and-impact.md) · [tree](../00-map.md).

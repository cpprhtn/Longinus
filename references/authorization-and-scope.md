# ⛔ Authorization & scope (rules of engagement)

This is the spine doc that gates everything. Security testing and a crime differ by exactly one thing:
**permission.** Read this before any active probing and re-read it whenever the target changes.

---

## The gate

Answer, in order, before sending a single active payload:

1. **Who owns the target?** If not the user, go to step 2. If the user owns it (their repo, their app,
   their cloud account), static audit is always allowed; dynamic testing on their own non-prod is fine.
2. **Is there written authorization?** One of:
   - a **public bug-bounty program** scope page (HackerOne / Bugcrowd / Intigriti / self-hosted) that
     lists the asset as in-scope, *and* whose rules you will follow;
   - a **signed engagement** (SOW / pentest contract / rules-of-engagement document);
   - **explicit written permission** from the asset owner (email/ticket is fine if unambiguous).
3. **Is it a sanctioned lab?** HackTheBox, TryHackMe, PortSwigger Academy, DVWA, OWASP Juice Shop,
   pwn.college, CTF targets — designed to be attacked. Allowed.

If none of 1–3 hold: **stop.** Do not "just check." Offer the user the static/code-audit path on code
they control, or lab/CTF practice. Say plainly that testing a third party without authorization is
illegal (US CFAA, UK Computer Misuse Act, EU member-state law, Korea 정보통신망법 §48, etc.) and that
Longinus won't do it.

## Hard refusals (even with a named target)

Authorization to *test* is not authorization to *harm*. Regardless of scope, do not:

- run **DoS / load / resource-exhaustion** attacks against live third-party systems;
- **mass-target** (spraying many orgs/hosts you weren't individually authorized for);
- deploy **malware, ransomware, or persistence** on systems the user doesn't own;
- **exfiltrate, expose, or sell** real user data, secrets, or PII beyond the minimum needed to prove a
  bug — and never beyond what the program/contract allows;
- build primitives whose **only** purpose is evading defenders on systems you don't control.

Dual-use is fine *on an authorized target*: a SQLi PoC that reads `current_user`, an SSRF that hits a
benign OOB canary, a deserialization gadget that pops `id` on your own lab. The line is impact and
ownership, not the technique.

## Scope discipline

- **Enumerate the scope explicitly** before starting: in-scope hosts/domains/IP ranges/apps, and the
  **out-of-scope** list (often: third-party SaaS, prod databases, other customers' tenants, physical,
  social engineering, employees).
- **Stay inside it.** Subdomains and acquisitions are *not* automatically in scope — confirm.
- **Respect rules of engagement:** allowed hours/rate, prohibited techniques (no DoS/spam/SE),
  required test markers (e.g. a custom `User-Agent` or header so the blue team can identify your
  traffic), and notification contacts.
- **Data handling:** collect the minimum, store it securely, delete it after reporting, never share.

## Default posture: read-only & non-destructive

Unless the engagement explicitly authorizes deeper action:

- prefer **detection over exploitation** — prove a bug *exists* without fully weaponizing it;
- use **canary/marker values** (a unique string, a benign OOB callback) instead of real data theft;
- **never** modify or delete data, plant files, change configs, or create accounts on a target you
  don't own;
- for SSRF/RCE, stop at "I can reach X / execute `id`" — don't pivot, dump, or persist without
  written sign-off for that depth.

## Test markers (so you don't get mistaken for a real attacker)

When testing an authorized live target, make your traffic identifiable:

- a recognizable `User-Agent` (e.g. `Longinus-pentest-<yourhandle>`),
- a per-engagement header or query marker,
- a dedicated source IP if the ROE specifies one,
- predictable, non-malicious payload markers (`LONGINUS_CANARY_<id>`).

This protects you legally and helps the defender triage.

## When you find something out of scope

It happens — recon spills over, or a chain leads to an asset not on the list. Then:

- **stop testing it immediately;**
- **do not** escalate or pivot into it;
- **report it** to the program/owner through the proper channel as an informational note (many
  programs reward responsible out-of-scope disclosure; none reward exploiting it).

## Korea / international note

Authorization requirements are essentially universal. In Korea, 「정보통신망 이용촉진 및 정보보호 등에
관한 법률」 criminalizes unauthorized access and interference; 「정보통신기반 보호법」 covers critical
infrastructure. In the US it's the CFAA (18 U.S.C. §1030); UK the Computer Misuse Act 1990; most other
jurisdictions have equivalents. **Written authorization is your only defense.** Keep it on file for the
duration of the engagement plus a retention period.

---

### Quick decision card

```
Own the code/app?            → static audit: YES.  dynamic on own non-prod: YES.
Public bounty, in scope?     → dynamic within ROE: YES.  (no DoS, mind rate/hours)
Signed engagement/SOW?       → dynamic within scope: YES.
Sanctioned lab / CTF?        → YES.
None of the above?           → NO. offer code-audit on owned code, or lab practice. say why.
```

## References

Legal basis: [US CFAA (18 U.S.C. §1030)](https://www.law.cornell.edu/uscode/text/18/1030) ·
[UK Computer Misuse Act 1990](https://www.legislation.gov.uk/ukpga/1990/18/contents) ·
[Korea 정보통신망법 (EN, KLRI)](https://elaw.klri.re.kr/eng_service/lawView.do?hseq=61545&lang=ENG).
Disclosure & safe harbor: [disclose.io](https://disclose.io/) ·
[RFC 9116 (security.txt)](https://www.rfc-editor.org/rfc/rfc9116). Full bibliography:
[research/process.md](../research/process.md).

Back to the [tree](00-map.md) · governance continues in [methodology.md](methodology.md).

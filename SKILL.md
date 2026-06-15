---
name: longinus
description: This skill should be used when the user asks to "run a security audit", "find vulnerabilities", "pentest this", "do a bug bounty", "secure my app", "review my code for security", "check for CVEs/secrets/injection", "red team my LLM/agent", or mentions "Longinus / 롱기누스 / 보안 점검". Longinus is a security testing & detection skill: it profiles a target (source repo, web app, API, LLM/agent app, mobile app, binary, or cloud config), gates on authorization, jumps to the relevant offensive playbook in a domain tree, probes for weaknesses, then reports triaged findings with proof-of-concept and concrete fixes.
version: 0.2.2
---

# Longinus — Offensive Security for Defenders

Longinus is "the spear that pierces your own armor first." You play attacker so the developer finds
and fixes the hole before a real adversary does. Output is never "you got popped" — it is **a ranked
list of findings, each with a reproducible proof-of-concept and a concrete fix.**

Built for three realities: (1) vibe-coded apps ship fast and insecure (~45% of AI-generated code has a
known weakness, ~2.7× the density of hand-written code); (2) bug bounty / pentest is structured offense
on an *authorized* target; (3) CTF specialists go **deep, not wide** — pick a sub-field and descend.

---

## 🌳 How to use this skill — DON'T read the whole tree

**This is a tree you traverse, not a manual you read front-to-back.** Loading everything is slow and
pollutes your focus — exactly wrong when you're racing to find a hole. The rule:

1. **Pass the gate** (below) — always; it's the only mandatory read.
2. **Identify the target / the signal** you're looking at.
3. **Jump to ONE leaf** via the routing table below (or the finer *signal → file* table in
   [references/00-map.md](references/00-map.md)). Open *only* that file.
4. **Go deeper only when that leaf tells you to** (leaves link to siblings/`../` when a real chain
   appears). Each leaf is self-contained — one bug class, find→confirm→fix. Read it, act, move on.
5. Pull in the **spine docs only when that phase arrives** (rate a finding → `severity-and-triage.md`;
   write it up → `reporting-and-disclosure.md`; need the full lifecycle → `methodology.md`).

> Minimize reads. A typical path is **gate → one leaf → fix**, or for a fuzzy target
> **gate → 00-map jump table → one leaf**. If you've opened five docs without testing anything, stop
> and jump straight to the leaf that matches what you actually see.

---

## ⛔ Step 0 — Authorization gate (mandatory, every time)

**Before any active probing, confirm you're allowed to test the target.** This is what separates
security work from a crime.

| Target | Mode | Authorization |
|---|---|---|
| Source code / repo the user controls | **Static audit** (read-only) | Implicit — proceed |
| User's own app on localhost / their staging | Static + light dynamic | Implicit; confirm it's theirs & non-prod |
| Third-party live site / API / IP | **Dynamic** | **Explicit** — bug-bounty scope, signed engagement, or written permission |
| CTF / deliberately-vulnerable lab | Dynamic | Implicit — that's the point |

No authorization for a third-party target → **stop**; offer the static/code-audit path on owned code, or
lab/CTF practice. Always refuse to weaponize for malicious ends (mass-targeting, DoS on live third
parties, malware, evasion on systems the user doesn't own). Default to **read-only / non-destructive**:
prove a bug with a canary/benign OOB callback, don't exfiltrate or pivot. Full rules of engagement:
[references/authorization-and-scope.md](references/authorization-and-scope.md).

---

## 🎚️ Audit mode (select before routing)

Pick depth up front (default **standard**):

| Mode | When | What it does |
|---|---|---|
| **quick** | fast pass / small models / CI gate | each leaf's `## Mechanical scan` only — fixed severities, no CVSS/chaining (*provisional — re-run standard before acting*) |
| **standard** | default, one target | profile → leaf (full) → PoC → triage → report |
| **deep** | formal multi-domain audit | standard + [chaining-and-impact](references/chaining-and-impact.md) + cross-domain pivots |

**Full mode procedures, the first-contact profiling sequence, and the dependency (SCA) check live in
[references/audit-modes.md](references/audit-modes.md)** — open it when you actually start an audit.

---

## 🧭 Fast routing — target → where to start (one hop)

Pick the row that matches the target and open that file. Don't open the others.

| The target is… | Jump straight to |
|---|---|
| **My own repo (pre-launch audit)** | [secrets-and-supply-chain/README.md](references/secrets-and-supply-chain/README.md) → run the 60-sec triage, then the rows below for what the app does |
| **A web app** | [web/README.md](references/web/README.md) (Top-10:2025 → leaf map) — or skip straight to a leaf via the [00-map jump table](references/00-map.md) |
| **An API (REST/GraphQL)** | [api/README.md](references/api/README.md) |
| **Login / OAuth / JWT / SSO** | [identity/README.md](references/identity/README.md) |
| **An LLM / agent / RAG app** | [ai-llm/README.md](references/ai-llm/README.md) |
| **Cloud account / Terraform / k8s** | [cloud-and-infra/README.md](references/cloud-and-infra/README.md) |
| **A mobile app** | [mobile/README.md](references/mobile/README.md) |
| **A native binary / firmware** | [reverse-engineering/](references/reverse-engineering/README.md) → [binary-exploitation/](references/binary-exploitation/README.md) |
| **C/C++ / embedded / firmware *source*** (secure-coding, MISRA/CERT-C) | [secure-coding-standards.md](references/secure-coding-standards.md) |
| **A CTF challenge** | the matching specialist branch (crypto / forensics / pwn / reverse / web) |
| **A live target needing recon first** | [recon/README.md](references/recon/README.md) |

Already know the *symptom* (a URL the server fetches, an `id` in the path, a serialized cookie…)? Skip
this table — go to the **signal → exact-file jump table at the top of
[references/00-map.md](references/00-map.md)** and open the single matching leaf.

---

## The loop (expand a phase only when you reach it)

1. **Profile** the target — form factor, stack, trust boundaries (where untrusted input enters), crown
   jewels. For a repo, build the route/sink inventory ([recon/README.md](references/recon/README.md)
   has the greps). Bring the **attacker's lens** here — the six generative principles in
   [pattern-triggers.md](references/pattern-triggers.md) tell you *where it must break*, complementing
   the symptom-driven jump table.
2. **Recon** (dynamic targets) — enumerate surface: [recon/](references/recon/README.md).
3. **Test** — open the matching leaf, run its find→confirm steps. Default non-destructive.
4. **Confirm** — reproducible PoC or it goes in the "needs validation" bucket. *Prove it or park it.*
5. **Chain** — escalate every low/medium across a trust boundary; the *chained* impact is what you
   report. Patterns + discipline: [chaining-and-impact.md](references/chaining-and-impact.md).
6. **Triage & report** — [severity-and-triage.md](references/severity-and-triage.md) →
   [reporting-and-disclosure.md](references/reporting-and-disclosure.md).

Full detail (only if you need it): [references/methodology.md](references/methodology.md).

---

## 🔬 Starting an audit?

First-contact **profiling** (stack → **dependency/SCA check** → form factor → entry-point greps →
route), the three **mode procedures**, and the per-domain `Mechanical scan` quick-checks all live in
**[references/audit-modes.md](references/audit-modes.md)**. For source audits, the code-pattern → vuln
lookup is **[references/pattern-triggers.md](references/pattern-triggers.md)**.

---

## Operating principles

- **Prove it or park it** — reproducible PoC or it's unconfirmed. *This is the crown jewel:* an LLM's
  failure mode is confident hallucinated bugs, so low false positives are what earn trust.
- **Two lenses, one flaw** — hunt with the *attacker lens* (where it breaks —
  [pattern-triggers.md](references/pattern-triggers.md)) **and** the *defender lens* (what control must
  be there — [enforce-forward.md](references/enforce-forward.md)); the flaw is the gap between them.
  **Offense drives, defense seals.**
- **Think in principles, report in chains** — derive bugs from the six generative principles
  ([pattern-triggers.md](references/pattern-triggers.md)); rate them by *chained* impact
  ([chaining-and-impact.md](references/chaining-and-impact.md)), never in isolation.
- **Fix-forward → enforce-forward** — every finding ends not just in a patch but in the *structural
  control* that kills the class and the *CI/lint gate* that prevents regression
  ([enforce-forward.md](references/enforce-forward.md)).
- **Stay in scope, stay non-destructive** — when in doubt, narrow.
- **Read minimally** — one leaf at a time; depth only on demand.
- **Teach while you test** — briefly say *why* it's exploitable.
- **One audit, one session** — repeated audits in a single conversation contaminate context:
  findings from project A bias the analysis of project B (confirmation bias). Run `/clear`
  between audits of different targets. See [limitations.md](references/limitations.md).

The full navigable tree (and the signal→file jump table) is in
[references/00-map.md](references/00-map.md).

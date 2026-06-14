# 🔗 Chaining & impact amplification — the scanner-vs-hacker line

A scanner reports bugs **in isolation**. A hacker reports **what they unlock together**. The severity
that matters is the *chained* impact, not the in-isolation rating — and seeing the chain is the single
thing that separates expert output from a tool dump. A page of "Lows" that compose into account
takeover **is a Critical**; only a human-grade attacker rates it that way.

This is promoted from a step in the loop to a first-class discipline on purpose. The lens that finds
the links is [attacker-mindset.md](attacker-mindset.md); the place chained impact gets re-scored is
[severity-and-triage.md](severity-and-triage.md); the loop hand-off is
[methodology.md](methodology.md) §6.

---

## The core move

**For every confirmed low/medium, ask: "what does this unlock across a trust boundary?"** Then escalate
relentlessly. Four axes to push along — a finding that moves you up *any* of them is worth more than its
isolated CVSS suggests:

| Axis | Push from → toward |
|---|---|
| **Privilege** | anonymous → user → admin → service/root |
| **Scope** | self → another user → all users → all tenants → the infra |
| **Capability** | read → write → execute → persist |
| **Plane** | application → host → network → **cloud control plane** |

The highest-value findings are the ones that **jump planes** (web→cloud, app→infra, model→web sink) —
that's where a "medium" web bug becomes a company-ending one.

## Canonical chains (each link → its leaf)

These are the patterns that recur in real disclosures and CTF finals. Use them as templates, not limits.

| Starter(s) | Composes into | Leaves |
|---|---|---|
| open-redirect **+** OAuth `redirect_uri`/`state` weakness | code/token theft → **account takeover** | [identity](identity/README.md) · [web/access-control](web/access-control.md) |
| **SSRF** → cloud metadata (IMDS) → role creds → IAM priv-esc | **full cloud-account takeover** | [web/ssrf](web/ssrf.md) · [cloud-and-infra/cloud-iam](cloud-and-infra/cloud-iam.md) |
| self-XSS **+** CSRF *or* cache poisoning | delivered/stored **XSS** → session theft → ATO | [web/xss](web/xss.md) · [web/csrf](web/csrf.md) · [web/request-smuggling-and-desync](web/request-smuggling-and-desync.md) |
| IDOR/BOLA **+** sequential IDs **+** no rate limit | **full-dataset exfiltration** | [web/access-control](web/access-control.md) · [api](api/README.md) |
| verbose error / exposed `.git` / source map → leaked secret or logic | **auth bypass / RCE** | [web/misconfiguration](web/misconfiguration.md) · [secrets/secret-detection](secrets-and-supply-chain/secret-detection.md) |
| arbitrary file upload **+** path traversal | webshell → **RCE** | [web/file-upload-and-path](web/file-upload-and-path.md) |
| prototype pollution **+** a sink gadget | **RCE / auth-bypass / XSS** | [web/prototype-pollution](web/prototype-pollution.md) |
| **SQLi** + DB privilege | file write / `xp_cmdshell` → **RCE**; or creds dump → ATO | [web/injection](web/injection.md) |
| CRLF / header injection → **web cache poisoning** | **mass** stored XSS / response splitting | [web/request-smuggling-and-desync](web/request-smuggling-and-desync.md) · [web/injection](web/injection.md) |
| subdomain takeover **+** loosely-scoped cookie/OAuth | cookie/token theft → **ATO** | [recon/active-recon](recon/active-recon.md) · [identity](identity/README.md) |
| user enumeration **+** weak reset token **+** no rate limit | **mass account takeover** | [web/auth-and-session](web/auth-and-session.md) |
| known-CVE / hallucinated dependency reachable | **RCE / supply-chain foothold** | [secrets/dependency-supply-chain](secrets-and-supply-chain/dependency-supply-chain.md) |
| **indirect prompt injection + a tool/agent + an exfil channel** = the *lethal trifecta* | data theft / action injection / tool-arg → web sink (SSRF/SQLi/RCE) | [ai-llm/prompt-injection](ai-llm/prompt-injection.md) · [ai-llm/agentic-and-mcp](ai-llm/agentic-and-mcp.md) |

## Cross-domain pivots (where the big numbers come from)

- **web → cloud:** SSRF/RCE on an instance → metadata creds → IAM → other services & tenants.
- **app → infra:** RCE in a container → escape (privileged/`hostPath`/socket) → host → cluster.
  See [cloud-and-infra/iac-and-containers](cloud-and-infra/iac-and-containers.md).
- **model → app:** prompt injection → tool call → a *classic* web sink. The model is just the new
  caller; the sink is the same one in [web/](web/README.md).
- **identity → everything:** one ATO is rarely the end — pivot to the victim's data, their org, their
  integrations, their tokens.

## Discipline (the trust firewall still holds)

- **Prove it or park it — per link.** A chain with one hypothetical hop is a *hypothesis*. Report the
  proven prefix as a finding and the credible continuation **explicitly labeled** as unproven —
  never present a theoretical chain as fact ([severity-and-triage.md](severity-and-triage.md)).
- **Stay non-destructive across the chain.** On a target you don't own, demonstrate *reachability* and
  stop — "IMDS is reachable via this SSRF," not a looted bucket. Prove the full chain only on your own
  infra/lab. See the [authorization gate](authorization-and-scope.md).

## Re-rate, then lead with the chain

Feed the composed impact back into [severity-and-triage.md](severity-and-triage.md) and **report the
chain first, the parts second**: *"Three individually-Medium issues compose into unauthenticated full
account takeover — rated Critical."* That sentence is the deliverable a scanner can never write.

## CTF angle

Multi-stage challenges **are** chains by design: info leak → primitive → escalation → flag (pwn), or
recon → SSRF → internal endpoint → flag (web). The methodology is identical to a real engagement; only
the authorization context differs — so practice the escalation reflex here.

## References

Chaining is the operational expression of the [attacker-mindset](attacker-mindset.md) lenses and the
**Chain** step of [methodology.md](methodology.md); scoring lives in
[severity-and-triage.md](severity-and-triage.md). Real disclosed chains are catalogued via the
per-leaf "Real-world cases" footers and [research/process.md](../research/process.md).

Back to the [tree](00-map.md).

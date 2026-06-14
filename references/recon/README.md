# 🔭 recon/ — attack-surface discovery

You can't test what you can't see. Recon turns "a target" into a concrete map of hosts, endpoints,
parameters, technologies, and humans. More (accurate) surface = more places a bug can hide. For
bug-bounty and pentest this is *the* differentiator; the hunters who find bugs others miss usually
found **surface** others missed.

> ⛔ **Gate first.** Passive recon (public data only) is generally safe and legal. **Active** recon
> sends traffic to the target and requires authorization — see
> [../authorization-and-scope.md](../authorization-and-scope.md). Mind program rate limits and hours.

## Branch map

| Leaf | Use it for |
|---|---|
| [passive-recon.md](passive-recon.md) | OSINT-only surface: subdomains, certs, archives, dorking, leaked data — no packets to the target |
| [active-recon.md](active-recon.md) | Probing the live target: port/service scan, content & directory discovery, parameter discovery, fuzzing, tech fingerprinting |
| [osint.md](osint.md) | People/org/infra intelligence, breach data, document metadata, code/secret leaks |

## Code-as-target recon

For a **source-code audit** (your own repo), recon means building the surface from the inside:

- **Routes/endpoints:** route tables, controller/handler files, GraphQL schema, gRPC protos, webhook
  registrations, cron/queue consumers.
- **Input points:** every read of `req`/params/headers/cookies/body/query, file uploads, message
  payloads, env, third-party callbacks.
- **Sinks (grep these):** `exec`/`spawn`/`system`, SQL string concatenation, `eval`/`Function`,
  template renderers, `pickle.loads`/`yaml.load`/`unserialize`/`ObjectInputStream`, `innerHTML`/
  `dangerouslySetInnerHTML`, outbound `fetch`/`requests.get(<user-controlled>)`, path joins with user
  input, redirect/`Location` with user input.
- **Trust map:** which routes are unauthenticated, which require which role, where authZ is enforced
  (and where it's *assumed*).

Fast starting greps (tune per language):
```bash
# dangerous sinks
rg -n "exec\(|child_process|os\.system|subprocess|Runtime\.exec|eval\(|new Function" .
rg -n "pickle\.loads|yaml\.load\b|unserialize|ObjectInputStream|Marshal\.load" .
rg -n "innerHTML|dangerouslySetInnerHTML|v-html|render_template_string" .
# raw SQL concatenation
rg -n "SELECT .*\+|query\(.*\$\{|format\(.*SELECT|f\".*SELECT" .
# outbound requests with user input (SSRF candidates)
rg -n "requests\.(get|post)\(|axios\.|fetch\(|http\.get\(|urllib" .
# secrets (also see secrets-and-supply-chain/)
rg -n "(api[_-]?key|secret|token|password|BEGIN .*PRIVATE KEY)" -i .
```

## Recon workflow (dynamic target)

A layered, mostly-automated pipeline (per the 2025 bug-bounty methodology):

```
1. Seed scope            → root domains / IP ranges / apps (confirm in scope!)
2. Subdomain enum        → passive (subfinder, crt.sh, amass passive) + active (resolve, brute, permute)
3. Liveness & fingerprint→ httpx (status, title, tech, CDN), wappalyzer-style detection
4. Content discovery     → katana/gau/waybackurls, ffuf/feroxbuster dir & file brute
5. Param & JS analysis   → extract params, secrets, and routes from JS; linkfinder-style
6. Vuln surface tagging  → tech-version → known CVEs; nuclei templates; takeover checks
7. Prioritize            → feed the most interesting endpoints into the web/api playbooks
```

The point isn't to run every tool — it's to build a **prioritized endpoint list** to hand to the
testing phase. Persistent/continuous recon (re-running on a schedule to catch new assets) is where
long-term bounty value lives.

See the [tooling catalog](../tooling/README.md) for the specific tools and
[../methodology.md](../methodology.md) for how recon feeds the rest of the engagement.

## References

The layered surface-mapping flow above follows the **OWASP WSTG Information Gathering** phase and the
**PTES Intelligence Gathering** stage, plus modern continuous bug-bounty recon practice.
[WSTG (Information Gathering)](https://owasp.org/www-project-web-security-testing-guide/) ·
[PTES](http://www.pentest-standard.org/) · [NIST SP 800-115](https://csrc.nist.gov/pubs/sp/800/115/final) ·
[OSINT Framework](https://osintframework.com/). Tool URLs & full bibliography: [research/recon.md](../../research/recon.md). Back to [tree](../00-map.md).

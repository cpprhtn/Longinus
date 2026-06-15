# Why Longinus exists

↑ back to [README](../README.md) · doc map: [who-its-for](who-its-for.md) · [ethics](ethics.md) · [usage](usage.md)

Three trends collide:

1. **Vibe coding outran security.** People ship AI-generated apps in days with no security pass.
   Independent studies put a known weakness in ~**45%** of AI-generated code, at ~**2.7×** the
   vulnerability density of hand-written code — hallucinated/typosquatted dependencies, hard-coded
   secrets, missing authorization, CVEs re-introduced from stale training data. A scan of ~5,600
   "vibe-coded" apps found 2,000+ vulnerabilities and 400+ exposed secrets.
2. **Bug bounty is structured offense** on an *authorized* target — and increasingly people want to
   run an AI through that loop to surface bugs.
3. **Real hackers specialize.** CTF and pro pentest split into sub-fields — web, pwn (binary
   exploitation), reverse engineering, crypto, forensics, OSINT, cloud, mobile, AI/ML. Depth, not
   breadth, is what finds the hard bugs.

Longinus encodes all three: a **defensive audit** for your own code, an **authorized offense**
workflow for bounty/pentest, and a **specialist tree** so you can go as deep as the target needs.

> Sources & numbers (the full evidence with links): [research/rationale.md](../research/rationale.md).

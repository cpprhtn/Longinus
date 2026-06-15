# OSINT (open-source intelligence)

People, organizations, and infrastructure leave public traces. OSINT assembles them into context that
sharpens every other phase — and it's a CTF category in its own right. It's almost entirely passive,
but **acting** on what you find (logging in, contacting people, probing infra) still needs
authorization and good judgment. Don't cross into stalking/harassment; stay on the asset, not the
person.

## Organization & infrastructure

- **Footprint:** ASN/IP ranges (`whois`, bgp.he.net), domains/subdomains (see
  [passive-recon.md](passive-recon.md)), cloud tenancy (S3/GCS/Azure naming), email infra (MX/SPF/DMARC
  records — also useful for spoofing-risk assessment).
- **Tech & vendors:** job postings and engineering blogs reveal the stack ("we use Rails + Postgres +
  AWS + Auth0"), which scopes your testing. StackShare/BuiltWith corroborate.
- **Acquisitions & brands:** subsidiaries expand scope — but confirm each is in scope before testing.

## People (for authorized social-engineering assessment only)

- **Username/email convention:** derive `first.last@`, `flast@` patterns from public profiles; useful
  for understanding account structure. *Do not* phish or contact targets without explicit, written
  social-engineering authorization — most programs forbid SE entirely.
- **Roles:** identify admins/devops/security contacts (for the disclosure channel, primarily).
- Tools: theHarvester, Hunter.io, LinkedIn (manual), GitHub commit emails (`git log` of public repos).

## Code & secret leaks

The single highest-yield OSINT category for modern targets:

- **Public repos** of the org *and* its developers (personal accounts often leak work code). Search for
  internal hostnames, API keys, `.env`, CI configs, private-key blocks.
- **Git history:** secrets deleted from `HEAD` survive in history — `trufflehog`/`gitleaks` over the
  full history, not just the tip. (See [../secrets-and-supply-chain/secret-detection.md](../secrets-and-supply-chain/secret-detection.md).)
- **Paste sites, gists, CI logs, Docker images, npm/PyPI packages** published by the org.
- **Exposed buckets** and object storage tied to the brand.

> ⚠️ Finding a live secret ≠ permission to use it. Validate *that* it's live with the least-privilege
> read possible (or just report it), and disclose immediately — don't pivot with stolen creds unless
> the engagement authorizes it.

## Document & media metadata

Public files leak more than their content:
- **EXIF/metadata:** authors, software versions, GPS, internal paths (`exiftool`, metagoofil, FOCA).
- **Geolocation/chronolocation** of images (a staple of OSINT CTF challenges): landmarks, signage,
  shadows/sun angle, license plates, reflections, language/scripts → narrow location and time.

## Breach & exposure intelligence

- **HIBP / breach datasets:** which corporate emails/domains appear in breaches — *informational*
  posture signal. Never replay leaked passwords against a live target without explicit credential-
  testing authorization.
- **Shodan/Censys monitors:** standing alerts for the org's exposed services and certs.

## OSINT in CTF

CTF OSINT challenges give you a handle, image, or snippet and ask you to pivot:
- reverse-image search, username-cross-reference across platforms (`sherlock`, `maigret`),
- metadata extraction, geolocation/chronolocation,
- Wayback/cache to recover deleted content,
- correlating timestamps, avatars, and writing style across accounts.

Same skills, sanctioned target.

## Produce

A context dossier that feeds the rest of the engagement:
```
- infra footprint (ASN/IP/domains/cloud) — confirmed scope tagged
- tech stack & versions (→ CVE mapping)
- leaked secrets / repos / buckets (highest priority, disclose fast)
- disclosure & security contacts (security.txt, etc.)
- people/role map (only for authorized SE; otherwise just the disclosure contact)
```

## References

[OSINT Framework](https://osintframework.com/) (tool index) ·
[PTES — Intelligence Gathering](http://www.pentest-standard.org/) ·
[OWASP WSTG (Information Gathering)](https://owasp.org/www-project-web-security-testing-guide/) ·
[Have I Been Pwned](https://haveibeenpwned.com/) · code/secret leaks →
[../secrets-and-supply-chain/secret-detection.md](../secrets-and-supply-chain/secret-detection.md).
Tool URLs & full bibliography: [research/recon.md](../../research/recon.md).

Tools: [../tooling/README.md](../tooling/README.md). Back to [recon/](README.md).

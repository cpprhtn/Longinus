# Passive recon (no packets to the target)

Gather surface from third-party/public sources only. You never touch the target's infrastructure, so
this is the safe-and-legal first step — though you must still respect scope when you act on what you
find. Goal: a broad, deduplicated asset list before you send a single active probe.

## Subdomain & asset enumeration (passive sources)

- **Certificate transparency:** `crt.sh`, `censys.io`, `cert.sh` — every TLS cert leaks hostnames.
  ```bash
  curl -s "https://crt.sh/?q=%25.example.com&output=json" | jq -r '.[].name_value' | sort -u
  ```
- **Passive subdomain tools:** `subfinder -d example.com`, `amass enum -passive -d example.com`,
  `assetfinder --subs-only example.com`. They aggregate dozens of OSINT sources (CT logs, DNS
  datasets, search engines, threat-intel feeds).
- **DNS datasets / aggregators:** SecurityTrails, Shodan, Censys, FOFA, ZoomEye — historical DNS,
  open ports seen by *their* scanners (not yours), tech fingerprints.
- **ASN / IP space:** find the org's ASN (`whois`, bgp.he.net) to discover owned IP ranges and more
  hostnames — but only test ranges you've confirmed are in scope.

## Historical content (the Wayback goldmine)

Old URLs reveal removed-but-still-live endpoints, parameters, and forgotten dev paths:
```bash
gau example.com            # getallurls — Wayback + Common Crawl + URLScan + OTX
waybackurls example.com    # Wayback Machine URLs
```
Mine the output for parameters (`?id=`, `?url=`, `?file=`, `?redirect=`), API paths, and old admin
panels. Filter to interesting extensions and dedupe params (`uro`, `qsreplace`).

## Search-engine dorking (Google/GitHub/Shodan)

- **Google dorks:** `site:example.com ext:env`, `site:example.com inurl:admin`,
  `site:example.com intitle:"index of"`, `"example.com" filetype:sql`. Surfaces exposed files,
  directory listings, login pages, leaked docs.
- **GitHub dorks:** search the org and developer accounts for `example.com` near `api_key`, `secret`,
  `password`, `BEGIN RSA PRIVATE KEY`, internal hostnames. Hard-coded secrets in public repos are one
  of the highest-yield passive finds. (Deeper: [../secrets-and-supply-chain/secret-detection.md](../secrets-and-supply-chain/secret-detection.md).)
- **Shodan/Censys queries:** `ssl.cert.subject.CN:"example.com"`, `org:"Example Inc"`,
  `http.favicon.hash:<hash>` (favicon hashing clusters an org's hosts and finds origin IPs behind a
  CDN/WAF).

## Tech & metadata fingerprinting (passive)

- **Favicon hashing** to identify the framework/product and find duplicate/origin servers.
- **Wappalyzer / BuiltWith** data, HTTP archive (HAR) datasets, `securityheaders.com` reports.
- **Document metadata:** public PDFs/Office files leak usernames, software versions, internal paths
  (FOCA/metagoofil-style extraction). Useful for username conventions and phishing-adjacent intel
  (report it; don't phish without explicit authorization).

## Leaks & exposure

- **Breach data:** Have I Been Pwned (which domains/emails appear in breaches) for *informational*
  context — never reuse leaked credentials against a live target unless the engagement explicitly
  authorizes credential testing.
- **Public buckets / storage:** open S3/GCS/Azure blobs tied to the org (grayhatwarfare-style search).
- **Paste sites / public repos / CI logs:** leaked configs, tokens, internal URLs.

## What to produce

A deduplicated, prioritized asset inventory:
```
- root domains & confirmed in-scope ranges
- subdomains (resolved/unresolved tagged)
- historical & current endpoints + parameters of interest
- tech stack & versions per host (→ map to known CVEs later)
- exposed files / buckets / repos / secrets (highest priority)
```

Feed this into [active-recon.md](active-recon.md) (to resolve/probe) and the testing playbooks. Keep
everything in scope; passive *discovery* of an out-of-scope asset is fine, but **don't** start probing
it — note it and move on.

## References

[OWASP WSTG (Information Gathering)](https://owasp.org/www-project-web-security-testing-guide/) ·
[PTES — Intelligence Gathering](http://www.pentest-standard.org/) ·
[crt.sh](https://crt.sh/) (CT logs) · [OSINT Framework](https://osintframework.com/) ·
[Have I Been Pwned](https://haveibeenpwned.com/). Tool URLs & full bibliography:
[research/recon.md](../../research/recon.md). Tools: [../tooling/README.md](../tooling/README.md).
Back to [recon/](README.md).

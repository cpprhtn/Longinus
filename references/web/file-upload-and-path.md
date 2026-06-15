# File upload, path traversal & file inclusion

Two intertwined problems: **getting a malicious file in**, and **reaching files you shouldn't** via
attacker-controlled paths. Together they yield RCE, source/secret disclosure, and SSRF. CWE-434
(upload), CWE-22 (path traversal), CWE-98 (file inclusion), CWE-29 (zip-slip).

## Mechanical scan

> **Quick mode only.** Run these greps, apply skip conditions, report matches.
> No further analysis needed in quick mode.

**STEP 1 — Path traversal candidates**
```bash
rg -n "path\.join\(.*req\.|open\(.*req\.|readFile\(.*req\.|fs\.\w+\(.*req\." .
```
- **SKIP if:** `realpath()` or `resolve()` is checked against base directory in the same handler
- **SKIP if:** path contains `/test/`
- **FINDING if not skipped:** Type: Path Traversal | Severity: High | Fix: Canonicalize path with realpath() and verify it's within the allowed base directory

**STEP 2 — File upload without validation**
```bash
rg -n "multer\(|upload\.|\.save\(.*file|move_uploaded_file|FileField|ImageField" .
```
- **SKIP if:** content-type validation and non-webroot storage are confirmed in the same handler
- **FINDING if not skipped:** Type: Unrestricted File Upload | Severity: High | Fix: Validate content-type server-side; store outside webroot with random filenames

**Output template (quick mode):**
```
| File:Line | Type | Severity | Pattern | Fix |
|---|---|---|---|---|
```

## Path traversal (LFI-style file read)

**Find:** any parameter that names a file/path — `?file=`, `?page=`, `?template=`, `?download=`,
`?lang=`, image/avatar paths, log viewers, export/report names. Inject traversal:
```
../../../../etc/passwd        ..%2f..%2fetc%2fpasswd        ....//....//etc/passwd
%2e%2e%2f (encoded)           ..%252f (double-encoded)      ..\..\ (Windows)
/etc/passwd%00 (null, legacy) file:///etc/passwd            php://filter/... (PHP wrappers)
```
**Confirm:** read a known file (`/etc/passwd`, `web.config`, app source, `.env`) — for a PoC a benign
file is enough; don't bulk-read secrets on a target you don't own. Try app config/source for high
impact (it often contains DB creds → bigger chain).

**Bypass tests (for evaluating a filter):** URL/double-URL encoding, `....//` (filter strips one
`../`), absolute paths, null bytes (legacy), and missing extension-append (`?file=../../etc/passwd` vs
forced `.php` suffix → use wrappers/null).

**Fix:** don't put user input in file paths. If unavoidable: map an **allowlist of identifiers → fixed
paths** (never the raw name); **canonicalize then verify** the resolved path stays under the intended
base dir (`realpath(path).startsWith(baseDir)`); strip path separators; serve downloads by ID from a
store, not by filesystem path.

## Local/Remote File Inclusion (LFI/RFI)

Mostly PHP-era but still appears: user input passed to `include`/`require`/`fopen`/template loaders.
- **LFI:** include local files → source disclosure, log poisoning → RCE, PHP wrappers
  (`php://filter/convert.base64-encode/resource=index.php` to read source; `data://`/`expect://` for
  exec where enabled).
- **RFI:** include a remote URL → direct RCE (needs `allow_url_include`).
**Fix:** never include user-controlled paths; allowlist; disable URL inclusion; use a fixed view/router
map.

## Unrestricted / unsafe file upload

**Find & test** every upload (avatar, attachment, import, profile media). Probe controls in order:
1. **Extension/MIME enforcement:** upload `shell.php`, `shell.php.jpg`, `shell.pHp`, `shell.php5`,
   `shell.phtml`, `.svg`, `.html`, `.jsp`, `.aspx`; trailing dot/space/`::$DATA` (Windows);
   double-extension; null-byte (legacy). Does the server validate by content or just the name?
2. **Content vs claimed type:** does it check magic bytes? Embed a payload in a valid image
   (polyglot) or in EXIF; rename to bypass MIME.
3. **Where does it land, and is it executable/served?** Upload to a web-served, script-executing dir →
   request the file → RCE. Even if not executable: **stored XSS** via `.html`/`.svg` served inline,
   **SSRF/XXE** via SVG, **content-type sniffing** XSS.
4. **Path control in filename:** `../../` in the filename → write outside the upload dir (overwrite
   configs/web files).
5. **Size/zip handling:** decompression bombs (DoS) and **zip-slip** (archive entries with `../`
   paths that escape extraction dir → arbitrary write).
6. **Image libraries:** known ImageMagick/`convert` RCE (ImageTragick), Ghostscript, ffmpeg SSRF —
   map the processing library to known CVEs.

**Confirm (non-destructive):** show the uploaded file is served and executes a benign marker
(`<?php echo 'LNG'.7*7; ?>` returning `LNG49`), or that an `.svg`/`.html` fires `alert(document.domain)`
for a viewer. On a target you don't own, prove reachability, not full RCE.

**Fix:**
- Validate by **allowlist of extensions *and* content/magic bytes**; re-encode images server-side
  (strip metadata, normalize) rather than trusting the upload.
- Store outside the webroot (or in object storage) with **server-generated random names**; serve via a
  handler that sets `Content-Disposition: attachment` and a correct, non-sniffable `Content-Type`
  (`X-Content-Type-Options: nosniff`); never execute the upload dir.
- Enforce size limits; safe archive extraction (validate each entry path stays in-dir); cap
  decompression ratio.
- Patch/sandbox image-processing libraries; disable dangerous coders (ImageMagick policy.xml).
- Antivirus/scan for user-shared files where appropriate.

## CTF angle

Upload-a-webshell-to-RCE (bypass the extension filter), LFI to read the flag/source or log-poison to
RCE, PHP wrapper source disclosure, and zip-slip to overwrite a config are classic. Identify the
server's exact filter, then pick the matching bypass.

## Real-world cases

Disclosed HackerOne reports with PoCs:
[file upload](https://github.com/reddelexc/hackerone-reports/blob/master/docs/tops_by_bug_type/TOPUPLOAD.md) ·
[file read / path traversal / LFI](https://github.com/reddelexc/hackerone-reports/blob/master/docs/tops_by_bug_type/TOPFILEREADING.md).

## References

WSTG-BUSL-09 (upload), WSTG-ATHZ-01 (traversal) · CWE-[434](https://cwe.mitre.org/data/definitions/434.html)/[22](https://cwe.mitre.org/data/definitions/22.html)/[98](https://cwe.mitre.org/data/definitions/98.html)/[29](https://cwe.mitre.org/data/definitions/29.html) ·
[OWASP File Upload Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/File_Upload_Cheat_Sheet.html) ·
[PortSwigger: Path traversal](https://portswigger.net/web-security/file-path-traversal) / [File upload](https://portswigger.net/web-security/file-upload).
Overlaps [access-control.md](access-control.md). Full bibliography: [research/web.md](../../research/web.md). Back to [web/](README.md).

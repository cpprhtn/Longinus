# 🧪 forensics/ — DFIR & forensics

Recovering truth from artifacts: what's hidden in this file, what happened on this machine, what's in
this network capture. A core CTF category and the real-world discipline of **DFIR** (digital forensics
& incident response). For Longinus it's both a CTF specialty and the "what did the attacker do?" side
of a security program.

> ⛔ For real incidents: preserve evidence (work on copies/images, keep hashes & chain of custody), and
> only handle data you're authorized to.
>
> **Scope & depth.** A map across the DFIR sub-fields, not a full procedure set. Use it to match an
> artifact to the right tool/workflow; go deep with **[CTF Wiki (misc/forensics)](https://ctf-wiki.mahaloz.re/misc/introduction/)**,
> the **[Volatility](https://www.volatilityfoundation.org/)** docs, and
> **[SANS DFIR](https://www.sans.org/posters/?focus-area=digital-forensics-incident-response)** resources.

## Sub-fields & workflow

### File analysis & carving
Identify and pull data out of files:
- `file`, `binwalk` (embedded/appended files — a ZIP/PNG hidden after a JPEG is classic), `foremost`/
  `scalpel` (carve by signature), `xxd`/hex editor (inspect magic bytes & structure), `exiftool`
  (metadata).
- **Trick:** fix corrupted/“wrong” magic bytes; find appended data after EOF; extract from polyglots,
  archives, office docs (`unzip` a `.docx`), PDFs (`pdf-parser`, `peepdf`).

### Steganography
Data hidden in media:
- Images: `zsteg` (PNG/BMP LSB), `steghide` (JPEG/WAV, often passworded — crack with `stegcracker`),
  `stegsolve` (bit-plane/channel view), LSB scripts, `exiftool`/strings, hidden in color channels or
  metadata.
- Audio: spectrogram analysis (Audacity/Sonic Visualiser — hidden images/text in frequency domain),
  LSB, DTMF/morse, slowed/reversed tracks.
- Always run `strings`/`binwalk`/`exiftool` first; stego is often layered.

### Memory forensics
Analyze a RAM dump (the most "DFIR" sub-field):
- **Volatility 3** (and 2): `windows.pslist`/`pstree`, `cmdline`, `netscan`, `filescan`/`dumpfiles`,
  `hashdump`/`lsadump` (creds), `malfind` (injected code), `windows.registry.*`. Find the malicious
  process, dumped files, network connections, and secrets in memory.
- Linux/Mac profiles likewise. Memory often holds plaintext keys, clipboard, and the flag.

### Disk & filesystem forensics
- Mount/loopback images read-only; **Autopsy/Sleuth Kit (`tsk`)** for timelines, deleted-file recovery,
  filesystem metadata; `testdisk`/`photorec` for recovery; `mmls`/`fls`/`icat` to navigate partitions
  and extract files. Examine browser history, `$MFT`, registry hives, journals, slack space.

### Network forensics (PCAP)
- **Wireshark**/`tshark`, `tcpdump`, NetworkMiner. Follow TCP/HTTP streams, **export objects**
  (transferred files), decode protocols, extract credentials/tokens, reassemble exfiltrated data,
  decrypt TLS if keys/`SSLKEYLOGFILE` provided. Spot C2 beacons, DNS tunneling, and odd ports.

### Log & timeline analysis
- Parse web/auth/system logs to reconstruct an attack: find the malicious requests
  ([../web/](../web/README.md) payloads in access logs), failed-login bursts, the privilege escalation,
  the data accessed. Build a timeline; correlate across sources. (Defensive flip side of
  [../web/misconfiguration.md](../web/misconfiguration.md) A09 logging.)

## CTF forensics workflow

1. `file` + `exiftool` + `strings` + `binwalk` on whatever you're given — surprisingly often solves it.
2. Match artifact type to sub-field: image/audio → stego; `.raw`/`.vmem`/`.dmp` → Volatility; `.pcap` →
   Wireshark; disk image → Autopsy/tsk; archive/doc → extract & recurse.
3. Recurse — forensics challenges layer (a PCAP carries a ZIP holding a stego PNG hiding the flag).
4. Look for the flag format throughout (`grep`/`rg` for the prefix at every stage).

## Real-world DFIR mindset

Same skills, applied to "we were breached":
- **Acquire** (image disk/RAM, collect logs) without altering evidence; hash everything.
- **Analyze** to answer: initial access, what executed, what was accessed/exfiltrated, persistence,
  lateral movement, timeline (map to **MITRE ATT&CK**).
- **Report** findings and IOCs; feed detections back into monitoring.

## References

[Volatility](https://www.volatilityfoundation.org/) · [Sleuth Kit/Autopsy](https://www.autopsy.com/) ·
[Wireshark](https://www.wireshark.org/) · [CTF Wiki (misc/forensics)](https://ctf-wiki.mahaloz.re/misc/introduction/) ·
zsteg/steghide/stegsolve · [SANS DFIR](https://www.sans.org/posters/?focus-area=digital-forensics-incident-response) ·
[MITRE ATT&CK](https://attack.mitre.org/). Tool URLs & full bibliography: [research/ctf.md](../../research/ctf.md).
Back to [tree](../00-map.md).

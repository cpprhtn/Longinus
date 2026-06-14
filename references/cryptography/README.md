# 🔐 cryptography/ — crypto attacks (CTF + real-world misuse)

Crypto rarely breaks because the math is broken — it breaks because it's **misused**: wrong mode, weak
randomness, reused nonces, unauthenticated ciphertext, rolled-your-own, or a side channel. This branch
covers the attacks (a major CTF category, per CryptoHack/Cryptopals) and the real-world misuse patterns
that map to **OWASP A04:2025 (Cryptographic Failures)**.

> ⛔ Apply to CTF challenges, your own systems, or authorized targets.

## Real-world crypto failures (A04:2025) — audit checklist

For a code/app audit, look for these first (they're common in vibe-coded apps):
- **Secrets/PII not encrypted** at rest or in transit; missing TLS/HSTS (see
  [../web/misconfiguration.md](../web/misconfiguration.md)).
- **Weak/legacy algorithms:** MD5/SHA1 for integrity, DES/3DES/RC4, ECB mode, `Math.random()` for
  tokens.
- **Passwords hashed wrong:** plain/`md5`/`sha256` instead of **argon2id/bcrypt/scrypt** (see
  [../web/auth-and-session.md](../web/auth-and-session.md)).
- **Hardcoded keys/IVs**, keys in source/repo ([../secrets-and-supply-chain/secret-detection.md](../secrets-and-supply-chain/secret-detection.md)).
- **Unauthenticated encryption** (encrypt-without-MAC; use AEAD like AES-GCM/ChaCha20-Poly1305).
- **Predictable randomness** for tokens/IVs/nonces (non-CSPRNG); **nonce/IV reuse**.
- **Custom crypto** / homemade protocols — almost always broken.
- **JWT algorithm flaws** → [../identity/README.md](../identity/README.md).

```bash
rg -n -i "md5|sha1\b|DES|RC4|ECB|Math\.random|new Random\(|createCipher\b|hardcoded|iv\s*=\s*['\"]" .
```

## Attack catalog (CTF + applied)

### Symmetric / block-cipher mode abuse
- **ECB:** identical plaintext blocks → identical ciphertext blocks. Detect (repeating 16-byte blocks),
  **ECB cut-and-paste** to forge structured plaintext (e.g. a token), **byte-at-a-time ECB decryption**
  of an appended secret.
- **CBC padding oracle:** a server that reveals (directly or via timing/error) whether padding is valid
  → **decrypt and forge** ciphertext without the key. Classic high-impact real bug.
- **CBC bit-flipping:** flip ciphertext bytes to flip controlled plaintext bits in the next block →
  tamper with unauthenticated CBC data (e.g. `admin=0`→`admin=1`).
- **IV/nonce reuse:** CTR/GCM nonce reuse → keystream recovery / GCM auth-key recovery (forgery).
- **Fix:** authenticated encryption (AES-GCM/ChaCha20-Poly1305), random unique nonces, never ECB,
  constant-time padding handling.

### Stream-cipher / XOR
- **Key/keystream reuse** (two-time pad): XOR ciphertexts → recover via crib-dragging/statistics.
- **Single-byte / repeating-key XOR** (Cryptopals 1): frequency analysis, key-size detection.

### Hashing
- **Length-extension** (MD5/SHA1/SHA2 Merkle–Damgård with `H(secret‖msg)` MACs) → forge
  `H(secret‖msg‖pad‖ext)` without the secret (`hashpump`). **Fix:** use **HMAC**, not naive
  concatenation.
- **Weak/collidable hashes** (MD5/SHA1 collisions), unsalted/fast hashes for passwords (crack with
  `hashcat`/`john`).

### RSA (a CTF favorite)
- **Small `e`** (e=3) with small/no padding → cube-root; **Håstad** broadcast (same `m`, multiple
  keys). **Common modulus**, **shared prime** between moduli (batch-GCD). **Fermat** (close primes),
  small factors → **factordb**/`yafu`. **Wiener** (small `d`). **Textbook RSA malleability**.
  **Padding oracle / Bleichenbacher** on PKCS#1v1.5. Tools: `RsaCtfTool`, `sympy`, `pycryptodome`.
  **Fix:** OAEP padding, proper key sizes, unique strong primes, never textbook RSA.

### Other
- **Diffie-Hellman:** small-subgroup / parameter confusion, weak params.
- **ECC:** invalid-curve, nonce reuse in **ECDSA** (reused `k` → private-key recovery — the PS3 bug),
  bias attacks.
- **PRNG prediction:** non-crypto RNG (Mersenne Twister) state recovery → predict "random" tokens.
- **Timing/side channels:** non-constant-time comparisons (`==` on MACs/tokens) → byte-by-byte
  recovery. **Fix:** constant-time compare.

## CTF workflow

1. Identify the primitive (RSA? AES-CBC? XOR? a custom scheme?) and what you're given (ciphertext,
   public params, an oracle, source).
2. Match to a known attack from the catalog (the params usually scream which one — small `e`, a padding
   oracle endpoint, reused nonce, ECB-shaped output).
3. Implement with `pycryptodome`/`sympy`/`z3`/`sage`; for RSA, try `RsaCtfTool` first; for oracles,
   script the query loop.
4. Recover key/plaintext → flag.

## References

[CryptoHack](https://cryptohack.org/) · [Cryptopals](https://cryptopals.com/) ·
[OWASP A04:2025](https://owasp.org/Top10/2025/) +
[Cryptographic Storage Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Cryptographic_Storage_Cheat_Sheet.html) ·
[CTF Wiki (crypto)](https://ctf-wiki.mahaloz.re/crypto/introduction/) ·
[RsaCtfTool](https://github.com/RsaCtfTool/RsaCtfTool) / [hashcat](https://hashcat.net/hashcat/) /
[SageMath](https://www.sagemath.org/). Full bibliography: [research/ctf.md](../../research/ctf.md).
JWT crypto → [../identity/README.md](../identity/README.md). Back to [tree](../00-map.md).

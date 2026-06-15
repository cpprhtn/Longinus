# Model & ML supply chain — malicious model files, hubs & poisoning (LLM03/04)

**A model file is code, not data.** The most common serialization formats embed executable objects, so
`load`-ing an untrusted model is **arbitrary code execution on your machine** — before a single
inference runs. Add poisoned weights/data (backdoors, triggers) and namesquatted hub repos, and the ML
supply chain becomes a first-class RCE + integrity surface. This is **insecure deserialization**
([../web/deserialization.md](../web/deserialization.md)) wearing a `.pt` extension, plus the
dependency problems in [../secrets-and-supply-chain/dependency-supply-chain.md](../secrets-and-supply-chain/dependency-supply-chain.md).

## Mechanical scan

> **Quick mode only.** Run these greps, apply skip conditions, report matches.
> No further analysis needed in quick mode.

**STEP 1 — trust_remote_code enabled**
```bash
rg -n "trust_remote_code\s*=\s*True" .
```
- **SKIP if:** the repository is your own AND pinned to a verified commit hash
- **FINDING if not skipped:** Type: Arbitrary Code Execution on Model Load (LLM03) | Severity: Critical | Fix: Set trust_remote_code=False; audit remote code before enabling

**STEP 2 — Unsafe model deserialization**
```bash
rg -n "torch\.load\(|pickle\.load|joblib\.load|keras\.models\.load_model" .
```
- **SKIP if:** `weights_only=True` is specified (torch) or input is from a trusted internal source
- **SKIP if:** path contains `/test/`
- **FINDING if not skipped:** Type: Model Deserialization → RCE (LLM03) | Severity: Critical | Fix: Use torch.load(weights_only=True) or safe alternatives (safetensors)

**Output template (quick mode):**
```
| File:Line | Type | Severity | Pattern | Fix |
|---|---|---|---|---|
```

## Format risk matrix (know what executes on load)

| Format / loader | Risk on load | Why |
|---|---|---|
| **Pickle** `.pkl`/`.bin`, `pickle.load`, `joblib.load`, `numpy` `allow_pickle=True` | **RCE** | `__reduce__` runs arbitrary code during unpickling |
| **PyTorch** `.pt`/`.pth`, `torch.load` (legacy default) | **RCE** | a zip wrapping a pickle — same primitive (use `weights_only=True`) |
| **Keras** `.h5`/`.keras`, `load_model` | **RCE** | **`Lambda` layers** serialize arbitrary Python; custom objects execute |
| **TensorFlow SavedModel** | **RCE / side effects** | custom ops / `Lambda` / graph ops can run code |
| **HuggingFace `from_pretrained(..., trust_remote_code=True)`** | **RCE** | downloads & **executes** arbitrary repo code on load |
| **GGUF / ONNX** | mostly data | safer, but parsers have had memory-safety CVEs; still verify source |
| **`safetensors`** | **data only — the fix** | pure tensors, no code path; prefer everywhere |

## Where it hides

- `torch.load`, `pickle.load`/`pickle.loads`, `joblib.load`, `keras.models.load_model`,
  `numpy.load(..., allow_pickle=True)`, `from_pretrained`/`AutoModel...`, `dill`/`cloudpickle`.
- Any model pulled from **HuggingFace Hub / civitai / a model zoo / a random URL / a teammate's
  artifact**, fine-tune checkpoints, RAG embedding models, and **CI/CD that downloads models** at build.
- `trust_remote_code=True` anywhere — and unpinned model refs (`main` instead of a commit hash).

```bash
rg -n "torch\.load|pickle\.loads?|joblib\.load|load_model|allow_pickle\s*=\s*True|trust_remote_code\s*=\s*True|from_pretrained" .
# flag any whose source is a downloaded/3rd-party/user-supplied artifact, and any unpinned model ref
```

## Vulnerable ↔ fixed application code

```python
# ❌ VULNERABLE — loads user-uploaded model with pickle (RCE on load)
@app.route('/predict', methods=['POST'])
def predict():
    model_file = request.files['model']
    model = torch.load(model_file)     # __reduce__ runs arbitrary code during unpickling
    return jsonify(model(data).tolist())

# ✅ FIXED — only load safetensors from trusted, pinned sources
from safetensors.torch import load_file

MODEL_PATH = "/app/models/verified-model.safetensors"  # pre-verified, hash-pinned
model_weights = load_file(MODEL_PATH)                   # no code execution possible
```

---

## How to find it

1. **Inventory model sources & pinning** — every model the app loads: where from, pinned to a commit/
   digest? signed? Loaded with a pickle-based loader or `safetensors`? `trust_remote_code`?
2. **Scan the artifact statically** — `modelscan` / `picklescan` (flag dangerous opcodes/imports),
   **`fickling`** (Trail of Bits — decompile/inject/check pickles) to see what a model executes on load.
3. **Hub hygiene** — check for **model-name confusion / namesquatting** (the dependency-confusion analog
   for model repos), typosquatted org names, and "mirror" repos of a popular model.

## Data & model poisoning (LLM04)

Even a "safe" format can carry a **backdoor**: weights trained to misbehave on a trigger input, or a
model fine-tuned/RAG-fed on attacker-controlled data so it leaks, biases, or executes attacker intent
on a cue. **Test/Fix:** vet training/fine-tune/RAG data provenance and ingestion pipelines; evaluate on
trigger/backdoor test sets; treat any externally-sourced weights as untrusted until provenance is
established. RAG-document poisoning is indirect prompt injection → [prompt-injection.md](prompt-injection.md).

## How to confirm (PoC)

- Static: `fickling`/`modelscan` flags a `REDUCE`/`GLOBAL` opcode invoking `os.system`/`exec` in the
  artifact, or a Keras `Lambda` carrying code.
- Dynamic (**your own lab only**): a benign pickle whose `__reduce__` writes a canary file / DNS-pings a
  collaborator on `torch.load` — proves load-time execution. Never run an untrusted model outside a
  sandbox to "prove" it.

## How to fix

- **Prefer `safetensors`** for everything; convert/reject pickle-based weights from untrusted sources.
- `torch.load(..., weights_only=True)`; **`trust_remote_code=False`**; avoid `allow_pickle`.
- **Pin by commit/revision + verify digests/signatures** (HF model signing / Sigstore); treat models as
  dependencies with provenance ([../secrets-and-supply-chain/dependency-supply-chain.md](../secrets-and-supply-chain/dependency-supply-chain.md)).
- **Scan in CI** (`modelscan`/`picklescan`) and **isolate model loading** (sandbox/container, no creds,
  no network) so a malicious artifact can't reach anything.

## CTF angle

A challenge hands you a `.pkl`/`.pt`/`.h5` to "load this model" → craft a `__reduce__` (or a Keras
`Lambda`) for code exec and read the flag; or a malicious HF repo gated behind `trust_remote_code`.
Same primitive as Python pickle pwn in [../web/deserialization.md](../web/deserialization.md).

## Real-world cases

- **Pickle RCE is the canonical Python deserialization bug** — `__reduce__` → `os.system`; PyTorch and
  joblib/sklearn models inherit it. The reason `safetensors` exists.
- **Malicious models on HuggingFace** — repeated discoveries of pickle-backdoored models (e.g. the
  *nullifAI* technique smuggling payloads past scanners); drove **`modelscan`/`picklescan`** (Protect AI)
  and Hub malware scanning. [HF security docs](https://huggingface.co/docs/hub/en/security).
- **Keras `Lambda`-layer RCE** — a saved `.h5`/`.keras` with a `Lambda` layer runs arbitrary code on
  `load_model`; addressed by Keras "safe mode".

## References

[OWASP LLM03 Supply Chain / LLM04 Data & Model Poisoning:2025](https://genai.owasp.org/resource/owasp-top-10-for-llm-applications-2025/) ·
[MITRE ATLAS](https://atlas.mitre.org/) ·
[safetensors](https://github.com/huggingface/safetensors) ·
[modelscan](https://github.com/protectai/modelscan) · [fickling](https://github.com/trailofbits/fickling) ·
[HuggingFace security](https://huggingface.co/docs/hub/en/security). It's deserialization at heart →
[../web/deserialization.md](../web/deserialization.md). Full bibliography:
[research/ai-llm.md](../../research/ai-llm.md). Back to [ai-llm/](README.md).

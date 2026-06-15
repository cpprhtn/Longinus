# Validation & sanitization layer (the structural control for injection/XSS classes)

The class-killer for injection, XSS, mass-assignment, and bad-input bugs is a **central, fail-closed
validation layer at the trust boundary** — allowlist the shape, reject by default — plus **context-aware
output encoding** that's never disabled. Validate once, at the edge; never trust the client. Backs the
Web/API/Identity/AI rows of [../enforce-forward.md](../enforce-forward.md).

## TypeScript / Node — zod schema at the boundary

```ts
import { z } from "zod";
const CreateUser = z.object({
  email: z.string().email(),
  age: z.number().int().min(0).max(130),
  role: z.enum(["user"]),            // never accept "admin" from the client (mass-assignment guard)
}).strict();                          // .strict() rejects unknown keys

app.post("/users", (req, res) => {
  const parsed = CreateUser.safeParse(req.body);
  if (!parsed.success) return res.status(400).json({ error: "invalid input" });
  createUser(parsed.data);            // only the allowlisted, typed fields
});
```

## Python / FastAPI — pydantic model (auto-validates the request)

```python
from pydantic import BaseModel, EmailStr, conint, Field
class CreateUser(BaseModel):
    email: EmailStr
    age: conint(ge=0, le=130)
    role: str = Field("user", pattern="^user$")   # client cannot set admin
    model_config = {"extra": "forbid"}             # reject unknown fields

@app.post("/users")
def create_user(body: CreateUser):                 # FastAPI rejects bad input with 422
    save(body.model_dump())
```

## Framework-agnostic — JSON Schema / OpenAPI request validation

Define the contract once and enforce it as middleware so **every** route is validated:
- Node: `express-openapi-validator` (validates requests against your `openapi.yaml`).
- Python: `connexion` (drives routing from the spec) or `fastapi` (spec is generated from models).
- A CI **contract test** asserting every route has a schema turns "we validate" into a guarantee.

## Output side — encode in context, never disable auto-escaping

- HTML: keep template auto-escaping ON; for rich HTML sanitize with **DOMPurify** (JS) / **Bleach**
  (Python) immediately before render. Treat `dangerouslySetInnerHTML` / `| safe` / `mark_safe` as a
  finding unless the value is provably trusted ([../web/xss.md](../web/xss.md)).
- SQL: parameterized queries / ORM only — never string-build ([../web/injection.md](../web/injection.md)).
- LLM output is untrusted input: validate/encode it into its sink exactly like raw user input
  ([../ai-llm/prompt-injection.md](../ai-llm/prompt-injection.md)).

## The gate

- A unit/contract test that **fails if any endpoint lacks a request schema**.
- Semgrep rules that flag raw sinks (`innerHTML`, string-built SQL, `dangerouslySetInnerHTML`) →
  [ci-gates.md](ci-gates.md).

Back to [enforce-forward.md](../enforce-forward.md) · [templates](README.md).

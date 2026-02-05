# Quality Code — Universal Rules for AI Coding Agents

You are an AI coding agent writing production code. Follow these rules strictly.

## Method: Plan → Test → Code → Document

### Step 0: Understand

- **Read before writing.** Never change code you haven't read.
- **Ask if unclear.** Don't guess requirements — clarify with the user.
- Identify which layers and files are involved.

### Step 1: Plan the Structure

- Break work into todos. Each step = independently verifiable.
- **Before writing any function, decide where it lives.** Not "I'll refactor later" — place it correctly from the start.
- Plan the file structure: which files will be created/modified, what each will contain.
- If a new feature touches 3+ areas — sketch the architecture in todos first.

```
Example plan for "add Stripe payments":
1. models/payment.py       — Payment, PaymentStatus dataclasses
2. clients/stripe.py       — StripeClient: create_charge, get_balance, refund
3. services/payments.py    — process_payment, validate_amount (business logic)
4. tests/test_payments.py  — tests for services/payments.py
5. main.py                 — wire new /pay command
```

### Step 2: Write Tests First

- **Write the test before the implementation.** Even a simple one.
- The test defines the contract: what goes in, what comes out, what should fail.
- Start with the happy path, then add edge cases.
- Tests force you to design clean interfaces — if it's hard to test, the design is wrong.

```python
# Write this FIRST:
def test_calculate_price_basic():
    assert calculate_price(base=100, coefficient=0.7) == 70

def test_calculate_price_minimum():
    assert calculate_price(base=100, coefficient=0.01, min_price=5) == 5

# Then implement calculate_price to make tests pass.
```

- If full TDD is overkill for the task — at minimum write tests immediately after the code, not "later".
- Run tests after every meaningful change.

### Step 3: Write Minimal Code

- Only what makes the tests pass. Nothing extra.
- **Place code in the right file from the start.** Don't dump into one file and "split later".
- One function = one file section = one responsibility.
- Keep files focused. A long file with uniform content (all models, all endpoints) is fine. A file mixing different types of logic is not.

### Step 4: Document Changes

- Update project CLAUDE.md or README if you changed architecture or added a new module.
- If you added a new service/client/layer — note it in the project docs.
- Don't write essays. One line per change is enough:

```markdown
## Changes
- Added `clients/stripe.py` — Stripe API client (charge, refund)
- Added `services/payments.py` — payment processing logic
- Added `models/payment.py` — Payment, PaymentStatus models
```

### Step 5: Verify

- Re-read the modified code for correctness.
- Run tests. Run build. Run linter if available.
- Check that no file mixes unrelated logic.
- Mark todo as completed only after verification passes.

## Write Short Code

- Less code = fewer bugs. Strive for brevity without sacrificing clarity.
- If it can be done in 1 line clearly — do it in 1 line.
- Eliminate intermediate variables that add no meaning.
- Use language idioms: comprehensions, ternaries, destructuring, chaining — when they're readable.
- Don't pad code with blank lines, redundant comments, or ceremonial boilerplate.
- If a function body is one expression — consider keeping it as one expression.
- Remove dead code, unused imports, unused variables immediately.
- Prefer standard library over reinventing. Prefer builtins over imports.

```python
# BAD — verbose
result = []
for item in items:
    if item.is_active:
        result.append(item.name)
return result

# GOOD — concise
return [item.name for item in items if item.is_active]
```

```typescript
// BAD — verbose
let result: string;
if (user.name !== undefined) {
    result = user.name;
} else {
    result = "Anonymous";
}

// GOOD — concise
const result = user.name ?? "Anonymous";
```

## Architecture: File Organization

### Start Flat, Split When It Hurts

Don't create 6 folders on day one. Start simple, grow structure when complexity demands it.

### Phase 1: Small Project (up to ~10 files)

Keep it flat. One file per logical domain. No folders needed yet.

```
project/
  main.py              — entry point, CLI wiring
  config.py            — settings, env
  models.py            — all data structures (while few)
  payments.py          — payment logic + Stripe API calls (while it's one API)
  notifications.py     — email/push logic
  tests/
    test_payments.py
    test_notifications.py
```

**Rules at this phase:**
- Each file does one domain — can mix layers (API + logic) while it's all the same domain.
- `models.py` can hold all models in one file — split by domain only when models become unrelated to each other.
- No `utils.py` yet. Put helpers next to the code that uses them.

### Phase 2: When to Split

**Logic mixing matters more than line count.** A 400-line file with 30 similar model classes is OK. A 100-line file with API calls, business logic, and DB queries is not. But still — try to keep files under 500 lines. If a uniform file approaches that limit, look for a natural seam to split by subdomain.

Split when a file starts doing two distinct jobs:

| Sign | Action |
|---|---|
| File has API calls AND business logic | Extract API calls → `clients/{api}.py` |
| File has 5+ data classes | Extract → `models/{domain}.py` |
| File has DB queries AND business logic | Extract queries → `db/{domain}.py` |
| File has 3+ CLI commands | Extract → `commands/{group}.py` |
| Same helper used in 3+ files | Extract → `utils/{purpose}.py` |

**After splitting, the natural layers emerge:**

```
project/
  main.py              — entry point only
  config.py            — settings
  models/
    payment.py         — Payment, PaymentStatus
    user.py            — User, UserProfile
  services/
    payments.py        — business logic (no HTTP, no SQL)
  clients/
    stripe.py          — Stripe HTTP calls only
    sendgrid.py        — email HTTP calls only
  db/
    payments.py        — payment DB queries
  tests/
    test_payments.py
    test_stripe.py
```

### Key Principles (any phase)

- **One file = describable in one phrase.** If you need "and" — it's two files.
- **Name by domain, not pattern:** `payments.py` (good) vs `manager.py` (bad), `stripe.py` (good) vs `api.py` (bad).
- **Don't mix different external APIs in one file.** Stripe and SendGrid = two files, always.
- **Flat > nested.** Max 2 directory levels. No `services/api/clients/http/base/`.
- **No empty abstractions.** Don't create `__init__.py` / `index.ts` barrels until you have 4+ files in a directory.

## Design for Testability

- **Pure functions first.** Extract logic into pure functions (input → output, no side effects). Pure functions are trivially testable.
- **Inject dependencies.** Don't hardcode clients/DB inside functions. Pass them as parameters or via constructor.

```python
# BAD — impossible to test without real Stripe
def process_payment(amount):
    client = stripe.Client(os.environ["STRIPE_KEY"])
    return client.charge(amount)

# GOOD — testable with mock
def process_payment(amount, stripe_client):
    return stripe_client.charge(amount)
```

- **Separate I/O from logic.** Read data → process (pure) → write result. The "process" part should be testable without any I/O.
- **Small functions = small tests.** If a test needs 20 lines of setup — the function does too much.
- **No global state.** No module-level mutable variables. Pass state explicitly.

### Test File Organization

```
tests/
  test_payments.py       — mirrors services/payments.py
  test_stripe_client.py  — mirrors clients/stripe.py
  test_models.py         — model validation tests
  conftest.py            — shared fixtures (Python/pytest)
```

- One test file per source file. Name: `test_{source_file}.py` / `{source_file}.test.ts`.
- Fixtures/helpers in `conftest.py` (pytest) or `setup.ts` — not duplicated across tests.
- Test names describe behavior: `test_rejects_negative_amount`, not `test_payment_3`.

## Proactive File Hygiene

Before adding code to a file, ask: **does this belong here?**

- If the file is `payments.py` and you're adding email logic — wrong file.
- If the file has only models and you're adding API logic — wrong file.
- If the file has 20 similar functions and you're adding another similar one — that's fine, even if the file is long.

**The rule is about logic type first, size second.** Uniform files can be long — but cap at ~500 lines. Past that, find a subdomain boundary and split.

**Never create `utils.py`, `helpers.py`, or `misc.py` as a dumping ground.** If a helper is used in one place — keep it local. If used in 3+ places — create a file named by purpose (`date_utils.py`, `formatting.py`).

## Simplicity

- Boring obvious code > clever code.
- 3 similar lines > 1 premature abstraction.
- No helpers/wrappers/factories for things used once.
- No "just in case" parameters, configs, feature flags.
- No design patterns for a single use case.
- Don't design for hypothetical future requirements.

## Naming

- **What**, not **how**: `active_users`, not `filtered_list`.
- Functions: verb + noun — `fetch_user`, `validate_input`.
- Booleans: `is_`, `has_`, `can_` prefix.
- Collections: plural — `users`, `prices`.
- No abbreviations except universal ones (`url`, `id`, `http`).

## Functions

- One function = one job.
- Max 3-4 params. More → group into object/dataclass.
- Return early, guard clauses at top, avoid nesting.
- No side effects in pure computations. No computations in side-effect functions.
- Keep functions short. If > 20 lines — probably should split.

## Error Handling

- Handle at boundaries (user input, APIs, I/O). Trust internal code.
- Specific exceptions, not generic `Exception` / `Error`.
- Messages: what happened + what was expected + what to do.
- Don't silently swallow errors.

## Async/Await

- **Don't mix sync and async.** If a client is async — all callers are async. Don't wrap async in sync or vice versa.
- **Parallel when independent.** Independent calls → run concurrently, not sequentially.
  - Python: `asyncio.gather(*tasks, return_exceptions=True)`
  - JS/TS: `Promise.all([...])` or `Promise.allSettled([...])`
  - Go: `errgroup.Group`
- **Sequential when dependent.** If B needs the result of A — await A first, then B.
- **Always handle partial failures** from parallel calls. `return_exceptions=True` (Python) or `Promise.allSettled` (JS) — then check each result.
- **Never block the event loop** with CPU-heavy or sync I/O code inside async functions.

## API Clients: Timeouts & Retries

- **Always set timeouts.** Set once at client creation, not per-request. No timeout = potential hang forever.
- **Retry only idempotent operations.** GET, PUT, DELETE — safe to retry. POST — only with an idempotency key.
- **Retry only transient errors.** 429, 502, 503, 504, timeouts, network errors. Never retry 400, 401, 403, 404.
- **Exponential backoff.** 1s → 2s → 4s. Max 3 attempts. That's enough for most cases.
- **One retry helper for the whole project.** Don't copy-paste retry logic into every client. Write it once or use a library (`tenacity` for Python, `p-retry` for JS, go-retryablehttp for Go).

## Logging

- **Log at boundaries:** API client calls (url, method, status, duration), key business events (order created, payment processed), errors.
- **Don't log inside pure logic.** If a function is pure computation — it doesn't need logging.
- **Three levels:**
  - `ERROR` — something broke, needs attention
  - `WARNING` — retry, fallback, degraded but working
  - `INFO` — key business events
- **Structured format** (key=value or JSON). Not free-form strings. Easier to search and filter.
- **Never log:** tokens, passwords, API keys, full request/response bodies (may contain PII), sensitive user data.

## Don't Touch What's Not Yours

- Don't refactor adjacent code.
- Don't add comments/docstrings/types to unchanged code.
- Don't reformat, rename, or "improve" existing code.
- Don't convert print→logging, requests→httpx unless asked.
- Don't add `__all__`, re-exports, barrel files unless project uses them.
- Don't create README/docs unless asked.

## Security

- Never hardcode secrets.
- Parameterized queries, never string interpolation for SQL.
- Sanitize user input at boundaries.
- Don't disable SSL, CORS, or auth "temporarily".
- Don't log tokens, passwords, PII.

## Language Defaults

### Python
- Type hints on signatures. f-strings. `pathlib.Path`.
- Pydantic / dataclass for structured data, not raw dicts.
- Comprehensions over loops when readable.

### TypeScript
- `const` default. `let` if reassigned. Never `var`.
- `async/await` over `.then()`.
- Strict mode. Named exports.
- Template literals over concatenation.

### Any Language
- Follow existing project conventions first, these rules second.
- Match existing style: tabs/spaces, naming, patterns.

## Context Efficiency

- Read only relevant files. Don't scan the whole project.
- Use Explore agent for broad questions.
- Use Grep for specific symbols. Glob for file patterns.
- For large files — read the specific section needed.
- Complete each step before starting the next.

## Communication

- Be direct. File path + line number when referencing code.
- Don't over-explain obvious things.
- If unsure — say so and investigate, don't guess.

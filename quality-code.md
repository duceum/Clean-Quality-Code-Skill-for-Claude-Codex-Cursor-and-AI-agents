# Quality Code — Universal Rules for AI Coding Agents

You are an AI coding agent writing production code. Follow these rules strictly.

## Method: Plan → Test → Code → Document

### Step 0: Understand & Propose Architecture

- **Read before writing.** Never change code you haven't read.
- **Ask if unclear.** Don't guess requirements — clarify with the user.
- **For new projects or features — propose the project structure first.** Before writing any code, present to the user:
  1. What framework/stack is being used (or should be used)
  2. What the folder structure will look like
  3. Where each piece of logic will live
  4. What the entry point is
- **Follow the framework's conventions.** If it's Next.js — use `app/` or `pages/`. If it's Django — use apps. If it's Express — use `routes/`. Don't invent your own structure when the framework has one.
- **For existing projects — read the structure first** and follow established patterns. Don't introduce a new organization style.

### Step 1: Plan the Implementation

- Break work into todos. Each step = independently verifiable.
- **Before writing any function, decide where it lives.** Not "I'll refactor later" — place it correctly from the start.
- Plan which files will be created/modified and what each will contain.

```
Example plan for "add Stripe payments" (framework-agnostic):
1. src/models/payment       — Payment, PaymentStatus types/classes
2. src/clients/stripe        — StripeClient: create_charge, get_balance, refund
3. src/services/payments    — process_payment, validate_amount (business logic)
4. tests/payments            — tests for services/payments
5. entry point              — wire new /pay command or route
```

### Step 2: Write Tests First

- **Write the test before the implementation.** Even a simple one.
- The test defines the contract: what goes in, what comes out, what should fail.
- Start with the happy path, then add edge cases.
- Tests force you to design clean interfaces — if it's hard to test, the design is wrong.

```
# Write this FIRST:
test "calculates price with coefficient":
    result = calculate_price(base=100, coefficient=0.7)
    assert result == 70

# Then implement calculate_price to make the test pass.
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
- Added `src/clients/stripe` — Stripe API client (charge, refund)
- Added `src/services/payments` — payment processing logic
- Added `src/models/payment` — Payment, PaymentStatus types
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

```
# BAD — verbose loop where a one-liner works
result = []
for item in items:
    if item.is_active:
        result.append(item.name)

# GOOD — use the language's idiomatic way to filter + map
result = items.filter(active).map(name)

# BAD — unnecessary branching
if user.name != null { result = user.name } else { result = "Anonymous" }

# GOOD — use null-coalescing operator (or equivalent idiom)
result = user.name ?? "Anonymous"
```

## Architecture: File Organization

### Rule #1: Follow the Framework

If the project uses a framework — **follow its conventions first**. Don't invent custom structure.

| Framework | Follow its structure |
|---|---|
| Next.js | `app/` or `pages/`, `components/`, `lib/` |
| Django | apps with `models.py`, `views.py`, `urls.py` |
| Rails | `app/models/`, `app/controllers/`, `app/services/` |
| Express/Fastify | `routes/`, `middleware/`, `controllers/` |
| FastAPI | `routers/`, `schemas/`, `services/` |
| Spring Boot | `controller/`, `service/`, `repository/`, `model/` |
| Flutter | `lib/screens/`, `lib/widgets/`, `lib/services/` |
| SwiftUI | follow Xcode groups convention |

If no framework or it doesn't dictate structure — use the generic structure below.

### Rule #2: Source Code Goes in `src/`

Keep the project root clean. Entry point and config at root, everything else in `src/` (or `lib/`, `app/` — whatever the ecosystem prefers).

```
project/
  src/                   — all source code lives here
    models/              — data structures, types, schemas
    services/            — business logic, use cases
    clients/             — external API integrations (one file per API)
    db/                  — database queries, repositories
    routes/ or commands/ — HTTP routes or CLI commands
  tests/                 — mirrors src/ structure
  entry point            — main file at root or src/ (index.ts, main.py, main.go)
  config                 — settings, env loading
```

**Don't dump source files in the project root.** Only entry point, config, and tooling configs (package.json, pyproject.toml, etc.) belong at root level.

### Rule #3: Split by Logic Type, Not by Size

**Logic mixing matters more than line count.** A 400-line file with 30 similar model classes is OK. A 100-line file with API calls, business logic, and DB queries is not. But still — try to keep files under 500 lines. If a uniform file approaches that limit, look for a natural seam to split by subdomain.

Split when a file starts doing two distinct jobs:

| Sign | Action |
|---|---|
| File has API calls AND business logic | Extract API calls → `clients/{api_name}` |
| File has 5+ unrelated data types | Extract → `models/{domain}` |
| File has DB queries AND business logic | Extract queries → `db/{domain}` |
| File has 3+ CLI commands or routes | Extract → `commands/` or `routes/{group}` |
| Same helper used in 3+ files | Extract → `utils/{purpose}` |

### Key Principles

- **One file = describable in one phrase.** If you need "and" — it's two files.
- **Name by domain, not pattern:** `payments` (good) vs `manager` (bad), `stripe` (good) vs `api` (bad).
- **One external API = one file.** Stripe and SendGrid never share a file.
- **Max 2-3 directory levels.** No `src/services/api/clients/http/base/`.
- **No barrel files** (index re-exports) until a directory has 4+ files that external code imports from.

## Design for Testability

- **Pure functions first.** Extract logic into pure functions (input → output, no side effects). Pure functions are trivially testable.
- **Inject dependencies.** Don't hardcode clients/DB inside functions. Pass them as parameters or via constructor.

```
# BAD — hardcoded dependency, impossible to test
process_payment(amount):
    client = new StripeClient(ENV.STRIPE_KEY)
    return client.charge(amount)

# GOOD — dependency injected, testable with mock
process_payment(amount, stripe_client):
    return stripe_client.charge(amount)
```

- **Separate I/O from logic.** Read data → process (pure) → write result. The "process" part should be testable without any I/O.
- **Small functions = small tests.** If a test needs 20 lines of setup — the function does too much.
- **No global state.** No module-level mutable variables. Pass state explicitly.

### Test File Organization

- Test directory mirrors source directory structure.
- One test file per source file. Follow the language's naming convention for tests.
- Shared fixtures/helpers in one place — not duplicated across tests.
- Test names describe behavior: `rejects_negative_amount`, not `test_3`.

## Proactive File Hygiene

Before adding code to a file, ask: **does this belong here?**

- If the file is `payments` and you're adding email logic — wrong file.
- If the file has only models and you're adding API logic — wrong file.
- If the file has 20 similar functions and you're adding another similar one — that's fine, even if the file is long.

**The rule is about logic type first, size second.** Uniform files can be long — but cap at ~500 lines. Past that, find a subdomain boundary and split.

**Never create `utils`, `helpers`, or `misc` as a dumping ground.** If a helper is used in one place — keep it local. If used in 3+ places — create a file named by purpose (`date_utils`, `formatting`).

## Simplicity

- Boring obvious code > clever code.
- 3 similar lines > 1 premature abstraction.
- No helpers/wrappers/factories for things used once.
- No "just in case" parameters, configs, feature flags.
- No design patterns for a single use case.
- Don't design for hypothetical future requirements.

## Naming

- **What**, not **how**: `active_users`, not `filtered_list`.
- **Functions/methods:** verb + noun — `fetch_user`, `validate_input`, `calculate_total`.
- **Booleans:** `is_`, `has_`, `can_`, `should_` prefix — `is_valid`, `has_access`.
- **Collections:** plural — `users`, `prices`, `order_items`.
- **Constants:** UPPER_SNAKE in most languages — `MAX_RETRIES`, `DEFAULT_TIMEOUT`.
- **Classes/types:** PascalCase, noun — `PaymentResult`, `UserProfile`. Never `PaymentManager` or `UserHelper`.
- **Files/modules:** match the primary thing they export. File has `PaymentService` → name it `payment_service` or `paymentService`.
- **Callbacks/handlers:** `on` + event — `on_click`, `on_payment_complete`, `handle_error`.
- **Iterators:** meaningful name if the body is >1 line. Single letter (`i`, `x`) only for trivial one-liners.
- No abbreviations except universal ones (`url`, `id`, `http`, `db`, `api`).
- **When unsure** — longer and clear beats short and cryptic. `remaining_attempts` > `rem`.

## Functions

- One function = one job.
- Max 3-4 params. More → group into an object/struct/dataclass.
- Return early, guard clauses at top, avoid nesting deeper than 2-3 levels.
- No side effects in pure computations. No computations in side-effect functions.
- If a function does two things that can be named separately — split it into two.

## Error Handling

- Handle errors at boundaries (user input, APIs, I/O). Trust internal code.
- Specific error types, not generic catch-all errors.
- Error messages: what happened + what was expected + what to do about it.
- Don't silently swallow errors. If you catch — either handle meaningfully or re-throw.
- **Catch at the right level.** Don't wrap every function in try/catch. Catch where you can actually do something useful (retry, fallback, show error to user). Let errors bubble up through pure logic.
- **When debugging — read the error message and stack trace first.** Don't guess. Trace the actual execution path.

## Async/Await

- **Don't mix sync and async.** If a client is async — all callers are async. Don't wrap async in sync or vice versa.
- **Parallel when independent.** Independent calls → run concurrently (gather, Promise.all, errgroup, etc.), not sequentially.
- **Sequential when dependent.** If B needs the result of A — await A first, then B.
- **Always handle partial failures** from parallel calls. Use the language's mechanism to collect errors without aborting all tasks.
- **Never block the event loop** with CPU-heavy or sync I/O code inside async functions.

## API Clients: Timeouts & Retries

- **Always set timeouts.** Set once at client creation, not per-request. No timeout = potential hang forever.
- **Retry only idempotent operations.** GET, PUT, DELETE — safe to retry. POST — only with an idempotency key.
- **Retry only transient errors.** 429, 502, 503, 504, timeouts, network errors. Never retry 400, 401, 403, 404.
- **Exponential backoff.** 1s → 2s → 4s. Max 3 attempts. That's enough for most cases.
- **One retry helper for the whole project.** Don't copy-paste retry logic into every client. Write it once or use a well-known retry library for your language.

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
- Don't swap libraries or patterns (e.g. change HTTP client, logging framework) unless asked.
- Don't add re-exports or barrel files unless the project already uses them.
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

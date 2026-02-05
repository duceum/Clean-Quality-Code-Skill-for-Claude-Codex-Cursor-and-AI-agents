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
- **Follow the framework's conventions.** Don't invent your own structure when the framework has one.
- **For existing projects — read the structure first** and follow established patterns.

### Step 1: Plan the Implementation

- Break work into todos. Each step = independently verifiable.
- **Before writing any function, decide where it lives.** Not "I'll refactor later" — place it correctly from the start.

```
Example plan for "add Stripe payments":
1. src/models/payment      — Payment, PaymentStatus types
2. src/clients/stripe      — StripeClient: charge, refund, get_balance
3. src/services/payments   — process_payment, validate_amount (business logic)
4. tests/payments          — tests for services/payments
5. entry point             — wire new /pay command or route
```

### Step 2: Write Tests First

- **Write the test before the implementation.** The test defines the contract: what goes in, what comes out, what should fail.
- Start with the happy path, then add edge cases.
- If full TDD is overkill — at minimum write tests immediately after the code, not "later".

```
test "calculates price with coefficient":
    assert calculate_price(base=100, coefficient=0.7) == 70
```

### Step 3: Write Minimal Code

- Only what makes the tests pass. Nothing extra.
- **Place code in the right file from the start.** Don't dump into one file and "split later".

### Step 4: Document Changes

- Update project docs if you changed architecture or added a new module. One line per change.

### Step 5: Verify

- Re-read the modified code. Run tests. Run build/linter if available.
- Mark todo as completed only after verification passes.

## Write Short Code

- Less code = fewer bugs. Brevity without sacrificing clarity.
- Use language idioms: comprehensions, ternaries, destructuring, chaining — when readable.
- Remove dead code, unused imports, unused variables immediately.
- Prefer standard library over reinventing.

```
# BAD
result = []
for item in items:
    if item.is_active:
        result.append(item.name)

# GOOD
result = items.filter(active).map(name)

# BAD
if user.name != null { result = user.name } else { result = "Anonymous" }

# GOOD
result = user.name ?? "Anonymous"
```

## Architecture: File Organization

### Rule #1: Follow the Framework

If the project uses a framework — **follow its conventions first**. Don't invent custom structure. If no framework — use the generic structure below.

### Rule #2: Source Code in `src/`

Keep root clean. Entry point and config at root, everything else in `src/` (or `lib/`, `app/` — whatever the ecosystem prefers).

```
project/
  src/
    models/              — data structures, types, schemas
    services/            — business logic, use cases
    clients/             — external API integrations (one file per API)
    db/                  — database queries, repositories
    routes/ or commands/ — HTTP routes or CLI commands
  tests/                 — mirrors src/ structure
  entry point            — main file at root or src/
  config                 — settings, env loading
```

### Rule #3: Split by Logic Type, Not by Size

A 400-line file with similar model classes — OK. A 100-line file mixing API calls, business logic, and DB queries — not OK. Cap at ~500 lines; past that, find a subdomain seam.

Before adding code to a file, ask: **does this belong here?** Never create `utils`, `helpers`, or `misc` as a dumping ground.

| Sign | Action |
|---|---|
| File has API calls AND business logic | Extract → `clients/{api_name}` |
| File has 5+ unrelated data types | Extract → `models/{domain}` |
| File has DB queries AND business logic | Extract → `db/{domain}` |
| Same helper used in 3+ files | Extract → `utils/{purpose}` |

### Rule #4: Business Logic Is the Core

Business logic takes data in, returns data out. It must not depend on external services, frameworks, or I/O. Everything external connects to the business logic — not the other way around.

```
# BAD — business logic knows about HTTP
calculate_shipping(order):
    rates = http.get("https://shipping-api.com/rates")
    return pick_cheapest(order, rates)

# GOOD — pure, caller provides data
calculate_shipping(order, rates):
    return pick_cheapest(order, rates)
```

### Key Principles

- **One file = describable in one phrase.** If you need "and" — it's two files.
- **Name by domain, not pattern:** `payments` not `manager`, `stripe` not `api`.
- **One external API = one file.**
- **Max 2-3 directory levels.**
- **Inject dependencies.** Pass clients/DB as parameters, don't instantiate inside business logic.

## Testability

- **Pure functions first.** Extract logic into pure functions (input → output, no side effects). Pure functions are trivially testable.
- **Separate I/O from logic.** Read data → process (pure) → write result. The "process" part should be testable without any I/O.
- **If a test needs 20 lines of setup** — the function does too much. Split it.
- **No global state.** No module-level mutable variables. Pass state explicitly.
- **Test directory mirrors source directory.** One test file per source file.
- **Shared fixtures/helpers in one place** — not duplicated across tests.
- **Test names describe behavior:** `rejects_negative_amount`, not `test_3`.

## Core Principles

- **Immutability by default.** Declare variables as immutable. Don't mutate function arguments — return new data.
- **Composition over inheritance.** Prefer composing small functions/objects. No deep inheritance chains.
- **Fail fast.** Validate inputs at the entry point. Fail immediately with a clear error, don't return null and check later.
- **Explicit over implicit.** No hidden side effects, no magic strings/numbers. Use named constants.
- **Single source of truth.** Every piece of knowledge in exactly one place. Don't duplicate logic — extract it.
- **Consistent patterns.** Same problem = same solution across the codebase. Follow existing patterns when adding new code.

## Simplicity

- Boring obvious code > tricky "smart" code.
- 3 similar lines > 1 premature abstraction.
- No helpers/wrappers/factories for things used once.
- No "just in case" parameters, configs, feature flags.
- No design patterns for a single use case.

## Naming

- **What**, not **how**: `active_users`, not `filtered_list`.
- **Functions:** verb + noun — `fetch_user`, `validate_input`.
- **Booleans:** `is_`, `has_`, `can_`, `should_` prefix.
- **Collections:** plural — `users`, `prices`.
- **Constants:** `UPPER_SNAKE` — `MAX_RETRIES`, `DEFAULT_TIMEOUT`.
- **Classes/types:** PascalCase noun — `PaymentResult`, `UserProfile`. Never `PaymentManager`.
- **Files:** match the primary export — `payment_service` or `paymentService`.
- **Callbacks:** `on_` + event — `on_click`, `handle_error`.
- **Iterators:** meaningful name if body >1 line. Single letter only for trivial one-liners.
- When unsure — longer and clear > short and cryptic. `remaining_attempts` > `rem`.

## Functions

- One function = one job.
- Max 3-4 params. More → group into an object/struct.
- Return early, guard clauses at top, nesting max 2-3 levels.
- If a function does two things that can be named separately — split it.

## Error Handling

- Handle at boundaries (user input, APIs, I/O). Trust internal code.
- Specific error types, not generic catch-all.
- Error messages: what happened + what was expected + what to do.
- **Catch at the right level.** Don't wrap every function in try/catch. Catch where you can do something useful (retry, fallback, show to user).
- **When debugging — read the error message and stack trace first.** Don't guess.

## Async/Await

- **Don't mix sync and async.** If a client is async — all callers are async.
- **Parallel when independent.** Run concurrently, not sequentially.
- **Sequential when dependent.** If B needs A's result — await A first.
- **Handle partial failures** from parallel calls.
- **Never block the event loop** with CPU-heavy or sync I/O inside async functions.

## API Clients: Timeouts & Retries

- **Always set timeouts** at client creation. No timeout = potential hang forever.
- **Retry only idempotent operations.** GET, PUT, DELETE — safe. POST — only with idempotency key.
- **Retry only transient errors:** 429, 502, 503, 504, timeouts. Never 400, 401, 403, 404.
- **Exponential backoff.** 1s → 2s → 4s. Max 3 attempts.
- **One retry helper for the whole project.** Don't copy-paste retry logic.

## Logging

- **Log at boundaries:** API calls (url, method, status, duration), key business events, errors.
- **Don't log inside pure logic.**
- **Three levels:** `ERROR` (broken), `WARNING` (degraded), `INFO` (key events).
- **Structured format** (key=value or JSON), not free-form strings.
- **Never log:** tokens, passwords, API keys, PII.

## Git Hygiene

- **Atomic commits.** One logical change per commit. Don't mix refactoring with feature work.
- **Meaningful commit messages.** What changed and why, not "fix" or "update".
- **Don't commit:** secrets, credentials, .env files, build artifacts, large binaries.
- **Review your diff before committing.** Read what's actually staged.

## Don't Touch What's Not Yours

- Don't refactor adjacent code.
- Don't add comments/docstrings/types to unchanged code.
- Don't reformat, rename, or "improve" existing code.
- Don't swap libraries or patterns unless asked.
- Don't create README/docs unless asked.

## Security

- Never hardcode secrets.
- Parameterized queries, never string interpolation for SQL.
- Sanitize user input at boundaries.
- Don't disable SSL, CORS, or auth "temporarily".
- Don't log tokens, passwords, PII.

## Language

- Follow existing project conventions first, these rules second.
- Match existing style: tabs/spaces, naming, patterns.
- Use the language's idiomatic constructs and best practices.

## Context Efficiency

- Read only relevant files. Don't scan the whole project.
- Use Explore agent for broad questions.
- Use Grep for specific symbols. Glob for file patterns.
- For large files — read the specific section needed.

## Communication

- Be direct. File path + line number when referencing code.
- Don't over-explain obvious things.
- If unsure — say so and investigate, don't guess.

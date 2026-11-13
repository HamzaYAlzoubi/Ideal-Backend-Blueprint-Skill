---
name: ideal-backend-blueprint
description: "Mandatory blueprint for building any backend: API, server, database, auth, or background-work layer, in any project of any size. Use this skill whenever the user asks to create, extend, or review a backend, REST or GraphQL API, server, database layer, login or auth, payments, or any code that handles requests and stored data, even if they never say 'backend' or describe the project as small. It defines the fixed components, conditional components, and rules that keep the system secure, stable, correct under concurrent requests, fast, scalable, and maintainable. Do not use for pure frontend or UI work, static sites, or one-off scripts with no server or stored data."
metadata:
  version: "1.6"
---

# Ideal Backend Blueprint

## How to use this blueprint

You build the backend; the user only describes the idea. The architecture decisions below are settled: do not reopen them or treat them as options.

- Apply it to **every** project that has a backend, small or large. Small projects grow, and a project built without these foundations gets rebuilt later.
- Goal: a real backend that is secure, stable, correct under concurrent requests, fast, and able to grow without a rewrite.
- Scope is architecture only. It does not choose the language, framework, folder layout, or code style. Follow the project's other instructions for those, and pick common, stable tools instead of inventing. Use the chosen framework's built-in features (CSRF protection, sessions, migrations, validation) instead of hand-rolling them, as long as they meet the rules below.

## The seven pillars (the yardstick for every decision)

1. **Security**: protect data and access.
2. **Reliability**: keep running under load and failures, and recover.
3. **Data integrity and concurrency**: data stays correct when many requests hit at once.
4. **Performance and efficiency**: fast responses, most requests per CPU.
5. **Scalability**: handle more users by adding resources, without redesign.
6. **Maintainability**: change and test safely.
7. **Operability and observability**: deploy, configure, and see what the system is doing.

Every decision must serve at least one pillar. Do not add anything that serves none.

## Governing principles

1. **Size changes the implementation, never the presence.** The 12 fixed components exist in every project. Reason: a model that treats them as optional for small projects produces fragile ones. A minimal form is smaller but fully working, never a stub, a TODO, or a disabled check; if you cannot implement a rule, tell the user instead of faking it.
2. **Boundaries are fixed; implementations grow.** Write each boundary fully on day one, start with the simplest implementation behind it, and swap it later without touching business logic. Only five capabilities get a swappable boundary: rate-limit store, cache, background-job dispatch, file storage, external providers. Each is an interface your app owns; code talks to the interface, not the tool. Everything else uses its tools directly, because extra abstraction is a burden.
3. **Build the irreversible decisions fully from the start**, because changing them after data and users accumulate means a rewrite:
   - database schema, constraints, and migrations
   - API response shape, `/v1` versioning, and pagination
   - authentication model and per-object permissions
   - unguessable ID format
   - stateless processes
   - idempotency keys
   - structured logs with request IDs
4. **Simplest implementation that satisfies the pillar.** Add no tool, service, or conditional component "just in case": no microservices, Kubernetes, queues, or Redis without a measured signal (see the upgrade ladder). Each extra moving part is another thing that fails.
5. **When a rule conflicts with the project, or you are unsure:** neither skip it silently nor apply it blindly. Pick the simplest solution that still satisfies the pillar, and tell the user your decision and why in two lines.
6. **Do not sink time into small details.** File names, variable names, formatting, and equivalent library choices are settled quickly by common convention. Spend your effort on the architectural decisions here.

## Before writing code

Write the user a summary of at most ten lines: main entities, key endpoints, whether the four conditional components are needed and why, and anything in the idea that conflicts with this blueprint. Then build.

## Build in verified steps

Build in this order. After each step, run install, build, and tests, and fix every failure before moving on: (1) skeleton, config, error handler, health checks; (2) schema and migrations; (3) request guard and authentication; (4) domain endpoints; (5) concurrency protections and their tests; (6) container and CI.

---

## The 12 fixed components

For each: why it exists, core rules, and the minimal form for small projects.

### 1. Request guard
Why: everything from outside is untrusted; this is the first line of defense.
- HTTPS only (it may terminate at a reverse proxy).
- Rate limiting, stricter on login. Limits on body size and request timeout.
- CORS with an explicit origin list; security headers.
- Authentication: hash passwords with Argon2id (at least 19 MiB memory, 2 iterations, parallelism 1), or bcrypt with cost 10 or higher if Argon2id is unavailable, never plain SHA or MD5. Short-lived access tokens; if refresh tokens are used, rotate them, keep them revocable server-side, and revoke them on logout and password change. One generic failure message that does not reveal whether an account exists.
- If authentication uses cookies: set HttpOnly, Secure, and SameSite, require a CSRF token or Origin check on every state-changing request, and never change state on GET. Bearer tokens sent in the Authorization header do not need CSRF protection.
- Authorization: deny by default. Check the role AND the user's right to that specific object (owner, tenant member, or access-list entry), so users cannot read each other's data. In multi-tenant systems, scope every query by tenant.
- Validate every input (type, length, range) and reject the unexpected.
Minimal form: in-memory rate limiting (move to a shared store when running multiple instances). All other rules apply.

### 2. API layer (routes and controllers)
Why: it is the contract with the frontend and costly to change.
- Documented endpoint list under `/v1`.
- Controllers read validated input, call business logic, and shape the response. **No business logic inside controllers.**
- One consistent success shape and error shape. Responses are built from explicitly chosen fields, never raw database rows. Create and update accept only the fields allowed for that operation, so no one can set role, balance, or ownership fields.
- Every list is paginated with a maximum page size.
Minimal form: same rules.

### 3. Business logic
Why: separates rules from HTTP so they can be tested and reused.
- Rules and calculations live here, not in controllers. This layer knows nothing about HTTP or the database tool.
- Multi-step workflows (for example order: new, paid, shipped) use a state machine that rejects invalid transitions.
- Transaction boundaries are decided here: what must succeed or fail as one unit.
Minimal form: one service per domain area.

### 4. Data access
Why: one place for all queries makes them easy to secure and tune.
- Parameterized queries only; never concatenate input into SQL.
- Connection pool with a maximum size and timeouts; a query timeout.
- Fetch only the needed columns; avoid N+1 queries.
Minimal form: same rules.

### 5. Database
Why: the last guard of data correctness, even if the app has a bug.
- Primary and foreign keys; NOT NULL, UNIQUE, and CHECK where needed.
- Indexes on foreign keys and on filter and sort columns.
- Versioned, immutable migrations stored in the repository (never edit one that has been applied).
- App account with least privilege.
- Backups with a documented, tested restore.
Minimal form: one database; all rules apply.

### 6. Concurrency and duplicate prevention
Why: concurrent clicks and automatic client retries cause duplicates and conflicts: two orders decrementing the same stock, a double charge.
- Sensitive operations (payment, order creation, booking) accept an idempotency key protected by a UNIQUE constraint in the database. Scope the key to the user and operation, store a fingerprint of the request and reject the same key with a different payload, and return the original result on a repeat.
- Concurrent updates use optimistic locking (version column) or an atomic conditional update, never read-then-write.
- Use row locks where strict ordering is required, such as inventory.
- Never rely on an in-memory check alone: it cannot stop two simultaneous requests.
Minimal form: same rules; no extra tools needed.

### 7. Config and secrets
Why: config differs per environment while code does not, and one leaked secret breaks everything.
- All config comes from environment variables through one config module; no values in code.
- Validate at startup and fail fast if anything is missing.
- Secrets never enter Git, images, or logs; pass them at runtime, not build time. Keep a `.env.example` with placeholders. Create `.gitignore` and `.dockerignore` that exclude `.env`, dependency folders, and build output, and never copy env files into an image.
Minimal form: same rules.

### 8. Error handling
Why: exposed errors leak information; silent errors hide failures.
- One central handler.
- Expected errors (validation, not found, forbidden) get clear codes.
- Unexpected errors return a generic 500; details go to logs only.
- Return a request ID to the client and log it.
- An unhandled exception is logged with full context, then the process exits gracefully so the supervisor restarts it. Never swallow it and keep running.
Minimal form: same rules.

### 9. Observability
Why: you cannot fix what you cannot see, and it drives every upgrade decision.
- Structured JSON logs to stdout with request ID, timestamp, and level.
- Log: authentication failures, authorization violations, validation failures, exceptions, admin actions.
- **Never log** passwords, tokens, secrets, card data, or personal information.
- Per-route latency (p95 and p99), traffic, and errors, plus service-level saturation (CPU, memory, connection pool).
- Two health checks: a light liveness check, and a readiness check that verifies the database.
Minimal form: logs, both health checks, one simple metric per route.

### 10. Process lifecycle
Why: avoid losing in-flight requests on every deploy or restart.
- On start: verify config and database connectivity before accepting traffic.
- On stop signal: stop accepting new requests, finish in-flight ones within a timeout, close connections, then exit.
- Stateless process: no sessions or important data in process memory. This is what allows horizontal scaling.
Minimal form: same rules.

### 11. Automated tests
Why: the safety net that allows changes without fear.
- Unit tests for business logic.
- Integration tests on a real database for critical paths.
- Security tests: a user accessing another user's object, and access without authentication, must be rejected. Concurrency test: two identical simultaneous requests yield one result.
- These tests go through the real HTTP endpoints with real tokens and a real database, and must fail if the protection is removed. A test that mocks what it checks is invalid.
- Run automatically before every merge.
Minimal form: cover critical paths only (login, sensitive or financial operations, concurrency), not every line.

### 12. Build and deploy
Why: what runs on your machine must run the same in production.
- Container with a multi-stage build, a pinned base image version (never `latest`), a non-root user, and a restart policy.
- Dependency lock files.
- CI that runs tests and a dependency vulnerability scan before deploy.
- Ability to roll back a release.
Minimal form: one Dockerfile and one CI pipeline.

---

## The 4 conditional components

Each has a fixed boundary from day one. Add the real tool only when its trigger appears.

### 13. Cache
- Boundary: a `cache` interface the code uses; the first implementation is in-process memory or a no-op.
- Add the tool when: repeated reads are slow after indexes and N+1 fixes are exhausted. With more than one instance, an in-process cache per instance is fine if stale data is tolerable; use a shared store when instances must see the same cached data.
- Rules: cache-aside with a TTL and a clear invalidation path. Include the user in the key for private content. The cache is never the source of truth, and losing it must not break the system. No sensitive data inside.

### 14. Background jobs
- Boundary: a job-dispatch interface; the first implementation runs asynchronously inside the same process. Jobs that must not be lost (payments, orders, anything promised to the user) are recorded in a database table in the same transaction as the business change, and unfinished ones are resumed at startup; running in-process alone is acceptable only for work that can be lost.
- Add the tool when: any operation exceeds about two seconds, depends on a slow external party (email, file processing, reports), or is scheduled. Never run these inside the HTTP request.
- Rules when moving to a queue: workers separate from the API; every job safe to run twice (delivery is at-least-once); retries with growing delay and a cap; a dead-letter queue for failures.

### 15. External integrations
- Boundary: one adapter per provider, behind an interface your app owns.
- Add when: calling any external service (payments, email, SMS, storage).
- Rules: a timeout on every call; bounded retries for safe operations only; a circuit breaker on sustained failure; no external call inside an open database transaction; treat provider responses as untrusted input; fetch user-supplied URLs only through an allowlist, verify the resolved IP is not private or internal (re-check after any redirect) to prevent SSRF; for webhooks, verify the signature and prevent double processing.

### 16. File uploads
- Boundary: a file-storage interface your app owns; the first implementation writes to local disk outside the web root. Local disk suits a single instance; switch to shared or object storage before running more than one.
- Add when: the project accepts uploaded files (images, documents).
- Rules: only authorized users upload; allow only the file types the feature needs; check the real file content, not just the extension or the client-sent type; store under a generated name, never the user's filename; enforce a size limit; never execute uploaded files; serve them through your own handler or signed URLs, not from a public folder.

---

## Upgrade ladder: measure, do not guess

Upgrade an implementation only on a measured signal from component 9, in this order. Never jump ahead.
1. Indexes and query tuning (signal: slow lists or search).
2. Remove N+1 queries.
3. Cache (signal: high latency with idle CPU).
4. Tune the connection pool size, or add an external pooler (signal: "too many connections" errors). A pool itself is already part of component 4.
5. Background jobs (signal: long requests tie up workers).
6. Scale the app horizontally by adding instances (signal: CPU saturation). This works because processes are stateless.
7. Bigger database, then read replicas.
8. Sharding: last resort, only with user approval, because it is irreversible.

## Never do

- Business logic in controllers.
- Unpaginated lists.
- State or sessions in process memory.
- Secrets or sensitive data in code, Git, or logs.
- External calls inside an open transaction.
- Unhashed or fast-hashed (SHA, MD5) passwords.
- Adding a tool or service with no signal that justifies it.
- Dropping a fixed component because the project is "small".
- Stubbed, commented-out, or "for MVP" security checks, and documentation that claims what the code does not do.

## Definition of done

Before saying the work is finished, run: install from the lock file, build or type-check, migrations, start the app, both health checks, and the test suite. Show the real output to the user. If a step fails, fix it or report it as failing, and never call a feature working unless you ran it. README and decision notes describe only what exists in the code, and list anything unimplemented under "Not implemented".

## Pre-delivery checklist

Verify each item. Tell the user about any that fails, and why.
- [ ] Every protected route requires authentication, object ownership is checked, and cookie-based auth has CSRF protection.
- [ ] Every input is validated against allowed fields, every query is parameterized, and responses never return raw database rows.
- [ ] Rate limiting is on, especially on login.
- [ ] Every list is paginated with a maximum size.
- [ ] Constraints, indexes, and migrations exist.
- [ ] Sensitive operations are protected against duplicates and concurrency, and a test proves it.
- [ ] No secrets in code, Git, or logs, and `.gitignore` and `.dockerignore` exist.
- [ ] A 500 error reveals no internal details.
- [ ] Structured logs with request IDs, plus liveness and readiness checks.
- [ ] Graceful shutdown works and no state lives in the process.
- [ ] Tests for critical paths pass.
- [ ] Dockerfile and CI exist.
- [ ] The project installed, built, started, and passed its tests, with real output shown, and README and decision notes match the code.
- [ ] You told the user about your conditional-component decisions and any conflict with this blueprint.

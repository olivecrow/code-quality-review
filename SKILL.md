---
name: code-quality-review
description: Read-only, evidence-backed code quality review for repositories, pull requests, diffs, modules, files, web services, frontend/mobile/desktop apps, Unity or game projects, libraries, CLIs, and data or automation code. Use when Codex is asked to review code quality, correctness, architecture, refactoring risks, code smells, module and layer boundaries, dependency structure, shared helper/type/schema reuse, hidden coupling, error handling, test strategy, reliability, performance, security, or technical debt, especially when findings and recommendations should come before implementation.
---

# Code Quality Review

## Core Contract

Produce a read-only, evidence-backed code review that can guide later refactoring, security hardening, bug fixing, or test improvement.

During the review pass, do not modify source files, generated files, lockfiles, tests, configuration, documentation, git state, or external systems. Do not stage, commit, push, run formatters with write mode, run auto-fix commands, install or upgrade dependencies, regenerate snapshots, rewrite files, or create a patch as part of the review.

If the user explicitly asks for both review and implementation, complete the review first and keep the findings separate from the later implementation work. After the review, continue to implementation only when the user's request and host instructions authorize edits; keep changes scoped to the reviewed findings. If the user asks for analysis only, stop at findings, recommendations, and verification ideas.

Prefer findings over broad advice. A useful review identifies the affected code, explains why it matters, estimates impact and confidence, and gives the smallest practical direction for remediation or refactoring.

## Review Workflow

1. Establish scope.
   - Identify whether the target is a whole repository, branch, pull request, diff, directory, module, file, or specific concern.
   - Inspect project instructions, architecture notes, package manifests, dependency manifests, build/test configuration, and relevant recent diffs before judging intent.
   - Use read-only inspection commands. Prefer `rg`, `git status`, `git diff`, `git log`, package manifests, lockfiles, dependency graphs when already available, and targeted file reads.
   - Tests, builds, type checks, linters, and static analyzers are allowed only as validation aids. Avoid commands that auto-fix, update snapshots, rewrite lockfiles, install packages, or mutate external systems. If a validation command produces local artifacts or caches, report them or clean only clearly disposable outputs.

2. Map the system before criticizing it.
   - Identify entry points, core data flows, external integrations, persistence boundaries, authentication or permission boundaries, background jobs, user-facing paths, and runtime lifecycle hooks.
   - Note intended layers and ownership: UI, presentation, transport, API, domain logic, state management, infrastructure, persistence, integrations, shared utilities, tests, build tooling, assets, and deployment/runtime code.
   - Identify owners for shared functions, helpers, types, schemas, DTOs, validation shapes, constants, adapters, protocol objects, events, and assets. Treat repeated reinvention of these artifacts as a first-class refactoring signal.
   - Sketch dependency direction in words: which layers are allowed to depend on which other layers, where cross-layer shortcuts appear, and whether directory hierarchy makes ownership visible.
   - Separate actual behavior visible in code from inferred design intent.

3. Follow real paths.
   - Trace code from public inputs to meaningful outputs, side effects, storage, rendering, network calls, engine callbacks, process boundaries, or security-sensitive sinks.
   - Review tests against observable contracts, not only implementation details.
   - Avoid isolated style complaints that do not affect behavior, maintainability, security, performance, reliability, or future refactoring.

4. Calibrate findings.
   - Rank findings by impact and likelihood, not personal style preference.
   - Mark uncertain items as questions or hypotheses. Do not present speculation as confirmed.
   - For security findings, identify the trust boundary, attacker-controlled input or sensitive asset, vulnerable operation, exploit preconditions, and realistic impact.

5. Report for action.
   - Put findings first, ordered by severity.
   - Include evidence, impact, direction, and useful validation ideas for each actionable issue.
   - Use host-native inline comments only when supported and when a finding is tied to a specific file and narrow line range.

## High-Signal Failure Modes

Actively look for these review signals, then explain them through the relevant review lens below:

- Reimplemented helper/type/schema/shape: a local utility, data structure, validator, mapper, adapter, event, or protocol object duplicates an existing owner instead of replacing or extending it.
- Hidden coupling: callers depend on import side effects, initialization order, global mutation, ambient context, shared singletons, cache state, serialized object state, or an unvalidated data shape.
- Boundary collapse: UI, transport, domain, persistence, vendor, infrastructure, engine, or asset logic are mixed so a small change requires edits across unrelated layers.
- Brittle tests: tests assert source text, function names, private helper calls, or internal steps instead of observable behavior, state transition, output, failure, or side effect.
- Missing edge cases: empty input, missing values, malformed data, boundary values, duplicate input, concurrency, reentrancy, cancellation, partial failure, cleanup, and retry exhaustion are not covered.
- Complexity inflation: nested control flow, boolean flag branches, long functions, large files, broad switch/case logic, or state machines hidden in conditionals make the real contract hard to see.
- Ownership fog: flat hierarchy, broad `shared` folders, re-export chains, wildcard exports, or asset folders hide ownership and dependency direction.
- Silent failure: empty catch blocks, broad fallbacks, valid-looking defaults, catch-and-continue behavior, or repeated caller-side defensive checks hide defects and inflate code size.

## Review Lenses

### Correctness and Behavior

- Check whether code implements the visible product, API, command, library, or runtime contract.
- Look for invalid states, ordering problems, null or missing values, resource leaks, race conditions, partial failure behavior, retry behavior, cleanup, idempotency, and inconsistent state transitions.
- Verify that tests cover the contract users or callers rely on, not just the current implementation shape.

### Architecture and Design

- Check module, package, component, scene, asset, and layer boundaries against the project structure and actual imports.
- Use SOLID only as a diagnostic lens, not as a slogan. Flag Single Responsibility, Open/Closed, Liskov Substitution, Interface Segregation, or Dependency Inversion issues only when they create concrete maintenance, testing, correctness, or extension cost.
- Flag oversized modules, God classes, God objects, broad service objects, overgrown components, and orchestration code that centralizes unrelated product logic, state, persistence, validation, integration, or lifecycle behavior.
- Prefer existing project patterns and local ownership over generic architecture advice.

### Dependencies and Boundaries

- Inspect imports, dependency manifests, lockfiles, dependency injection, service locators, global state, shared singletons, module initialization side effects, package boundaries, and generated code boundaries.
- Look for circular dependencies, unstable dependency direction, framework leakage into domain code, duplicated adapters, unnecessary production dependencies, and transitive dependencies that become accidental public contracts.
- Check whether boundaries are enforced by lint, tests, build rules, package boundaries, dependency-cruiser/import-linter-style rules, assembly definitions, project references, or CI. If enforcement is absent, recommend a scoped mechanical rule before relying on convention.
- Identify fan-in and fan-out hotspots: modules imported by many unrelated areas, modules that import across too many layers, and "shared" folders that collect unrelated behavior.
- Flag re-export chains when they hide ownership, cause accidental public APIs, create circular dependencies, or let feature code bypass layer boundaries. Do not require eliminating all re-exports; require explicit, narrow ownership.
- Distinguish dependency hygiene from dependency vulnerability. Do not claim a package is vulnerable unless verified through a lockfile, advisory, official changelog, or current authoritative source.

### Error Handling and Fallbacks

- Locate the intended error boundary: request handler, job runner, command boundary, UI boundary, engine boundary, integration adapter, library API, or process supervisor.
- Flag empty catch blocks, catch-all defaults, logs that omit context, fallback values that look valid to callers, and repeated defensive checks that compensate for unclear contracts.
- Distinguish resilience from silence. A fallback should name the failure it handles, preserve enough context for debugging, and be tested through the boundary that owns the policy.
- Check whether validation happens at the boundary and whether internal functions can rely on explicit typed or validated inputs.

### Security

- Trace data from untrusted sources to sensitive sinks. Cover user input, uploaded files, URLs, headers, cookies, tokens, environment variables, database rows, third-party webhooks, save files, mod/plugin inputs, model or tool outputs, and generated content where relevant.
- Check authentication, authorization, tenant isolation, injection, command execution, path traversal, SSRF, XSS, CSRF, insecure deserialization, unsafe redirects, file upload handling, secret exposure, weak cryptography, logging of sensitive data, permissive CORS, rate limiting, and unsafe defaults.
- For dependency security, inspect manifests and lockfiles first. Use current authoritative advisories when a package vulnerability claim matters.
- Separate confirmed vulnerabilities from hardening suggestions. State exploit preconditions and realistic impact.

### Reliability, Performance, and Operations

- Check timeouts, cancellation, retries, idempotency, backpressure, concurrency, memory growth, caching, pagination, batching, cleanup, logging, metrics, diagnostics, and failure visibility.
- Flag performance issues only when the code path, expected data size, frame budget, render pattern, allocation pattern, or call frequency makes the risk credible.
- Note operational risks that would complicate refactoring, rollback, debugging, incident response, asset delivery, version migration, or support.

### Tests and Observability

- Identify missing tests around behavior, security boundaries, error paths, integration contracts, migrations, asset loading, engine lifecycle behavior, and regression-prone code.
- Flag tests that mock internal modules, private helpers, or implementation steps. Prefer mocking external boundaries such as network, clock, filesystem, database, vendor SDK, process execution, browser APIs, game engine APIs when necessary, or model/tool providers.
- Check side effects explicitly: persisted state, emitted events, logs or metrics where contract-relevant, cache invalidation, retries, cleanup, idempotency, rendering state, and asset or scene changes.
- Watch for production code made more defensive only to satisfy unrealistic tests. Recommend fixing the test contract or moving validation to the correct boundary instead of adding caller-by-caller guards.

## Domain Overlays

Use these overlays only when relevant to the reviewed project. Do not force every review to cover every domain.

### Web Services and APIs

- Review request validation, response contracts, authentication, authorization, tenant isolation, rate limiting, pagination, migrations, transaction boundaries, background jobs, webhooks, idempotency keys, retries, queue semantics, and deployment/runtime configuration.
- Check database access for injection, missing indexes on hot paths, N+1 query patterns, transaction scope mistakes, unbounded queries, and inconsistent authorization filters.
- Check external integrations for timeout, retry, circuit breaker, signature verification, replay handling, and partial failure behavior.

### Frontend, Mobile, and Desktop Apps

- Review state ownership, rendering boundaries, navigation, form validation, async loading states, error states, offline behavior, permission prompts, local storage, cache invalidation, accessibility, responsiveness, and platform-specific lifecycle behavior.
- Check whether user-visible behavior is covered by component, integration, end-to-end, or platform tests rather than implementation-only mocks.
- Look for performance risks from unnecessary rerenders, expensive derived state, large bundles, layout thrashing, memory leaks, unbounded listeners, and resource cleanup gaps.

### Unity and Game Projects

- Review engine lifecycle methods, scene loading, prefab references, serialized fields, ScriptableObject shared state, asset ownership, Addressables or resource loading, coroutines, async tasks, physics update boundaries, save/load paths, input handling, and platform-specific build settings when present.
- Treat per-frame paths such as `Update`, `FixedUpdate`, render callbacks, and input polling as performance-sensitive. Look for repeated allocations, broad searches, reflection, blocking IO, expensive logging, and unnecessary component lookups.
- Check gameplay systems for deterministic state transitions, pause/time-scale behavior, object pooling, event subscription cleanup, scene transition cleanup, and editor-only code leaking into runtime builds.
- For Unity packages, inspect assembly definitions, public API surface, sample assets, editor/runtime separation, package manifest constraints, and compatibility assumptions.

### Libraries, SDKs, CLIs, and Tools

- Review public API stability, backward compatibility, semantic versioning assumptions, error contracts, configuration loading, environment handling, command exit codes, streaming or large-file behavior, extensibility hooks, and documentation/test alignment.
- Check that internal helpers are not accidentally exported, public types are not coupled to private implementation details, and examples exercise realistic failure paths.

### Data, Automation, and Model-Integrated Code

- Review schema drift handling, input validation, batch boundaries, retries, idempotency, audit trails, backfills, sampling assumptions, prompt/model boundary handling, unsafe tool output trust, and personally identifiable or sensitive data handling.
- Check that metrics, transformations, and generated outputs have reproducible definitions and tests for malformed, partial, duplicated, or out-of-order inputs.

## Refactoring Readiness

Use this checklist when the review is meant to guide later implementation:

- Design: module and layer boundaries are named; dependency direction is visible; shared utilities, types, schemas, assets, and protocol objects have one owner.
- Enforcement: strict typing or documented runtime validation is active where applicable; circular dependencies and boundary violations can fail CI; complexity, nesting, and function-size limits are enforced where local tooling supports them.
- Change discipline: new functions, helpers, schemas, adapters, and assets are created only after searching for existing owners; errors are handled at boundaries; fallback behavior is explicit.
- Review discipline: duplicate replacements, dead code, commented-out code, hidden coupling, re-export fog, fan-in/fan-out hotspots, and initialization-order dependencies are checked before recommending changes.
- Test discipline: edge cases are listed before implementation; tests assert behavior and side effects; mocks stay at external boundaries; tests do not force production-only defensive clutter.

If a repository lacks enforcement, recommend incremental guardrails in this order: detect circular imports or dependency cycles, enforce layer import rules, enable strict typing or typed boundary validation, then add complexity/depth/function-size limits. Avoid proposing a large rule set that would bury the team in unrelated violations; scope rules to changed or high-risk areas when needed.

## Finding Standards

Each actionable finding should include:

- Severity: `Critical`, `High`, `Medium`, `Low`, or `Info`.
- Confidence: `High`, `Medium`, or `Low`.
- Category: for example `Security`, `Correctness`, `Architecture`, `Dependency`, `Code smell`, `Testing`, `Performance`, `Reliability`, or `Operations`.
- Evidence: exact file and line references where possible.
- Impact: the concrete bug, risk, maintenance cost, or refactoring blocker.
- Direction: the smallest practical refactoring, fix, boundary change, or test strategy that would address it later.
- Validation: the test, check, scenario, or manual verification that should prevent recurrence.

Use `Critical` only for issues that can plausibly cause severe security compromise, data loss, systemic outage, or broadly exploitable behavior. Use `Info` for useful context that is not a defect.

## Host-Native Output

When the host supports inline code comments, emit one native comment for each actionable finding tied to a specific file and narrow line range. Keep the range tight and point at the root cause or misleading contract, not every downstream usage.

For Codex inline comments, use `::code-comment{...}` with:

- `title`: short severity-prefixed title, such as `[P1] Authorization bypass in project lookup`.
- `body`: one paragraph with evidence, impact, and remediation direction. Mention that this is a review finding and do not include patch text unless implementation was requested.
- `file`: absolute file path when available; otherwise a workspace-resolvable path.
- `start` and `end`: 1-based line numbers. Omit `end` for a single-line finding.
- `priority`: `0` for critical, `1` for high, `2` for medium, and `3` for low or informational findings.

Do not emit inline comments for broad architectural observations that cannot honestly be tied to a narrow location. Put those in the report instead. When a finding appears as an inline comment, still include a concise entry in the findings list so the review is readable without the comments UI.

## Report Shape

Start with findings. Keep summary and process notes short until after the findings.

Recommended structure:

```markdown
**Findings**
- [High][Security][High confidence] Title
  Evidence: `path/to/file.ext:123` and the code path involved.
  Impact: What can go wrong and under what conditions.
  Direction: What a later refactor or fix should change.
  Validation: Test, check, or manual verification to add later.

**Refactoring Map**
- Quick wins: duplicate helpers/schemas to remove, dead code, shallow nesting cleanup, narrow re-export cleanup, local tests to rewrite.
- Structural refactors: boundary, layering, ownership, directory hierarchy, dependency direction, fan-in/fan-out, lifecycle, initialization-order, or asset ownership changes.
- Enforcement: lint, type, dependency, complexity, package, assembly, and CI rules that should prevent recurrence.
- Security hardening: confirmed vulnerabilities and defense-in-depth items.
- Test coverage: edge-case, behavior, side-effect, failure-path, lifecycle, and integration tests that should exist before or during changes.

**Coverage and Limits**
- Inspected: files, modules, diffs, commands, and documents reviewed.
- Not inspected: material areas outside scope.
- Residual risk: what could still be missed and how to reduce that risk.
```

If no actionable findings are found, say so directly, then report the inspected scope and remaining risk. Do not invent minor issues just to fill the report.

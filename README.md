# Code Quality Review

`code-quality-review` is a Codex skill for read-only, evidence-backed code quality reviews.
It is designed to review repositories, pull requests, diffs, modules, files, web services,
apps, Unity or game projects, libraries, CLIs, and data or automation code before an
implementation phase begins.

The skill focuses on actionable findings rather than broad advice. It asks Codex to inspect
real code paths, identify concrete risks, rank findings by severity and confidence, and report
remediation directions that can guide later refactoring, security hardening, bug fixing, or
test improvement.

## What It Reviews

- Correctness, behavior, and edge cases
- Architecture, ownership, and module boundaries
- Dependency direction, circular dependencies, and accidental public APIs
- Shared helper, type, schema, validation shape, adapter, and protocol reuse
- Hidden coupling, initialization-order dependence, global state, and side effects
- Error handling, fallback behavior, and failure visibility
- Security boundaries, untrusted input flows, sensitive sinks, and dependency risk
- Reliability, performance, operations, and diagnostics
- Test quality, mock boundaries, observability, and regression coverage

## Domain Coverage

The skill includes domain overlays for:

- Web services and APIs
- Frontend, mobile, and desktop apps
- Unity and game projects
- Libraries, SDKs, CLIs, and tools
- Data, automation, and model-integrated code

These overlays are conditional. Codex should use only the overlays relevant to the project
being reviewed instead of forcing every review through every domain checklist.

## Review Style

The skill keeps review work separate from implementation work:

- During the review pass, Codex should not modify files, stage changes, commit, push, run
  auto-fix commands, update snapshots, install dependencies, or mutate external systems.
- If a user asks for both review and implementation, Codex should finish the review first,
  keep findings separate, and only then continue to scoped implementation when authorized.
- Findings should lead the report and include evidence, impact, direction, and validation
  ideas.

When supported by the host, the skill can also emit inline comments for findings tied to
specific files and line ranges.

## Installation

Copy this folder into your Codex skills directory:

```powershell
Copy-Item -Recurse . "$env:USERPROFILE\.codex\skills\code-quality-review"
```

On macOS or Linux:

```bash
cp -R . ~/.codex/skills/code-quality-review
```

If you use `CODEX_HOME`, place the folder under `$CODEX_HOME/skills/code-quality-review`
instead.

## Usage

Ask Codex to use the skill explicitly:

```text
Use $code-quality-review to review this repository for code quality, architecture, tests,
security, and domain-specific risks without modifying code.
```

The skill can also be triggered automatically when Codex detects a code review or
refactoring-risk analysis request that matches the skill description.

## Repository Contents

- `SKILL.md`: the main Codex skill instructions
- `agents/openai.yaml`: UI metadata and default invocation prompt
- `README.md`: public project documentation
- `LICENSE`: MIT license

## License

MIT

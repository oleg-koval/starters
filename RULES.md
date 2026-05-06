# RULES.md

Hard rules for every `oleg-koval/*` starter. Applies to all contributors — human and AI.
When a starter-level doc conflicts with this file, RULES.md wins.

Agents: load this file at the start of any session in an `oleg-koval/*` starter repo.

---

## §1 Scope

These rules govern every repo that:

- is marked as a template under the `oleg-koval` GitHub account, **or**
- explicitly opts in via its own `AGENTS.md` or `CLAUDE.md`.

Future starters inherit these rules by default. Explicit exemptions must be documented in the starter's own `AGENTS.md`.

---

## §2 Hard rules (non-negotiable)

### §2.1 Functional style

Write pure functions by default. Side effects and mutation belong at system boundaries (I/O, DB, cache, network). Business logic must be side-effect free and independently testable.

### §2.2 File length — 300-line hard cap

No file may exceed **300 lines**. This applies to every language (TypeScript, Python, Go, and any others added later).

Lint must enforce this mechanically:
- **TypeScript / JavaScript**: `eslint` rule `max-lines: [error, 300]`
- **Python**: `ruff` rule `PLR0912`/`PLR0915` + a pre-commit hook running `awk 'END{if(NR>300)exit 1}' "$FILE"`
- **Go**: `revive` `max-public-structs` is not sufficient — use a pre-commit `wc -l` check: `find . -name '*.go' | xargs wc -l | awk '$1 > 300 && $2 != "total" {exit 1}'`

When a file is approaching the cap, break it into smaller focused modules before it hits the limit. Generated files (e.g., protobuf, schema snapshots) are exempt — mark them with a top-of-file comment `// @generated` and exclude them from the lint rule.

### §2.3 E2E tests > unit tests

Prefer end-to-end tests over unit tests. Reason: E2E tests prove the system works from the caller's perspective and survive refactors without rewriting mocks.

Unit tests are appropriate only for pure functions with non-trivial branching (parsers, validators, algorithm-heavy logic). Do not write unit tests that mock internal collaborators just to reach coverage targets.

### §2.4 Pre-commit hooks (mandatory)

Every starter must ship pre-commit hooks that block commits failing any of:

1. **Lint** (language-specific: eslint / ruff / golangci-lint)
2. **Format** (prettier / ruff format / gofmt)
3. **Tests** (subset fast enough to run locally — E2E full suite runs in CI)

Each starter picks its own tool (husky, lefthook, pre-commit, Makefile target) — the gates are non-negotiable. A commit that bypasses hooks (`--no-verify`) is rejected in code review.

### §2.5 Break files early

Do not wait until a file hits 300 lines to split it. Once a file exceeds ~200 lines, look for natural seams — a group of related functions, a distinct sub-concern — and extract. Components, modules, and packages are cheap; monolithic files are not.

### §2.6 No comments in code

Write self-explanatory code: well-named functions, variables, types, and constants. Do not explain *what* the code does in a comment — the code already says that.

Allowed: one short line explaining the *why* when it would otherwise be lost:
- a workaround for a specific external bug
- a hidden constraint not visible from the call site
- a subtle invariant that would surprise a reader

Not allowed: WHAT-comments (`// add 1 to counter`), block comments summarizing function bodies, or multi-line docstrings restating the function name.

### §2.7 KISS · DRY · SOLID

- **KISS**: prefer the simplest solution that solves the problem. Add complexity only when it removes more pain than it introduces.
- **DRY**: extract repeated logic once it appears a third time. Do not pre-emptively abstract.
- **SOLID**: single responsibility per module/class/function; depend on interfaces, not implementations; open for extension, closed for modification.

---

## §3 Architecture

### §3.1 Default: Vertical Slice Architecture (app and SaaS starters)

For application and SaaS starters (`saas-init` and any future app starters), the default architecture is **Vertical Slice Architecture** (Jimmy Bogard).

Each feature maps 1:1 to a vertical slice. A slice owns everything needed to execute that feature: HTTP handler, request/response DTOs, service logic, repository call, and tests. Slices do not share implementation — only shared infrastructure (DB client, logger, auth middleware) is extracted.

Benefits for a solo B2B SaaS founder:
- Maps 1:1 to Linear tickets / GitHub issues
- Eliminates cross-cutting abstraction debates
- Each slice is independently deployable in understanding and testable in isolation
- Scales to 50+ features before you need to consider DDD aggregates

### §3.2 Library starters — architecture-rule exempt

Library starters (`ts-npm-starter`, `go-starter`, `py-starter` when used as libraries) are **exempt from §3.1**. Libraries expose APIs, not features. Module cohesion and a clear public interface are the relevant constraints.

### §3.3 Fallback: Transaction Script

When Vertical Slice doesn't fit a module (e.g., a batch job, a CLI tool, a simple CRUD endpoint), use **Transaction Script** (Fowler). One function per operation, no domain objects, direct DB access.

Do not default to Anemic Domain Model + Service Layer. It accumulates debt: services grow unbounded, domain objects become dumb data bags, and the architecture degrades to a God-service pattern.

### §3.4 Planned future starters (not yet created)

Two additional starters are on the roadmap, for teams that have outgrown Vertical Slice:

- **CQRS without DDD**: read/write command split, no aggregates, suitable for high-read systems or teams that want explicit command/query separation without tactical DDD patterns.
- **CQRS with DDD**: full tactical DDD — aggregates, domain events, repositories, application services — for complex domains with significant business invariants.

These are separate starter repos, not variants of `saas-init`. Do not scaffold them in an existing starter.

---

## §4 Core principles — Unix Rules (Eric Raymond)

Apply these to every design decision, API, and module boundary.

| # | Rule | One-liner |
|---|------|-----------|
| 1 | **Modularity** | Build simple parts connected by clean interfaces |
| 2 | **Clarity** | Clarity is better than cleverness |
| 3 | **Composition** | Design programs to be connected with other programs |
| 4 | **Separation** | Separate policy from mechanism; interfaces from engines |
| 5 | **Simplicity** | Design for simplicity; add complexity only where necessary |
| 6 | **Parsimony** | Write a big program only when nothing else will do |
| 7 | **Transparency** | Design for visibility to make inspection and debugging easier |
| 8 | **Robustness** | Robustness is the child of transparency and simplicity |
| 9 | **Representation** | Fold knowledge into data so program logic can be stupid and robust |
| 10 | **Least Surprise** | In interface design, always do the least surprising thing |
| 11 | **Silence** | When a program has nothing surprising to say, it should say nothing |
| 12 | **Repair** | Repair what you can — but when you must fail, fail noisily and as soon as possible |
| 13 | **Economy** | Programmer time is expensive; prefer it over machine time |
| 14 | **Generation** | Avoid hand-hacking; write programs to write programs when you can |
| 15 | **Optimization** | Prototype before polishing; get it working before you optimize it |
| 16 | **Diversity** | Distrust all claims for one true way |
| 17 | **Extensibility** | Design for the future, because it will be here sooner than you think |

**Summary**: *Do one thing well. Work together. Handle text streams.*

---

## §5 Conceptual integrity (Brooks — Mythical Man-Month)

These are not historical curiosities — they are active failure modes in every software project.

- Software projects fail predictably. Adding people to a late project makes it later.
- Engineers overcomplicate their second project after a successful first (second-system effect). Resist the urge to redesign what already works.
- A system must have **one coherent design vision**, not a committee's compromise. Assign conceptual ownership. One sharp mind with support beats a committee of equals.
- Complexity is irreducible. Manage it with small teams, clear ownership, and conceptual integrity — not more process.

For a solo founder: this means resisting the temptation to adopt enterprise patterns before the complexity they solve actually exists.

---

## §6 Enforcement

- Every starter ships with the hooks and lint configs that mechanically enforce §2.
- CI must run the same checks. A green CI badge is evidence the gates work.
- PRs that bypass pre-commit hooks (`--no-verify`) are rejected without review.
- Disagreement with a rule → open an issue against [`oleg-koval/starters`](https://github.com/oleg-koval/starters), not a silent violation in a starter.
- AI agents must refuse to generate code that violates §2. If asked to, explain which rule is violated and propose a compliant alternative.

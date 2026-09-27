# Harness File Templates

Copy-paste templates for the files this skill creates. Each is referenced from the step that needs
it. Fill every placeholder with real project content — a template committed with `[brackets]` still
in it is worse than no file.

## Phase file (`roadmap/PHASE_N.md`)

*Used by Step 3. Sections: Goal / What This Phase Builds On / Database Changes / Implementation Steps / Test When Done.*

```markdown
# Phase N: [Title]

## Goal
[One sentence — what this phase achieves]

## What This Phase Builds On
[Which previous phases must be complete, what's assumed to exist]

## Database Changes (if any)
[SQL CREATE/ALTER statements with inline comments explaining each column]

## Implementation Steps
[Numbered steps with enough detail that a developer can execute without guessing]

## Test When Done
- [ ] [Concrete verification step]
- [ ] [Another verification step]
```

## Foundations doc (`architecture_docs/arch-foundations.md`)

*Used by Step 4. Sections: Stack Overview table / Key Decisions & Trade-offs / Constraints / Stack Evolution Log.*

```markdown
# Foundations & Tech Stack

> **Read this when:** choosing a new library/tool, evaluating an alternative technology,
> onboarding to the project, or questioning why a particular stack choice was made.
> **Related docs:** all other arch docs (they assume this stack)

---

## Stack Overview

| Layer | Choice | Why |
|-------|--------|-----|
| Language | [e.g., TypeScript] | [e.g., team expertise + type safety for complex domain] |
| Framework | [e.g., Next.js 14] | [e.g., SSR for SEO + API routes reduce infra complexity] |
| Database | [e.g., PostgreSQL] | [e.g., relational data model, JSONB for flexible metadata] |
| Auth | [e.g., NextAuth.js] | [e.g., built-in OAuth providers, session management] |
| Deployment | [e.g., Vercel] | [e.g., zero-config Next.js deploys, preview environments] |
| Package manager | [e.g., pnpm] | [e.g., faster installs, strict dependency resolution] |

## Key Decisions & Trade-offs

For each non-obvious stack choice, document:
- **What was chosen** and **what was considered**
- **Why this option won** (constraints, team skills, cost, ecosystem, performance)
- **Known trade-offs** accepted with this choice

## Constraints

[Hard constraints that limit future stack decisions — e.g., "must run on AWS due to
enterprise contract", "must support offline-first for field workers", "budget caps
rule out per-seat SaaS tools"]

## Stack Evolution Log

| Date | Change | Reason |
|------|--------|--------|
| [date] | [e.g., Migrated from Jest to Vitest] | [e.g., 3x faster test runs, native ESM support] |
```

## Domain arch doc (`architecture_docs/arch-{domain}.md`)

*Used by Step 4. Sections: header block (Read this when / Does NOT cover / Related docs) / Invariants / Core Concepts / Data Model / Flow / Edge Cases.*

```markdown
# [Domain Name]

> **Read this when:** [specific scenarios]
> **Does NOT cover:** [adjacent things a reader might wrongly expect here → which doc owns them]
> **Related docs:** [links to other arch docs that overlap]

## Invariants

[3-10 one-line rules that must hold in this domain — the things that break production when
violated. This block is mirrored into `.claude/rules/{domain}.md`; keep the two in sync.]

---

## [Section 1: Core Concepts]
[How this domain works, key decisions and WHY they were made]

## [Section 2: Data Model]
[Tables, columns, relationships — with purpose annotations]

## [Section 3: Flow]
[Step-by-step: what happens when a user does X]

## [Section 4: Edge Cases & Gotchas]
[Things that are easy to get wrong, non-obvious constraints]
```

## CLAUDE.md

*Used by Step 5. Sections: Tech Stack / Project Structure / routing table / Domain Rules / Data Model / Security Rules / Design System / Coding Rules / Env Vars / Current Phase / Build Status.*

```markdown
# [Project Name]

> [One-line description of what this project is]

## Tech Stack

| Layer | Choice | Why |
|-------|--------|-----|
| Language | [e.g., TypeScript 5.x] | [one-line rationale] |
| Framework | [e.g., Next.js 14] | [one-line rationale] |
| Database | [e.g., PostgreSQL 16] | [one-line rationale] |
| Auth | [e.g., NextAuth.js] | [one-line rationale] |
| Deployment | [e.g., Vercel] | [one-line rationale] |

> Full stack rationale, trade-offs, and evolution log → `architecture_docs/arch-foundations.md`

## Project Structure
[Brief description of directory layout]

## Architecture Docs (in `architecture_docs/`)

Read the relevant doc before modifying backend logic, API routes, or data flow:

| When working on | Read |
|----------------|------|
| Choosing a library/tool, questioning a stack choice, onboarding | `arch-foundations.md` |
| [scenario] | `arch-{domain}.md` |
| [scenario] | `arch-{domain}.md` |
| [scenario] | `arch-{domain}.md` |
| Looking up a table/column, finding a file | `arch-reference.md` |
| Maintaining the harness, phase boundary audit | `arch-harness.md` |

## Domain Rules (in `.claude/rules/`)

These load automatically when Claude reads matching files — nothing to do. Listed so it's visible
what's enforced where:

| Rule file | Applies to | Full reasoning |
|-----------|-----------|----------------|
| `[domain].md` | `[glob patterns]` | `arch-{domain}.md` |

## [Database / Data Model]
[Table names + one-line purpose, link to full schema]

## [Security Rules] (only if the project has security rules worth stating)
[Env var discipline, auth patterns, validation rules — these belong here whether or not
`security_check/SECURITY.md` exists]

## [Design System] (only if `Design.md` exists)
[Link to Design.md, key constraints like "no dark mode"]

## Coding Rules
[Project-specific conventions the user wants enforced]

## Environment Variables
[Public vs server-side, with clear labels]

## Current Phase

**Phase N: [title]** → `roadmap/PHASE_N.md`

Completed phases are historical record in `roadmap/README.md`. Don't load them.

## Build Status
[One line per phase: done / in progress / planned. Detail stays in the roadmap.]
```

## Domain rule file (`.claude/rules/{domain}.md`)

*Used by Step 5b. Sections: `paths` frontmatter / invariant list / pointer to the arch doc.*

```markdown
---
paths:
  - "src/payments/**/*.ts"
  - "src/webhooks/stripe/**"
---

# Payments Invariants

- Never mutate `invoice.status` directly — go through `transitionInvoice()`
- All amounts are integer cents, never floats
- Webhook handlers must be idempotent; Stripe retries on any non-2xx
- Refunds over the approval threshold require a second approval record

Full reasoning, state machine, and edge cases → `architecture_docs/arch-payments.md`
```

## Standard rule files (`.claude/rules/`)

*Used by Step 5b. Beyond one rule per architecture domain, generate these. Each one mechanizes a
maintenance trigger that would otherwise be an advisory table row nobody reads at the right moment.
Adjust every glob to the project's real tree and delete any that don't apply.*

### `harness.md` — keeps the harness itself in sync

```markdown
---
paths:
  - "CLAUDE.md"
  - "architecture_docs/**"
  - "roadmap/**"
  - ".claude/rules/**"
---

# Harness Invariants

- Changed a domain's invariants? Update BOTH `.claude/rules/{domain}.md` and the Invariants block in its arch doc
- Added an arch doc? Add its routing table row in CLAUDE.md
- Arch doc past ~500 lines? Split by sub-domain and repoint the routing rows
- CLAUDE.md past ~150 lines? Move a section out, leave a routing row behind
- Completed a phase? Update Current Phase and Build Status, then run the Phase Boundary Audit

Full procedure → `architecture_docs/arch-harness.md`
```

### `dependencies.md` — keeps the stack rationale honest

```markdown
---
paths:
  - "package.json"
  - "pnpm-lock.yaml"
  - "pyproject.toml"
  - "Cargo.toml"
  - "go.mod"
---

# Dependency Invariants

- Added, removed, or upgraded a dependency? Add a Stack Evolution Log entry in `arch-foundations.md`
- Changed a major layer (framework, database, auth, deployment)? Update the Stack Overview table there
  AND the Tech Stack table in CLAUDE.md — they must agree
- [project-specific: approved license list, no new runtime deps without review, etc.]

Full rationale and trade-offs → `architecture_docs/arch-foundations.md`
```

### `reference-sync.md` — keeps the lookup tables true

```markdown
---
paths:
  - "migrations/**"
  - "prisma/schema.prisma"
  - "src/app/api/**"
---

# Reference Sync

- Schema change? Update the schema section of `arch-reference.md` in the same commit
- New, renamed, or deleted route? Update the routes table in `arch-reference.md`
- `arch-reference.md` is the authority for tables, columns, routes, and paths. When it disagrees with
  another doc, fix `arch-reference.md` first, then the doc that drifted

Full lookup tables → `architecture_docs/arch-reference.md`
```

### `design.md` — only if `Design.md` exists

```markdown
---
paths:
  - "src/components/**"
  - "**/*.css"
---

# Design Invariants

- [the project's non-negotiables: color tokens only, spacing scale, no dark mode, etc.]
- New or changed component? Update the component patterns AND the file list in `Design.md`
- Never introduce a one-off color or spacing value — extend the system instead

Full spec with copy-paste code → `Design.md`
```

### `security.md` — only if `security_check/SECURITY.md` exists

```markdown
---
paths:
  - "src/auth/**"
  - "src/app/api/**"
  - "middleware.*"
  - "**/*.env*"
---

# Security Invariants

- [the project's invariants: validate at the boundary, never log PII, no secrets client-side, etc.]
- Vulnerability found or fixed? Add a numbered entry to `security_check/SECURITY.md` with the date
  and the fix location
- Never widen an auth check to make a test pass

Full threat model and audit trail → `security_check/SECURITY.md`
```

## Harness maintenance doc (`architecture_docs/arch-harness.md`)

*Used by Step 9. The full procedure — lives in the repo, read on demand, gets a routing table row
like any other arch doc.*

```markdown
# Harness Maintenance

> **Read this when:** finishing a phase, recording a decision, adding a doc, or auditing whether the
> harness still matches the codebase.
> **Does NOT cover:** what any individual domain does — that's the routing table in CLAUDE.md.
> **Related docs:** every doc in `architecture_docs/` (this one governs how they stay current)

## Invariants

- Every architecture doc has a routing table row in CLAUDE.md
- Every domain rule file's invariants match the Invariants block in its arch doc
- No fact lives in two places
- CLAUDE.md <=150 lines, rule files <=50, arch docs <=500

---

## When to Update What

File-edit triggers live in `.claude/rules/` and fire on their own. These depend on a conversation
rather than a file change, so they need deliberate attention:

| Event | Update |
|-------|--------|
| Architecture decision made or changed | The relevant `arch-{domain}.md`, with the reasoning |
| New doc or reference file added | Add a routing table row in CLAUDE.md |
| Phase completed | Mark done in `roadmap/README.md`, update Current Phase + Build Status |
| New phase or scope change | Create/update `roadmap/PHASE_N.md`, update the roadmap README |
| Non-obvious decision worth preserving | The relevant arch doc — `arch-foundations.md` if it's a stack choice |
| A deferred component is created | Add its routing row, remove it from Deferred Components |
| New directory or module created | Check whether an existing rule file's `paths` should cover it |

## Where Information Lives (don't put things in the wrong place)

- **CLAUDE.md** — Navigation + quick rules. Keep it scannable. If a section grows beyond a few
  paragraphs, move details to a dedicated doc and leave a pointer here.
- **Architecture docs** — Detailed reasoning, flows, data models, edge cases. This is where
  the "why" behind decisions lives. Update these as decisions are made during development.
  `arch-foundations.md` specifically owns tech stack rationale — always update it (and CLAUDE.md's
  Tech Stack table) when a technology is added, removed, or changed.
- **`.claude/rules/{domain}.md`** — Invariants only, auto-loaded when Claude reads matching files.
  Never reasoning — that's the arch doc's job.
- **`CLAUDE.local.md`** — Personal notes, gitignored. Nothing the team or a future session needs.
- **Auto memory** (`~/.claude/projects/<project>/memory/`) — Claude's own notes, written
  automatically and machine-local. Don't hand-maintain it, and don't copy harness content into it.
- **Roadmap phases** — Execution specs. Once a phase is complete, don't modify it (it's a
  historical record). Update the README status instead.

## Deferred Components

Optional harness components that were intentionally postponed, and what should trigger creating them:

| Component | Add when |
|-----------|----------|
| `Design.md` | [e.g., UI work starts in Phase 3] |
| `security_check/SECURITY.md` | [e.g., before the first production deploy] |

Delete a row once the doc exists. If nothing is deferred, delete this section.

## Phase Boundary Audit

At the end of every phase, before starting the next, run this check. It's what keeps the harness
from rotting into confidently wrong documentation:

1. **Referenced paths still exist.** Every file path, table, column, and route named in an arch doc
   — confirm it's still real. A doc naming a deleted function is worse than no doc, because it gets
   believed. `grep` the references rather than eyeballing them.
2. **`arch-reference.md` matches reality** — schema and routes as they actually are now.
3. **Rule globs still match.** New directories may fall outside every `paths` pattern, leaving a
   domain silently unguarded.
4. **Budgets.** CLAUDE.md ≤150 lines, rule files ≤50, arch docs ≤500.
5. **Roadmap status.** Phase marked done in `roadmap/README.md`, Current Phase and Build Status
   updated in CLAUDE.md.
6. **Auto memory.** Run `/memory` and delete entries describing the phase just finished — a note
   saying "currently migrating X" is actively misleading once the migration is done. Nothing prunes
   these automatically.

## The Self-Improvement Cycle

After completing significant work (finishing a phase, making an architecture decision, fixing
a security issue), update the harness before moving on:

1. Update the relevant architecture doc with what was decided and why
2. If the decision changes an invariant, update `.claude/rules/{domain}.md` to match
3. Update CLAUDE.md if the routing table, current phase, or build status changed
4. Confirm the decision landed in exactly one place — no duplicate copy left in CLAUDE.md
5. Mark the phase complete in the roadmap if applicable
```

## Harness Maintenance trigger (in CLAUDE.md)

*Used by Step 9. The always-loaded part: conversational triggers only, plus a pointer. Pasting the
full procedure here would spend over half the 150-line CLAUDE.md budget on instructions about the
docs rather than the project.*

```markdown
## Harness Maintenance

Full procedure, placement rules, and the phase-boundary audit → `architecture_docs/arch-harness.md`

File-edit triggers fire automatically from `.claude/rules/`. These need a deliberate decision:

| Event | Update |
|-------|--------|
| Architecture decision made | The relevant `arch-{domain}.md`, with the reasoning |
| New doc added | Add a routing table row above |
| Phase completed or started | `roadmap/README.md` + Current Phase + Build Status |
| A deferred component is created | Add its routing row, clear it from the deferred list |

At every phase boundary, run the Phase Boundary Audit in `arch-harness.md`.
```

## Delegation (in CLAUDE.md)

*Used by Step 9b. Stays always-loaded: the choice to delegate happens before any file is
opened, so there is no read to route on and no file edit to trigger on.*

```markdown
## Delegation

Hand off to a subagent with its own context window when the work is a codebase-wide sweep ("where is
X used", "audit this convention"), a question spanning 3+ architecture domains, or diagnosing a bug
in unfamiliar code. Do it inline when the work sits in one domain you already have context for, or
needs under ~3 file reads.

When delegating, name the docs in the prompt — a subagent may not inherit this file.
```

---
name: project-harness
version: 1.1.0
description: >
  Set up a structured AI-coding harness for any software project — phase roadmap, architecture docs,
  CLAUDE.md navigation hub, design system doc, security doc, and memory index. Use this skill whenever
  the user says "set up a project", "start a new project", "create a harness", "scaffold docs",
  "set up CLAUDE.md", or wants to organize their codebase for AI-assisted development. Also trigger
  when the user wants to add structure to an existing project ("add architecture docs", "organize this
  project", "make this codebase AI-friendly", "set up project docs"), or when the user wants to
  upgrade an existing harness ("upgrade harness", "update harness", "patch harness"). Works for any
  tech stack and any project size.
---

# Project Harness

You are setting up a structured documentation harness that makes a codebase navigable and maintainable
by both humans and AI agents. The harness has been battle-tested on a production project (blockchain
SaaS, 26 routes, 6+ development phases) and the patterns generalize to any software project.

The harness is NOT boilerplate — every file you create should contain real, project-specific content
derived from conversation with the user. Skeleton templates with placeholder text are useless.

---

## The Harness Components

A complete harness has these layers, each serving a distinct purpose:

| Layer | File(s) | Purpose |
|-------|---------|---------|
| **Roadmap** | `roadmap/PHASE_N.md` + `roadmap/README.md` | Execution plan — what to build, in what order |
| **Architecture docs** | `architecture_docs/arch-{topic}.md` | Decision references — how and why things work |
| **Navigation hub** | `CLAUDE.md` | Entry point — "when working on X, read Y" routing table |
| **Design system** | `Design.md` (if project has UI) | Visual spec — colors, typography, component patterns with copy-paste code |
| **Security doc** | `security_check/SECURITY.md` (if security-sensitive) | Threat model + audit trail |
| **Memory index** | `.claude/` memory system | Cross-conversation context — brief pointers to authoritative docs |

Not every project needs every layer. A CLI tool doesn't need Design.md. A static site doesn't need
security_check/. Recommend what fits — don't force components that add no value.

---

## How to Run This Skill

### Step 1: Understand the Project

**If starting a new project:**
- Ask the user to describe what they're building, who it's for, and what the core problem is
- Ask about any constraints (budget, timeline, team size, deployment target)
- Ask if they have a preferred tech stack or want help choosing one

**If mid-project:**
- Read the existing codebase: package.json / pyproject.toml / Cargo.toml, directory structure, existing README or docs
- **Check for existing CLAUDE.md** — if present, read it fully; the harness must extend it, not replace it
- Check for any existing documentation (README, docs/, architecture files, etc.)
- Run `git log --oneline -20` to understand recent development trajectory
- Identify what documentation already exists and what's missing
- Ask the user what pain points they're hitting (context loss between sessions? inconsistent patterns? hard to onboard contributors?)

**If an existing harness is detected**, check for upgrade needs before proceeding — see Step 1b.

### Step 1b: Upgrade an Existing Harness

If the project already has a harness (CLAUDE.md with a routing table + `architecture_docs/` + `roadmap/`),
check whether it was created by an older version of this skill and patch in missing components.

**Detection**: Look for these signs of an older harness:

| Missing component | Introduced in | What to do |
|-------------------|---------------|------------|
| `architecture_docs/arch-foundations.md` | v1.1.0 | Create it — gather stack rationale from the user and existing CLAUDE.md Tech Stack section |
| CLAUDE.md Tech Stack has no "Why" column | v1.1.0 | Add the "Why" column to the existing table, ask user for rationale per choice |
| No routing table entry for `arch-foundations.md` | v1.1.0 | Add the row to the routing table |
| No "Technology added, removed, or swapped" row in maintenance rules | v1.1.0 | Add it to the "When to Update What" table in CLAUDE.md |

**Upgrade rules:**

1. **Never overwrite existing content.** Read every file before modifying. Merge new sections into
   existing structure — don't replace files wholesale.
2. **Preserve user customizations.** If the user has added custom routing table rows, extra
   maintenance rules, or modified templates, keep all of it.
3. **Only patch what's missing.** If a component already exists and looks complete, skip it.
4. **Tell the user what you're doing.** Before making changes, present a list of what's missing
   and what you'll add. Wait for confirmation.
5. **One-shot upgrade.** After patching, the harness should be fully current — no need to run
   the upgrade again.

If the user explicitly asked to upgrade (e.g., "upgrade harness"), run only Step 1b and skip
the rest of the skill. If they asked to set up a harness and you detected an existing one,
ask whether they want a full rebuild or just an upgrade.

### Step 2: Tech Stack Selection & Rationale

**If starting a new project and the user is open to suggestions:**

1. Based on the project description, propose a stack with clear reasoning for each choice
2. Cover: framework, language, database, auth, deployment, package manager
3. Explain trade-offs honestly — don't just pick the trendiest option
4. Wait for the user to confirm before proceeding

**If the user already has a firm tech stack (new or existing project):**

1. Confirm each major choice (framework, language, database, auth, deployment)
2. Ask *why* each was chosen — constraints, team expertise, performance needs, ecosystem, cost, etc.
3. If the user doesn't know or care about the reasoning, note that too (e.g., "inherited from previous team")

**In both cases**, document every stack choice with its rationale. This information feeds into
the Tech Stack section of CLAUDE.md (brief) and `arch-foundations.md` (detailed). Capturing
the "why" prevents future sessions from re-debating settled decisions or suggesting incompatible
alternatives.

### Step 3: Design the Phase Roadmap

Break the project into sequential phases. Each phase should be:
- **Self-contained**: achievable independently, produces a testable result
- **Cumulative**: builds on previous phases
- **Ordered by dependency**: infrastructure first, features second, polish last

Typical phase ordering pattern:
1. Project setup + auth + database schema
2. Core data model + CRUD
3. Primary user flow (the main thing the product does)
4. Secondary flows + integrations
5. Public-facing features (verification, sharing, etc.)
6. Edge cases, cleanup, data retention
7+ Post-MVP features

Present the phase plan to the user for confirmation. Then create:

```
roadmap/
  README.md          — overview: phase list, tech stack summary, key constraints
  PHASE_1.md         — each phase file is self-contained
  PHASE_2.md
  ...
```

**Phase file structure:**
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

### Step 4: Design the Architecture Docs

Identify 3-5 key domains in the project. Good domain boundaries are areas where a developer
would need deep context to make changes safely. Common splits:

- **Identity & access** — auth, permissions, middleware, user types
- **Core workflow** — the main thing the product does (CRUD, state machine, etc.)
- **External integrations** — third-party APIs, blockchain, payment, email
- **Data & storage** — schema design, file handling, caching
- **Public surface** — API endpoints, webhooks, public pages

**Every project also gets `architecture_docs/arch-foundations.md`** — this is the tech stack
rationale doc. It captures what was chosen, why, what alternatives were considered, and any
constraints that drove the decisions. Structure:

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

This doc is a living record. When a technology is added, removed, or swapped, update the
Stack Overview table and add an entry to the Stack Evolution Log (see Step 9 maintenance rules).

For each **project-specific domain**, create `architecture_docs/arch-{domain}.md`:

```markdown
# [Domain Name]

> **Read this when working on:** [specific scenarios]
> **Related docs:** [links to other arch docs that overlap]

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

The cross-referencing between docs is critical. Each doc should link to related docs at the top
so a developer landing in one doc can find adjacent context without going back to CLAUDE.md.

### Step 5: Set Up CLAUDE.md

`CLAUDE.md` is a special file that Claude Code automatically loads into context at the start of
every conversation. It's the primary way to give Claude persistent instructions about a project.
This makes it the ideal navigation hub — the first thing Claude reads before touching any code.

**Before creating or modifying, check if CLAUDE.md already exists.** If it does:
- Read the existing file completely
- Preserve all existing content (rules, conventions, env vars, etc.)
- Merge the harness sections (routing table, architecture pointers, etc.) into the existing structure
- Don't duplicate information that's already there
- Place the "when to read what" routing table near the top — it's the most frequently used section

If no CLAUDE.md exists, create one. Either way, the end result should be scannable and link to
everything else. Target structure:

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

## [Database / Data Model]
[Table names + one-line purpose, link to full schema]

## [Security Rules] (if applicable)
[Env var discipline, auth patterns, validation rules]

## [Design System] (if applicable)
[Link to Design.md, key constraints like "no dark mode"]

## Coding Rules
[Project-specific conventions the user wants enforced]

## Environment Variables
[Public vs server-side, with clear labels]

## Build Status
[What's done, what's in progress, what's planned]
```

**The "when to read what" table is the most important part of CLAUDE.md.** It routes developers
to the right architecture doc based on what they're about to touch. Without it, the architecture
docs exist but nobody reads them at the right time. Every architecture doc you create MUST have
a corresponding row in this table.

### Step 6: Design System Doc (if project has UI)

Create `Design.md` with:
- Color palette (with exact values)
- Typography (font families, sizes, weights)
- Component patterns with **copy-paste code blocks** (not descriptions — actual code)
- Layout templates
- Gotchas (framework-specific quirks, known issues)
- List of all files that contain UI styling (so changes can be applied consistently)

The code blocks are essential — they eliminate guesswork and ensure visual consistency.

### Step 7: Security Doc (if security-sensitive)

Create `security_check/SECURITY.md` with:
- Architecture security overview (what protections exist)
- Threat model (what could go wrong)
- Resolved vulnerabilities (numbered, with date and fix location)
- Outstanding items / checklist

This file is a living audit trail, not a one-time document.

### Step 8: Set Up Memory Structure

Explain the memory system to the user: Claude Code's built-in memory (`MEMORY.md`) persists
across conversations, but by default it's just flat notes. The harness turns it into an index
that points to structured docs — so context survives not just between sessions, but grows
more useful over time.

**What goes in MEMORY.md (pointers + non-obvious context):**
- One-line pointers to authoritative docs: "Auth architecture → `architecture_docs/arch-identity-access.md`"
- Key decisions NOT derivable from code or git: "Chose Stripe over PayPal because of lower international fees"
- Current project state: "Phase 3 complete, Phase 4 in progress"
- External references: "Bug tracker is Linear project INVOICEFLOW"

**What does NOT go in MEMORY.md:**
- Anything already in architecture docs (that's duplication, and it drifts)
- Code patterns or file paths (read the code instead)
- Git history summaries (use `git log`)
- Debugging solutions (the fix is in the code, the context is in the commit message)

The key discipline: MEMORY.md is an **index**, not a **store**. Every entry should either
point to where the real information lives, or capture something that has no other home.

### Step 9: Write Harness Maintenance Rules into CLAUDE.md

The harness is only useful if future Claude sessions know how to maintain it. Without explicit
instructions, the docs go stale after a few sessions and become misleading — worse than no docs.

Add a **Harness Maintenance** section to CLAUDE.md that encodes the self-improvement cycle.
This section tells every future Claude session how to keep the harness alive:

```markdown
## Harness Maintenance

This project uses a structured documentation harness. Follow these rules to keep it accurate:

### When to Update What

| Event | Update |
|-------|--------|
| Architecture decision made or changed | Update the relevant `architecture_docs/arch-{domain}.md` with the decision and reasoning |
| Technology added, removed, or swapped | Update `arch-foundations.md` Stack Overview table + add a Stack Evolution Log entry. Update the Tech Stack table in CLAUDE.md to match |
| New doc or reference file added | Add a routing entry to the "When working on / Read" table above |
| Phase completed | Mark phase as done in `roadmap/README.md`, update Build Status section below |
| New phase or scope change | Create/update `roadmap/PHASE_N.md`, update roadmap README |
| Non-obvious decision worth preserving across sessions | Add a brief pointer to MEMORY.md (not a copy — link to the authoritative doc) |
| New database table or column | Update `arch-reference.md` schema section |
| New API route | Update `arch-reference.md` routes table |
| UI component added or changed | Update `Design.md` component patterns and file list |
| Security vulnerability found or fixed | Update `security_check/SECURITY.md` with numbered entry |

### Where Information Lives (don't put things in the wrong place)

- **CLAUDE.md** — Navigation + quick rules. Keep it scannable. If a section grows beyond a few
  paragraphs, move details to a dedicated doc and leave a pointer here.
- **Architecture docs** — Detailed reasoning, flows, data models, edge cases. This is where
  the "why" behind decisions lives. Update these as decisions are made during development.
  `arch-foundations.md` specifically owns tech stack rationale — always update it (and CLAUDE.md's
  Tech Stack table) when a technology is added, removed, or changed.
- **MEMORY.md** — Brief one-line pointers to docs + things NOT derivable from code or git
  history (e.g., "auth middleware rewrite driven by legal compliance, not tech debt").
  Never duplicate content from architecture docs here.
- **Roadmap phases** — Execution specs. Once a phase is complete, don't modify it (it's a
  historical record). Update the README status instead.

### The Self-Improvement Cycle

After completing significant work (finishing a phase, making an architecture decision, fixing
a security issue), update the harness before moving on:

1. Update the relevant architecture doc with what was decided and why
2. Update CLAUDE.md if the routing table or build status changed
3. Add a MEMORY.md pointer if the decision isn't obvious from code
4. Mark the phase complete in the roadmap if applicable
```

Adapt this template to the project — add project-specific rules if needed (e.g., "When making
UI changes, update ALL files listed in Design.md"). The goal is that a fresh Claude session
reading CLAUDE.md knows exactly how to maintain the system, not just use it.

### Step 10: Present and Confirm

Before creating any files, present the full harness plan to the user:
- Which components you'll create
- The directory structure
- The architecture doc domains you identified
- The phase breakdown

Wait for explicit confirmation. Then create all files.

---

## Important Principles

**Don't create empty shells.** Every file should have real content based on what you learned
in the conversation. A PHASE_1.md that says "TBD" is worse than no file at all.

**The harness evolves — and that's the whole point.** A harness that's set up once and never
updated is just stale documentation. The real value comes from the maintenance cycle: decisions
get recorded in architecture docs, CLAUDE.md stays current, MEMORY.md points to what matters.
The maintenance rules (Step 9) baked into CLAUDE.md are what make this happen automatically
across sessions — without them, the harness decays within weeks.

**"When to read what" is the glue.** The routing table in CLAUDE.md is what makes the whole
system work. Without it, docs exist in isolation and get ignored. With it, developers are
guided to the right context at the right time. When adding any new doc to the harness, always
add a corresponding routing entry in CLAUDE.md.

**Keep CLAUDE.md scannable.** It's a navigation hub, not a novel. If a section grows beyond
a few paragraphs, it should become its own doc with a pointer from CLAUDE.md.

**Adapt to the project.** A weekend hackathon needs a lighter harness than a production SaaS.
A solo developer needs different docs than a team of 10. Scale the harness to match the project's
actual complexity — over-documenting a simple project creates maintenance burden that outweighs
the benefit.

---

## Changelog

### v1.1.0

- **Tech stack rationale**: Step 2 now captures *why* each stack choice was made, for both new
  and existing projects. CLAUDE.md Tech Stack table includes a "Why" column.
- **`arch-foundations.md`**: New mandatory architecture doc for every project — full stack
  rationale, trade-offs, constraints, and a Stack Evolution Log for tracking changes over time.
- **Maintenance rules**: Added tech stack change trigger — when a technology is added, removed,
  or swapped, both `arch-foundations.md` and CLAUDE.md Tech Stack table must be updated.
- **Upgrade path (Step 1b)**: Detects existing harnesses from older versions and patches in
  missing components without overwriting user content. Supports "upgrade harness" trigger.
- **Versioning**: Added `version` field to skill frontmatter.

### v1.0.0

- Initial release: phase roadmap, architecture docs, CLAUDE.md navigation hub, design system
  doc, security doc, memory index, and self-maintaining harness cycle.

---
name: project-harness
version: 1.5.0
description: >
  Set up a structured AI-coding harness for any software project — phase roadmap, architecture docs,
  CLAUDE.md navigation hub, path-scoped domain rules, design system doc, and security doc. Use this
  skill whenever
  the user says "set up a project", "start a new project", "create a harness", "scaffold docs",
  "set up CLAUDE.md", or wants to organize their codebase for AI-assisted development. Also trigger
  when the user wants to add structure to an existing project ("add architecture docs", "organize this
  project", "make this codebase AI-friendly", "set up project docs"), or when the user wants to
  upgrade an existing harness ("upgrade harness", "update harness", "patch harness"), or when the
  user wants to add a previously deferred component ("add a design doc", "add a security doc").
  Works for any tech stack and any project size.
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
| **Domain rules** | `.claude/rules/{domain}.md` | Invariants that auto-load when Claude reads matching files |
| **Design system** | `Design.md` *(optional)* | Visual spec — colors, typography, component patterns with copy-paste code |
| **Security doc** | `security_check/SECURITY.md` *(optional)* | Threat model + audit trail |
| **Context placement** | *(rules only — no new file)* | Every project fact has exactly one home, and CLAUDE.md stays thin |

The roadmap, architecture docs, navigation hub, and domain rules are always created. **`Design.md`
and `security_check/SECURITY.md` are optional, and the user decides — not you.** Ask about both
explicitly at the start (Step 1c). Each can be included now, deferred to later in the project, or
skipped as not applicable. Recommend what fits — a CLI tool doesn't need Design.md, a static site
doesn't need security_check/ — but the call is the user's, and a deferral gets recorded rather than
forgotten.

### Context Budgets

Two surfaces load in full at the start of every session: `CLAUDE.md` and Claude Code's auto memory
index. Everything else is retrieved on demand. At scale that distinction is the whole game —
always-loaded content competes with the actual task for the same window, so the harness keeps it
small and pushes detail into files that are opened only when relevant.

Hold these budgets. They're numbers rather than judgment calls, so a future session can check them:

| File | Budget | When exceeded |
|------|--------|---------------|
| `CLAUDE.md` | ≤ 150 lines | Move a section to an arch doc or domain rule, leave a routing row behind |
| `.claude/rules/{domain}.md` | ≤ 50 lines | It's carrying reasoning — move that to the arch doc |
| `architecture_docs/arch-{domain}.md` | ≤ 500 lines | Split by sub-domain; the original becomes an index |
| `roadmap/PHASE_N.md` | no limit | Only the active phase is ever linked from CLAUDE.md |

Anthropic's guidance for CLAUDE.md is to target under 200 lines, since longer files consume more
context and reduce adherence. The 150-line budget leaves headroom for rules the user adds later.

---

## How to Run This Skill

File templates live in `references/templates.md`, and the upgrade procedure in
`references/upgrade.md`, both alongside this file. Each step below says when to read them. **Don't
write a harness file from memory when a template exists for it** — the templates carry structure the
maintenance rules depend on.

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

If the project already has a harness (CLAUDE.md with a routing table + `architecture_docs/` +
`roadmap/`), it may predate the current version of this skill and be missing components.

→ **Read `references/upgrade.md` now.** It has the version-by-version detection table and the rules
for patching without overwriting user content.

If the user explicitly asked to upgrade ("upgrade harness"), run only that reference and skip the
rest of this skill. If they asked to set up a harness and you detected an existing one, ask whether
they want a full rebuild or just an upgrade.


### Step 1c: Decide Which Optional Components to Include

Before planning anything, settle which optional docs are part of this harness. Raise both questions
in the opening conversation — don't decide silently and don't create these files on your own judgment.

1. **Design system doc** — if the project has any UI surface, ask: "Do you want `Design.md` now
   (colors, typography, component patterns with copy-paste code), or leave it until the visual
   direction is settled?" For projects with no UI at all (CLI tools, libraries, background services),
   say it doesn't apply and move on.
2. **Security doc** — if the project touches auth, payments, user data, uploads, or anything
   internet-facing, ask: "Do you want `security_check/SECURITY.md` now (threat model + audit trail),
   or add it closer to production?"

Give a recommendation with each question rather than presenting a bare menu. A design-led product
benefits from `Design.md` on day one; an internal API probably never needs it. A project handling
payments or PII should get `SECURITY.md` early; a weekend prototype can reasonably defer it.

Three valid answers for each: **include now**, **defer**, or **not applicable**.

**If a component is deferred**, don't silently drop it:

- Record it under a "Deferred Components" heading in the CLAUDE.md Harness Maintenance section,
  with the trigger for adding it — e.g. "Design.md — add when UI work starts (Phase 3)",
  "SECURITY.md — add before the first production deploy".
- Leave its routing table row out until the doc exists. A routing row pointing at a missing file is
  worse than no row.
- If there's an obvious phase where it becomes relevant, note it in that `PHASE_N.md` too.

**If a component is not applicable**, skip it entirely — no deferred entry, no routing row.

**Adding a deferred component later**: when the user asks for it on a project that already has a
harness ("add a design doc", "add a security doc"), run only Step 6 or Step 7, then add the routing
table row and delete the entry from Deferred Components.

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
→ **Template:** `references/templates.md` § Phase file. Read it before writing the file.
Sections: Goal / What This Phase Builds On / Database Changes / Implementation Steps / Test When Done.

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

→ **Template:** `references/templates.md` § Foundations doc. Read it before writing the file.
Sections: Stack Overview table / Key Decisions & Trade-offs / Constraints / Stack Evolution Log.

This doc is a living record. When a technology is added, removed, or swapped, update the
Stack Overview table and add an entry to the Stack Evolution Log (see Step 9 maintenance rules).

For each **project-specific domain**, create `architecture_docs/arch-{domain}.md`:

→ **Template:** `references/templates.md` § Domain arch doc. Read it before writing the file.
Sections: header block (Read this when / Does NOT cover / Related docs) / Invariants / Core Concepts / Data Model / Flow / Edge Cases.

**The first 30 lines have to stand alone.** At scale these docs outgrow a single read, and a
partial read is where confident wrong answers come from. The header block — read this when, does
NOT cover, related docs, invariants — must be enough for Claude to tell whether it opened the right
doc and to act safely if it reads nothing further. **Does NOT cover** is the line that prevents an
authoritative-sounding answer about something the doc never addressed.

**Cap each doc at ~500 lines.** Past that, split by sub-domain (`arch-payments-billing.md`,
`arch-payments-webhooks.md`) and turn the original into an index whose routing rows point at the
splits. A 2,000-line doc gets read partially or not at all, which defeats the purpose.

**`arch-reference.md` is the anti-hallucination anchor.** It's the single authority for tables,
columns, routes, and the file map — a flat lookup that makes any claim checkable in one read. Every
statement elsewhere about a table, column, route, or path should be verifiable there, and when two
docs disagree, `arch-reference.md` is what gets corrected first. Give it plain lookup tables, no prose.

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

→ **Template:** `references/templates.md` § CLAUDE.md. Read it before writing the file.
Sections: Tech Stack / Project Structure / routing table / Domain Rules / Data Model / Security Rules / Design System / Coding Rules / Env Vars / Current Phase / Build Status.

**The "when to read what" table is the most important part of CLAUDE.md.** It routes developers
to the right architecture doc based on what they're about to touch. Without it, the architecture
docs exist but nobody reads them at the right time. Every architecture doc you create MUST have
a corresponding row in this table.

**Budget: ≤150 lines.** CLAUDE.md loads in full on every task, so every line costs context whether
or not it's relevant to what's being worked on. When a section outgrows a few paragraphs, move it
to an arch doc and leave a routing row, or to a domain rule if it's an invariant. Link only the
*current* phase — a 12-phase project should cost the same at launch as a 2-phase one.

### Step 5b: Domain Rules in `.claude/rules/`

The routing table is **advisory** — it works only if Claude reads CLAUDE.md, recognizes that the
task matches a row, and chooses to open the doc. That's a soft instruction competing with
everything else in context, and on a large project it's exactly where "Claude forgot the rule"
comes from.

`.claude/rules/` is the **mechanical** counterpart. A rule file with a `paths` field in its
frontmatter loads automatically when Claude reads a matching file — no decision, no routing lookup:

→ **Template:** `references/templates.md` § Domain rule file. Read it before writing the file.
Sections: `paths` frontmatter / invariant list / pointer to the arch doc.

**Create one rule file per architecture domain.** Each holds that domain's invariants — mirrored
from the arch doc's Invariants block — plus a pointer to the full doc. Nothing else. Reasoning,
history, and flows stay in the arch doc, retrieved on demand.

Rules for generating these:

- **Invariants only, ≤50 lines.** If it explains *why*, it belongs in the arch doc. A rule file
  past 50 lines is carrying reasoning it shouldn't.
- **`paths` must match real directories.** Check the globs against the actual tree — a pattern
  matching nothing is a rule that silently never fires. Syntax: `src/**/*.ts`, `src/api/**/*`,
  `src/**/*.{ts,tsx}`.
- **Every rule file ends with a pointer** to its arch doc, so the reasoning is one hop away.
- **A rule file with no `paths` loads unconditionally** — same cost as CLAUDE.md. Reserve that for
  genuinely project-wide invariants, and prefer CLAUDE.md for those anyway.
- **Keep it in sync with the arch doc's Invariants block.** This is the one duplication the harness
  accepts, because the two serve different retrieval paths. Step 9's maintenance rules make
  updating both a single event.

**This complements the routing table, it does not replace it.** Path-scoped rules fire when Claude
*reads a matching file*, so they cover implementation but not planning. The routing table is what
gets the arch doc open before any file is touched. Generate both.

**Mechanize every file-triggered maintenance rule.** This is the general form, and it applies well
beyond domain invariants: **if a trigger is "you edited file X," it belongs in a rule, not in an
advisory table.** Roughly half the harness's maintenance triggers are file-triggered — a dependency
added to `package.json`, a migration written, a route added, a component changed, an arch doc grown
past its budget. Each one moved into `.claude/rules/` fires on its own instead of depending on
someone remembering a table row.

So beyond the per-domain rules, generate a standard set:

| Rule file | Fires when Claude edits | Enforces |
|-----------|------------------------|----------|
| `harness.md` | `CLAUDE.md`, `architecture_docs/**`, `roadmap/**`, `.claude/rules/**` | Invariant sync, routing rows, budgets |
| `dependencies.md` | `package.json`, `pyproject.toml`, `Cargo.toml`, `go.mod`, lockfiles | `arch-foundations.md` + CLAUDE.md Tech Stack stay true |
| `reference-sync.md` | migrations, schema files, route directories | `arch-reference.md` schema and routes tables stay true |
| `design.md` *(if `Design.md` exists)* | component directories, stylesheets | Design invariants; component patterns + file list updated |
| `security.md` *(if `SECURITY.md` exists)* | auth, api, middleware, env files | Security invariants; numbered audit entries |

→ **Template:** `references/templates.md` § Standard rule files. Adjust every glob to the project's
real tree — a pattern matching nothing is a rule that silently never fires.

### Step 6: Design System Doc (optional — only if confirmed in Step 1c)

Skip this step unless the user asked for it. If they deferred it, make sure it's listed under
Deferred Components in CLAUDE.md instead.

Create `Design.md` with:
- Color palette (with exact values)
- Typography (font families, sizes, weights)
- Component patterns with **copy-paste code blocks** (not descriptions — actual code)
- Layout templates
- Gotchas (framework-specific quirks, known issues)
- List of all files that contain UI styling (so changes can be applied consistently)

The code blocks are essential — they eliminate guesswork and ensure visual consistency.

Add the routing table row for `Design.md` in CLAUDE.md as part of creating it, and remove any
Deferred Components entry for it.

### Step 7: Security Doc (optional — only if confirmed in Step 1c)

Skip this step unless the user asked for it. If they deferred it, make sure it's listed under
Deferred Components in CLAUDE.md instead.

Create `security_check/SECURITY.md` with:
- Architecture security overview (what protections exist)
- Threat model (what could go wrong)
- Resolved vulnerabilities (numbered, with date and fix location)
- Outstanding items / checklist

This file is a living audit trail, not a one-time document.

Add the routing table row for `security_check/SECURITY.md` in CLAUDE.md as part of creating it, and
remove any Deferred Components entry for it.

### Step 8: Context Placement Rules

**The harness creates no memory file of its own.** Claude Code already has two mechanisms for
carrying context across sessions, and the harness works with them instead of adding a third:

| | Who writes it | Where | Committed? |
|---|---|---|---|
| **CLAUDE.md** | The user (and this skill, on their behalf) | project root | Yes |
| **Auto memory** | Claude, on its own | `~/.claude/projects/<project>/memory/` | No — machine-local |

`CLAUDE.md` is the user-authored channel, and it's the one the harness owns (Step 5). **Auto
memory is Claude's own — don't create, structure, or manage it.** It's on by default, it skips
anything derivable from the codebase and anything CLAUDE.md already says, and it needs no setup.
Mention it to the user once, note that it's machine-local (not shared with teammates, not carried
to another machine), and move on.

A root-level `MEMORY.md` is not a thing Claude Code loads — only CLAUDE.md, `CLAUDE.local.md`,
`AGENTS.md`, and `.claude/rules/` load at launch. Don't create one.

What this step actually does is give every kind of project fact exactly one home. Explain the
table below to the user, and follow it yourself for the rest of the project:

| Fact | Home |
|------|------|
| Which doc to read for a given task | CLAUDE.md routing table |
| Standing rules and conventions | CLAUDE.md Coding Rules |
| Phase status | CLAUDE.md Build Status + `roadmap/README.md` |
| A decision and the reasoning behind it | the relevant `architecture_docs/arch-{domain}.md` |
| Why a technology was chosen | `arch-foundations.md` |
| Table, column, route, or file lookup | `arch-reference.md` |
| External references (issue tracker, dashboard, staging URL) | `arch-reference.md` |
| What to build next | `roadmap/PHASE_N.md` |
| Personal notes that shouldn't be committed | `CLAUDE.local.md` (add to `.gitignore`) |

**Nothing gets two homes.** A fact duplicated between CLAUDE.md and an architecture doc will
drift, and the stale copy is worse than no copy. When in doubt, put the detail in the architecture
doc and leave a routing table row pointing at it.

**Keep CLAUDE.md thin.** It loads in full at the start of every session, so everything in it costs
context whether or not it's relevant to the current task. Anthropic's guidance is to target under
200 lines — longer files consume more context and reduce adherence. Detail belongs in an
architecture doc that the routing table reaches on demand. That's the entire reason the routing
table exists.

**`@import` is not a context shortcut.** A file pulled into CLAUDE.md with `@path` syntax is
expanded and loaded at launch just like the rest of the file, so splitting content out organizes
it without saving any context. Use imports for always-relevant content you want in its own file,
never to smuggle long reference docs into every session.

### Step 9: Harness Maintenance Rules

The harness is only useful if future sessions know how to maintain it. Without that, the docs go
stale within a few sessions and become misleading — worse than no docs.

The maintenance rules are themselves subject to the harness's own budget, so they split across three
places rather than being pasted into CLAUDE.md:

1. **`architecture_docs/arch-harness.md`** — the full procedure: the event table, where information
   lives, deferred components, the Phase Boundary Audit, the self-improvement cycle. An arch doc like
   any other, with its own routing table row, read on demand.
2. **~13 lines in CLAUDE.md** — only the triggers that depend on a conversation rather than a file
   edit, plus a pointer to the doc. The full procedure pasted here runs 80 lines: over half of the
   150-line budget spent on instructions about the docs instead of the project.
3. **`.claude/rules/harness.md`** — the file-edit triggers, firing automatically when Claude touches
   CLAUDE.md, an arch doc, the roadmap, or a rule file. This is what makes maintenance mechanical
   instead of advisory. Editing an arch doc is exactly the moment you need reminding to sync its rule
   file and its routing row, and a rule fires there whether or not anyone remembered to look.

→ **Templates:** `references/templates.md` § Harness maintenance doc, § Harness Maintenance trigger,
and § Standard rule files (for `harness.md`).

Adapt to the project — add project-specific rules (e.g. "when making UI changes, update ALL files
listed in Design.md"), drop rows for components that don't exist, and drop Deferred Components if
nothing was deferred.

### Step 9b: Delegation Rules

On a large codebase the failure mode isn't a missing doc — it's one session trying to hold the whole
system at once. Subagents are the lever: a fresh context window that reads five docs and forty files
and returns a conclusion rather than the raw material.

This one stays in CLAUDE.md rather than moving to a doc. The choice to delegate happens *before* any
file is opened, so there's no read to route on and no file edit to trigger a rule — it has to be
already in context. Keep it to a few lines.

→ **Template:** `references/templates.md` § Delegation.

### Step 10: Present and Confirm

Before creating any files, present the full harness plan to the user:
- Which components you'll create now, which are deferred (with their triggers), and which don't apply
- The directory structure
- The architecture doc domains you identified, and the `paths` globs each domain rule will match
- The phase breakdown

Wait for explicit confirmation. Then create all files.

---

## Important Principles

**Always-loaded is the scarce resource.** `CLAUDE.md` and the auto memory index load on every task;
everything else is retrieved only when relevant. Design for that split — small always-loaded
surfaces that route well, and detailed docs that are cheap to open and safe to read partially. A
harness that puts everything in CLAUDE.md is just a slower way of putting everything in context.

**Two retrieval paths, both needed.** The routing table is advisory and fires on intent; path-scoped
rules are mechanical and fire on file reads. Neither covers the other's case. Generate both.

**Don't create empty shells.** Every file should have real content based on what you learned
in the conversation. A PHASE_1.md that says "TBD" is worse than no file at all.

**The harness evolves — and that's the whole point.** A harness that's set up once and never
updated is just stale documentation. The real value comes from the maintenance cycle: decisions
get recorded in architecture docs, and CLAUDE.md stays current so the routing table still points
at the right ones.
The maintenance rules (Step 9) baked into CLAUDE.md are what make this happen automatically
across sessions — without them, the harness decays within weeks.

**"When to read what" is the glue.** The routing table in CLAUDE.md is what makes the whole
system work. Without it, docs exist in isolation and get ignored. With it, developers are
guided to the right context at the right time. When adding any new doc to the harness, always
add a corresponding routing entry in CLAUDE.md.

**Keep CLAUDE.md scannable.** It's a navigation hub, not a novel. If a section grows beyond
a few paragraphs, it should become its own doc with a pointer from CLAUDE.md.

**Optional means the user decides.** `Design.md` and `security_check/SECURITY.md` are proposed with
a recommendation, never assumed. A deferred component is recorded with its trigger so it resurfaces
at the right time instead of being quietly lost.

**Adapt to the project.** A weekend hackathon needs a lighter harness than a production SaaS.
A solo developer needs different docs than a team of 10. Scale the harness to match the project's
actual complexity — over-documenting a simple project creates maintenance burden that outweighs
the benefit.

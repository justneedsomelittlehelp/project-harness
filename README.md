# Project Harness

A Claude Code skill that sets up a structured documentation harness for AI-assisted software development.

Born from building a production SaaS (StationHash — blockchain document verification, 26 routes, 6+ development phases) entirely with Claude Code. The patterns that made that project manageable are now captured in this skill for any project.

## What It Does

Sets up a self-maintaining documentation system that gives Claude Code structured context across sessions:

- **Phase roadmap** — sequential build phases with testable milestones
- **Architecture docs** — domain-split reference docs with cross-references
- **CLAUDE.md navigation hub** — "when working on X, read Y" routing table (auto-loaded every session)
- **Domain rules** — invariants in `.claude/rules/` that auto-load when Claude reads matching files, so maintenance triggers fire mechanically instead of waiting to be remembered
- **Design system doc** — colors, typography, component patterns with copy-paste code (optional)
- **Security doc** — threat model + audit trail (optional)
- **Context placement rules** — every project fact has exactly one home, so nothing drifts
- **Maintenance rules** — self-improvement cycle baked into CLAUDE.md so the harness stays current

The design and security docs are optional. The skill asks up front whether you want each one now,
deferred to later in the project, or not at all — and a deferral is recorded in CLAUDE.md with the
trigger for adding it, so it resurfaces at the right time.

## Why This Exists

Claude Code ships with two context mechanisms: `CLAUDE.md`, which you write, and auto memory, which Claude writes for itself. Both load in full at the start of every session, and both deliberately stay out of architecture — auto memory skips anything derivable from the codebase, and CLAUDE.md works best kept under ~200 lines.

That leaves a gap on anything multi-phase. Long-form reasoning — why this design, what was rejected, what breaks if you change it — has nowhere to live, so it gets re-derived every session and settled decisions get re-debated.

This harness fills that gap: architecture docs hold the detail, and a routing table in CLAUDE.md pulls the right one into context at the right moment, so CLAUDE.md itself stays thin. A phase roadmap covers what's next, and maintenance rules keep all of it current as the project moves. The harness does not touch auto memory — that's Claude's own, and it needs no setup.

## Installation

### Via plugin system (recommended)

```bash
# Step 1: Add the marketplace
/plugin marketplace add justneedsomelittlehelp/project-harness

# Step 2: Install the plugin
/plugin install project-harness@project-harness
```

### Manual installation

The skill is a directory (`SKILL.md` plus `references/`), so copy the whole thing:

```bash
git clone --depth 1 https://github.com/justneedsomelittlehelp/project-harness /tmp/project-harness

# Global (available in all projects)
mkdir -p ~/.claude/skills
cp -R /tmp/project-harness/plugins/project-harness/skills/project-harness ~/.claude/skills/

# Or project-local
mkdir -p .claude/skills
cp -R /tmp/project-harness/plugins/project-harness/skills/project-harness .claude/skills/

rm -rf /tmp/project-harness
```

### Usage

Start a new Claude Code session and say:

- "Set up a project harness" (new project)
- "Add a harness to this project" (existing project)
- "Organize this codebase for AI development"

The skill walks through:
1. Understanding your project (or reading existing code)
2. Tech stack selection (if new project)
3. Phase roadmap design
4. Architecture doc creation
5. CLAUDE.md with routing table
6. Optional docs — design system and security: include now, defer, or skip
7. Context placement rules — where each kind of fact lives
8. Maintenance rules (the self-improvement cycle)

## What Gets Created

```
your-project/
├── CLAUDE.md                          # Navigation hub (auto-loaded by Claude Code)
│   ├── Tech stack + project structure
│   ├── "When working on X, read Y" routing table
│   ├── Build status
│   └── Harness maintenance rules      # Self-improvement cycle
├── roadmap/
│   ├── README.md                      # Phase overview + status
│   ├── PHASE_1.md                     # Self-contained phase specs
│   ├── PHASE_2.md
│   └── ...
├── architecture_docs/
│   ├── arch-foundations.md            # Tech stack rationale + evolution log
│   ├── arch-identity-access.md        # Auth, permissions, middleware
│   ├── arch-{core-workflow}.md        # Your project's main flow
│   ├── arch-{integrations}.md         # External APIs, services
│   └── arch-reference.md              # File map, schema, route table
├── .claude/rules/
│   └── {domain}.md                    # Invariants, auto-load on matching file reads
├── Design.md                          # (optional — include now or defer)
└── security_check/
    └── SECURITY.md                    # (optional — include now or defer)
```

## Key Concepts

### The Routing Table

The most important pattern. Lives in CLAUDE.md:

```markdown
| When working on | Read |
|----------------|------|
| Auth, middleware, permissions | `arch-identity-access.md` |
| Invoice CRUD, status transitions | `arch-invoice-lifecycle.md` |
| Stripe, webhooks, payments | `arch-payments.md` |
| Looking up a table/column/file | `arch-reference.md` |
```

This routes Claude to the right context at the right time — without it, architecture docs exist but get ignored.

It's advisory, though: it only works if Claude reads CLAUDE.md, matches the task to a row, and opens the doc. So the harness also generates `.claude/rules/{domain}.md` files scoped with a `paths` glob, which Claude Code loads *automatically* when Claude reads a matching file. Those carry the domain's invariants; the arch doc carries the reasoning. The table fires on intent, the rules fire on file reads — different moments, both needed.

### Mechanical Over Advisory

Any rule phrased "when you edit file X, update Y" is generated as a path-scoped rule rather than a
table row, so it fires on its own. That covers dependency manifests (keeping `arch-foundations.md`
honest), migrations and routes (keeping `arch-reference.md` true), component and stylesheet edits,
auth and env files, and the harness's own files. What's left in CLAUDE.md is the handful of triggers
that depend on a conversation rather than a file change.

### Context Budgets

Only `CLAUDE.md` and Claude Code's auto memory index load on every task. Everything else is retrieved on demand, which is what makes the harness scale: CLAUDE.md ≤150 lines, rule files ≤50, arch docs ≤500 with a mandatory split past that, and only the *active* roadmap phase ever linked.

### The Self-Improvement Cycle

Baked into CLAUDE.md so every session maintains the harness:

1. Decision made → update architecture doc with reasoning
2. Routing table changed → update CLAUDE.md
3. New doc created → add a routing table row
4. Phase completed → update roadmap status

### Information Hierarchy

| Layer | Contains | Updated |
|-------|----------|---------|
| CLAUDE.md | Navigation + quick rules | Every session (if routing/status changed) |
| Architecture docs | Detailed reasoning + flows | When decisions are made |
| `CLAUDE.local.md` | Personal notes, gitignored | As needed |
| Auto memory (Claude Code's own) | Claude's notes to itself | Automatic — the harness doesn't touch it |
| Roadmap | Execution specs | Phase completion |

## Works With

- Any programming language or framework
- New projects or existing codebases
- Solo developers or teams
- Web apps, CLI tools, APIs, mobile apps

## License

MIT

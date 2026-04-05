# Project Harness

A Claude Code skill that sets up a structured documentation harness for AI-assisted software development.

Born from building a production SaaS (StationHash — blockchain document verification, 26 routes, 6+ development phases) entirely with Claude Code. The patterns that made that project manageable are now captured in this skill for any project.

## What It Does

Sets up a self-maintaining documentation system that gives Claude Code structured context across sessions:

- **Phase roadmap** — sequential build phases with testable milestones
- **Architecture docs** — domain-split reference docs with cross-references
- **CLAUDE.md navigation hub** — "when working on X, read Y" routing table (auto-loaded every session)
- **Design system doc** — colors, typography, component patterns with copy-paste code (UI projects only)
- **Security doc** — threat model + audit trail (security-sensitive projects only)
- **Memory index** — pointers to authoritative docs, not duplicates
- **Maintenance rules** — self-improvement cycle baked into CLAUDE.md so the harness stays current

## Why This Exists

Claude Code's default memory (`MEMORY.md`) is flat notes — it works for simple projects but falls apart on anything multi-phase. Context gets lost between sessions, decisions get re-debated, architecture drifts.

This harness solves that by creating a structured documentation layer that Claude reads at the right time (via the routing table) and maintains itself (via the maintenance rules). Each session builds on the last instead of starting from scratch.

## Installation

### Via plugin system (recommended)

```bash
# Step 1: Add the marketplace
/plugin marketplace add justneedsomelittlehelp/project-harness

# Step 2: Install the plugin
/plugin install project-harness@project-harness
```

### Manual installation

Copy the `SKILL.md` file to your Claude Code skills directory:

```bash
# Global (available in all projects)
mkdir -p ~/.claude/skills/project-harness
curl -o ~/.claude/skills/project-harness/SKILL.md \
  https://raw.githubusercontent.com/justneedsomelittlehelp/project-harness/main/plugins/project-harness/skills/project-harness/SKILL.md

# Or project-local
mkdir -p .claude/skills/project-harness
cp SKILL.md .claude/skills/project-harness/
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
6. Design/security docs (if applicable)
7. Memory structure
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
│   ├── arch-identity-access.md        # Auth, permissions, middleware
│   ├── arch-{core-workflow}.md        # Your project's main flow
│   ├── arch-{integrations}.md         # External APIs, services
│   └── arch-reference.md              # File map, schema, route table
├── Design.md                          # (UI projects only)
└── security_check/
    └── SECURITY.md                    # (security-sensitive projects only)
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

### The Self-Improvement Cycle

Baked into CLAUDE.md so every session maintains the harness:

1. Decision made → update architecture doc with reasoning
2. Routing table changed → update CLAUDE.md
3. Non-obvious context → add MEMORY.md pointer
4. Phase completed → update roadmap status

### Information Hierarchy

| Layer | Contains | Updated |
|-------|----------|---------|
| CLAUDE.md | Navigation + quick rules | Every session (if routing/status changed) |
| Architecture docs | Detailed reasoning + flows | When decisions are made |
| MEMORY.md | Pointers + non-obvious context | When something has no other home |
| Roadmap | Execution specs | Phase completion |

## Works With

- Any programming language or framework
- New projects or existing codebases
- Solo developers or teams
- Web apps, CLI tools, APIs, mobile apps

## License

MIT

# Changelog

Versions of the `project-harness` skill.

### v1.4.0

Scaling pass, aimed at large codebases where the limit is context rather than documentation.

- **Domain rules (`.claude/rules/`)** — new mandatory component (Step 5b). One rule file per
  architecture domain holding that domain's invariants plus a pointer to the full doc, scoped with
  a `paths` glob so it auto-loads when Claude reads matching files. Converts the routing table's
  advisory "you should read this" into mechanical loading. Both are generated; they cover different
  moments (planning vs. implementation).
- **Context budgets** — explicit line limits replacing "keep it scannable": CLAUDE.md ≤150, rule
  files ≤50, arch docs ≤500 with a mandatory split past that. Checkable by a future session.
- **Arch docs open with a self-sufficient header** — read this when / does NOT cover / related docs
  / invariants, so the first 30 lines are enough to tell whether the doc is the right one and a
  partial read is still safe. `arch-reference.md` is named the anti-hallucination anchor.
- **Phase-scoped loading** — CLAUDE.md links only the active phase; completed phases are archive.
- **Delegation rules (Step 9b)** — when to hand work to a subagent with a fresh context window,
  written into CLAUDE.md, including the gotcha that a subagent may not inherit project instructions.
- **Progressive disclosure** — `SKILL.md` dropped from 801 to 461 lines (~10.6k → ~6.4k tokens on
  invocation) by moving the seven file templates to `references/templates.md`, the upgrade procedure
  to `references/upgrade.md`, and this changelog out of the skill. Decision content stays inline;
  boilerplate is fetched at the step that needs it. Manual installation now copies the skill
  directory rather than a single file.
- **Phase Boundary Audit** — operationalizes the anti-rot claim: verify referenced paths still
  exist, `arch-reference.md` matches the schema, rule globs still match, budgets hold, and prune
  stale auto memory with `/memory`.

### v1.3.0

- **Step 8 no longer creates a memory file.** It was reimplementing what Claude Code's built-in
  context mechanisms already do, and a root-level `MEMORY.md` isn't loaded at launch anyway. Step 8
  is now **Context Placement Rules**: a single-home table mapping each kind of project fact to the
  file that owns it, plus the reasoning for keeping CLAUDE.md thin.
- **Corrected the premise.** Earlier versions claimed Claude Code's built-in memory is "just flat
  notes." Auto memory has been an index-plus-topic-files system since v2.1.59 (Feb 2026). The skill
  now describes both built-in channels accurately and states that the harness leaves auto memory
  alone — it's Claude-written and machine-local.
- **Removed the duplication trap.** Every category the old Step 8 asked for already had a home
  elsewhere in the harness (routing table, Build Status, `arch-foundations.md`, `arch-reference.md`),
  so the step's own "never duplicate" rule argued against its existence.
- **Added `CLAUDE.local.md`** as the documented place for uncommitted personal notes.

### v1.2.0

- **Optional components are now user-decided**: `Design.md` and `security_check/SECURITY.md` are no
  longer created on the skill's own judgment. New Step 1c asks the user up front, with a
  recommendation, and accepts three answers: include now, defer, or not applicable.
- **Deferred Components tracking**: deferrals are recorded in the CLAUDE.md Harness Maintenance
  section with the trigger for adding them, so they resurface instead of being forgotten. Routing
  table rows are only added once the doc actually exists.
- **Later-addition path**: "add a design doc" / "add a security doc" on an existing harness runs
  just Step 6 or Step 7, then wires up the routing row and clears the deferred entry.

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

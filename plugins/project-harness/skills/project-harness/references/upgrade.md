# Upgrading an Existing Harness

Read this when a project already has a harness (CLAUDE.md with a routing table +
`architecture_docs/` + `roadmap/`) and may be missing components from a newer version of the skill.

If the project already has a harness (CLAUDE.md with a routing table + `architecture_docs/` + `roadmap/`),
check whether it was created by an older version of this skill and patch in missing components.

**Detection**: Look for these signs of an older harness:

| Missing component | Introduced in | What to do |
|-------------------|---------------|------------|
| `architecture_docs/arch-foundations.md` | v1.1.0 | Create it — gather stack rationale from the user and existing CLAUDE.md Tech Stack section |
| CLAUDE.md Tech Stack has no "Why" column | v1.1.0 | Add the "Why" column to the existing table, ask user for rationale per choice |
| No routing table entry for `arch-foundations.md` | v1.1.0 | Add the row to the routing table |
| No "Technology added, removed, or swapped" row in maintenance rules | v1.1.0 | Add it to the "When to Update What" table in CLAUDE.md |
| `Design.md` / `security_check/SECURITY.md` missing with no record of the decision | v1.2.0 | Ask the user (Step 1c): include now, defer, or not applicable. Record any deferral under a "Deferred Components" section in the maintenance rules |
| A root-level `MEMORY.md` created by an older harness | v1.3.0 | Nothing loads it. Move each entry to the file that owns it per the Step 8 placement table, then ask the user before deleting it. Update the maintenance rules and "Where Information Lives" to drop MEMORY.md references |
| No `.claude/rules/` directory | v1.4.0 | Create one rule file per existing architecture domain (Step 5b): mirror its invariants, add a `paths` glob verified against the real tree, end with a pointer to the arch doc |
| Arch docs with no Invariants block or "Does NOT cover" line | v1.4.0 | Add both to each doc's header so the first 30 lines stand alone |
| CLAUDE.md has no Current Phase section | v1.4.0 | Add it, pointing at the active phase only |
| No delegation rules in CLAUDE.md | v1.4.0 | Add the Step 9b section |
| No Phase Boundary Audit in the maintenance rules | v1.4.0 | Add it, and run it once against the existing docs — expect to find stale paths |
| Any file over its Context Budget | v1.4.0 | Report the overage to the user, then split per the budget table |
| An 80-line Harness Maintenance section pasted into CLAUDE.md | v1.5.0 | Move the procedure to `architecture_docs/arch-harness.md`, leave the ~13-line trigger + a routing row, and create `.claude/rules/harness.md` |
| No standard rule files (`harness.md`, `dependencies.md`, `reference-sync.md`) | v1.5.0 | Create them (Step 5b), and delete the maintenance table rows they now cover |
| Maintenance table rows that are file-triggered | v1.5.0 | Move each into the rule file whose `paths` cover that edit; keep only conversational triggers in CLAUDE.md |

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

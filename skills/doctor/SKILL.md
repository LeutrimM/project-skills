---
name: doctor
description: Read-only consistency audit of the project workflow data — cross-checks spec statuses against section content, roadmap registration and feature IDs, dependency order, and data hygiene, then routes every finding to the skill or manual edit that fixes it. Use when the project state looks inconsistent, after hand-editing PLAN.md or specs, or before a release.
---

# Project Doctor

## Role
You are a workflow auditor. You diagnose inconsistencies in the project's planning data (PLAN.md, feature specs) and hand the user the exact fix. You check **workflow data consistency** — not code quality and not feature behavior (that is `/project:qa`'s job).

## The Cardinal Rule: READ-ONLY
NEVER edit any file. Diagnose, explain, route — never repair. Every fix belongs to an owner: a skill (`/project:refine`, `/project:architecture`, …) or a small manual edit you describe precisely. Running this skill must leave the project byte-identical.

Never invent content to "fix" a gap (e.g. fabricating a missing Tech Design) — that would create unapproved decisions, which is the exact failure this doctor exists to catch.

## Before Starting
Work from the project root — `PLAN.md`, `features/`, and `docs/` paths are relative to it.

1. Read `PLAN.md` fully — roadmap rows: IDs, titles, dependencies, phases
2. List `features/*.md` and read each spec fully: **Status** line (canonical definitions: the `STATUS LIFECYCLE` comment at the top of each spec), Dependencies, AC-N criteria, EC-N edge cases, Open Questions, Tech Design, Implementation Notes, QA Test Results
3. If the project has code: `git ls-files` for a quick existence check

**If PLAN.md doesn't exist (or is empty):** report "Project not initialized — run `/project:new-project` first" and stop. There is nothing to audit.

No interview: doctor audits — it does not interrogate the user about features. Everything the audit needs is on disk. The single permitted question is the D2 status clarification (which state the feature is really in) — never guess a status yourself.

## Checks

Severity: **❌ blocking** — the feature's next step cannot be trusted until fixed · **⚠️ informational** — worth cleaning, doesn't block the workflow.

"The latest QA run" (A5, A6) = the `### QA Run N — YYYY-MM-DD` block with the highest N inside the spec's QA Test Results section — a run block spans from its heading to the next `### QA Run` heading or the end of the section, so its `###` sub-headings (Bugs Found, Summary, …) belong to the run above them; if the section has no run headings (legacy spec), treat its content as the single run. An **open bug** (A5, A6) is a bug in that run whose **Status** is not `Fixed (Run N)` and not `Deferred (user decision)`.

### A) Status ↔ content mismatches
| Check | Finding | Fix |
|-------|---------|-----|
| A1 | Status `Architected`+ but Tech Design section is empty/placeholder | ❌ re-run `/project:architecture {id}` |
| A2 | Status `Planned` but Tech Design section has content (stale design after a `/project:refine` revert) | ⚠️ re-run `/project:architecture {id}` to redo it, or `/project:refine {id}` if the revert was unintended |
| A3 | Status `Implemented`+ but the Implementation Notes section is empty or still the template placeholder | ❌ re-run `/project:implement {id}` |
| A4 | Status `In Review`/`Approved` but the QA Test Results section is empty or still the template placeholder (the section heading exists in every spec from the template — check the content, not the heading) | ❌ run `/project:qa {id}` |
| A5 | Status `Approved` but the latest QA run says Production Ready: NO or lists open Critical/High bugs | ❌ fix via `/project:implement {id}`, then `/project:qa {id}` |
| A6 | Status `In Review` but the latest QA run has no open bugs | ⚠️ re-run `/project:qa {id}` to finalize the status |
| A7 | Duplicated sections (two Tech Design / two Implementation Notes / two QA Test Results sections — note: `### QA Run N` blocks inside one QA section are correct, not duplicates) | ⚠️ manual: delete the stale copy, or re-run the owner skill (it replaces, per its rules) |
| A8 | Status `Planned` but Implementation Notes or QA Test Results have content (a `/project:refine` revert moved the feature back past the design stage — the stale content is history, not truth) | ⚠️ re-run the chain `/project:architecture {id}` → `/project:implement {id}` → `/project:qa {id}` to bring content back in sync, or `/project:refine {id}` if the revert was unintended |

### B) Roadmap ↔ spec structure
Note: a roadmap row with no spec file is NORMAL (that is the "Roadmap" state) — never flag it. The reverse is not.
| Check | Finding | Fix |
|-------|---------|-----|
| B1 | Spec file exists with no matching PLAN.md roadmap row (orphan — e.g. a `/project:refine` split that was never registered) | ❌ `/project:refine {id}` to register it in the roadmap, or manual: add the row / remove the spec if it was created by mistake |
| B2 | Duplicate feature IDs — two spec files claiming the same ID, or two roadmap rows with the same ID (every skill resolves features by `features/{id}-*.md`, so this silently breaks them) | ❌ manual: give one side the next free ID, rename its file, update its roadmap row and any references to it |
| B3 | Feature ID not in `N.M` format, or the ID in the spec filename differs from its roadmap row ID | ⚠️ manual: correct the format or rename the file to match the row |

### C) Dependency violations
| Check | Finding | Fix |
|-------|---------|-----|
| C1 | Spec's Dependencies lists feature X, but X has no spec file, and this spec's status is past `Planned` | ❌ spec X first: `/project:write-spec X` |
| C2 | A feature's status is ahead of its dependency's (progress order: spec exists < `Architected` < `Implemented` < `In Review` < `Approved`) — e.g. dependent is `Implemented` but a dependency is still `Planned` | ❌ complete the dependency first, or `/project:refine {id}` to correct the dependency edge |
| C3 | PLAN.md roadmap dependencies contradict the spec's Dependencies section | ⚠️ `/project:refine {id}` to reconcile |
| C4 | Dependency references an ID that exists nowhere (typo / deleted feature) | ❌ `/project:refine {id}` or manual correction |

### D) Data hygiene
| Check | Finding | Fix |
|-------|---------|-----|
| D1 | Unfilled `YYYY-MM-DD` placeholders in a spec | ⚠️ manual: set the real dates |
| D2 | Missing Status line, or a Status value not in the STATUS LIFECYCLE comment | ❌ manual: set the correct status (ask the user which state the feature is really in) |
| D3 | Acceptance criteria not numbered `AC-N` / edge cases not `EC-N` (legacy spec) | ⚠️ manual renumber, or close naturally via `/project:refine {id}` |
| D4 | Spec filename doesn't match `features/{id}-{kebab-name}.md` (ID keeps its dot) | ⚠️ manual rename |
| D5 | Duplicate AC-N or EC-N IDs within one spec | ⚠️ manual renumber |
| D6 | Open Questions still `- [ ]` unchecked | ⚠️ informational: `/project:refine {id}` to close them |

**Backlog (intentionally not checked):** *semantic* drift — PLAN.md row title/description no longer matching the spec's wording. Keeping both sides in sync is `/project:refine`'s duty; comparing natural language reliably is not possible, and a strict check would nag on every hand-edit. The *structural* half of roadmap↔spec consistency is checked: B1–B3.

## Report Format
Report in chat — never write a report file.

If nothing found:
> ✅ Doctor report: no inconsistencies. N specs checked, all statuses match their content, dependencies are in order.

Otherwise:

### Doctor Report
| # | Severity | Feature | Check | Finding | Fix |
|---|----------|---------|-------|---------|-----|
| 1 | ❌ | 1.2 | A3 | Status `Implemented`, no Implementation Notes | `/project:implement 1.2` |
| 2 | ⚠️ | 2.1 | D1 | `Created: YYYY-MM-DD` unfilled | manual: set real date |

**Fix order:** all ❌ first, in dependency order (lowest feature ID first); ⚠️ at leisure.

## Re-verify Loop
After the user applies fixes, they re-run `/project:doctor` — it must report clean. A fix isn't done until the doctor agrees.

## Checklist
- [ ] No files modified (Cardinal Rule respected)
- [ ] PLAN.md and every spec read fully
- [ ] All checks A1–A8, B1–B3, C1–C4, D1–D6 evaluated
- [ ] Every finding has severity + an owner (exact skill command or precise manual steps)
- [ ] Clean bill reported when applicable
- [ ] Re-run suggested ("fix → doctor again → clean")

## What NOT to do
- Do NOT edit files, "normalize" statuses, or fill in gaps
- Do NOT interview or discuss features — that is `/project:refine`
- Do NOT test or review the implementation — that is `/project:qa`
- Do NOT decide which status a feature "should" have — ask the user which state it is really in (D2), then they set it

## Handoff
> "Fix the ❌ findings first (commands above, lowest feature ID first), then re-run `/project:doctor` — it should report clean. Then continue with `/project:help` for your next step."

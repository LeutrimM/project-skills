---
name: help
description: Context-aware guide that tells you where you are in the project workflow and what to do next. Use anytime you're unsure.
---

# Project Help Guide

You are a helpful project assistant. Your job is to analyze the current project state and tell the user exactly where they are and what to do next.

## The Workflow

This skill family drives a project through a fixed pipeline, one feature at a time:

```
/project:new-project → /project:write-spec → /project:architecture → /project:implement → /project:qa
                  ↑                                    │
                  └────────── /project:refine (anytime) ←──────┘
```

`/project:autopilot` drives this same pipeline unattended — fresh `claude -p` sessions per step on an `autopilot/*` branch, no questions asked, branch handed back for review.

Feature status lives in each spec's **Status** line:
`Planned` → `Architected` → `Implemented` → `In Review` → `Approved`

The canonical definition of each status and its transitions is the `STATUS LIFECYCLE` comment at the top of every spec file (created from the spec template) — if this summary ever disagrees with it, the comment wins.

**The Status line is the single source of truth** for the next action — read it, not the presence or absence of sections (a spec reverted by `/project:refine` keeps a stale Tech Design section but is "Planned" again). "Roadmap" is not a file status: it means a PLAN.md row with no `features/{id}-*.md` file yet.

## When Invoked
Work from the project root — `PLAN.md`, `features/`, and `docs/` paths are relative to it.

### Step 1: Analyze Current State

1. **Check PLAN.md** (project root)
   - Missing or empty? → Project not initialized yet
   - Exists? → Note the roadmap features and phases

2. **Check feature specs:** list `features/*.md`
   - None? → No features specified yet
   - Exist? → Read each one's **Status** line and check for section markers:
     - "Tech Design (Solution Architect)" — added by `/project:architecture`
     - "Implementation Notes" — added by `/project:implement`
     - "QA Test Results" — added by `/project:qa`
     - Unchecked "Open Questions" — candidates for `/project:refine`

3. **Check the roadmap for unspecced features:** PLAN.md rows with no matching `features/{id}-*.md` file are still "Roadmap"

4. **Check codebase:** quick scan of what exists — `git ls-files`, recent commits

### Step 2: Determine Next Action

**No PLAN.md (or empty):**
> The project hasn't been initialized yet.
> Run `/project:new-project` with a description of what you want to build.
> Example: `/project:new-project a task management CLI for small teams`

**PLAN.md exists, no feature specs:**
> The plan is set but no feature has been specified yet.
> Recommended first feature: **[1.1 from the roadmap]** — run `/project:write-spec 1.1`

**A feature has status "Planned" (spec written; design not done yet, or reverted by `/project:refine`):**
> Feature {id} is ready for technical design.
> Run `/project:architecture {id}`

**A feature has status "Architected":**
> Feature {id} has a design and is ready for implementation.
> Run `/project:implement {id}`

**A feature has status "Implemented" (implemented, no QA results):**
> Feature {id} is implemented and ready for independent testing.
> Run `/project:qa {id}` to test it against its acceptance criteria and audit it

**A feature has status "In Review" (QA in progress or QA bugs open):**
("the latest QA run" = the `### QA Run N — YYYY-MM-DD` block with the highest N in the spec's QA Test Results section; a run block spans from its heading to the next `### QA Run` heading or the end of the section; an "open bug" is one whose **Status** is not `Fixed (Run N)` and not `Deferred (user decision)` — a Deferred bug is the user's choice and never routes to `/project:implement`)
- If the latest QA run lists open bugs:
  > Feature {id} has open QA bugs. Fix them with `/project:implement {id}`, then re-run `/project:qa {id}`.
  > List the bugs from the spec's QA Test Results section.
- If the latest QA run lists no open bugs (QA was started but interrupted before the verdict):
  > Feature {id} has QA in progress but no open bugs recorded. Re-run `/project:qa {id}` to finish the run and finalize the status — do not send it to `/project:implement`.

**All specified features are "Approved":**
> All current features passed QA and are ready to release. You can:
> - Run `/project:write-spec` for the next roadmap feature ({next Roadmap feature from PLAN.md})
> - Run `/project:refine {id}` to close open questions or adjust a shipped feature

**Unresolved Open Questions or invalidated designs anywhere:**
> Feature {id} has N open questions{ / a spec that changed after its design}.
> Run `/project:refine {id}` to resolve them.

### Step 3: Answer User Questions

If the user asked a specific question (via arguments), answer it in the context of the current project state. Common questions:

- "What skills are available?" → List the eight skills with one-line descriptions (six workflow skills, `/project:doctor`, plus `/project:autopilot` for unattended runs)
- "How do I add a new feature?" → `/project:write-spec` (it adds unplanned features to PLAN.md automatically); `/project:new-project` only if the project isn't set up
- "The design no longer matches the spec" → `/project:refine {id}` reverts the status; then re-run `/project:architecture {id}`
- "Something looks off / inconsistent" → `/project:doctor` — read-only audit that cross-checks statuses, sections, and dependencies and routes every finding to its fix
- "Can it just build this while I'm away?" → `/project:autopilot <id|phase N|next>` — runs spec → architecture → implement → qa in fresh `claude -p` sessions on an `autopilot/*` branch, never asking questions, and hands the branch back for review
- "What's the project structure?" → Explain the layout: PLAN.md, `features/`, `docs/`, plus the code layout from `git ls-files`
- "Where do decisions get recorded?" → Each spec has a Decision Log (Product by `/project:write-spec`/`/project:refine`, Technical by `/project:architecture`/`/project:implement`); resolved questions get checked off in Open Questions

## Output Format

### Current Project Status
_Brief summary of where the project stands_

### Features Overview
| ID | Feature | Status | Next action |
|----|---------|--------|-------------|
| 1.1 | ... | Approved | — |
| 1.2 | ... | Planned | `/project:architecture 1.2` |
| 2.1 | ... | Roadmap | `/project:write-spec 2.1` |

### Recommended Next Step
_The single most important thing to do next, with the exact command_

### Other Available Actions
_Other things the user could do right now_

If the user asked a specific question, answer that FIRST, then show the status overview.

## Important
- Be concise and actionable
- If a spec's Status contradicts its content (e.g. `Architected` with no Tech Design), do not guess which is right — recommend `/project:doctor` for the full audit
- Always give the exact command to run, with the concrete feature ID
- Reference specific file paths
- Don't re-explain skill internals unless asked
- Focus on: "Here's where you are, here's what to do next"

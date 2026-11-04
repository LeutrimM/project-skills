---
name: implement
description: Implement a feature from its spec and tech design — data layer, logic, and UI in one workflow, using the project's own stack and conventions. Use after architecture is designed. Pass a feature name or ID as argument.
---

# Implementer

## Role
You are an experienced software developer. You read a feature spec + tech design and implement the feature end to end — data layer, logic, interfaces, UI — using the project's existing stack, patterns, and conventions.

## Before Starting
Work from the project root — `PLAN.md`, `features/`, and `docs/` paths are relative to it.

1. Read `PLAN.md` — project type, tech stack, constraints
2. Read the feature spec `features/{id}-*.md` — user stories, acceptance criteria, edge cases, **Tech Design** section
3. Explore what exists — use the repo, not assumptions:
   - `git ls-files` for the current module/package/directory layout
   - Recent commits for patterns to follow: `git log --oneline -10`
   - `docs/` for design system or architecture notes
4. Identify the project's tooling — build, run, test commands (from `pyproject.toml`, `package.json`, `Makefile`, `scripts/`, README). Never assume a framework; discover it.

**Status is the source of truth.** If the spec's **Status** is "Approved":
> "This feature is already implemented and approved. If it needs changes, run `/project:refine {id}` — the refine reverts the status and the architecture → implement → qa chain re-verifies it."
→ Stop here. Do not route an Approved feature back to the design stage.

If the spec's **Status** is not "Architected", "Implemented", or "In Review" (e.g. still "Planned" — including after `/project:refine` reverted an invalidated design, even if a stale Tech Design section is still present):
> "This feature needs a technical design before implementation. Run `/project:architecture` first."
→ Stop here.

**Check the Tech Design content, not just the Status.** If the "Tech Design (Solution Architect)" section is empty or still the template placeholder (`_To be added by /project:architecture_`), stop even when the Status allows it:
> "The Status says {status} but the Tech Design section is still empty/placeholder — the design is missing (doctor flags this as A1). Run `/project:architecture {id}` first."
→ Stop here. Never implement without a real design — inventing design decisions during implementation is `/project:architecture`'s job.

## Bug-Fix Round (Status "In Review")
If the spec's Status is "In Review", this run is a QA bug-fix round:
1. Read the spec's **QA Test Results** and list every **open** bug in the **latest QA run** only — the `### QA Run N` block with the highest N, spanning from its heading to the next `### QA Run` heading or the end of the section. An open bug is one whose **Status** is not `Fixed (Run N)` and not `Deferred (user decision)`; bugs recorded only in earlier run blocks, or marked Fixed/Deferred, are history — do not re-fix them
2. Fix them in severity order (Critical and High first)
3. Add or adjust automated tests that reproduce each fixed bug
Then continue with the normal workflow below; setting Status back to "Implemented" (see After Completion) tells the user to re-run `/project:qa`.

## Workflow

### 1. Read Spec + Design
- Understand the module structure, data model, and tech decisions from the Tech Design
- List the acceptance criteria — these are your definition of done
- Note dependencies on other features and whether they are implemented

### 2. Clarify Open Points (only if needed)
If the design leaves implementation-scoping open (validation rules, concurrency handling, permission boundaries, rate limiting), resolve it — **one question at a time, always with a recommended answer**. Skip anything the spec, design, or codebase already answers.

If a design decision turns out to be wrong or impractical while implementing: **do not silently deviate.** Record the deviation in the spec's Technical Decisions log with rationale. If the deviation is structural (changes the data model or module boundaries), stop and tell the user to re-run `/project:architecture`.

### 3. Implement — Layer by Layer
Build in dependency order, matching the design's structure tree:
1. **Data / storage layer** first (schema, files, persistence)
2. **Core logic / services** (business rules, validation, error handling)
3. **Interfaces** (API endpoints, CLI commands, library functions)
4. **UI last** (only if the feature has one)

Rules:
- Follow the existing code's conventions — naming, module layout, error handling, logging
- Reuse existing modules and helpers before writing new ones
- Validate all external input; handle error and empty states explicitly, not just the happy path
- Add new dependencies only if the design lists them — otherwise use what the project already has

**For UI work:** check for a design system first: `docs/design-system.md` (or files in `design/`, `mockups/`, `assets/`). If one exists, apply its colors, typography, and component guidelines; don't ask about choices it already covers. If none exists, ask the user for style/brand preferences before styling.

### 4. Write Automated Tests
Add tests as you go, in the project's existing test framework and layout (e.g. `tests/`, co-located files):
- Happy path per acceptance criterion
- Validation errors and edge cases from the spec
- Authorization/permission checks where applicable
If a test already covers a criterion, extend it — never create a parallel duplicate. This suite is the one `/project:qa` will later extend, so keep it the single home for per-criterion coverage.
Run the suite to confirm everything passes before proceeding.

### 5. Verify Against Acceptance Criteria
Before handing off, exercise the feature through its real surface (run the app/CLI/server, call the endpoint) and walk through EVERY acceptance criterion. Mark each pass/fail. Tick each passing criterion in the spec (`- [x] AC-N: …`); leave failures unchecked. Fix failures now — this is implementation, so fixing is your job here.

### 6. User Review
- Show the user how to run/exercise the feature (exact command or URL)
- Summarize what was built vs the design's structure tree
- Show test results and acceptance-criteria walkthrough
- Ask: "Does it behave as specified? Any changes needed?"

## Context Recovery
If your context was compacted mid-task:
1. Re-read the feature spec and Tech Design
2. Run `git diff` and `git status` to see what you've already changed
3. Check the spec for existing "Implementation Notes"
4. Continue from where you left off — don't restart or duplicate work

## After Completion
Update the spec file:
- Fill in the spec's **Implementation Notes** section (the spec template ships with a placeholder after "Tech Design (Solution Architect)"; replace it — never add a second section). It is a top-level section, NOT a subsection of Tech Design (Tech Design belongs to `/project:architecture` and gets replaced on design re-runs; your notes must survive that). Content: actual files/modules created or modified, any deviations from the design (also logged in Technical Decisions), and how to run/exercise the feature. On a re-run, replace the Implementation Notes content with the current state — do not append a second copy
- Update **Last Updated** and set **Status** to "Implemented" (canonical status definitions: the `STATUS LIFECYCLE` comment at the top of the spec)

## Checklist
- [ ] Spec + Tech Design read (Tech Design has real content, not the template placeholder); acceptance criteria listed as definition of done
- [ ] Existing patterns and tooling identified from the repo (not assumed)
- [ ] Open implementation questions resolved (one at a time, with recommendations)
- [ ] Implemented in layer order: data → logic → interface → UI
- [ ] Conventions followed; existing modules reused
- [ ] Deviations from design logged in Technical Decisions (structural ones sent back to /project:architecture)
- [ ] Input validation, error states, empty states handled
- [ ] Design system applied (if present)
- [ ] Automated tests written; project test suite passes
- [ ] Every acceptance criterion manually verified through the real surface
- [ ] Passing acceptance criteria ticked in the spec (`- [x] AC-N`)
- [ ] Implementation Notes section filled; status set to "Implemented"
- [ ] User has reviewed and accepted

## Handoff
> "Implementation is done and the test suite passes. Next step: run `/project:qa` to independently test this feature against its acceptance criteria and audit it."

## Git Commit
```
feat({id}): Implement [feature name]
```
On a bug-fix round (Status was "In Review"), use instead:
```
fix({id}): Fix QA bugs for [feature name]
```

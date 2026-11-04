# QA Test Results Template

Fill this structure into the feature spec's existing **QA Test Results** section in `features/{id}-{feature-name}.md` — the spec template ships with a placeholder line in that section; replace it rather than adding a second "QA Test Results" section. Every run is written as its own `### QA Run N — YYYY-MM-DD` block — the first run is `### QA Run 1`. On a re-run (e.g. after a bug-fix round), keep previous runs: append the block below at the end of the existing **QA Test Results** section with its heading renumbered to `### QA Run N — YYYY-MM-DD` (next N); copy the block as-is, it contains no `##` heading and no separator. The new run must account for every bug that was Open in the previous run: re-test each and re-list it (or summarize it) with its current **Status** — `Fixed (Run N)` when verified fixed, still `Open` when broken, `Deferred` when the user chose not to fix it. "The latest QA run" (referenced by `/project:help` and `/project:doctor`) means the `### QA Run N` block with the highest N; a run block spans from its heading to the next `### QA Run` heading or the end of the section, so its `###` sub-headings (`Acceptance Criteria Status`, `Bugs Found`, `Summary`, …) belong to the run above them.

```markdown
### QA Run 1 — YYYY-MM-DD

**Tested:** YYYY-MM-DD
**How exercised:** <URL / command / API call used, e.g. http://localhost:8088/ui or `curl ...`>
**Tester:** QA Engineer (AI)

### Acceptance Criteria Status

#### AC-1: [Criterion Name]
- [x] Sub-criterion passed
- [ ] BUG: Sub-criterion failed (describe what went wrong)

#### AC-2: [Criterion Name]
- [x] All sub-criteria passed

### Edge Cases Status

#### EC-1: [Edge Case Name]
- [x] Handled correctly

#### EC-2: [Edge Case Name]
- [ ] BUG: Not handled (describe expected vs actual behavior)

### Security Audit Results
<!-- Adapt checks to the project's attack surface; drop what doesn't apply -->
- [x] Authentication: cannot access without credentials (if applicable)
- [x] Authorization: no cross-user/cross-resource access
- [x] Input validation: injection / malformed input rejected
- [x] Guard/whitelist bypass attempts blocked
- [x] No secrets or sensitive data in logs, errors, or responses
- [ ] BUG: [Security issue description]

### Automated Regression Tests
- **Suite run:** `<test command>` — all pass / N failures
- **New tests added:** <file paths — only gaps not already covered by `/project:implement`'s suite; extend existing per-criterion tests instead of duplicating them>

### Bugs Found

#### BUG-1: [Bug Title]
- **Severity:** Critical | High | Medium | Low
- **Status:** Open | Fixed (verified in Run N) | Deferred (user decision — a deferred bug is not open and does not block routing)
- **Steps to Reproduce:**
  1. Go to / run [entry point]
  2. Do [action]
  3. Expected: [what should happen]
  4. Actual: [what actually happens]
- **Evidence:** [screenshot / log excerpt / request-response, if applicable]
- **Automated coverage:** [reproduction test `file::test_name` | NOT covered by automated tests — must be verified manually after the fix]
- **Priority:** Fix before approval | Fix in next sprint | Nice to have

### Summary
- **Acceptance Criteria:** X/Y passed
- **Bugs Found:** N total (C critical, H high, M medium, L low)
- **Security:** [Pass / Issues found]
- **Production Ready:** YES / NO
- **Recommendation:** [Approve & release / Fix bugs first]
```

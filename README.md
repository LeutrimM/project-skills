# project — Claude Code plugin

Spec-driven development workflow for Claude Code, adapted from the `skills/project/` set in
[atz3n/agent-tools](https://github.com/atz3n/agent-tools) (MIT, upstream commit `33dd102`).

```
/project:new-project → /project:write-spec → /project:architecture → /project:implement → /project:qa
                              ↑                                              │
                              └────────────── /project:refine (anytime) ←────┘
```

| Command | Purpose |
|---|---|
| `/project:new-project <idea>` | Interview → `PLAN.md` (phases, features, dependencies) |
| `/project:write-spec <id>` | Feature spec in `features/{id}-*.md` (user stories, AC, edge cases) |
| `/project:architecture <id>` | High-level technical design, no code |
| `/project:implement <id>` | Implement spec + design |
| `/project:qa <id>` | Test against acceptance criteria + security audit |
| `/project:refine <id>` | Change / challenge an existing spec |
| `/project:doctor` | Read-only consistency audit of PLAN.md + specs |
| `/project:help` | Where am I, what's next? |
| `/project:autopilot <id \| phase N \| next [N]> [--dry-run]` | Run the pipeline unattended in fresh `claude -p` sessions on an `autopilot/*` branch |

Status per spec: `Planned → Architected → Implemented → In Review → Approved`.

## Install

```bash
claude plugin marketplace add <github-user>/project-skills
claude plugin install project@project-skills
```

Or from a local checkout: `claude plugin marketplace add /path/to/project-skills`.
Inside Claude Code the same works via `/plugin`.

For cloud sessions (claude.ai/code), add to the project repo's `.claude/settings.json`:

```json
{
  "extraKnownMarketplaces": {
    "project-skills": { "source": { "source": "github", "repo": "<github-user>/project-skills" } }
  },
  "enabledPlugins": { "project@project-skills": true }
}
```

## Autopilot notes

- Child sessions run `claude -p --permission-mode bypassPermissions --output-format stream-json --verbose`.
  Run the autopilot in a sandbox/container or a throwaway clone — children have full tool access.
  `bypassPermissions` is refused as root outside a sandbox; the pre-flight smoke test catches that.
- Each step: prompt file → detached session → `.rc` exit-code sentinel → log in `/tmp/autopilot-*.log`.
- Optional flags you can ask for: model (`--model`), `--max-turns`, `--allowedTools`/`--disallowedTools`.
- Nothing is merged or pushed; the branch is handed back for review.

## Changes vs. upstream

- Skills renamed to short names inside the `project` plugin namespace (`project-qa` → `/project:qa`); all cross-references updated.
- `autopilot` rewritten for Claude Code: `claude -p` instead of `pi -p`, prompt via stdin, stream-JSON `result` event instead of `agent_settled`, `tool_result` parsing, CLI smoke test + plugin check in pre-flight, `gh pr create` instead of the github-proxy skill, cost column in the report.

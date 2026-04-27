# CLAUDE.md

This file provides guidance to Claude Code when working with the ACS Patch Release workflow.

## Architecture

This is a **thin wrapper** around the patch-release skill in `stackrox/skills` (internal repo).

- **This directory** (`patch-release/`) contains the ACP workflow config (ambient.json),
  the command file, and the CLAUDE.md. This is public.
- **`stackrox/skills`** contains the reference data, procedures, lessons learned, and
  scripts. This is internal (not public).

At session start, the `/patch-release` command locates or clones the skills repo
and sets `PATCH_RELEASE_DIR` to point to the plugin's reference data.

## Workflow Purpose

Stateless, resumable workflow for ACS Z-stream (patch) releases. Detects progress
from external signals (git tags, GitHub milestones, Jira, Konflux, PRs) so any
engineer can pick up a release at any point.

## Commands

- `/patch-release VERSION` — Perform a patch release (e.g. `/patch-release 4.8.11`).
  Works for both fresh starts and resuming — detects state automatically.

## Directory Structure

```
patch-release/                          # This directory (public, in ambient-workflows)
├── .ambient/
│   └── ambient.json                    # Workflow config and system prompt
├── .claude/
│   └── commands/
│       └── patch-release.md            # Command: clones skills repo, runs workflow
└── CLAUDE.md                           # This file

${PATCH_RELEASE_DIR}/                   # Located/cloned at runtime (internal, from stackrox/skills)
├── reference/
│   ├── phase-procedures.md
│   ├── lessons-learned.md
│   ├── advisory-rules.md
│   ├── upgrade-test-procedure.md
│   └── secrets-and-access.md
├── scripts/
│   └── acs-operator-test.sh
└── skills/patch-release/SKILL.md
```

## Critical Constraints

1. **Never paste tokens in chat** — sessions are shared, use workspace secrets
2. **Never modify Jira** — query only via MCP
3. **Prod release is NOT re-runnable** — escalate on failure
4. **Finish Release defaults to dry-run** — always dry-run first
5. **Jira is source of truth** for advisory content, not git commits

## Testing in ACP

1. Push branch to your fork
2. In ACP, select "Custom Workflow..."
3. Enter path: `patch-release`
4. Run `/patch-release X.Y.Z`

The workflow will clone `stackrox/skills` automatically (requires GitHub
integration configured in Settings > Integrations).

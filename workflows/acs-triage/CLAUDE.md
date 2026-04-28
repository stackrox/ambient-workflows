# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with the ACS Triage Workflow.

## Repository Purpose

This is a **single-purpose workflow** for automated triage of StackRox/ACS JIRA issues. It analyzes CI failures, security vulnerabilities, and flaky tests to generate actionable reports with intelligent team assignment.

## Key Features

- **Multi-Strategy Team Assignment**: 5-strategy priority system with 95%-70% confidence scores
- **Version Awareness**: Detects mismatches between issue versions and current codebase
- **Specialized Analysis**: Custom decision trees for CI failures, vulnerabilities, and flaky tests
- **READ-ONLY Mode**: Generates reports without modifying JIRA automatically

## Workflow Commands

The workflow provides 2 main commands:

- `/triage` - Complete end-to-end triage pipeline: setup → fetch → classify → analyze → assign → report (READ-ONLY)
- `/triage --comment` - Full triage pipeline + post analysis comments to JIRA (⚠️ WRITES to JIRA)
- `/comment-issues` - Standalone command to add triage comments to JIRA (requires prior /triage run)

**Simplified Design:** All triage steps are consolidated into a single `/triage` command for ease of use.

## Directory Structure

```
workflows/acs-triage/
├── .ambient/
│   └── ambient.json              # Workflow configuration
├── .claude/
│   └── commands/                 # 2 command files
│       ├── triage.md             # Main triage pipeline
│       └── comment-issues.md     # Optional JIRA commenting
├── reference/                    # StackRox domain knowledge
│   ├── teams.md
│   ├── CODEOWNERS-patterns.md
│   ├── error-signatures.md
│   ├── team-mappings.md
│   ├── vulnerability-decision-tree.md
│   ├── flaky-test-patterns.md
│   └── constants.md
├── templates/                    # Report templates
│   ├── triage-report.md
│   ├── report.html
│   └── slack-summary.md
├── CLAUDE.md                     # This file
├── README.md                     # Complete documentation
└── FIELD_REFERENCE.md           # Field definitions
```

## StackRox/ACS Domain Knowledge

See `reference/teams.md` for complete team list and responsibilities.

See `reference/error-signatures.md` for error patterns and confidence scores.

See `reference/constants.md` for all confidence thresholds and configuration values.

## Critical Constraints

1. **READ-ONLY MODE**: Generate reports only, never modify JIRA automatically
2. **Timeout**: Complete within 300 seconds (5 minutes)
3. **Issue Limit**: Process 10-20 issues per session
4. **Version Awareness**: Adjust confidence for version mismatches
5. **High Confidence Threshold**: ≥80% for recommended assignments

## Automated Execution

The workflow is configured in `.ambient/ambient.json`:

```json
{
  "config": {
    "jira": {
      "project": "ROX",
      "filter": 103399
    },
    "timeout": 300,
    "maxIssues": 20
  }
}
```

**Simplified Execution**: The `/triage` command internally handles all phases sequentially, with parallel execution for type-specific analysis (CI/vuln/flaky) to save 60-80 seconds.

## Version Mismatch Handling

The triage workflow clones latest `main` branch from StackRox repo. Issues with older `affectedVersions` may have:
- Different file ownership (CODEOWNERS changes)
- Moved/renamed components
- Changed team structures

**Impact:**
- Confidence scores adjusted (see `reference/constants.md` for thresholds)
- Reports flag with ⚠️ symbol

## Reference Data Sources

The workflow uses reference data from:

- `/tmp/triage/stackrox/.github/CODEOWNERS` - File path → team mappings (cloned during setup phase)
- `reference/*.md` - Local domain knowledge files for error patterns, team mappings, decision trees

## Output Locations

All artifacts go to `artifacts/acs-triage/`:

- `setup-info.json` - Setup metadata
- `issues.json` - Raw and enriched issue data
- `triage-report.md` - Detailed markdown report
- `report.html` - Interactive HTML dashboard
- `slack-summary.md` - Slack notification
- `summary.json` - Machine-readable summary

## Development Guidelines

### When Modifying Commands

1. Read the command file first to understand current logic
2. Preserve the structure: Purpose, Prerequisites, Process, Output, Notes
3. Update both command file and README.md if changing behavior
4. Test changes using "Custom Workflow" in ACP before committing

### When Adding Reference Files

1. Extract data from source (stackrox repo or triage-prompt-v2.md)
2. Use markdown format with clear sections
3. Include confidence levels for team assignments
4. Add examples for clarity
5. Update command files to reference new data

### When Updating Decision Trees

1. Document all decision steps clearly
2. Include exit conditions (when to CLOSE vs ASSIGN)
3. Note confidence scores for each path
4. Provide examples of each decision outcome
5. Keep ProdSec workflow aligned with team practices

## Testing in ACP

To test this workflow:

1. Push branch to your fork
2. In ACP, select "Custom Workflow..."
3. Enter:
   - URL: `https://github.com/{your-username}/ambient-workflows.git`
   - Branch: `{your-branch-name}`
   - Path: `.` (root of repository)
4. Run commands interactively or in sequence

## Common Pitfalls to Avoid

1. **Don't modify JIRA automatically** - Use `/triage` (READ-ONLY) or `/triage --comment` (explicit write)
2. **Don't skip version detection** - Critical for confidence adjustment (handled automatically)
3. **Don't ignore low confidence** - Flag for manual review
4. **Don't exceed timeout** - Limit to 10-20 issues (configured in ambient.json)
5. **Don't assume CODEOWNERS is current** - Version mismatches are detected and flagged

## Key Files

- **`.ambient/ambient.json`** - Workflow configuration (read this first)
- **`README.md`** - Complete user documentation
- **`FIELD_REFERENCE.md`** - Issue field definitions
- **`.claude/commands/*.md`** - Command implementations
- **`reference/*.md`** - StackRox domain knowledge

## Support

For questions or issues:
- Read `README.md` for complete documentation
- Check `.claude/commands/` for command-specific logic
- Review `FIELD_REFERENCE.md` for data structure
- Consult StackRox CODEOWNERS for team ownership

---

**Workflow Type:** Domain-Specific Triage
**Target:** StackRox/ACS JIRA Issues
**Mode:** READ-ONLY (Reports Only)
**Version:** 1.0.0

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

The workflow provides 7 slash commands:

- `/setup` - Clone StackRox repository for CODEOWNERS and reference data
- `/fetch-issues` - Retrieve untriaged JIRA issues from filters 103399 and 95004
- `/classify` - Categorize issues and detect version mismatches
- `/analyze-ci` - Deep analysis of CI failures
- `/analyze-vuln` - Apply ProdSec decision tree for vulnerabilities
- `/analyze-flaky` - Pattern matching for flaky tests
- `/assign-team` - Multi-strategy team assignment with confidence scores
- `/generate-report` - Create markdown, HTML, and Slack reports

## Directory Structure

```
ambient-workflows/
├── .ambient/
│   └── ambient.json              # Workflow configuration
├── .claude/
│   └── commands/                 # 7 command files
│       ├── setup.md
│       ├── fetch-issues.md
│       ├── classify.md
│       ├── analyze-ci.md
│       ├── analyze-vuln.md
│       ├── analyze-flaky.md
│       ├── assign-team.md
│       └── generate-report.md
├── reference/                    # StackRox domain knowledge (to be created)
│   ├── CODEOWNERS-patterns.md
│   ├── error-signatures.md
│   ├── team-mappings.md
│   ├── vulnerability-decision-tree.md
│   └── flaky-test-patterns.md
├── templates/                    # Report templates (to be created)
│   ├── triage-report.md
│   ├── report.html
│   └── slack-summary.md
├── CLAUDE.md                     # This file
├── README.md                     # Complete documentation
└── FIELD_REFERENCE.md           # Field definitions (to be created)
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

## Version Mismatch Handling

The `/setup` command clones latest `main` branch. Issues with older `affectedVersions` may have:
- Different file ownership (CODEOWNERS changes)
- Moved/renamed components
- Changed team structures

**Impact:**
- Confidence scores adjusted (see `reference/constants.md` for thresholds)
- Reports flag with ⚠️ symbol

## Reference Data Sources

After `/setup`, reference data is loaded from cloned repositories:

- `/tmp/triage/stackrox/.github/CODEOWNERS` - File path → team mappings
- `/tmp/triage/stackrox/.claude/agents/stackrox-ci-failure-investigator.md` - Error patterns
- `/tmp/triage/skills/plugins/rhacs-patch-eval/` - Vulnerability assessment

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

1. **Don't modify JIRA automatically** - This is READ-ONLY
2. **Don't skip version detection** - Critical for confidence adjustment
3. **Don't ignore low confidence** - Flag for manual review
4. **Don't exceed timeout** - Limit to 10-20 issues
5. **Don't assume CODEOWNERS is current** - Check version mismatch
6. **Don't forget to run /setup** - Reference data needed

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

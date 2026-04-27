# ACS Triage Workflow

Automated triage for StackRox/ACS JIRA issues with intelligent team assignment using multi-strategy confidence scoring. Analyzes CI failures, security vulnerabilities, and flaky tests to generate actionable reports.

## Overview

This workflow provides systematic triage of untriaged StackRox issues using:

- **Multi-Strategy Team Assignment**: 5-strategy priority system with 95%-70% confidence scores
- **Specialized Analysis**: Custom decision trees for CI failures, vulnerabilities, and flaky tests
- **Version Awareness**: Detects mismatches between issue versions and current codebase
- **READ-ONLY Mode**: Generates reports and recommendations without modifying JIRA
- **Multiple Output Formats**: Markdown, interactive HTML, Slack summaries, and JSON

### Workflow Phases

1. **Setup** - Clone StackRox repository for CODEOWNERS and reference data
2. **Fetch Issues** - Retrieve untriaged issues from JIRA filters (103399, 95004)
3. **Classify** - Categorize as CI_FAILURE, VULNERABILITY, FLAKY_TEST, or UNKNOWN
4. **Analyze** - Apply specialized analysis based on type
5. **Assign Team** - Multi-strategy assignment with confidence scoring
6. **Generate Reports** - Create markdown, HTML, and Slack outputs
7. **Review** - Human review of recommendations
8. **Comment** (Optional) - Add triage comments to JIRA
9. **Execute** - Manual JIRA updates based on report

## Getting Started

### Prerequisites

- JIRA MCP connection configured
- Access to JIRA filters 103399 (current untriaged) and 95004 (previous duty)
- Git access to github.com/stackrox/stackrox and github.com/stackrox/skills
- Permission to write to /tmp directory

### Quick Start

```bash
# Phase 1: Initialize
/setup

# Phase 2: Fetch untriaged issues
/fetch-issues

# Phase 3: Classify issues by type and detect version mismatches
/classify

# Phase 4-6: Run specialized analysis (automatically determined by type)
/analyze-ci       # For CI_FAILURE issues
/analyze-vuln     # For VULNERABILITY issues
/analyze-flaky    # For FLAKY_TEST issues

# Phase 7: Assign teams with confidence scoring
/assign-team

# Phase 8: Generate reports
/generate-report
```

### Automated Mode (Read-Only)

For scheduled execution (weekdays 3 PM UTC), run all commands in sequence:

```bash
/setup && /fetch-issues && /classify && /analyze-ci && /analyze-vuln && /analyze-flaky && /assign-team && /generate-report
```

### Semi-Automated Mode (With Comments)

⚠️ **Warning:** This mode WRITES to JIRA

```bash
# Generate reports first
/setup && /fetch-issues && /classify && /analyze-ci && /analyze-vuln && /analyze-flaky && /assign-team && /generate-report

# Review the report, then add comments (dry run first)
/comment-issues

# If satisfied, post comments
/comment-issues --confirm
```

## Workflow Phases

### Phase 1: Setup (`/setup`)

**Purpose:** Clone StackRox repository for CODEOWNERS and reference data

**Process:**
- Clone stackrox and skills repositories to /tmp/triage/
- Extract current version for version mismatch detection
- Verify key files exist (CODEOWNERS, stackrox-ci-failure-investigator.md)

**Output:** `artifacts/acs-triage/setup-info.json`

**⚠️ Important:** Clones latest `main` branch - issues with older `affectedVersions` may have outdated file ownership

### Phase 2: Fetch Issues (`/fetch-issues`)

**Purpose:** Retrieve untriaged JIRA issues from filters

**Process:**
- Query filters 103399 and 95004
- Limit to 10-20 issues (300s timeout constraint)
- Extract: key, summary, description, labels, components, priority, affectedVersions, fixVersions, comments

**Output:** `artifacts/acs-triage/issues.json`

### Phase 3: Classify (`/classify`)

**Purpose:** Categorize issues and detect version mismatches

**Classification Logic:**
- **VULNERABILITY**: CVE-* labels, CVE patterns in summary
- **FLAKY_TEST**: flaky-test labels, known flaky test patterns
- **CI_FAILURE**: CI_Failure labels, stack traces in description
- **UNKNOWN**: No patterns match

**Version Detection:**
- Compare issue `affectedVersions` against current stackrox version
- Flag mismatches (e.g., issue affects 4.5.x, using 4.7.x CODEOWNERS)

**Output:** Updated `issues.json` with type and version mismatch fields

### Phase 4-6: Specialized Analysis

#### `/analyze-ci` - CI Failure Analysis

**For:** CI_FAILURE issues

**Process:**
- Extract build metadata, error messages, stack traces, file paths
- Classify error type (GraphQL, panic, timeout, network, image, infrastructure)
- Match error signatures from stackrox-ci-failure-investigator.md
- Check for known flaky patterns

**Output:** `ci_analysis` field with error_type, file_paths, error_signature_match

#### `/analyze-vuln` - Vulnerability Analysis

**For:** VULNERABILITY issues

**ProdSec Decision Tree (6 steps):**
1. Version support check → CLOSE if unsupported
2. Severity threshold → CLOSE if Low or Moderate <7.0 CVSS
3. Container applicability → CLOSE if database with npm/Go (false positive)
4. Duplicate detection → DUPLICATE if CVE already exists
5. Impact analysis → CLOSE if VEX false positive
6. Team assignment by container

**Output:** `vuln_analysis` field with decision tree results and team assignment

#### `/analyze-flaky` - Flaky Test Analysis

**For:** FLAKY_TEST issues

**Process:**
- Match known flaky patterns (GlobalSearch, PolicyFieldsTest, NetworkFlowTest, etc.)
- Estimate frequency from JIRA history (>10/month = High, 3-10 = Medium, <3 = Low)
- Assign to test owner using CODEOWNERS

**Output:** `flaky_analysis` field with pattern match and frequency data

### Phase 7: Team Assignment (`/assign-team`)

**Purpose:** Multi-strategy team assignment with confidence scoring

Uses 5 strategies in priority order. See `reference/constants.md` for confidence thresholds and version mismatch adjustments.

**Output:** `team_assignment` field with assigned_team, confidence, strategy, evidence

### Phase 8: Generate Reports (`/generate-report`)

**Purpose:** Create multiple report formats

**Outputs:**

1. **triage-report.md** - Detailed markdown with:
   - Executive summary
   - Statistics by type/team/confidence
   - Main triage table
   - High-confidence recommendations (≥80%)
   - Version mismatch warnings

2. **report.html** - Interactive dashboard with:
   - Stats cards (type, team, confidence counts)
   - Filters (type, team, confidence, version mismatch)
   - Sortable table (click headers to sort)
   - Search box (filter by key/summary)

3. **slack-summary.md** - Concise summary for Slack:
   - High-confidence recommendations (≥90%)
   - Issue counts by team
   - Version mismatch count
   - Links to full reports

4. **summary.json** - Machine-readable data for automation

### Phase 9 (Optional): Add Triage Comments (`/comment-issues`)

⚠️ **Warning:** This phase WRITES to JIRA

**Purpose:** Add structured triage comments to JIRA issues with team recommendations

**Process:**
- Read triage results from `artifacts/acs-triage/issues.json`
- Generate formatted comment with team, confidence, reasoning, evidence
- Post comment to JIRA using JIRA MCP
- Log results (success/failure/skipped)

**Safety Features:**
- Dry run by default (preview without posting)
- Minimum confidence filter (default: ≥70%)
- Rate limiting (10 issues/minute)
- Idempotency check (skip if comment already exists)

**Usage:**
```bash
# Dry run (preview only)
/comment-issues

# Post comments (requires --confirm)
/comment-issues --confirm

# Higher confidence threshold
/comment-issues --min-confidence=80 --confirm
```

**Output:** `artifacts/acs-triage/comment-results.json`

## Team Assignment Strategies

See `reference/constants.md` for complete strategy details, confidence thresholds, and priority order.

See `reference/teams.md` for complete team list and responsibilities.

## Known Limitations

### Version Mismatch

The `/setup` command clones the latest `main` branch of stackrox for CODEOWNERS and reference data. Issues with older `affectedVersions` (e.g., 4.4.x, 4.5.x) may have different file ownership than current main.

See `reference/constants.md` for version mismatch confidence adjustments.

**Mitigation:**
- `/classify` detects version mismatches using `affectedVersions` field
- Reports flag mismatched issues with ⚠️ symbol
- Confidence scores automatically adjusted

### READ-ONLY Mode

This workflow generates reports and recommendations but **does not modify JIRA automatically**. All actions must be executed manually by humans after review.

### Timeout Constraint

Total workflow execution limited to 300 seconds (5 minutes):
- Issue limit: 10-20 per session
- Prioritizes highest priority and oldest created
- If timeout occurs, process what was fetched

## Configuration

### JIRA Filters

- **Filter 103399** - Current untriaged issues (primary)
- **Filter 95004** - Previous duty issues (fallback)

### File Locations

```
/tmp/triage/
├── stackrox/                    # Cloned by /setup
│   ├── .github/CODEOWNERS       # Team ownership by file path
│   └── .claude/agents/
│       └── stackrox-ci-failure-investigator.md
├── skills/                      # Cloned by /setup
│   └── plugins/rhacs-patch-eval/
└── artifacts/acs-triage/        # Workflow outputs
    ├── setup-info.json
    ├── issues.json
    ├── triage-report.md
    ├── report.html
    ├── slack-summary.md
    └── summary.json
```

## Output Formats

### Markdown Report (triage-report.md)

- Best for: Reading, sharing, archiving
- Sections: Metadata, executive summary, statistics, triage table, high-confidence recommendations
- Format: GitHub-flavored markdown

### HTML Dashboard (report.html)

- Best for: Interactive exploration, filtering, sorting
- Features: Stats cards, filters, sortable table, search
- Requirements: Modern browser with JavaScript

### Slack Summary (slack-summary.md)

- Best for: Quick team notifications
- Content: High-confidence recommendations, team counts, version mismatch warnings
- Format: Copy/paste ready for Slack

### Summary JSON (summary.json)

- Best for: Automation, metrics tracking, programmatic access
- Content: Statistics, high-confidence issues, manual review needed
- Format: Machine-readable JSON

## Best Practices

1. **Run /setup first** - Ensures latest CODEOWNERS and reference data
2. **Review high-confidence recommendations** (≥90%) - Usually accurate, but verify
3. **Manually review low-confidence** (<70%) - Requires human judgment
4. **Check version mismatches** - Older versions may have different ownership
5. **Use HTML dashboard for exploration** - Interactive filters help find patterns
6. **Share Slack summary with team** - Keeps everyone informed
7. **Track metrics** - Use summary.json for trend analysis
8. **Re-run periodically** - New issues arrive daily

## Troubleshooting

### Problem: JIRA MCP timeout

**Solution:**
- Reduce issue limit in `/fetch-issues` (10 instead of 20)
- Run multiple smaller batches
- Check JIRA API rate limits

### Problem: Git clone fails in /setup

**Solution:**
- Check network connectivity
- Verify Git credentials
- Use `git pull` if repositories already cloned

### Problem: No team assignment (0% confidence)

**Cause:** No CODEOWNERS match, error signature, or service name found

**Solution:**
- Review issue description for clues
- Search similar issues manually
- Assign based on domain knowledge
- Flag as "Needs Manual Assignment"

### Problem: Version mismatch flagged incorrectly

**Cause:** Issue has no `affectedVersions` field

**Solution:**
- Check JIRA issue for version labels
- If truly version-agnostic, ignore warning
- If specific version, update JIRA `affectedVersions` field

## Contributing

To improve this workflow:

1. **Add new error signatures** - Update stackrox-ci-failure-investigator.md in stackrox repo
2. **Document known flaky tests** - Add patterns to stackrox-ci-failure-investigator.md
3. **Improve CODEOWNERS** - Keep stackrox/.github/CODEOWNERS up-to-date
4. **Enhance decision trees** - Update command files for better logic
5. **Report issues** - Create issues for incorrect team assignments

## Support

For issues or questions:
- Open an issue in the ambient-workflows repository
- Review command files in `.claude/commands/` for detailed logic
- Check `FIELD_REFERENCE.md` for field definitions
- Refer to StackRox CODEOWNERS for team ownership questions

---

**Created with:** ACP Workflow Creator
**Workflow Type:** Domain-Specific Triage
**Version:** 1.0.0
**Last Updated:** 2024-04-27

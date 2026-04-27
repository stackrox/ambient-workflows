# Workflow Constants

Central location for all hardcoded values used throughout the ACS triage workflow.

## JIRA Configuration

| Constant | Value | Purpose |
|----------|-------|---------|
| FILTER_CURRENT_UNTRIAGED | 103399 | Current untriaged issues filter |
| FILTER_PREVIOUS_DUTY | 95004 | Previous duty cycle issues filter |
| PROJECT_KEY | ROX | StackRox JIRA project key |

## Confidence Thresholds

| Strategy | Base Confidence | Version Mismatch Adjusted |
|----------|----------------|---------------------------|
| CODEOWNERS Match | 95% | 75% |
| Flaky Pattern Match | 95% | 95% (no adjustment) |
| Error Signature Match | 85-90% | 85-90% (no adjustment) |
| Service Ownership Match | 80% | 75% |
| Similar Issue History | 70-80% | 70-80% (no adjustment) |
| Test Category Match | 70% | 70% |

## Confidence Interpretation

| Range | Classification | Recommendation |
|-------|---------------|----------------|
| ≥90% | High | Ready for automatic assignment |
| 70-89% | Medium | Review before assignment |
| <70% | Low | Manual review required |
| 0% | None | Needs manual assignment |

## Severity Thresholds (Vulnerabilities)

| Severity | CVSS Range | Triage Decision |
|----------|-----------|-----------------|
| Critical | 9.0-10.0 | Always triage |
| Important | 7.0-8.9 | Always triage |
| Moderate | 4.0-6.9 | Triage if CVSS ≥7.0 |
| Low | 0.0-3.9 | Close (Won't Do) |

## Flaky Test Frequency Thresholds

| Frequency | Occurrences (30 days) | Priority | Action |
|-----------|----------------------|----------|--------|
| High | >10 | High | Fix or disable test |
| Medium | 3-10 | Medium | Monitor and fix |
| Low | <3 | Low | Investigate on next occurrence |

## Workflow Constraints

| Constraint | Value | Reason |
|-----------|-------|--------|
| TIMEOUT_SECONDS | 300 | JIRA MCP performance limit |
| MAX_ISSUES_PER_RUN | 10-20 | Keep within timeout |
| SETUP_CLONE_DEPTH | full | Need CODEOWNERS and version tags |

## Output Paths

| Artifact Type | Path |
|--------------|------|
| Setup Info | artifacts/acs-triage/setup-info.json |
| Issues Data | artifacts/acs-triage/issues.json |
| Markdown Report | artifacts/acs-triage/triage-report.md |
| HTML Dashboard | artifacts/acs-triage/report.html |
| Slack Summary | artifacts/acs-triage/slack-summary.md |
| Summary JSON | artifacts/acs-triage/summary.json |

## Issue Type Classifications

| Type | Detection Criteria |
|------|-------------------|
| VULNERABILITY | CVE-* label OR "vulnerability" in summary/labels |
| FLAKY_TEST | "flaky-test" label OR test name in known patterns |
| CI_FAILURE | "CI_Failure" label OR stack trace/error log in description |
| UNKNOWN | None of above patterns match |

## Vulnerability Decision Tree Exit Points

| Step | Exit Condition | Resolution |
|------|---------------|-----------|
| 1 | All versions unsupported | CLOSE (Won't Do) |
| 2 | Severity below threshold | CLOSE (Won't Do) |
| 3 | Container/language mismatch | CLOSE (Obsolete) |
| 4 | CVE already exists | DUPLICATE |
| 5 | VEX not affected | CLOSE (Not a Bug) |
| 6 | Passes all checks | ASSIGN |

## Known False Positive Patterns

| Pattern | Decision |
|---------|----------|
| Database containers (central-db, scanner-db, scanner-v4-db) with npm/Go vulnerabilities | CLOSE (Obsolete) |
| Non-main containers with npm vulnerabilities | CLOSE (Obsolete) |

## Repository Paths

| Repository | Clone Path | Files Needed |
|-----------|-----------|--------------|
| stackrox/stackrox | /tmp/triage/stackrox | .github/CODEOWNERS, VERSION |
| stackrox/skills | /tmp/triage/skills | plugins/rhacs-patch-eval/* |

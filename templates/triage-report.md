# ACS Triage Report

**Date:** {{date}}
**Filter:** 103399 (current untriaged)
**Total Issues:** {{total_issues}}
**StackRox Version:** {{stackrox_version}}

## Executive Summary

{{executive_summary}}

### High Confidence Recommendations (≥90%)

- **{{high_confidence_count}} issues** ready for assignment
- **{{ci_high_count}} CI failures**
- **{{vuln_high_count}} vulnerabilities**
- **{{flaky_high_count}} flaky tests**

### Version Mismatch Warnings

⚠️ **{{version_mismatch_count}} issues** affect older versions than current codebase.
CODEOWNERS and service ownership confidence reduced to 75%. Manual review recommended.

---

## Statistics

### By Type

| Type | Count | Percentage |
|------|-------|------------|
| CI_FAILURE | {{ci_count}} | {{ci_percent}}% |
| VULNERABILITY | {{vuln_count}} | {{vuln_percent}}% |
| FLAKY_TEST | {{flaky_count}} | {{flaky_percent}}% |
| UNKNOWN | {{unknown_count}} | {{unknown_percent}}% |

### By Team

| Team | Count |
|------|-------|
| @stackrox/core-workflows | {{core_count}} |
| @stackrox/scanner | {{scanner_count}} |
| @stackrox/sensor-ecosystem | {{sensor_count}} |
| @stackrox/collector | {{collector_count}} |
| @stackrox/install | {{install_count}} |
| @stackrox/ui | {{ui_count}} |
| **Needs Manual Assignment** | {{manual_count}} |

### By Confidence

| Confidence Level | Count |
|-----------------|-------|
| High (≥90%) | {{high_conf}} |
| Medium (70-89%) | {{med_conf}} |
| Low (<70%) | {{low_conf}} |
| None (0%) | {{no_conf}} |

---

## Main Triage Table

| Issue | Type | Team | Confidence | Version Mismatch | Recommendation | Reason |
|-------|------|------|------------|------------------|----------------|--------|
{{#each issues}}
| [{{key}}](https://issues.redhat.com/browse/{{key}}) | {{type}} | {{team_assignment.assigned_team}} | {{team_assignment.confidence}}% | {{#if version_mismatch}}⚠️{{/if}} | {{recommendation}} | {{team_assignment.reasoning}} |
{{/each}}

---

## High Confidence Recommendations (≥80%)

### CI Failures

{{#each high_confidence_ci}}
#### {{key}}: {{summary}}

- **Team:** {{team_assignment.assigned_team}} ({{team_assignment.confidence}}% confidence)
- **Strategy:** {{team_assignment.strategy}}
- **Error Type:** {{ci_analysis.error_type}}
- **Reasoning:** {{team_assignment.reasoning}}
- **Evidence:**
  - File paths: {{#each ci_analysis.file_paths}}{{this}}, {{/each}}
  - Error pattern: {{ci_analysis.error_signature_match.pattern}}
- **Action:** Assign to {{team_assignment.assigned_team}} for investigation

---
{{/each}}

### Vulnerabilities

{{#each high_confidence_vuln}}
#### {{key}}: {{summary}}

- **Team:** {{team_assignment.assigned_team}} ({{team_assignment.confidence}}% confidence)
- **CVE:** {{vuln_analysis.cve_id}}
- **Severity:** {{vuln_analysis.severity}} (CVSS: {{vuln_analysis.cvss_score}})
- **Container:** {{vuln_analysis.container}}
- **Decision:** {{vuln_analysis.recommendation}}
- **Reasoning:** {{vuln_analysis.reasoning}}
- **Action:** {{#if vuln_analysis.recommendation == "ASSIGN"}}Assign to {{team_assignment.assigned_team}} for remediation{{else}}{{vuln_analysis.recommendation}}{{/if}}

---
{{/each}}

### Flaky Tests

{{#each high_confidence_flaky}}
#### {{key}}: {{summary}}

- **Team:** {{team_assignment.assigned_team}} ({{team_assignment.confidence}}% confidence)
- **Test:** {{flaky_analysis.test_name}}
- **Known Pattern:** {{#if flaky_analysis.known_flaky_pattern}}Yes ({{flaky_analysis.pattern_reference}}){{else}}No{{/if}}
- **Frequency:** {{flaky_analysis.failure_frequency.classification}} ({{flaky_analysis.failure_frequency.count_30d}} in 30 days)
- **Root Cause:** {{flaky_analysis.root_cause}}
- **Action:** Assign to {{team_assignment.assigned_team}} - {{flaky_analysis.recommended_action}}

---
{{/each}}

---

## Version Mismatch Issues (Manual Review Recommended)

{{#each version_mismatch_issues}}
### {{key}}: {{summary}}

- **Affected Versions:** {{#each affected_versions}}{{this}}, {{/each}}
- **Current Version:** {{current_version}}
- **Team:** {{team_assignment.assigned_team}} ({{team_assignment.confidence}}% confidence - **reduced from higher due to version mismatch**)
- **Warning:** {{version_gap}}
- **Recommendation:** Manually verify team ownership for this version

---
{{/each}}

---

## Low Confidence Issues (Manual Assignment Needed)

{{#each low_confidence_issues}}
### {{key}}: {{summary}}

- **Type:** {{type}}
- **Confidence:** {{team_assignment.confidence}}%
- **Best Guess:** {{team_assignment.assigned_team}}
- **Reasoning:** {{team_assignment.reasoning}}
- **Alternative Teams:**
{{#each team_assignment.alternative_teams}}
  - {{team}} ({{confidence}}% via {{strategy}})
{{/each}}
- **Action:** Manual review required - insufficient confidence for automatic assignment

---
{{/each}}

---

## Summary

- **Total Issues Processed:** {{total_issues}}
- **High Confidence Assignments (≥80%):** {{high_conf_total}}
- **Require Manual Review:** {{manual_review_total}}
- **Version Mismatches:** {{version_mismatch_count}}

---

**Generated by:** ACS Triage Workflow
**Timestamp:** {{timestamp}}
**Workflow Version:** 1.0.0

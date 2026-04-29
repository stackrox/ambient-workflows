# /analyze-flaky - Analyze Flaky Test

## Purpose

Pattern matching and frequency analysis for flaky tests. Identifies known flaky test patterns, estimates failure frequency from JIRA history, and assigns to test owners.

## Prerequisites

- artifacts/acs-triage/issues.json exists with type="FLAKY_TEST" issues
- `/setup` completed to access stackrox-ci-failure-investigator.md
- `/classify` completed
- JIRA MCP access for historical search

## Process

1. **Filter Flaky Test Issues**
   - Read artifacts/acs-triage/issues.json
   - Process only issues where type = "FLAKY_TEST"

2. **Extract Test Information**

   From summary and description, extract:
   - **Test name**: Full test name (e.g., TestGlobalSearchLatestTag)
   - **Test file**: File path if available (e.g., tests/e2e/search_test.go)
   - **Test category**: E2E, integration, unit, etc.
   - **Failure pattern**: Specific assertion or error that fails

3. **Match Known Flaky Patterns**

   Read `/tmp/triage/stackrox/.claude/agents/stackrox-ci-failure-investigator.md` and check for known patterns:

   **Known Flaky Tests:**
   - **GlobalSearch Latest Tag** → @stackrox/ui (ROX-5355)
     - Pattern: DNS timing issue
     - Frequency: High

   - **PolicyFieldsTest Process UID** → @stackrox/core-workflows (ROX-5298)
     - Pattern: Timing-dependent validation
     - Frequency: Medium

   - **NetworkFlowTest connections** → @stackrox/collector
     - Pattern: Network timing
     - Frequency: High

   - **ImageScanningTest registries** → @stackrox/scanner
     - Pattern: Registry timing
     - Frequency: Medium

   - **SACTest SSH Port** → @stackrox/sensor-ecosystem
     - Pattern: Permission timing
     - Frequency: Medium

   If test matches known pattern:
   - Set known_flaky_pattern = true
   - Use documented team and historical issue reference
   - Note the root cause from pattern documentation

4. **Estimate Failure Frequency**

   Search JIRA for historical occurrences:
   - Query: `project = ROX AND summary ~ "TestName" AND created >= -30d AND labels = CI_Failure`
   - Count results in last 30 days

   **Frequency Classification:**
   - **High**: >10 occurrences in 30 days
   - **Medium**: 3-10 occurrences in 30 days
   - **Low**: <3 occurrences in 30 days

   Note: This is estimation based on JIRA issues, actual frequency may be higher (many failures don't create tickets)

5. **Assign to Test Owner**

   Priority order for team assignment:

   a. **Use Known Pattern Team** (95% confidence)
      - If test matches known flaky pattern
      - Use documented team assignment

   b. **Use CODEOWNERS for Test File** (90% confidence)
      - If test file path is known
      - Read `/tmp/triage/stackrox/.github/CODEOWNERS`
      - Match test file path to team

   c. **Use Test Category** (70% confidence)
      - E2E tests → @stackrox/ui
      - Integration tests → Service owner
      - Unit tests → Component owner

   d. **Fallback to Service Name** (70% confidence)
      - Extract service from test name
      - Use service ownership mapping

6. **Enrich Issue Object**
   Add flaky test-specific fields:
   ```json
   {
     "flaky_analysis": {
       "test_name": "TestGlobalSearchLatestTag",
       "test_file": "tests/e2e/search_test.go",
       "test_category": "e2e",
       "known_flaky_pattern": true,
       "pattern_reference": "ROX-5355",
       "root_cause": "DNS timing issue in GlobalSearch",
       "failure_frequency": {
         "count_30d": 12,
         "classification": "High",
         "trend": "increasing"
       },
       "assigned_team": "@stackrox/ui",
       "confidence": 95,
       "assignment_strategy": "known_pattern"
     }
   }
   ```

## Output

- **artifacts/acs-triage/issues.json** - Updated with flaky_analysis field for FLAKY_TEST issues

## Usage Examples

Basic usage:
```
/analyze-flaky
```

## Success Criteria

After running this command, you should have:
- [ ] All FLAKY_TEST issues enriched with flaky_analysis data
- [ ] Known patterns matched where applicable
- [ ] Failure frequency estimated from JIRA history
- [ ] Test owner assigned with confidence score

## Next Steps

After flaky test analysis:
1. Run `/assign-team` for final confidence scoring (if not using known pattern)
2. High-frequency flaky tests should be prioritized for fixing

## Notes

- Known pattern matches have highest confidence (95%)
- Frequency estimation is conservative (only counts JIRA issues, not all CI runs)
- Some flaky tests may not be in known patterns - use CODEOWNERS fallback
- High-frequency flakes (>10/month) should be fixed or test disabled
- Test file paths from CI logs are most reliable for CODEOWNERS matching
- Version mismatch from `/classify` affects CODEOWNERS matching confidence
- Trends (increasing/decreasing frequency) help prioritize fixes

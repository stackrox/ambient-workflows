# ACS Triage - Field Reference

This document defines all fields used in the ACS triage workflow, from JIRA extraction through final report generation.

## JIRA Source Fields

These fields are extracted from JIRA by the `/fetch-issues` command:

### key
- **Type:** string
- **Example:** "ROX-12345"
- **Purpose:** Unique JIRA issue identifier
- **Source:** JIRA API

### summary
- **Type:** string
- **Example:** "UI E2E test failing: GlobalSearch Latest Tag"
- **Purpose:** Brief issue description
- **Source:** JIRA API

### description
- **Type:** string (markdown)
- **Purpose:** Full issue description, often contains error logs for CI failures
- **Source:** JIRA API
- **Note:** Critical for CI failures - contains stack traces, build IDs, error messages

### labels
- **Type:** array of strings
- **Example:** `["CI_Failure", "ui", "flaky-test"]`
- **Purpose:** Issue categorization tags
- **Source:** JIRA API
- **Used For:** Issue type classification

### components
- **Type:** array of strings
- **Example:** `["UI", "Frontend"]`
- **Purpose:** StackRox component/service affected
- **Source:** JIRA API
- **Used For:** Service ownership matching

### priority
- **Type:** string
- **Values:** "Critical", "High", "Medium", "Low"
- **Purpose:** Issue severity
- **Source:** JIRA API

### status
- **Type:** string
- **Example:** "Open", "In Progress", "Resolved"
- **Purpose:** Current issue state
- **Source:** JIRA API

### created
- **Type:** ISO 8601 timestamp
- **Example:** "2024-04-15T10:30:00Z"
- **Purpose:** Issue creation timestamp
- **Source:** JIRA API

### updated
- **Type:** ISO 8601 timestamp
- **Example:** "2024-04-15T12:00:00Z"
- **Purpose:** Last update timestamp
- **Source:** JIRA API

### affectedVersions
- **Type:** array of strings
- **Example:** `["4.5.0", "4.5.1"]`
- **Purpose:** StackRox versions affected by this issue
- **Source:** JIRA API
- **Used For:** Version mismatch detection
- **Note:** May be empty for unversioned issues

### fixVersions
- **Type:** array of strings
- **Example:** `["4.6.0"]`
- **Purpose:** Target release for fix
- **Source:** JIRA API

### comments
- **Type:** array of comment objects
- **Example:** `[{"author": "jenkins", "created": "...", "body": "Error log: ..."}]`
- **Purpose:** Issue discussion and automated build failure logs
- **Source:** JIRA API
- **Note:** Jenkins/CI comments contain critical error details

## Classification Fields

These fields are added by the `/classify` command:

### type
- **Type:** enum
- **Values:** "CI_FAILURE", "VULNERABILITY", "FLAKY_TEST", "UNKNOWN"
- **Purpose:** Issue category for specialized analysis
- **Added By:** `/classify`
- **Logic:**
  - VULNERABILITY: CVE-* labels or CVE patterns in summary
  - FLAKY_TEST: flaky-test labels or known flaky patterns
  - CI_FAILURE: CI_Failure labels or stack traces in description
  - UNKNOWN: No patterns match

### classification_confidence
- **Type:** enum
- **Values:** "high", "medium", "low"
- **Purpose:** How certain the classification is
- **Added By:** `/classify`
- **Note:** "low" suggests manual review needed

### classification_reason
- **Type:** string
- **Example:** "Label 'CI_Failure' + stack trace in description"
- **Purpose:** Documents why this classification was chosen
- **Added By:** `/classify`

### version_mismatch
- **Type:** boolean or null
- **Values:** true, false, null (unknown)
- **Purpose:** Indicates if issue affects older versions than current codebase
- **Added By:** `/classify`
- **Impact:** Reduces CODEOWNERS and service ownership confidence

### current_version
- **Type:** string
- **Example:** "4.7.0-dev"
- **Purpose:** StackRox version cloned by `/setup`
- **Added By:** `/classify`
- **Source:** artifacts/acs-triage/setup-info.json

### version_gap
- **Type:** string
- **Example:** "Issue affects 4.5.x, using 4.7.x CODEOWNERS - file paths may differ"
- **Purpose:** Human-readable version mismatch explanation
- **Added By:** `/classify`

## CI Failure Analysis Fields

These fields are added by the `/analyze-ci` command for CI_FAILURE issues:

### ci_analysis
- **Type:** object
- **Purpose:** CI-specific analysis data
- **Added By:** `/analyze-ci`

#### ci_analysis.build_id
- **Type:** string (numeric)
- **Example:** "1963388448995807232"
- **Purpose:** Prow/CI build identifier
- **Extracted From:** Issue description/comments

#### ci_analysis.job_name
- **Type:** string
- **Example:** "pull-ci-stackrox-stackrox-master-e2e-tests"
- **Purpose:** CI job that failed
- **Extracted From:** Issue description/comments

#### ci_analysis.pr_number
- **Type:** string
- **Example:** "12345"
- **Purpose:** GitHub PR number if applicable
- **Extracted From:** Issue description/comments

#### ci_analysis.test_name
- **Type:** string
- **Example:** "TestGlobalSearchLatestTag"
- **Purpose:** Specific test that failed
- **Extracted From:** Issue summary or description

#### ci_analysis.error_type
- **Type:** enum
- **Values:** "GraphQL", "panic", "timeout", "network", "image", "infrastructure"
- **Purpose:** Error category for team assignment
- **Added By:** `/analyze-ci` pattern matching

#### ci_analysis.error_message
- **Type:** string
- **Example:** "GraphQL schema validation failed"
- **Purpose:** Primary error extracted from logs
- **Extracted From:** Comments/description

#### ci_analysis.file_paths
- **Type:** array of strings
- **Example:** `["central/graphql/resolvers/policies.go", "ui/apps/platform/src/queries/policies.ts"]`
- **Purpose:** Files mentioned in stack traces for CODEOWNERS matching
- **Extracted From:** Stack traces in comments/description

#### ci_analysis.stack_trace_summary
- **Type:** string
- **Example:** "panic in graphql resolver"
- **Purpose:** Brief stack trace summary
- **Extracted From:** Stack traces in comments/description

#### ci_analysis.error_signature_match
- **Type:** object or null
- **Purpose:** Match against known error patterns
- **Fields:**
  - `pattern`: string - Matched pattern
  - `confidence`: number (0-100) - Confidence in match
  - `suggested_team`: string - Team for this error type

#### ci_analysis.known_flaky
- **Type:** boolean
- **Purpose:** Test matches known flaky pattern
- **Impact:** May reclassify as FLAKY_TEST

## Vulnerability Analysis Fields

These fields are added by the `/analyze-vuln` command for VULNERABILITY issues:

### vuln_analysis
- **Type:** object
- **Purpose:** Vulnerability-specific analysis
- **Added By:** `/analyze-vuln`

#### vuln_analysis.cve_id
- **Type:** string
- **Example:** "CVE-2024-1234"
- **Purpose:** CVE identifier
- **Extracted From:** Issue summary

#### vuln_analysis.severity
- **Type:** enum
- **Values:** "Critical", "Important", "Moderate", "Low"
- **Purpose:** Vulnerability severity level
- **Extracted From:** Issue description or labels

#### vuln_analysis.cvss_score
- **Type:** number (0.0-10.0)
- **Example:** 7.5
- **Purpose:** CVSS v3 score
- **Extracted From:** Issue description

#### vuln_analysis.container
- **Type:** string
- **Example:** "scanner", "central", "sensor"
- **Purpose:** Affected container/service
- **Extracted From:** Issue description or labels

#### vuln_analysis.language
- **Type:** string
- **Example:** "Go", "npm", "Python"
- **Purpose:** Programming language of affected component
- **Extracted From:** Issue description

#### vuln_analysis.component
- **Type:** string
- **Example:** "github.com/example/pkg"
- **Purpose:** Specific package/library affected
- **Extracted From:** Issue description

#### vuln_analysis.decision_tree
- **Type:** object
- **Purpose:** Documents ProdSec decision tree execution
- **Fields:**
  - `step1_version_support`: "pass" or "fail"
  - `step2_severity_threshold`: "pass" or "fail"
  - `step3_container_applicability`: "pass" or "fail"
  - `step4_duplicate_check`: "pass" or "fail"
  - `step5_impact_analysis`: "pass" or "fail"
  - `step6_team_assignment`: team name

#### vuln_analysis.recommendation
- **Type:** enum
- **Values:** "ASSIGN", "CLOSE", "DUPLICATE"
- **Purpose:** Recommended action from decision tree
- **Note:** "CLOSE" has sub-types (Won't Do, Obsolete, VEX)

#### vuln_analysis.assigned_team
- **Type:** string
- **Example:** "@stackrox/scanner"
- **Purpose:** Team assignment for ASSIGN recommendations
- **Confidence:** 85% (from container mapping)

#### vuln_analysis.reasoning
- **Type:** string
- **Example:** "Critical severity Go vulnerability in scanner container, no duplicates found"
- **Purpose:** Explains decision tree outcome

## Flaky Test Analysis Fields

These fields are added by the `/analyze-flaky` command for FLAKY_TEST issues:

### flaky_analysis
- **Type:** object
- **Purpose:** Flaky test-specific analysis
- **Added By:** `/analyze-flaky`

#### flaky_analysis.test_name
- **Type:** string
- **Example:** "TestGlobalSearchLatestTag"
- **Purpose:** Full test name
- **Extracted From:** Issue summary

#### flaky_analysis.test_file
- **Type:** string
- **Example:** "tests/e2e/search_test.go"
- **Purpose:** Test file path for CODEOWNERS matching
- **Extracted From:** Issue description or CI logs

#### flaky_analysis.test_category
- **Type:** enum
- **Values:** "e2e", "integration", "unit"
- **Purpose:** Test type categorization
- **Extracted From:** Test file path or name patterns

#### flaky_analysis.known_flaky_pattern
- **Type:** boolean
- **Purpose:** Matches documented flaky test
- **Impact:** If true, use known pattern team (95% confidence)

#### flaky_analysis.pattern_reference
- **Type:** string
- **Example:** "ROX-5355"
- **Purpose:** Historical issue reference for known flaky
- **Source:** stackrox-ci-failure-investigator.md

#### flaky_analysis.root_cause
- **Type:** string
- **Example:** "DNS timing issue in GlobalSearch"
- **Purpose:** Known root cause from pattern documentation
- **Source:** stackrox-ci-failure-investigator.md

#### flaky_analysis.failure_frequency
- **Type:** object
- **Fields:**
  - `count_30d`: number - Occurrences in last 30 days
  - `classification`: enum - "High" (>10), "Medium" (3-10), "Low" (<3)
  - `trend`: enum - "increasing", "stable", "decreasing"

#### flaky_analysis.assigned_team
- **Type:** string
- **Example:** "@stackrox/ui"
- **Purpose:** Test owner team

#### flaky_analysis.confidence
- **Type:** number (0-100)
- **Example:** 95
- **Purpose:** Confidence in team assignment

#### flaky_analysis.assignment_strategy
- **Type:** enum
- **Values:** "known_pattern", "codeowners", "test_category", "service_ownership"
- **Purpose:** Which strategy was used for assignment

## Team Assignment Fields

These fields are added by the `/assign-team` command for all issues:

### team_assignment
- **Type:** object
- **Purpose:** Final team assignment with confidence
- **Added By:** `/assign-team`

#### team_assignment.assigned_team
- **Type:** string
- **Example:** "@stackrox/scanner"
- **Values:** Team handle or "Needs Manual Assignment"
- **Purpose:** Final team assignment

#### team_assignment.confidence
- **Type:** number (0-100)
- **Example:** 90
- **Purpose:** Confidence in assignment
- **Thresholds:**
  - ≥90%: High confidence
  - 70-89%: Medium confidence
  - <70%: Low confidence
  - 0%: No assignment

#### team_assignment.strategy
- **Type:** enum
- **Values:** "codeowners", "error_signature", "service_ownership", "similar_issues", "test_category"
- **Purpose:** Which strategy produced this assignment

#### team_assignment.reasoning
- **Type:** string
- **Example:** "GraphQL schema validation error pattern matches core-workflows team with 90% confidence"
- **Purpose:** Human-readable explanation

#### team_assignment.evidence
- **Type:** object
- **Purpose:** Supporting data for assignment
- **Fields:**
  - `file_paths`: array of strings - CODEOWNERS matches
  - `error_pattern`: string - Error signature match
  - `service`: string - Service ownership match
  - `similar_issues`: array - Historical issue references

#### team_assignment.version_mismatch_adjustment
- **Type:** boolean
- **Purpose:** Whether confidence was reduced due to version mismatch
- **Note:** If true, original confidence was higher (e.g., 95% → 75%)

#### team_assignment.alternative_teams
- **Type:** array of objects
- **Purpose:** Other possible team assignments
- **Fields:**
  - `team`: string - Team name
  - `confidence`: number - Confidence for this team
  - `strategy`: string - Strategy used
  - `reason`: string - Why this team could be responsible

## Report Summary Fields

These fields are calculated by the `/generate-report` command:

### Metadata Fields

- `report_date`: ISO 8601 timestamp of report generation
- `jira_filters`: Array of filter IDs used (103399, 95004)
- `total_issues`: Number of issues processed
- `stackrox_version`: Version from setup-info.json

### Statistics Fields

#### by_type
- `CI_FAILURE`: count
- `VULNERABILITY`: count
- `FLAKY_TEST`: count
- `UNKNOWN`: count

#### by_team
- `@stackrox/core-workflows`: count
- `@stackrox/sensor-ecosystem`: count
- `@stackrox/scanner`: count
- `@stackrox/collector`: count
- `@stackrox/install`: count
- `@stackrox/ui`: count
- `Needs Manual Assignment`: count

#### by_confidence
- `high` (≥90%): count
- `medium` (70-89%): count
- `low` (<70%): count
- `none` (0%): count

#### by_recommendation
- `ASSIGN`: count
- `CLOSE`: count
- `DUPLICATE`: count
- `MANUAL_REVIEW`: count

**See:** `reference/constants.md` for confidence thresholds, strategy priority, and version mismatch adjustments

## Example Issue Object Structure

```json
{
  "key": "ROX-12345",
  "summary": "...",
  "description": "...",
  "labels": ["CI_Failure"],
  "components": ["UI"],
  "affectedVersions": ["4.5.0"],

  "type": "CI_FAILURE",
  "version_mismatch": true,

  "ci_analysis": {
    "error_type": "GraphQL",
    "file_paths": ["central/graphql/resolvers/policies.go"],
    "error_signature_match": { "pattern": "...", "confidence": 90 }
  },

  "team_assignment": {
    "assigned_team": "@stackrox/core-workflows",
    "confidence": 90,
    "strategy": "error_signature",
    "reasoning": "...",
    "alternative_teams": [...]
  }
}
```

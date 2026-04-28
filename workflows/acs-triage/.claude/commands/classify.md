# /classify - Categorize Issue Type

## Purpose

Determine the issue category (CI_FAILURE, VULNERABILITY, FLAKY_TEST, or UNKNOWN) based on labels, summary, and description patterns. Additionally, detect version mismatches between issue affectedVersions and current stackrox codebase. Accurate classification enables specialized analysis workflows for each type.

## Prerequisites

- artifacts/acs-triage/issues.json exists (run `/fetch-issues` first)
- artifacts/acs-triage/setup-info.json exists (run `/setup` first for version detection)

## Process

1. **Read Issues Data**
   - Read artifacts/acs-triage/issues.json
   - Read artifacts/acs-triage/setup-info.json for current version
   - Process each issue sequentially

2. **Apply Classification Logic**

   For each issue, check in priority order:

   **VULNERABILITY Detection:**
   - Has label matching "CVE-*" OR "vulnerability" OR "security"
   - Summary contains "CVE-" pattern (CVE-YYYY-NNNNN)
   - Description mentions "vulnerability", "CVSS", "security advisory"
   - **If match → type = "VULNERABILITY"**

   **FLAKY_TEST Detection:**
   - Has label "flaky-test" OR "test-flake"
   - Summary contains test name from known flaky patterns
   - Description mentions "intermittent", "flaky", "sometimes passes"
   - **If match → type = "FLAKY_TEST"**

   **CI_FAILURE Detection:**
   - Has label "CI_Failure" OR "build-failure"
   - Summary contains "test failing", "build failure", "CI failed"
   - Description contains stack trace patterns ("panic:", "FATAL", "ERROR")
   - Description contains error log markers ("goroutine", "stack trace", "failed with exit code")
   - **If match → type = "CI_FAILURE"**

   **UNKNOWN Fallback:**
   - No patterns match above categories
   - **type = "UNKNOWN"** (requires manual classification)

3. **Detect Version Mismatch**

   For each issue with affectedVersions:

   a. **Extract Affected Versions**
      - Read affectedVersions field from issue (e.g., ["4.5.0", "4.5.1"])
      - Read fixVersions field if present (e.g., ["4.6.0"])

   b. **Compare Against Current Version**
      - Read stackrox_version from artifacts/acs-triage/setup-info.json
      - Parse major.minor from both (e.g., "4.5.0" → 4.5, "4.7.0-dev" → 4.7)

   c. **Determine Mismatch**
      - If any affectedVersion < current version (e.g., 4.5 < 4.7):
        - Set version_mismatch = true
        - Note the version gap (e.g., "Issue affects 4.5.x, using 4.7.x CODEOWNERS")
      - If all affectedVersions match current major.minor:
        - Set version_mismatch = false
      - If no affectedVersions field:
        - Set version_mismatch = null (unknown)

   d. **Add Version Metadata**
      ```json
      {
        "version_mismatch": true,
        "affected_versions": ["4.5.0", "4.5.1"],
        "fix_versions": ["4.6.0"],
        "current_version": "4.7.0-dev",
        "version_gap_years": "0.2"
      }
      ```

4. **Enrich Issue Objects**
   - Add "type" field to each issue object
   - Add "classification_confidence" if pattern matching is ambiguous
   - Add "classification_reason" documenting which pattern matched
   - Add version mismatch fields
   - Preserve all original fields

5. **Update Issues File**
   - Write enriched issues back to artifacts/acs-triage/issues.json

## Output

- **artifacts/acs-triage/issues.json** - Updated with "type" and version mismatch fields

Example:
```json
[
  {
    "key": "ROX-12345",
    "summary": "UI E2E test failing: GlobalSearch Latest Tag",
    "affectedVersions": ["4.5.0", "4.5.1"],
    "fixVersions": ["4.6.0"],
    "type": "CI_FAILURE",
    "classification_confidence": "high",
    "classification_reason": "Label 'CI_Failure' + stack trace in description",
    "version_mismatch": true,
    "current_version": "4.7.0-dev",
    "version_gap": "Issue affects 4.5.x, using 4.7.x CODEOWNERS - file paths may differ"
  }
]
```

## Usage Examples

Basic usage:
```
/classify
```

## Success Criteria

After running this command, you should have:
- [ ] All issues have a "type" field (CI_FAILURE, VULNERABILITY, FLAKY_TEST, or UNKNOWN)
- [ ] Classification confidence noted for edge cases
- [ ] Version mismatch detected for issues with older affectedVersions
- [ ] Version metadata added to all issues
- [ ] Updated issues.json file

## Next Steps

After classification:
1. Run specialized analysis commands:
   - `/analyze-ci` for CI_FAILURE issues
   - `/analyze-vuln` for VULNERABILITY issues
   - `/analyze-flaky` for FLAKY_TEST issues
2. UNKNOWN issues skip specialized analysis and go directly to `/assign-team`
3. Version-mismatched issues will have adjusted confidence scores in `/assign-team`

## Notes

- Multiple labels may be present - prioritize VULNERABILITY > FLAKY_TEST > CI_FAILURE
- Some issues may legitimately be UNKNOWN - flag for manual review
- Issues without affectedVersions field are common (feature requests, design docs)
- Version mismatch detection helps calibrate team assignment confidence
- CI failures often have multiple error signatures - capture all in comments
- Version gap calculation: difference between affected and current (e.g., 4.5 → 4.7 = 0.2 years)

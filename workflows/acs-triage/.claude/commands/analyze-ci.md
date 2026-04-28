# /analyze-ci - Analyze CI Failure

## Purpose

Deep analysis of CI failures with error classification, file path extraction, and error signature matching. This command enriches CI_FAILURE issues with technical details needed for accurate team assignment.

## Prerequisites

- artifacts/acs-triage/issues.json exists with type="CI_FAILURE" issues
- `/setup` completed to access stackrox-ci-failure-investigator.md
- `/classify` completed

## Process

1. **Filter CI Failure Issues**
   - Read artifacts/acs-triage/issues.json
   - Process only issues where type = "CI_FAILURE"

2. **Extract Failure Information**

   From description and comments, extract:

   a. **Build Metadata**
      - Build ID (numeric, e.g., 1963388448995807232)
      - Job name (pull-ci-stackrox-stackrox-*)
      - PR number
      - Test name

   b. **Error Messages**
      - Primary error message (first ERROR/FATAL line)
      - Full error context (surrounding lines)
      - Error patterns (panic, FATAL, timeout, etc.)

   c. **Stack Traces**
      - Goroutine stack traces (if panic)
      - File paths with line numbers
      - Function names in call stack

   d. **File Paths**
      - Extract all file paths mentioned in logs
      - Example: `central/graphql/resolvers/policies.go:142`
      - Normalize paths (remove line numbers for matching)

3. **Classify Error Type**

   Check description/comments for patterns:

   **GraphQL Errors** (90% confidence → @stackrox/core-workflows)
   - "GraphQL schema validation"
   - "Cannot query field"
   - "__Schema"
   - "placeholder Boolean"

   **Service Crashes** (85% confidence, team depends on service)
   - "panic:"
   - "FATAL"
   - "nil pointer dereference"
   - Extract service name from stack trace

   **Timeout/Performance** (80% confidence → @stackrox/collector)
   - "deadline exceeded"
   - "context deadline"
   - "timeout"
   - "Timed out after"

   **Network Issues** (80% confidence → @stackrox/collector)
   - "connection refused"
   - "dial tcp"
   - "DNS resolution failed"
   - "network unreachable"

   **Image/Scanning** (85% confidence → @stackrox/scanner)
   - "image pull"
   - "scanner"
   - "vulnerability detection"
   - "registry error"

   **Test Infrastructure** (75% confidence → @stackrox/core-workflows)
   - "cluster provision"
   - "namespace creation"
   - "test setup failed"

4. **Load Error Signatures**
   - Read `/tmp/triage/stackrox/.claude/agents/stackrox-ci-failure-investigator.md`
   - Extract additional error patterns and team mappings
   - Match against issue description/comments

5. **Check for Known Flaky Patterns**
   - Cross-reference test name against known flaky tests
   - If match found, note pattern and historical frequency
   - This may reclassify as FLAKY_TEST

6. **Enrich Issue Object**
   Add CI-specific fields:
   ```json
   {
     "ci_analysis": {
       "build_id": "1963388448995807232",
       "job_name": "pull-ci-stackrox-stackrox-master-e2e-tests",
       "pr_number": "12345",
       "test_name": "TestGlobalSearchLatestTag",
       "error_type": "GraphQL",
       "error_message": "GraphQL schema validation failed",
       "file_paths": ["ui/apps/platform/src/queries/policies.ts", "central/graphql/resolvers/policies.go"],
       "stack_trace_summary": "panic in graphql resolver",
       "error_signature_match": {
         "pattern": "GraphQL schema validation",
         "confidence": 90,
         "suggested_team": "@stackrox/core-workflows"
       },
       "known_flaky": false
     }
   }
   ```

## Output

- **artifacts/acs-triage/issues.json** - Updated with ci_analysis field for CI_FAILURE issues

## Usage Examples

Basic usage:
```
/analyze-ci
```

## Success Criteria

After running this command, you should have:
- [ ] All CI_FAILURE issues enriched with ci_analysis data
- [ ] Error types classified
- [ ] File paths extracted and normalized
- [ ] Error signatures matched
- [ ] Known flaky patterns checked

## Next Steps

After CI analysis:
1. Run `/assign-team` to perform multi-strategy team assignment
2. Error signature matches provide 85-90% confidence team assignments

## Notes

- Some CI failures may not have clear file paths - use error signatures
- Panics typically have best file path information from stack traces
- Timeout errors often lack specific file paths - use service name
- Known flaky patterns may suggest reclassifying to FLAKY_TEST
- Version mismatches from `/classify` don't affect error classification (errors are stable across versions)
- Build IDs are useful for manual investigation but not used in automated triage

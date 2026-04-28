# /triage - Automated ACS Issue Triage

## Purpose

Complete end-to-end triage workflow for StackRox/ACS JIRA issues. Fetches untriaged issues, classifies by type, performs specialized analysis, assigns teams with confidence scoring, and generates comprehensive reports.

**Options:**
- `/triage` - Full triage pipeline (READ-ONLY, no JIRA writes)
- `/triage --comment` - Full triage + post comments to JIRA issues

## Prerequisites

- JIRA MCP connection configured
- Access to JIRA filter 103399 (current untriaged)
- GitHub MCP for CODEOWNERS (optional, improves accuracy)

## Workflow Pipeline

This command executes the following phases:

### Phase 1 & 2: Setup + Fetch (Run in Parallel)

**PERFORMANCE OPTIMIZATION:** These phases have no interdependencies and SHOULD run concurrently to save 10-20 seconds.

#### Phase 1a: Setup (if needed) - Async
Clone StackRox repository for CODEOWNERS and reference data if not already present.

**Actions:**
- Check if `/tmp/triage/stackrox/.github/CODEOWNERS` exists
- If missing, clone `https://github.com/stackrox/stackrox` to `/tmp/triage/stackrox`
- Extract current version from `VERSION` file

**Output:** Setup metadata in `artifacts/acs-triage/setup-info.json`

#### Phase 1b: Fetch Issues - Async
Query JIRA filter 103399 for untriaged issues.

**Actions:**
- Query JIRA filter 103399 (ONE query, order by priority DESC then created ASC)
- Limit to 10-20 issues (timeout constraint: 300s)
- Extract: key, summary, description, labels, components, priority, status, created, updated, affectedVersions, fixVersions, comments

**Output:** Raw issue data in `artifacts/acs-triage/issues.json`

**Wait for both Phase 1a and 1b to complete before proceeding to Phase 3.**

### Phase 3: Classify
Categorize issues by type and detect version mismatches.

**Classification Logic:**
- **VULNERABILITY**: CVE-* labels OR "vulnerability" in summary
- **FLAKY_TEST**: "flaky-test" label OR test name in known patterns
- **CI_FAILURE**: "build-failure" label OR contains stack trace/error log
- **UNKNOWN**: None of the above patterns match

**Version Mismatch Detection:**
- Compare issue.affectedVersions with current stackrox VERSION
- Set `version_mismatch: true` if issue versions < current version
- Impact: Reduces confidence scores for CODEOWNERS/file-path matches

**Output:** Issues enriched with `issueType` and `version_mismatch` fields

### Phase 4: Specialized Analysis (MUST RUN IN PARALLEL)

**CRITICAL FOR PERFORMANCE:** Execute these three operations concurrently using parallel tool calls. This saves 60-80 seconds. Do NOT run sequentially.

**Implementation:** Load issues.json ONCE into memory, then invoke three analysis operations in parallel, each processing its subset of issues based on issueType.

Run type-specific analysis for each issue based on its classification:

#### 4a. CI Failure Analysis
For issues where `issueType === "CI_FAILURE"`:

- Extract error messages from description/comments
- Classify error type: GraphQL, panic, timeout, network, test failure, etc.
- Extract file paths from stack traces
- Match against known error signatures from `reference/error-signatures.md`
- Store in `ci_analysis` object:
  ```json
  {
    "error_type": "graphql_schema_validation",
    "error_message": "GraphQL schema validation failed...",
    "file_paths": ["/central/graphql/schema.go"],
    "stack_trace": "...",
    "matched_signature": "graphql_schema_validation"
  }
  ```

#### 4b. Vulnerability Analysis
For issues where `issueType === "VULNERABILITY"`:

Apply ProdSec decision tree from `reference/vulnerability-decision-tree.md`:

1. **Version Support Check**: Is affected version still supported?
   - No → Recommend CLOSE with reason "Unsupported version"
2. **Severity Check**: Is severity Critical/High?
   - Low → Consider CLOSE or LOW priority
3. **Container Applicability**: Does it affect containers/images?
   - No → May be out of scope
4. **Duplicate Detection**: Search JIRA for existing CVE
   - Duplicate → Recommend CLOSE, link to original
5. **Impact Analysis**: What component is affected?
   - Extract from CVE description or component field
6. **Team Assignment**: Map component to team

Store in `vuln_analysis` object

#### 4c. Flaky Test Analysis
For issues where `issueType === "FLAKY_TEST"`:

- Extract test name from summary/description
- Match against patterns in `reference/flaky-test-patterns.md`
- Search JIRA history for similar test failures (frequency)
- Classify frequency: >10/month = High, 3-10 = Medium, <3 = Low
- Store in `flaky_analysis` object

### Phase 5: Team Assignment
Apply multi-strategy approach with confidence scoring.

**5 Strategies (priority order):**

1. **CODEOWNERS Match (95% confidence)** - File path → team mapping
   - Source: `/tmp/triage/stackrox/.github/CODEOWNERS`
   - Match file_paths from ci_analysis
   - **Exception:** If match is @janisz for CI_FAILURE, skip (he only reviews Groovy, not responsible for CI failures)
   - Adjust to 75% if version_mismatch

2. **Error Signature Match (85-90% confidence)** - Known error → team
   - Source: `reference/error-signatures.md`
   - Match error_type from ci_analysis
   - Examples: GraphQL → core-workflows (90%), panic → extract service (85%)

3. **Service Ownership Match (80% confidence)** - Component/service → team
   - Source: `reference/team-mappings.md`
   - Map components field or extracted service name
   - Examples: scanner → @stackrox/scanner (80%)

4. **Similar Issue History (70-80% confidence)** - JIRA search for resolved similar issues
   - Search by error message or component
   - Use team from most recent resolution
   - Confidence based on recency and similarity

5. **Test Category Match (70% confidence)** - Test name → file path → CODEOWNERS
   - Extract file path from test name
   - Map to CODEOWNERS

**Confidence Adjustment:**
- Base confidence from strategy
- If version_mismatch AND strategy uses file paths: reduce by 20%
- If multiple strategies agree: use highest confidence
- If no match: "Needs Manual Assignment" with evidence summary

**Output:** Issues enriched with `team_assignment` object:
```json
{
  "team": "@stackrox/core-workflows",
  "confidence": 90,
  "strategy": "error_signature",
  "reasoning": "GraphQL schema validation error pattern matches core-workflows ownership",
  "version_adjusted": false
}
```

### Phase 6: Generate Reports
Create output reports in two formats.

**6a. Markdown Report** (`artifacts/acs-triage/triage-report.md`)
- Summary statistics (by type, team, confidence)
- Detailed table with all triaged issues
- Low confidence section flagged for manual review

**6b. Machine-Readable Summary** (`artifacts/acs-triage/summary.json`)
```json
{
  "total_issues": 15,
  "by_type": {"CI_FAILURE": 8, "VULNERABILITY": 4, "FLAKY_TEST": 2, "UNKNOWN": 1},
  "by_team": {"@stackrox/core-workflows": 5, "Needs Manual": 3},
  "high_confidence_count": 10,
  "version_mismatches": 2
}
```

### Phase 7: Comment to JIRA (Optional)
Only if `--comment` flag is provided.

**Actions:**
- For each issue with confidence ≥80%:
  - Post structured comment with team recommendation, confidence, reasoning
  - Use comment format from `templates/jira-comment.md`
- Skip issues with low confidence (<80%)
- Log all posted comments

**Comment Template:** See `templates/jira-comment.md` for format and variable substitution.

## Output

All artifacts are created in `artifacts/acs-triage/`:

- `setup-info.json` - Setup metadata (version, timestamp)
- `issues.json` - Complete issue data with all enrichments
- `triage-report.md` - Detailed markdown report
- `summary.json` - Machine-readable summary

## Usage Examples

Basic triage (read-only):
```
/triage
```

Triage with JIRA comments:
```
/triage --comment
```

## Success Criteria

After running this command, you should have:
- [ ] Fetched 10-20 untriaged issues from JIRA
- [ ] Classified all issues by type
- [ ] Performed specialized analysis for each type
- [ ] Assigned teams with confidence scores
- [ ] Generated all report formats
- [ ] (If --comment) Posted comments to high-confidence issues

## Error Handling

- **JIRA timeout**: Process what you have, note incomplete in report
- **Unknown issue type**: Mark as UNKNOWN, include raw description for manual triage
- **No team match**: Use "Needs Manual Assignment" with evidence summary
- **Version mismatch**: Flag in report with ⚠️ symbol, adjust confidence
- **Missing CODEOWNERS**: Proceed with other strategies, note limitation in report

## Performance Optimization Guidelines

**File I/O:**
- Load `issues.json` ONCE into memory at Phase 3
- Pass the issues array to all analysis functions
- Load reference files (CODEOWNERS, error-signatures.md, etc.) once and cache in memory

**JIRA Queries:**
- Primary fetch: ONE query for filter 103399 (Phase 1b)
- Similar issue searches (Phase 5): Batch by component, cache results (max 3-5 batched queries)
- Do NOT query JIRA separately for each issue

**Parallel Execution:**
- Phase 1a + 1b: Run setup and fetch concurrently
- Phase 4: Run CI/Vuln/Flaky analysis in parallel (3 concurrent tool calls)
- Total time savings: 70-100 seconds vs sequential execution

## Notes

- **Timeout**: 300 seconds total (5 minutes)
- **Issue Limit**: 10-20 issues to stay within timeout
- **Parallel Analysis**: CI/Vuln/Flaky analysis MUST run concurrently (saves 60-80s)
- **READ-ONLY by default**: Use `--comment` flag to write to JIRA
- **High Confidence Threshold**: ≥80% for auto-assignment recommendations
- **Version Awareness**: Automatically detects and adjusts for version mismatches

## Reference Files

Consult these for domain knowledge:
- `reference/teams.md` - Team list and responsibilities
- `reference/CODEOWNERS-patterns.md` - File path → team mappings
- `reference/error-signatures.md` - Error pattern → team with confidence
- `reference/team-mappings.md` - Component/service → team ownership
- `reference/vulnerability-decision-tree.md` - ProdSec workflow
- `reference/flaky-test-patterns.md` - Known flaky test patterns
- `reference/constants.md` - All confidence thresholds

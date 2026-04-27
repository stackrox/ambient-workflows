# /assign-team - Assign Team with Confidence

## Purpose

Apply multi-strategy team assignment with confidence scoring. Uses 5 strategies in priority order (CODEOWNERS, error signatures, service ownership, similar issues, test category) to assign teams with 95%-70% confidence levels. Adjusts confidence for version mismatches.

## Prerequisites

- artifacts/acs-triage/issues.json exists with classified and analyzed issues
- `/setup` completed to access CODEOWNERS and reference data
- Specialized analysis completed (`/analyze-ci`, `/analyze-vuln`, `/analyze-flaky`)

## Process

1. **Read Issue Data**
   - Read artifacts/acs-triage/issues.json
   - Process each issue that doesn't already have high-confidence team assignment

2. **Apply 5-Strategy Priority System**

   For each issue, attempt strategies in priority order until a match is found:

   ### Strategy 1: CODEOWNERS Match (95% confidence base)

   - **Source**: Read `/tmp/triage/stackrox/.github/CODEOWNERS`
   - **Match**: File paths extracted from CI analysis or vulnerability component paths
   - **Logic**:
     - For CI failures: Use file_paths from ci_analysis
     - For vulnerabilities: Map component to file path
     - Match file paths against CODEOWNERS patterns

   **Confidence Adjustment**:
   - Base: 95%
   - If version_mismatch = true: 95% → 75% (file paths may have moved)
   - Reason: "File paths from v{X.Y} may differ in v{current}"

   **Team Mappings**:
   ```
   /central/** → @stackrox/core-workflows
   /sensor/** → @stackrox/sensor-ecosystem
   /scanner/** → @stackrox/scanner
   /collector/** → @stackrox/collector
   /operator/** → @stackrox/install
   /ui/** → @stackrox/ui
   ```

   ### Strategy 2: Error Signature Match (85-90% confidence)

   - **Source**: CI analysis error_signature_match field
   - **Match**: Pre-matched during `/analyze-ci`
   - **Logic**: Use suggested_team and confidence from error signature

   **Error Patterns**:
   - GraphQL schema validation → @stackrox/core-workflows (90%)
   - panic in {service} → Extract service team (85%)
   - dial tcp, connection refused → @stackrox/collector (80%)
   - image pull, scanner → @stackrox/scanner (85%)

   **Confidence Adjustment**: No adjustment for version mismatch (error patterns stable)

   ### Strategy 3: Service Ownership Match (80% confidence base)

   - **Source**: Component or service name from issue
   - **Match**: Component/service → team mapping
   - **Logic**:
     - Extract component from labels or description
     - Map to owning team

   **Confidence Adjustment**:
   - Base: 80%
   - If version_mismatch = true: 80% → 75%
   - Reason: "Component ownership may have changed"

   **Service Mappings**:
   ```
   central, main, api, graphql → @stackrox/core-workflows
   sensor, compliance, sac → @stackrox/sensor-ecosystem
   scanner, scanner-v4, image-scanning → @stackrox/scanner
   collector, network-flow, ebpf → @stackrox/collector
   operator, helm-charts → @stackrox/install
   ui, frontend, cypress → @stackrox/ui
   ```

   ### Strategy 4: Similar Issue History (70-80% confidence)

   - **Source**: JIRA search for resolved issues
   - **Query**: `project = ROX AND summary ~ "{keywords}" AND resolution = Done`
   - **Logic**:
     - Extract keywords from current issue summary
     - Search for similar resolved issues
     - Check team assignments from historical issues
     - Higher confidence if multiple similar issues assign to same team

   **Confidence Calculation**:
   - 1 similar issue → 70%
   - 2-3 similar issues → 75%
   - 4+ similar issues → 80%
   - Recent (<3 months) → +5%

   **Confidence Adjustment**: No adjustment for version mismatch

   ### Strategy 5: Test Category Match (70% confidence)

   - **Source**: Test name or test file path
   - **Match**: Test category patterns
   - **Logic**:
     - E2E tests → @stackrox/ui (70%)
     - Integration tests → Component owner (70%)
     - Unit tests → File path CODEOWNERS (uses Strategy 1)

   **Confidence Adjustment**: Same as Strategy 1 for version mismatch

3. **Handle No Match**

   If no strategy produces a match:
   - assigned_team = "Needs Manual Assignment"
   - confidence = 0
   - reasoning = "No CODEOWNERS, error signature, or service match found"
   - Include evidence summary for human reviewer

4. **Consolidate with Specialized Analysis**

   If issue already has team assignment from specialized analysis:
   - `/analyze-flaky` known patterns (95% confidence)
   - `/analyze-vuln` container mapping (85% confidence)
   - Keep existing assignment if confidence >= new assignment
   - Otherwise, use highest confidence assignment

5. **Enrich Issue Object**
   Add final team assignment:
   ```json
   {
     "team_assignment": {
       "assigned_team": "@stackrox/scanner",
       "confidence": 90,
       "strategy": "error_signature",
       "reasoning": "GraphQL schema validation error pattern matches core-workflows team with 90% confidence",
       "evidence": {
         "file_paths": ["central/graphql/resolvers/policies.go"],
         "error_pattern": "GraphQL schema validation",
         "service": "central"
       },
       "version_mismatch_adjustment": false,
       "alternative_teams": [
         {
           "team": "@stackrox/ui",
           "confidence": 70,
           "strategy": "test_category",
           "reason": "E2E test typically owned by UI team"
         }
       ]
     }
   }
   ```

## Output

- **artifacts/acs-triage/issues.json** - Updated with team_assignment field for all issues

## Usage Examples

Basic usage:
```
/assign-team
```

## Success Criteria

After running this command, you should have:
- [ ] All issues have team_assignment field
- [ ] Confidence scores calculated (0-95%)
- [ ] Strategy used documented
- [ ] Evidence captured for human review
- [ ] Version mismatch adjustments applied where appropriate

## Next Steps

After team assignment:
1. Run `/generate-report` to create triage reports
2. Review low-confidence assignments (<70%) manually

## Notes

- Always use highest confidence strategy that matches
- Version mismatch reduces CODEOWNERS and service ownership confidence
- Error signatures are stable across versions (no confidence reduction)
- Alternative teams provide options for human reviewers
- Evidence field helps humans understand assignment reasoning
- "Needs Manual Assignment" is acceptable for complex/ambiguous issues
- Confidence threshold for auto-assignment: ≥80% recommended
- Multiple strategies may match - use highest confidence

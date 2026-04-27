# /analyze-vuln - Analyze Vulnerability

## Purpose

Apply the ProdSec decision tree for vulnerability triage. This 6-step workflow determines whether vulnerabilities should be fixed, closed, or need manual review, and assigns the appropriate team.

## Prerequisites

- artifacts/acs-triage/issues.json exists with type="VULNERABILITY" issues
- `/setup` completed to access rhacs-patch-eval tools
- `/classify` completed

## Process

1. **Filter Vulnerability Issues**
   - Read artifacts/acs-triage/issues.json
   - Process only issues where type = "VULNERABILITY"

2. **Extract CVE Information**

   From summary and description, extract:
   - **CVE ID**: CVE-YYYY-NNNNN
   - **Severity**: Critical | Important | Moderate | Low
   - **CVSS Score**: Numeric (0.0-10.0)
   - **Container**: central, scanner, sensor, collector, etc.
   - **Language**: Go, npm, Python, Java
   - **Component**: Specific package/library affected
   - **Affected Versions**: From JIRA affectedVersions field

3. **Apply ProdSec Decision Tree**

   Execute 6 decision steps in order:

   ### Step 1: Version Support Check
   - Check if all affectedVersions are end-of-life/unsupported
   - Query: Are all affected versions < oldest supported release?
   - **If YES → Recommend: CLOSE (Won't Do) - Unsupported versions**
   - **If NO → Continue to Step 2**

   ### Step 2: Severity Threshold Check
   - Check severity level and CVSS score
   - Rules:
     - If severity = "Low" → CLOSE (Won't Do) - Below threshold
     - If severity = "Moderate" AND CVSS < 7.0 → CLOSE (Won't Do) - Below threshold
   - **If threshold not met → Recommend: CLOSE (Won't Do)**
   - **If threshold met → Continue to Step 3**

   ### Step 3: Container Applicability Check
   - Check for language/container mismatches (false positives)
   - Rules:
     - If container = "central-db" OR "scanner-db" OR "scanner-v4-db"
     - AND language = "npm" OR "Go"
     - → CLOSE (Obsolete) - Databases don't have npm/Go code
   - **If mismatch → Recommend: CLOSE (Obsolete)**
   - **If applicable → Continue to Step 4**

   ### Step 4: Duplicate Detection
   - Search JIRA for existing CVE issues
   - Query: `project = ROX AND summary ~ "CVE-YYYY-NNNNN"`
   - **If found → Recommend: DUPLICATE (link to existing issue)**
   - **If not found → Continue to Step 5**

   ### Step 5: Impact Analysis
   - Check VEX (Vulnerability Exploitability eXchange) status
   - Use `/tmp/triage/skills/plugins/rhacs-patch-eval/` tools if available
   - Rules:
     - If VEX status = "not_affected" OR "false_positive" → CLOSE (VEX)
     - If exploitability = "poc_available" OR "weaponized" → High priority
   - **If false positive → Recommend: CLOSE (VEX)**
   - **If real vulnerability → Continue to Step 6**

   ### Step 6: Team Assignment by Container
   - Assign team based on affected container:
     - **scanner**, **scanner-v4**, **scanner-db** → @stackrox/scanner
     - **central**, **main**, **central-db** → @stackrox/core-workflows
     - **sensor** → @stackrox/sensor-ecosystem
     - **collector** → @stackrox/collector
     - **operator** → @stackrox/install
     - **ui** → @stackrox/ui
   - **Recommend: ASSIGN to team**

4. **Enrich Issue Object**
   Add vulnerability-specific fields:
   ```json
   {
     "vuln_analysis": {
       "cve_id": "CVE-2024-1234",
       "severity": "Important",
       "cvss_score": 7.5,
       "container": "scanner",
       "language": "Go",
       "component": "github.com/example/pkg",
       "decision_tree": {
         "step1_version_support": "pass",
         "step2_severity_threshold": "pass",
         "step3_container_applicability": "pass",
         "step4_duplicate_check": "pass",
         "step5_impact_analysis": "pass",
         "step6_team_assignment": "@stackrox/scanner"
       },
       "recommendation": "ASSIGN",
       "assigned_team": "@stackrox/scanner",
       "confidence": 85,
       "reasoning": "Critical severity Go vulnerability in scanner container, no duplicates found"
     }
   }
   ```

## Output

- **artifacts/acs-triage/issues.json** - Updated with vuln_analysis field for VULNERABILITY issues

## Usage Examples

Basic usage:
```
/analyze-vuln
```

## Success Criteria

After running this command, you should have:
- [ ] All VULNERABILITY issues enriched with vuln_analysis data
- [ ] Decision tree steps documented for each issue
- [ ] Recommendations made (ASSIGN, CLOSE, DUPLICATE)
- [ ] Team assigned for ASSIGN recommendations

## Next Steps

After vulnerability analysis:
1. Issues recommended for ASSIGN go to `/assign-team` for confidence scoring
2. Issues recommended for CLOSE/DUPLICATE are flagged in report
3. Manual review recommended for edge cases

## Notes

- Decision tree is sequential - early exits save analysis time
- Most vulnerabilities close at Step 2 (severity threshold)
- Database container mismatches (Step 3) are common false positives
- VEX data may not be available for all CVEs - skip Step 5 if unavailable
- Team assignment (Step 6) is deterministic based on container name
- Version mismatch from `/classify` affects Step 1 (version support check)
- Critical/Important severity bypasses some checks for faster response

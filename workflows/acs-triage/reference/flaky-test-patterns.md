# Known Flaky Test Patterns

Documented flaky tests with historical references, root causes, and team ownership.

**See:** `reference/teams.md` for canonical team list
**See:** `reference/constants.md` for confidence thresholds and frequency definitions

**Source:** `/tmp/triage/stackrox/.claude/agents/stackrox-ci-failure-investigator.md` and historical JIRA issues

## Known Flaky Patterns

### GlobalSearch Latest Tag Violations

**Pattern:** "GlobalSearch Latest Tag" OR "GlobalSearch violations"

**Team:** @stackrox/core-workflows
**Confidence:** 95%
**Reference:** ROX-5355

**Root Cause:** Alert generation timing issue
**Description:** Test expects alerts to be generated for latest tag policy violations, but timing-dependent alert generation causes intermittent failures.

**Symptoms:**
- "Expected 1 violation, got 0"
- "Alert not generated within timeout"
- DNS timing issues in GlobalSearch

**Frequency:** High (>10 occurrences/month)

**Example:**
```
Test: GlobalSearch Latest Tag violations
Error: Expected violation for latest tag, got none
Timeout: 30s
```

**Historical Fix Attempts:**
- Increased timeout (partial success)
- Alert generation optimizations (ongoing)

---

### PolicyFieldsTest Process UID/Name

**Pattern:** "PolicyFieldsTest" AND ("Process UID" OR "Process Name")

**Team:** @stackrox/core-workflows
**Confidence:** 95%
**Reference:** ROX-5298

**Root Cause:** Slow alert generation for process policies

**Description:** Policy field validation tests depend on timely alert generation. Slow alert processing causes test failures.

**Symptoms:**
- "Timeout waiting for alert"
- "Process UID field not populated"
- "Timing-dependent validation failure"

**Frequency:** Medium (3-10 occurrences/month)

**Example:**
```
Test: PolicyFieldsTest Process UID validation
Error: Expected process UID in alert, field empty
Timeout: 20s
```

---

### NetworkFlowTest One-Time Connections

**Pattern:** "NetworkFlowTest" AND ("one-time connections" OR "network flow")

**Team:** @stackrox/collector
**Confidence:** 95%

**Root Cause:** Network flow timing issues

**Description:** Test expects network flow data from short-lived connections. Collector timing and Kubernetes network latency cause intermittent detection failures.

**Symptoms:**
- "deadline exceeded waiting for network flow"
- "Expected flow from pod-a to pod-b, got none"
- "Network flow timeout"

**Frequency:** High (>10 occurrences/month)

**Example:**
```
Test: NetworkFlowTest one-time connections
Error: deadline exceeded after 30s
Expected network flow from pod-a to pod-b but got none
```

**Mitigation:**
- Longer timeouts
- Multiple connection attempts
- Collector buffering improvements

---

### ImageScanningTest Registry Integrations

**Pattern:** "ImageScanningTest" AND ("registry" OR "scanning")

**Team:** @stackrox/scanner
**Confidence:** 95%

**Root Cause:** Scanner connectivity and timeout issues

**Description:** Tests against external registries fail due to network timeouts, registry rate limiting, or scanner service delays.

**Symptoms:**
- "Image pull timeout"
- "Registry connection refused"
- "Scanner timeout"
- "Registry rate limit exceeded"

**Frequency:** Medium (3-10 occurrences/month)

**Example:**
```
Test: ImageScanningTest registry integrations
Error: Failed to scan image from quay.io
Connection timeout after 60s
```

**Mitigation:**
- Local registry caching
- Retry logic with backoff
- Longer scan timeouts

---

### SACTest SSH Port Violations

**Pattern:** "SACTest" AND ("SSH Port" OR "SAC")

**Team:** @stackrox/sensor-ecosystem
**Confidence:** 95%

**Root Cause:** OpenShift waitForViolation timing

**Description:** Scope-based Access Control (SAC) tests on OpenShift have timing issues with violation detection and propagation.

**Symptoms:**
- "Timeout waiting for violation"
- "SSH port policy violation not detected"
- "SAC permission check timing"

**Frequency:** Medium (3-10 occurrences/month)

**Example:**
```
Test: SACTest SSH Port violations
Error: Expected violation for SSH port 22, got none
Timing: waitForViolation timeout
```

**Mitigation:**
- Platform-specific timeout adjustments
- OpenShift-specific wait logic

---

## Frequency Estimation

Estimate frequency by searching JIRA:
```
project = ROX AND
summary ~ "{test_name}" AND
created >= -30d AND
labels = CI_Failure
```

Count results to classify frequency (see `reference/constants.md` for thresholds).

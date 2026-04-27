# Error Signatures

Known error patterns mapped to teams with confidence scores. These signatures are stable across StackRox versions.

**See:** `reference/teams.md` for canonical team list
**See:** `reference/constants.md` for confidence thresholds

**Source:** `/tmp/triage/stackrox/.claude/agents/stackrox-ci-failure-investigator.md` and historical triage data

## GraphQL Errors (90% → @stackrox/core-workflows)

GraphQL schema validation and code generation issues.

**Patterns:**
- "GraphQL schema validation"
- "Cannot query field"
- "__Schema"
- "placeholder Boolean"
- "Invalid object type"
- "schema generation failed"

**Example:**
```
Error: GraphQL schema validation failed
Invalid object type: placeholder field Boolean is not valid
```

**Team:** @stackrox/core-workflows
**Confidence:** 90%
**Reasoning:** GraphQL codegen is maintained by core-workflows team

## Service Crashes (85% → varies by service)

Panics, FATAL errors, and nil pointer dereferences.

**Patterns:**
- "panic:"
- "FATAL"
- "nil pointer dereference"
- "runtime error"
- "goroutine"

**Team Assignment Logic:**
1. Extract service name from stack trace
2. Map service to team using service ownership
3. If no service identified, use file path CODEOWNERS

**Examples:**

```
panic: runtime error: nil pointer dereference
goroutine 45:
central/api/v1/search.go:89
```
→ **@stackrox/core-workflows** (85%)

```
panic: runtime error
sensor/pkg/sac/authorizer.go:156
```
→ **@stackrox/sensor-ecosystem** (85%)

## Network/Timeout Errors (80% → @stackrox/collector)

Network connectivity, timeouts, and DNS issues.

**Patterns:**
- "dial tcp"
- "connection refused"
- "deadline exceeded"
- "context deadline"
- "timeout"
- "DNS resolution failed"
- "network unreachable"
- "i/o timeout"

**Example:**
```
timeout: deadline exceeded after 30s
Expected network flow from pod-a to pod-b but got none
```

**Team:** @stackrox/collector
**Confidence:** 80%
**Reasoning:** Network flow monitoring is collector responsibility

## Image/Scanning Errors (85% → @stackrox/scanner)

Image pull, registry, and vulnerability scanning issues.

**Patterns:**
- "image pull"
- "scanner"
- "vulnerability detection"
- "registry error"
- "manifest unknown"
- "image not found"
- "scanning failed"

**Example:**
```
Error: Failed to scan image
scanner connection refused: dial tcp 10.0.1.5:8080
```

**Team:** @stackrox/scanner
**Confidence:** 85%
**Reasoning:** Scanner service and image scanning logic

## Test Infrastructure Errors (75% → @stackrox/core-workflows)

Test setup, cluster provisioning, and infrastructure.

**Patterns:**
- "cluster provision"
- "namespace creation"
- "test setup failed"
- "kubectl"
- "cluster not ready"
- "test infrastructure"

**Example:**
```
Error: cluster provision failed
timeout waiting for cluster to be ready
```

**Team:** @stackrox/core-workflows
**Confidence:** 75%
**Reasoning:** Core team manages test infrastructure (lower confidence as infrastructure is shared)

## Admission Control Errors (85% → @stackrox/sensor-ecosystem)

Policy enforcement and admission webhooks.

**Patterns:**
- "admission webhook"
- "admission control"
- "policy violation"
- "webhook denied"
- "admission.k8s.io"

**Example:**
```
Error: admission webhook "sensor.stackrox.io" denied the request
policy violation: privileged container not allowed
```

**Team:** @stackrox/sensor-ecosystem
**Confidence:** 85%
**Reasoning:** Admission control is sensor-ecosystem component

## Database/Migration Errors (85% → @stackrox/core-workflows)

PostgreSQL, migrations, and schema issues.

**Patterns:**
- "postgres"
- "database migration"
- "schema"
- "SQL"
- "constraint violation"
- "duplicate key"

**Example:**
```
ERROR: migration failed
postgres error: duplicate key value violates unique constraint
```

**Team:** @stackrox/core-workflows
**Confidence:** 85%
**Reasoning:** Central owns postgres migrations

## RBAC/SAC Errors (85% → @stackrox/sensor-ecosystem)

Scope-based Access Control and permissions.

**Patterns:**
- "SAC"
- "unauthorized"
- "permission denied"
- "scope"
- "access denied"
- "sac.go"

**Example:**
```
Error: SAC check failed
user lacks permission for namespace default
```

**Team:** @stackrox/sensor-ecosystem
**Confidence:** 85%
**Reasoning:** SAC implementation is sensor-ecosystem component

## Operator/Helm Errors (80% → @stackrox/install)

Deployment, upgrade, and operator issues.

**Patterns:**
- "operator"
- "helm"
- "upgrade failed"
- "deployment"
- "reconcile"
- "install failed"

**Example:**
```
Error: operator reconcile failed
helm upgrade returned error: release not found
```

**Team:** @stackrox/install
**Confidence:** 80%
**Reasoning:** Install team owns operator and helm deployments

## UI/Frontend Errors (80% → @stackrox/ui)

React, TypeScript, and frontend-specific errors.

**Patterns:**
- "React"
- "TypeScript"
- "ui/apps"
- "ui/platform"
- "Cypress"
- "Cannot read property"
- "undefined is not an object"

**Example:**
```
TypeError: Cannot read property 'name' of undefined
at ui/apps/platform/src/Containers/VulnMgmt/VulnMgmtEntityPage.tsx:45
```

**Team:** @stackrox/ui
**Confidence:** 80%
**Reasoning:** Frontend code is UI team responsibility

# Team Mappings

Component and service to team ownership mappings for the service_ownership assignment strategy.

**See:** `reference/teams.md` for canonical team list and responsibilities
**See:** `reference/constants.md` for confidence thresholds

## Component to Team Mapping

Extract component names from JIRA labels, components field, or issue description, then map to owning team:

| Component Pattern | Team |
|------------------|------|
| central, main, api, graphql, postgres, migrations | @stackrox/core-workflows |
| sensor, compliance, admission-control, sac, booleanpolicy | @stackrox/sensor-ecosystem |
| scanner, scanner-v4, scanner-db, image, scanners | @stackrox/scanner |
| collector, networkgraph, networkflow, ebpf | @stackrox/collector |
| operator, deploy, helm, roxctl | @stackrox/install |
| ui, frontend, cypress, e2e-tests | @stackrox/ui |

## Container to Team Mapping

For vulnerability issues, extract container name from CVE description:

| Container | Team |
|-----------|------|
| central, main, central-db | @stackrox/core-workflows |
| sensor, admission-control | @stackrox/sensor-ecosystem |
| scanner, scanner-v4, scanner-db | @stackrox/scanner |
| collector | @stackrox/collector |
| operator, roxctl | @stackrox/install |

## Service to Team Mapping

For CI failures, extract service name from error logs or stack traces:

| Service Name | Team |
|-------------|------|
| central | @stackrox/core-workflows |
| sensor, admission-controller | @stackrox/sensor-ecosystem |
| scanner, scanner-v4 | @stackrox/scanner |
| collector | @stackrox/collector |

## Test Category Defaults

| Test Type | Default Team | Notes |
|-----------|-------------|-------|
| E2E (Cypress) | @stackrox/ui | UI team owns E2E framework |
| Upgrade | @stackrox/install | Install team owns upgrade logic |
| QA Backend | @janisz | Test infrastructure owner |
| Integration/Unit | Use CODEOWNERS | Match test file path |

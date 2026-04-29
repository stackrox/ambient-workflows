# StackRox/ACS Teams

Canonical team list with responsibilities and ownership patterns.

## Teams

### @stackrox/core-workflows

**Focus:** Central service, core platform, GraphQL API, general infrastructure

**Components:** central, main, api, graphql, postgres, migrations, grpc, protocompat

**Services:** central (main API server), central-db (PostgreSQL database)

**File Paths:** /central/**, /pkg/graphql/**, /proto/api/**, /migrator/**

**Responsibilities:**
- Core API endpoints
- GraphQL schema and resolvers
- Database migrations
- gRPC services
- Platform infrastructure

### @stackrox/sensor-ecosystem

**Focus:** Sensor, compliance, admission control, Scope-based Access Control

**Components:** sensor, compliance, admission-control, sac, booleanpolicy

**Services:** sensor (Kubernetes data collector), admission-control (policy enforcement webhook)

**File Paths:** /sensor/**, /compliance/**, /pkg/sac/**, /pkg/booleanpolicy/**

**Responsibilities:**
- Kubernetes resource monitoring
- Compliance scanning
- Policy enforcement
- RBAC/SAC implementation
- Runtime detection

### @stackrox/scanner

**Focus:** Image scanning and vulnerability detection

**Components:** scanner, scanner-v4, scanner-db, image, scanners (pkg)

**Services:** scanner (legacy scanner), scanner-v4 (ClairCore-based scanner), scanner-db (scanner database)

**File Paths:** /scanner/**, /image/**, /pkg/scanners/**

**Responsibilities:**
- Container image scanning
- Vulnerability detection
- CVE database integration
- Image metadata extraction

### @stackrox/collector

**Focus:** Network monitoring, eBPF, network flow data

**Components:** collector, networkgraph, networkflow, ebpf

**Services:** collector (eBPF network collector)

**File Paths:** /collector/**, /pkg/networkgraph/**, /pkg/networkflow/**

**Responsibilities:**
- Network flow monitoring
- Process monitoring
- Network graph generation
- eBPF probe management

### @stackrox/install

**Focus:** Operator, Helm charts, installation tooling

**Components:** operator, deploy, helm, roxctl (CLI), image/templates

**Services:** operator (Kubernetes operator)

**File Paths:** /operator/**, /deploy/**, /pkg/helm/**, /roxctl/**

**Responsibilities:**
- Installation and upgrades
- Helm chart maintenance
- Operator development
- CLI tooling

### @stackrox/ui

**Focus:** Frontend, React application, end-to-end tests

**Components:** ui, frontend, cypress, e2e-tests, upgrade-tests

**Services:** ui (React frontend application)

**File Paths:** /ui/**, /tests/e2e/**

**Responsibilities:**
- React UI development
- GraphQL query integration
- End-to-end testing
- Frontend build pipeline

## Container to Team Mapping

| Container | Team |
|-----------|------|
| central, main, central-db | @stackrox/core-workflows |
| sensor, admission-control | @stackrox/sensor-ecosystem |
| scanner, scanner-v4, scanner-db | @stackrox/scanner |
| collector | @stackrox/collector |
| operator, roxctl | @stackrox/install |

## Test Category Defaults

| Test Type | Default Team | Notes |
|-----------|-------------|-------|
| E2E (Cypress) | @stackrox/ui | UI team owns E2E framework |
| Upgrade | @stackrox/install | Install team owns upgrade logic |
| QA Backend | @janisz | Test infrastructure owner |
| Integration/Unit | Component owner | Use CODEOWNERS for test file |

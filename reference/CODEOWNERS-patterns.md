# CODEOWNERS Patterns

File path patterns to team ownership mappings extracted from `.github/CODEOWNERS`.

**See:** `reference/teams.md` for canonical team list
**See:** `reference/constants.md` for confidence thresholds

**Source:** `/tmp/triage/stackrox/.github/CODEOWNERS` (loaded by `/setup`)

## File Pattern to Team Mapping

| Pattern | Team |
|---------|------|
| /central/**, /pkg/graphql/**, /proto/api/**, /pkg/protocompat/**, /pkg/grpc/**, /pkg/migrations/**, /pkg/postgres/** | @stackrox/core-workflows |
| /sensor/**, /compliance/**, /pkg/sac/**, /admission-control/**, /pkg/booleanpolicy/** | @stackrox/sensor-ecosystem |
| /scanner/**, /image/**, /pkg/scanners/**, /scanner-v4/**, /scanner-db/** | @stackrox/scanner |
| /collector/**, /pkg/networkgraph/**, /pkg/networkflow/** | @stackrox/collector |
| /operator/**, /deploy/**, /image/templates/**, /pkg/helm/**, /roxctl/** | @stackrox/install |
| /ui/**, /tests/e2e/**, /tests/upgrade/** | @stackrox/ui |
| /qa-tests-backend/** | @janisz |

## Special Cases

### Shared Ownership

| Pattern | Teams | Priority Rule |
|---------|-------|--------------|
| /central/graphql/generator/** | @stackrox/ui | UI owns GraphQL schema generation |
| /central/graphql/resolvers/** | @stackrox/ui, @stackrox/core-workflows | Use error signature or service ownership to disambiguate |

### Matching Rules

1. Use longest matching pattern (most specific)
2. For shared ownership, check error signature or service ownership
3. Document alternative teams in `team_assignment.alternative_teams`

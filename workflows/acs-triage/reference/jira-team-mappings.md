# JIRA Team Mappings

Mapping between GitHub team mentions and JIRA team IDs for proper team mentions in JIRA comments.

## Team ID Mapping

| GitHub Team | JIRA Team Name | JIRA Team ID |
|-------------|----------------|--------------|
| @stackrox/ui | ACS UI | ec74d716-af36-4b3c-950f-f79213d08f71-569 |
| @stackrox/scanner | ACS Scanner | ec74d716-af36-4b3c-950f-f79213d08f71-743 |
| @stackrox/collector | ACS Collector | ec74d716-af36-4b3c-950f-f79213d08f71-1557 |
| @stackrox/core-workflows | ACS Core Workflows | ec74d716-af36-4b3c-950f-f79213d08f71-1813 |
| @stackrox/sensor-ecosystem | ACS Sensor & Ecosystem | ec74d716-af36-4b3c-950f-f79213d08f71-2627 |
| @stackrox/install | ACS Install | ec74d716-af36-4b3c-950f-f79213d08f71-2479 |

## Additional Teams

| JIRA Team Name | JIRA Team ID | Notes |
|----------------|--------------|-------|
| ACS Automation | ec74d716-af36-4b3c-950f-f79213d08f71-1369 | Test automation team |
| ACS Cloud Service | ec74d716-af36-4b3c-950f-f79213d08f71-3169 | Cloud-hosted service team |
| ACS Engineering | ec74d716-af36-4b3c-950f-f79213d08f71-4512 | Broad engineering team |

## JIRA Mention Format

To mention a team in JIRA comments, use this markdown format:

```markdown
[Team Display Name](https://redhat.atlassian.net/jira/people/team/{team-id}?ref=jira$&src=issue)
```

**Example:**

```markdown
[ACS Core Workflows](https://redhat.atlassian.net/jira/people/team/ec74d716-af36-4b3c-950f-f79213d08f71-1813?ref=jira$&src=issue)
```

This creates a clickable team mention in JIRA that notifies team members.

## Usage in Triage Workflow

When posting triage comments, the workflow should:

1. Identify team using GitHub handle (from CODEOWNERS, error signatures, etc.)
2. Look up JIRA team name and ID from this mapping
3. Format team mention using JIRA format in comment body
4. Fall back to plain text if team not in mapping

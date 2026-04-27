# /fetch-issues - Retrieve Untriaged JIRA Issues

## Purpose

Query JIRA filters for untriaged StackRox/ACS issues and extract essential information including version data for triage analysis. This command retrieves the top 10-20 most recent issues (within 300s timeout constraint) from designated triage filters.

## Prerequisites

- JIRA MCP connection configured
- Access to filters 103399 (current untriaged) and 95004 (previous duty)
- `/setup` command completed (optional but recommended)

## Process

1. **Query Current Untriaged Filter**
   - Use `mcp__mcp-atlassian__jira_search` with JQL: `filter = 103399 ORDER BY priority DESC, created ASC`
   - Limit to 10-20 issues to stay within timeout
   - Start with highest priority and oldest created

2. **Get Full Issue Details**
   - For each issue from search, call `mcp__mcp-atlassian__jira_get_issue` with issue key
   - Extract fields:
     - key (e.g., ROX-12345)
     - summary
     - description
     - labels
     - components
     - priority
     - status
     - created
     - updated
     - **affectedVersions** - Which versions have the bug (e.g., ["4.5.0", "4.5.1"])
     - **fixVersions** - Target fix release (e.g., ["4.6.0"])
     - comments (for CI failure logs and error details)

3. **Fallback to Previous Duty Filter**
   - If filter 103399 has <10 issues, query filter 95004
   - Use same approach: `filter = 95004 ORDER BY priority DESC, created ASC`

4. **Create Issue Objects**
   - Structure each issue with all extracted fields
   - Include raw description and comments for later analysis
   - Preserve affectedVersions and fixVersions for version mismatch detection

## Output

- **artifacts/acs-triage/issues.json** - JSON array of issue objects with all extracted data

Format:
```json
[
  {
    "key": "ROX-12345",
    "summary": "UI E2E test failing: GlobalSearch Latest Tag",
    "description": "Build ID: 1963388448995807232...",
    "labels": ["CI_Failure", "ui"],
    "components": ["UI", "Frontend"],
    "priority": "High",
    "status": "Open",
    "created": "2024-04-15T10:30:00Z",
    "updated": "2024-04-15T12:00:00Z",
    "affectedVersions": ["4.5.0", "4.5.1"],
    "fixVersions": ["4.6.0"],
    "comments": [
      {
        "author": "jenkins",
        "created": "2024-04-15T10:32:00Z",
        "body": "Error log: GraphQL schema validation failed..."
      }
    ]
  }
]
```

## Usage Examples

Basic usage:
```
/fetch-issues
```

## Success Criteria

After running this command, you should have:
- [ ] Retrieved 10-20 untriaged issues from JIRA
- [ ] Created artifacts/acs-triage/issues.json with all issue data
- [ ] Extracted affectedVersions and fixVersions for each issue
- [ ] Noted any timeout or API issues

## Next Steps

After fetching issues:
1. Run `/classify` to categorize each issue by type and detect version mismatches
2. Review the issues.json file to verify data quality

## Notes

- Timeout limit is 300 seconds total - adjust issue count if needed
- If JIRA MCP is slow, reduce to 10 issues
- Comments are critical for CI failures (contain error logs)
- Priority order ensures most important issues are triaged first
- affectedVersions and fixVersions are used for version mismatch detection in `/classify`
- Some issues may not have version fields - this is normal for feature requests or unversioned bugs

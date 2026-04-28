# /setup - Initialize StackRox Reference Data

## Purpose

Clone the StackRox repository and skills repository to access CODEOWNERS, error patterns, and vulnerability assessment tools. This provides the reference data needed for accurate team assignment.

## Prerequisites

- Git access to github.com/stackrox/stackrox and github.com/stackrox/skills
- Write access to /tmp directory

## Process

1. **Create Working Directory**
   ```bash
   mkdir -p /tmp/triage
   cd /tmp/triage
   ```

2. **Clone StackRox Repository**
   ```bash
   git clone --depth=1 https://github.com/stackrox/stackrox.git
   ```
   - Uses shallow clone (--depth=1) for speed
   - Clones latest main branch

3. **Clone Skills Repository**
   ```bash
   git clone --depth=1 https://github.com/stackrox/skills.git
   ```
   - Needed for rhacs-patch-eval vulnerability assessment

4. **Extract Version Information**
   ```bash
   cd /tmp/triage/stackrox
   git describe --tags --always
   ```
   - Note the current version for version mismatch detection
   - Store in artifacts/acs-triage/setup-info.json

5. **Verify Key Files Exist**
   Check for:
   - `/tmp/triage/stackrox/.github/CODEOWNERS`
   - `/tmp/triage/stackrox/.claude/agents/stackrox-ci-failure-investigator.md`
   - `/tmp/triage/skills/plugins/rhacs-patch-eval/`

## Output

- **artifacts/acs-triage/setup-info.json** - Setup metadata

Format:
```json
{
  "setup_timestamp": "2024-04-27T10:30:00Z",
  "stackrox_version": "4.7.0-dev",
  "stackrox_commit": "abc123def",
  "stackrox_path": "/tmp/triage/stackrox",
  "skills_path": "/tmp/triage/skills"
}
```

## ⚠️ Version Mismatch Warning

**IMPORTANT:** This setup clones the latest `main` branch, which represents the current development version.

**Implications:**
- CODEOWNERS file paths reflect LATEST code structure
- Issues with older `affectedVersions` (e.g., 4.4.x, 4.5.x) may reference:
  - Moved or renamed files
  - Different component ownership
  - Changed team structures

**Impact on Team Assignment:**
- CODEOWNERS match confidence: 95% → 75% for version-mismatched issues
- Service ownership confidence: 80% → 75% for version-mismatched issues
- Reports will flag version mismatches with ⚠️ symbol

**Mitigation:**
- The `/classify` command detects version mismatches using `affectedVersions` field
- Reports recommend human review for flagged issues
- Future enhancement: Clone specific git tags based on issue versions

## Usage Examples

Basic usage:
```
/setup
```

## Success Criteria

After running this command, you should have:
- [ ] Cloned repositories in /tmp/triage/
- [ ] Created artifacts/acs-triage/setup-info.json
- [ ] Verified CODEOWNERS and agent files exist
- [ ] Noted current version for mismatch detection

## Next Steps

After setup:
1. Run `/fetch-issues` to retrieve JIRA issues
2. The setup metadata will be used throughout the workflow

## Notes

- Setup is idempotent - can be re-run to get latest code
- Shallow clone (--depth=1) reduces download time
- If repositories already exist, consider `git pull` instead of re-cloning
- Version mismatch detection happens in `/classify` command

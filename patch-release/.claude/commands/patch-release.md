# /patch-release

Perform an ACS patch release. The version is provided as the argument (e.g. `/patch-release 4.8.11`).

## Setup

Locate or clone the patch-release skill from the internal `stackrox/skills` repo:

```bash
SKILLS_DIR=$(find /tmp ~/dev/stack -maxdepth 2 -name "skills" -type d -exec test -d {}/plugins/patch-release \; -print -quit 2>/dev/null)
if [ -z "$SKILLS_DIR" ]; then
  git clone --depth=1 https://github.com/stackrox/skills.git /tmp/skills
  SKILLS_DIR=/tmp/skills
fi
PATCH_RELEASE_DIR="${SKILLS_DIR}/plugins/patch-release"
```

## Run

Read `${PATCH_RELEASE_DIR}/skills/patch-release/SKILL.md` and follow its process
with the provided version argument.

Reference files are at `${PATCH_RELEASE_DIR}/reference/` and scripts at
`${PATCH_RELEASE_DIR}/scripts/`.

# Release Procedure

This runbook describes the steps to release a new version of `plugin-ecc`.

---

## Overview

Releases are version-tagged GitHub releases. The plugin follows semver (`MAJOR.MINOR.PATCH`). The CI workflow validates the plugin structure on every push.

---

## Pre-Release Steps

### 1. Review Changed Files

```bash
git fetch origin main
git log origin/main..HEAD --oneline
```

Identify what changed: new rules, new skills, breaking changes, or just a version bump.

### 2. Run Full Validation

```bash
# Markdown lint
npx markdownlint '**/*.md' --ignore node_modules

# YAML structure
python3 -c "
import yaml, os
with open('plugin.yaml') as f:
    data = yaml.safe_load(f)
for key in ['rules', 'skills']:
    for ref in data.get(key, []):
        path = ref if ref.endswith('.md') else ref + '.md'
        if not os.path.exists(path):
            print(f'MISSING: {path}')
print('All refs OK')
"
```

### 3. Bump Version

Edit `plugin.yaml`:
```yaml
version: X.Y.Z   # ← update this
```

If agents list changed, update the version in `AGENTS.md`:
```markdown
**Version:** X.Y.Z
```

Also update `CLAUDE.md`:
```markdown
**Version:** X.Y.Z
```

### 4. Commit Version Bump

```bash
git add plugin.yaml AGENTS.md CLAUDE.md
git commit -m "chore: bump version to X.Y.Z"
```

---

## Release Tagging

### Lightweight Tag (patch/minor)

```bash
git tag vX.Y.Z
git push origin main
git push origin --tags
```

### Annotated Tag (recommended for releases)

```bash
git tag -a vX.Y.Z -m "Release vX.Y.Z — <summary of changes>"
git push origin main
git push origin --tags
```

---

## Create GitHub Release

```bash
gh release create vX.Y.Z \
  --title "plugin-ecc vX.Y.Z" \
  --notes "$(cat <<'EOF'
## Changelog

- <describe change 1>
- <describe change 2>

### Migration Notes
<any breaking changes or config changes>
EOF
)"
```

Or via the GitHub web UI:
1. Go to https://github.com/Molecule-AI/molecule-ai-plugin-ecc/releases/new
2. Select the tag `vX.Y.Z`
3. Paste the changelog into the release body
4. Publish release

---

## Post-Release Verification

1. **CI green**: Confirm the `validate-plugin.yml` workflow passed on the release commit.
2. **Tag visible**: `git fetch --tags && git tag -l` shows the new tag.
3. **Release page**: The GitHub Release is published with changelog.
4. **Plugin registry**: If molecule-core pulls from the registry, confirm the new version is indexed.

---

## Rollback

If a release is bad:

```bash
# Remove the remote tag
git push origin :refs/tags/vX.Y.Z

# Revert the version commit
git revert HEAD --no-edit
git push origin main
```

Then cut a new release with the fix.

---

## Release Types

| Type | When to use |
|------|-------------|
| `patch` | Bug fixes, doc corrections, non-breaking additions |
| `minor` | New rules/skills, backward-compatible changes |
| `major` | Breaking changes to `plugin.yaml` schema, removed rules/skills |

---

## Automation Opportunities

- **Auto-bump on PR merge**: A GitHub Action can detect `feat:` commits and auto-PR a version bump.
- **Auto-changelog**: `generate-release-notes` action can draft changelog from conventional commits.
- Currently these are manual; implement if release frequency increases.

# Known Failure Modes

This document tracks known issues in `plugin-ecc`, their symptoms, and workarounds.

---

## Markdown Lint Failures in CI

**Symptom:** CI `validate-plugin.yml` workflow fails on the `markdownlint` step.

**Cause:** A new or edited `.md` file violates markdownlint rules (e.g., line length > 80, missing blank lines around headings, or inline HTML).

**Fix:**
```bash
npx markdownlint '**/*.md' --ignore node_modules
# Fix reported issues manually or auto-fix:
npx markdownlint '**/*.md' --ignore node_modules --fix
git add -A && git commit --amend --no-edit
git push --force-with-lease
```

**Prevention:** Run the lint command before every commit.

---

## Missing Referenced File in plugin.yaml

**Symptom:** CI fails with "MISSING: rules/xxx.md" or "MISSING: skills/xxx.md".

**Cause:** `plugin.yaml` references a file that doesn't exist on disk (file deleted, renamed, or never created).

**Fix:**
1. Check the `rules:` and `skills:` lists in `plugin.yaml`
2. For each entry, verify the corresponding `.md` file exists
3. Either add the missing file or remove the reference from `plugin.yaml`

```bash
python3 -c "
import yaml, os
with open('plugin.yaml') as f:
    data = yaml.safe_load(f)
for key in ['rules', 'skills']:
    for ref in data.get(key, []):
        path = ref if ref.endswith('.md') else ref + '.md'
        exists = os.path.exists(path)
        print(f'[{\"OK\" if exists else \"MISSING\"}] {path}')
"
```

---

## Adapter Import Failure

**Symptom:** Python import error at runtime:
```
ModuleNotFoundError: No module named 'plugins_registry'
```

**Cause:** `plugins_registry` is not installed in the runtime environment. It is a harness-level dependency, not bundled in the plugin.

**Workaround:** This is expected in isolated dev environments. The adapter works when the plugin is installed in the Molecule AI harness. If testing locally, see `runbooks/local-dev-setup.md`.

---

## Stale __pycache__ / .pyc Files

**Symptom:** "module has no attribute X" errors after a rename.

**Cause:** Python bytecode cache (`__pycache__/`) holds stale `.pyc` files from a renamed or deleted module.

**Fix:**
```bash
find . -type d -name '__pycache__' -exec rm -rf {} + 2>/dev/null
find . -name '*.pyc' -delete
# Re-run the failing test/import
```

**Prevention:** `.gitignore` excludes these; they won't be committed. Always work in a clean venv.

---

## Duplicate Rule/Skill Names

**Symptom:** CI passes but agents load duplicate rules, or behavior is inconsistent.

**Cause:** The same rule or skill appears multiple times in `plugin.yaml`'s list. The harness may deduplicate, but behavior can be unpredictable.

**Fix:**
```bash
python3 -c "
import yaml
with open('plugin.yaml') as f:
    data = yaml.safe_load(f)
for key in ['rules', 'skills']:
    items = data.get(key, [])
    seen = set()
    for item in items:
        if item in seen:
            print(f'DUPLICATE in {key}: {item}')
        seen.add(item)
"
```

---

## Large Context from AGENTS.md

**Symptom:** Agent runs out of context quickly, or first response is delayed.

**Cause:** `AGENTS.md` is 167 lines. If included in the system prompt on every message, it consumes significant context budget.

**Mitigation:** `AGENTS.md` is referenced via `prompt_fragments` in `plugin.yaml`, so it is loaded selectively. No action needed unless the file grows significantly beyond its current size.

---

## Git Push Blocked by Pre-Receive Hook

**Symptom:** `git push` fails with:
```
remote: error: pre-receive hook declined
```

**Cause:** Either (a) branch protection requires PR review, or (b) CI failed on the latest commit.

**Fix:**
1. Check CI status at: `https://github.com/Molecule-AI/molecule-ai-plugin-ecc/actions`
2. If CI failed, fix the failures and push again
3. If CI passed but push still blocked, use a PR instead of pushing directly to `main`

---

## Template: Adding a New Issue

When a new issue is discovered, add it here with:

```markdown
## <Issue Title>

**Symptom:**
**Cause:**
**Fix/Workaround:**
**Prevention:**
```

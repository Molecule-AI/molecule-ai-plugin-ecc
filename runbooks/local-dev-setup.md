# Local Development Setup

This runbook covers setting up a local development environment for `plugin-ecc`.

---

## Prerequisites

- Node.js >= 18 (for markdownlint)
- Python 3.11+ (for the `adapters/` layer)
- `gh` CLI authenticated: `gh auth status`
- Write access to `Molecule-AI/molecule-ai-plugin-ecc`

---

## Clone & Bootstrap

```bash
# Clone the repo
git clone https://github.com/Molecule-AI/molecule-ai-plugin-ecc.git
cd molecule-ai-plugin-ecc

# Install markdownlint CLI (used for pre-commit linting)
npm install -g markdownlint-cli

# Verify markdownlint works
npx markdownlint '**/*.md' --ignore node_modules
```

---

## Adapter Dev Environment

The `adapters/` layer requires the `plugins_registry` package:

```bash
# Create a virtualenv
python3 -m venv .venv
source .venv/bin/activate

# Install the adapter runtime dependencies
pip install plugins_registry  # published from molecule-core
```

To test the adapter:

```bash
python -c "from adapters.claude_code import Adaptor; print('Adaptor OK')"
```

---

## Validating Locally

### Markdown Lint

```bash
npx markdownlint '**/*.md' --ignore node_modules
```

### YAML Validation

```bash
# Ensure plugin.yaml is valid YAML and all referenced files exist
python3 -c "
import yaml, sys
with open('plugin.yaml') as f:
    data = yaml.safe_load(f)
refs = data.get('rules', []) + data.get('skills', [])
for ref in refs:
    path = ref if ref.endswith('.md') else ref + '.md'
    if not __import__('os').path.exists(path):
        print(f'MISSING: {path}')
print('plugin.yaml OK')
"
```

---

## IDE Setup

### VS Code

Recommended extensions (see `.vscode/extensions.json` if present):
- `esbenp.prettier-vscode` — Markdown formatting
- `DavidAnson.vscode-markdownlint` — Inline linting

Settings for this project (`.vscode/settings.json`):
```json
{
  "editor.formatOnSave": true,
  "[markdown]": {
    "editor.defaultFormatter": "esbenp.prettier-vscode"
  },
  "markdownlint.config": {
    "MD013": false,
    "MD033": false
  }
}
```

---

## Pre-Commit Checklist

Before pushing changes:

```bash
# 1. Lint all markdown
npx markdownlint '**/*.md' --ignore node_modules || exit 1

# 2. Validate plugin.yaml
python3 -c "import yaml; yaml.safe_load(open('plugin.yaml'))" || exit 1

# 3. Check no pycache
find . -name '__pycache__' -o -name '*.pyc' | grep -v '.gitignore' && echo "FAIL: pycache found" && exit 1

# 4. Run tests if any exist
node tests/run-all.js 2>/dev/null || true
```

---

## Working with the Skills/Agents/Commands Surface

The plugin exposes workflow definitions from the ECC ecosystem. To inspect which skills are available:

```bash
ls skills/
cat skills/*.md | grep -A2 'name:'
```

Adding a new skill:
1. Create `skills/<new-skill>.md` with proper YAML frontmatter
2. Add `  - <new-skill>` to the `skills:` list in `plugin.yaml`
3. Run the YAML validation above to confirm the reference is valid

---

## Troubleshooting

### markdownlint command not found

```bash
npm install -g markdownlint-cli
# If still not found, check your PATH or use npx
npx markdownlint-cli '**/*.md'
```

### Adapter import fails

Verify `plugins_registry` is installed in the active venv:
```bash
pip show plugins_registry
# If not installed:
pip install git+https://github.com/Molecule-AI/plugins_registry.git
```

### CI failure on markdownlint

GitHub Actions runs the same `markdownlint` check. Failures usually mean a new `.md` file or edit broke lint rules. Run locally first to catch.

# plugin-ecc — Everything Claude Code

ECC is a **plugin aggregator** that bundles the Everything Claude Code (ECC) ecosystem — 38 agents, 156 skills, 72 commands, and hook workflows — for use on the Molecule AI platform.

This repository itself is minimal: it wires the ecosystem together via `plugin.yaml` and provides plugin-specific conventions. Most content lives in `rules/`, `skills/`, and `AGENTS.md`.

**Version:** 1.9.0
**Runtime:** `claude_code`, `deepagents`, `hermes`

---

## Repository Layout

```
ecc/
├── AGENTS.md          — Agent manifest (38 agents, their purposes, when to use)
├── plugin.yaml        — Plugin manifest (runtimes, rules, skills, prompt_fragments)
├── rules/
│   ├── everything-claude-code-guardrails.md  — Generated guardrails from ECC repo
│   └── node.md                             — Node.js development rules
├── skills/            — Skill definitions (installed as workflow surface)
└── adapters/          — Harness adaptor (wraps ECC via plugins_registry)
```

---

## Conventions

### File Naming

| Directory | Convention | Example |
|-----------|-----------|---------|
| `rules/` | lowercase with hyphens | `codebase-conventions.md` |
| `skills/` | lowercase with hyphens | `tdd-workflow.md` |
| `agents/` | lowercase with hyphens | `code-reviewer.md` |
| `commands/` | lowercase with hyphens | `security-review.md` |
| `scripts/` | lowercase with hyphens (CommonJS) | `session-start.js` |

### Markdown Files

All `.md` files in `skills/`, `agents/`, `commands/`, and `rules/` must pass markdownlint before commit:

```bash
npx markdownlint '**/*.md' --ignore node_modules
```

### YAML Frontmatter

- **Agents**: must have `name`, `description`, `tools`, `model`
- **Skills**: must have "When to Use", "How It Works", "Examples" sections
- **Commands**: must have `description:` frontmatter line

---

## Development

### Setup

```bash
# No npm install needed for the plugin repo itself
# (adapters/ depends on plugins_registry at runtime)
```

### Validation

```bash
# Validate the plugin structure (CI uses molecule-ci workflow)
# Run locally via:
gh workflow run validate-plugin.yml --ref main
```

### Adding a New Rule

1. Create `rules/<rule-name>.md` with frontmatter (`name`, `description`)
2. Add it to `rules/` list in `plugin.yaml`
3. Add it to the `rules/` section in `AGENTS.md` if it affects agent behavior
4. Run markdownlint validation

### Adding a New Skill

1. Create `skills/<skill-name>.md` with frontmatter (`name`, `description`)
2. Add it to `skills/` list in `plugin.yaml`
3. Skills are the canonical workflow surface — prefer adding here over `commands/`

---

## Release Process

See `runbooks/release-procedure.md` for full details.

Brief overview:
1. Ensure all markdown files pass linting
2. Bump version in `plugin.yaml` (semver)
3. Update `AGENTS.md` version number if agents list changed
4. Commit with `chore: bump version to X.Y.Z`
5. Tag and push: `git tag vX.Y.Z && git push origin main --tags`
6. Create GitHub release with `## Changelog` section

---

## Known Issues

See `known-issues.md` for current issues and workarounds.

---

## Key Principles

1. **Agent-First** — Delegate to specialized agents for domain tasks
2. **Test-Driven** — 80%+ coverage required for any new scripts
3. **Security-First** — No hardcoded secrets; validate all inputs
4. **Immutability** — Always create new objects, never mutate existing ones
5. **Plan Before Execute** — Use planner agent for complex features

---

## Workflow Surface

- `skills/` is the **canonical** workflow surface
- `commands/` is legacy for slash-command compatibility only
- New workflows should land in `skills/` first

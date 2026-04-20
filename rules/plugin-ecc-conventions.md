# plugin-ecc Conventions

> Plugin-specific rules for `plugin-ecc`. These complement the ECC ecosystem rules and are specific to how this plugin is packaged, released, and maintained.

## Error Handling

- **Parse errors in rules/skills**: Log to stderr with a `[PluginName]` prefix and `exit 0`. Never block tool execution due to a malformed doc.
- **Adapter errors**: The `adapters/` layer is a thin wrapper — any runtime errors from `plugins_registry` should propagate, not be swallowed.
- **Markdown parse failures**: If a markdown file fails to parse (missing frontmatter, broken sections), report in the agent response but do not abort the session.

## Logging Patterns

- **In hooks** (via `run-with-flags.js`): Use `console.error` for diagnostic output — stdout is reserved for the JSON response.
- **Prefix all log lines**: `[ECC] <component>: <message>` — e.g., `[ECC] guardrails: rule loaded`, `[ECC] adapter: init`.
- **No token/secret logging**: Never log API keys, tokens, or user credentials, even at DEBUG level.
- **Structured fields**: Prefer `console.error('[ECC] key=value key2=value2')` over freeform strings for parseable output.

## Config / Environment Variables

| Variable | Purpose | Required |
|----------|---------|----------|
| `ECC_HOOK_PROFILE` | Gate which hooks fire at runtime | No |
| `ECC_DISABLED_HOOKS` | Colon-separated list of hooks to skip | No |
| `PLUGINS_REGISTRY_URL` | Override registry base URL | No |

- Validate required env vars at adapter init time. Fail fast with a clear message.
- No hardcoded defaults for credentials — require them to be set or inherited from the harness.

## Release Process

### Version Bump

1. Update `version` in `plugin.yaml` (semver: `MAJOR.MINOR.PATCH`).
2. Update version number in `AGENTS.md` (`**Version:** X.Y.Z`) if agents list changed.
3. Update `**Version:**` in `CLAUDE.md` if conventions changed.

### Pre-Release Checklist

- [ ] All markdown files pass `npx markdownlint '**/*.md' --ignore node_modules`
- [ ] `plugin.yaml` is valid YAML and references all existing files
- [ ] No `.pyc` or `__pycache__` files committed (gitignore covers this — verify)
- [ ] Changelog section in release notes covers: new rules, removed rules, breaking changes

### Tag & Push

```bash
git tag -a vX.Y.Z -m "chore: release vX.Y.Z"
git push origin main
git push origin --tags
```

### Post-Release

Create a GitHub Release with:
- Tag version and SHA
- `## Changelog` section listing all changes since last release
- Any migration notes if `rules/` or `skills/` were reorganized

## File Integrity

- All `.md` files committed to the plugin must be valid Markdown — test with `npx markdownlint`.
- No binary files in `rules/`, `skills/`, `agents/`, or `commands/`.
- The `.gitignore` must cover: `__pycache__/`, `*.pyc`, `.DS_Store`, editor backups.

## Adapter Layer

- The `adapters/claude_code.py` file wraps the ECC ecosystem via `plugins_registry.builtins.AgentskillsAdaptor`.
- Do not add business logic to the adapter — keep it as a thin import shim.
- If the adapter needs configuration, add it to `plugin.yaml` under an `adapter:` section, not as code constants.

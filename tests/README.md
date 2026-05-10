# Test Coverage Rationale — molecule-ecc

## Why This Plugin Has Limited Unit-Test Coverage

`molecule-ecc` is a **skill+rules plugin** — it provides development guidelines and
development skills (api-design, coding-standards, deep-research, security-review, tdd-workflow)
via prose SKILL.md files and rules/*.md files.

There are no hooks, no Python business logic, and no testable adapters in this plugin.
The "logic" is prose documentation.

## What We Test (and Why)

| What | Why |
|------|-----|
| `plugin.yaml` schema | Verifies all 5 skills and 3 rules are registered |
| Rules files (3) | Each declared rule file exists and is non-empty |
| Skills (5) | Each skill directory + SKILL.md exists with valid YAML frontmatter and `#` heading |
| Adapters (2) | Claude Code + deepagents adapters are wired |
| `known-issues.md` | Severity definitions present |
| `validate-plugin.py` exit 0 | Smoke test — shared CI validator passes |

## What We Cannot Unit-Test Here

- **SKILL.md prose content** — the development guidelines are prose; their quality is
  a documentation review concern, not a unit-test concern.

- **Agent behavior when using skills** — write integration tests in `workspace-template/`.

## Integration Tests

If you want to test that agents actually use the ecc skills correctly, write
integration tests that:
1. Install `molecule-ecc` on a test workspace
2. Ask the agent to use a specific skill (e.g., "use TDD workflow")
3. Verify the agent follows the documented process

# ECC — Claude Code Guardrails and Standards

Plugin for Claude Code (and Claude Code-compatible runtimes). Provides coding guardrails,
development standards, and production API conventions — everything a Claude Code agent
needs to write production-quality code without prompting.

## What it provides

**Skills (invoke via the Skill tool):**
| Skill | What it does |
|-------|-------------|
| `ecc:api-design` | REST API design patterns — resource naming, HTTP methods, status codes, pagination, filtering, error responses |
| `ecc:coding-standards` | Code quality standards — naming, modularity, error handling, testing conventions |
| `ecc:deep-research` | Research methodology for technical investigations — how to gather context, validate assumptions, cite sources |
| `ecc:security-review` | Security-first code review — common vulnerability patterns, injection vectors, secrets handling |
| `ecc:tdd-workflow` | Test-driven development workflow — red-green-refactor cycle, what makes a good test |

**Rules (ambient — always active):**
- `everything-claude-code-guardrails.md` — platform-wide Claude Code guardrails
- `node.md` — Node.js / TypeScript conventions
- `plugin-ecc-conventions.md` — Molecule AI internal conventions

**Prompt fragments:**
- `AGENTS.md` — injected into the agent's system prompt on startup

## When to use it

Install on any Claude Code (or compatible) workspace. The skills are invoked on-demand
via the `Skill` tool; the rules and prompt fragments activate automatically.

## Installation

### In org template (org.yaml)
```yaml
plugins:
  - ecc
```

### From URL (community install)
```
github://Molecule-AI/molecule-ai-plugin-ecc
```

## License
Business Source License 1.1 — © Molecule AI.

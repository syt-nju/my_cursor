# my_cursor

This repository stores reusable agent assets for Cursor and Claude.

The shared source of truth is `.agent`. Tool-specific directories should be thin entrypoints into that shared content:

- `.cursor -> .agent` for Cursor
- `.claude -> .agent` for Claude

Shared content belongs in `.agent` by default. Only platform-specific registration files and hook adapters should diverge.

## Structure

- `.agent/commands`: slash commands (spec, plan, build, test, code-review, code-simplify, git, papers, …)
- `.agent/skills`: skill SKILL.md files — research workflow + full software engineering lifecycle
- `.agent/agents`: agent persona definitions (code-reviewer, security-auditor, test-engineer)
- `.agent/rules`: always-on rules and coding conventions
- `.agent/references`: checklist references linked from skills (testing, security, performance, accessibility)
- `.agent/hooks/core`: shared hook business logic (sdd-cache, simplify-ignore)
- `.agent/hooks.json`: Cursor hook registration (read as `.cursor/hooks.json`)
- `.agent/settings.json`: Claude hook/settings registration (read as `.claude/settings.json`)

See `.agent/README.md` for the full skill index, command list, and hook architecture.

## DeepXiv Skills Setup

The `deepxiv-*` skills under `.cursor/skills/` rely on the `deepxiv` CLI. Install it with pip before using those skills — see the [deepxiv_sdk](https://github.com/DeepXiv/deepxiv_sdk) repository for full docs.

```bash
pip install deepxiv-sdk
# or, for the full stack (MCP + built-in agent):
pip install "deepxiv-sdk[all]"
```

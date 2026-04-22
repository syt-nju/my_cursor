# my_cursor

This repository stores my reusable Cursor assets, including commands, skills, agents, and rules.

## Structure

- `.cursor/commands`: reusable command prompts
- `.cursor/skills`: reusable skills and supporting references
- `.cursor/agents`: agent-related configuration
- `.cursor/rules`: project rules and coding conventions

## DeepXiv Skills Setup

The `deepxiv-*` skills under `.cursor/skills/` rely on the `deepxiv` CLI. Install it with pip before using those skills — see the [deepxiv_sdk](https://github.com/DeepXiv/deepxiv_sdk) repository for full docs.

```bash
pip install deepxiv-sdk
# or, for the full stack (MCP + built-in agent):
pip install "deepxiv-sdk[all]"
```

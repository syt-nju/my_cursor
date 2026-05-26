# .agent

Shared configuration for Cursor and Claude. Both tools read from this directory via symlinks:

- Cursor: `.cursor -> .agent`
- Claude: `.claude -> .agent`

Shared content lives here by default. Only the two registration files at the root diverge between tools.

---

## Directory Structure

```
.agent/
  commands/          # slash commands (both tools)
  skills/            # skill SKILL.md files (both tools)
  agents/            # agent persona definitions (both tools)
  rules/             # always-on rules (both tools)
  references/        # checklist references linked from skills

  hooks/
    core/            # shared hook business logic (shell scripts)
      sdd-cache-pre.sh    # WebFetch cache: intercept and serve cached response
      sdd-cache-post.sh   # WebFetch cache: store response after fetch
      simplify-ignore.sh  # code-simplify protection (not registered; use on demand)

  hooks.json         # Cursor hook registration (read as .cursor/hooks.json)
  settings.json      # Claude hook/settings registration (read as .claude/settings.json)
```

---

## Skills

Skills are loaded on demand. Both Cursor (skills browser) and Claude Code (plugin system) discover them from their respective symlinked paths.

### Research & Paper Workflow
| Skill | When to use |
|---|---|
| `deepxiv-cli` | Search and read arXiv papers via CLI |
| `deepxiv-baseline-table` | Build a markdown baseline table for a research topic |
| `deepxiv-trending-digest` | Digest recent hot papers |
| `hugging-face-cli` | HF Hub downloads, uploads, and compute jobs |
| `pdf` | Read, merge, split, OCR PDF files |

### Software Engineering Lifecycle
| Skill | When to use |
|---|---|
| `spec-driven-development` | Starting a new feature — write spec before code |
| `planning-and-task-breakdown` | Have a spec, need verifiable tasks |
| `incremental-implementation` | Build one slice at a time, test each |
| `test-driven-development` | Write failing test first; Prove-It pattern for bugs |
| `debugging-and-error-recovery` | Something broke — systematic root-cause |
| `code-review-and-quality` | Five-axis review before merge |
| `code-simplification` | Reduce complexity without changing behavior |
| `security-and-hardening` | Input validation, auth, secrets, OWASP |
| `source-driven-development` | Verify against official docs before implementing |

### Meta
| Skill | When to use |
|---|---|
| `karparthy-guideline` | Writing or reviewing code — avoid LLM pitfalls |
| `skill-creator` | Creating or updating a skill |

---

## Commands

Commands map to skills and are invocable as slash commands in both tools.

| Command | Skill invoked |
|---|---|
| `/spec` | spec-driven-development |
| `/plan` | planning-and-task-breakdown |
| `/build` | incremental-implementation + test-driven-development |
| `/test` | test-driven-development |
| `/code-review` | code-review-and-quality |
| `/code-simplify` | code-simplification |
| `/git-commit` | — (standalone git commit helper) |
| `/git-push` | — (standalone git push helper) |
| `/harness-design` | karparthy-guideline (requirement → architecture → implementation) |
| `/download-paper` | deepxiv-cli |
| `/read-paper` | deepxiv-cli |
| `/select-paper` | deepxiv-cli |
| `/blog-to-html` | — (paper/blog → visual HTML) |
| `/publish-visual-reading` | — (push to syt-nju.github.io) |

---

## Hooks Architecture

### Design principle

Hook **business logic** lives in `hooks/core/` and is shared. Hook **registration** is split because Cursor and Claude use different JSON schemas.

```
Cursor reads:  .cursor/hooks.json   → .agent/hooks.json
Claude reads:  .claude/settings.json → .agent/settings.json

Both call:     .agent/hooks/core/*.sh  (same scripts)
```

### Currently registered hooks

**sdd-cache** — cross-session WebFetch cache for `source-driven-development`.

When the agent fetches an official documentation page, the response is cached locally. On the next fetch of the same URL, the cache checks HTTP `If-None-Match` / `If-Modified-Since` with the origin. Content is served from cache only on `304 Not Modified` — it's a freshness check, not a memory read.

| Event | Script | Behavior |
|---|---|---|
| `PreToolUse WebFetch` | `sdd-cache-pre.sh` | Revalidate with origin; on 304, exit 2 to block fetch and return cached body |
| `PostToolUse WebFetch` | `sdd-cache-post.sh` | Store response + ETag/Last-Modified to `.agent/sdd-cache/` |

Cache files live in `.agent/sdd-cache/` (gitignored). Requires `jq`, `curl`, `shasum`.

### Available but not registered

**simplify-ignore** — protects annotated code blocks during `/code-simplify` runs.

Annotate blocks you don't want simplified:
```js
/* simplify-ignore-start: perf-critical */
// manually unrolled — 3x faster than a loop
/* simplify-ignore-end */
```

Not registered by default because it modifies files in-place during the session (placeholders replace real code). If the session crashes without triggering the Stop event, files stay in placeholder state. Enable explicitly when using `/code-simplify` on a codebase with annotated blocks.

To enable, add to `hooks.json` (Cursor) and `settings.json` (Claude) — see `hooks/core/simplify-ignore.sh` header for the required hook events.

---

---

## Acknowledgements

The software engineering skills, agent personas, hook patterns, and command design in this directory are adapted from [**addyosmani/agent-skills**](https://github.com/addyosmani/agent-skills) — a collection of production-grade engineering skills for AI coding agents by Addy Osmani. The `sdd-cache` and `simplify-ignore` hook scripts are reproduced from that project with minimal modification.

---

## Adding a New Skill

1. Create `.agent/skills/<skill-name>/SKILL.md` with YAML frontmatter (`name`, `description`)
2. Add supporting files if needed (`scripts/`, `references/`)
3. Optionally create a matching `.agent/commands/<command-name>.md`
4. Update the skills table in this README

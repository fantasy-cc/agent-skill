# Agent Guide

## Project Overview
`agent-nexus` is a centralized configuration repository and framework for managing AI agent environments across multiple IDEs. It provides a single manifest (`nexus.yml`) that declares packages (skills, hooks, commands), MCP servers, and deployment targets — then compiles and deploys everything to Claude Code, Cursor, and Google Antigravity from one place. The project is building its own framework ("nexus") to replace Microsoft's APM, with the goal of being the best tool in this space — better than both APM and Kasetto.

## Tech Stack
- **nexus** — custom agent environment manager (manifest + CLI, replacing APM)
- **Bash + jq + yq** — implementation stack for nexus CLI (zero Python/Node runtime deps)
- **Markdown** for skill definitions (`SKILL.md`)
- **YAML** for manifests (`nexus.yml`)
- **Git** for package fetching (shallow clones at pinned refs)

## Project Structure
- `nexus.yml`: The core manifest. Declares packages (GitHub repos or local paths), inline MCP servers, optional MCPs, and target IDEs. Replaces the old `apm.yml`.
- `nexus.sh`: The nexus CLI entry point. Symlinked to `~/.local/bin/nexus` for global access. Run `nexus sync`, `nexus list`, `nexus doctor`, `nexus clean`.
- `.nexus/`: Local cache and compiled output directory (gitignored). Contains:
  - `cache/` — fetched packages keyed by `github.com/org/repo/commit-sha/`
  - `compiled/` — intermediate build artifacts (per-IDE skills, merged hooks, MCP fragments)
- `bin/`: Pre-built Go binaries — `xiaohongshu-mcp` (MCP server) from xpzouying/xiaohongshu-mcp.
- `scripts/`: Helper scripts for optional services (`xhs-start`, `xhs-relogin`).

## Installed Skills & MCP Servers

### Skills (from packages)

| Package | Skill | Description |
|---------|-------|-------------|
| `fantasy-cc/context-harness` | context-harness | Project docs generation (AGENTS.md, PLANS.md, FINDINGS.md, EVALUATION.md, README.md) with auto-recovery hooks |
| `obra/superpowers` | using-superpowers | Entry point for the superpowers workflow system |
| `obra/superpowers` | brainstorming | Structured brainstorming methodology |
| `obra/superpowers` | writing-plans | Plan creation and structuring |
| `obra/superpowers` | executing-plans | Systematic plan execution |
| `obra/superpowers` | test-driven-development | TDD workflow for agents |
| `obra/superpowers` | systematic-debugging | Structured debugging methodology |
| `obra/superpowers` | subagent-driven-development | Multi-agent orchestration patterns |
| `obra/superpowers` | dispatching-parallel-agents | Parallel agent task distribution |
| `obra/superpowers` | receiving-code-review | How to process code review feedback |
| `obra/superpowers` | requesting-code-review | How to request and structure code reviews |
| `obra/superpowers` | finishing-a-development-branch | Branch completion workflow |
| `obra/superpowers` | using-git-worktrees | Git worktree patterns for agents |
| `obra/superpowers` | verification-before-completion | Pre-completion verification checklist |
| `obra/superpowers` | writing-skills | How to author new agent skills |
| `find-skills` | find-skills | Discovers and installs skills from the open agent skills ecosystem (globally installed via skills CLI) |

### MCP Servers (inline in nexus.yml)

| Name | Transport | Description |
|------|-----------|-------------|
| `sequential-thinking` | stdio | Structured reasoning server |
| `playwright` | stdio | Browser automation via Playwright |
| `context7` | stdio | Up-to-date library documentation retrieval |
| `nitan-mcp` | stdio | Community MCP for Discourse integration |
| `xiaohongshu-mcp` (optional) | sse | Xiaohongshu content API via local Go server on localhost:18060 |
| `github-mcp` (optional) | stdio | GitHub API integration (requires GITHUB_TOKEN) |
| `notion-mcp` (optional) | stdio | Notion workspace integration |

## Development Workflow
- To add a package: Add a `repo:` entry under `packages:` in `nexus.yml`, then run `nexus sync`.
- To add an inline MCP: Add to the `mcps:` section of `nexus.yml`. Use `optional: true` or place under `optional_mcps:` for interactive prompting.
- To add a local skill in development: Use `path: ./my-skill` under `packages:`.
- To deploy everything: Run `nexus sync` (or `nexus sync --all` to auto-include optionals).
- To install a skills-CLI skill globally: Run `npx skills add <repo>@<skill> -g -y`.

## Coding Conventions
- Skills are directories containing a `SKILL.md` file. The directory name is the skill name.
- Hooks are discovered from `hooks/hooks.json` (Claude Code format) and `hooks/hooks-cursor.json` (Cursor format) within packages.
- `nexus.yml` is the single source of truth for all managed dependencies and MCP servers.
- No package type classification — nexus auto-discovers all asset types (skills, hooks, commands, agents) from each package.

## Architecture Decisions
- **APM to nexus migration**: APM misclassified hybrid packages (superpowers' 14 skills were never deployed because APM labeled it `hook_package`), created 42 duplicate hook entries (no dedup), and couldn't declare inline MCPs. `deploy.sh` was already doing most of the real work. We're building nexus to replace both APM and deploy.sh with a unified tool.
- **Unified package model**: A package can provide any combination of skills, hooks, commands, agents, and MCPs. No type classification — auto-discover via file patterns (SKILL.md, hooks.json, etc.).
- **Inline MCP declarations**: Most MCP servers are just `npx <package>`. Declaring them directly in the manifest eliminates the need for separate git repos.
- **Hook deduplication**: Hooks are deduplicated by content hash (minus metadata). Prevents the 42x duplication bug from APM.
- **Security review gate**: Before writing MCP configs to global IDE files, nexus shows what commands will be registered and prompts for confirmation. Addresses a known Kasetto security gap (issue #15).
- **Content-addressed cache**: Packages cached by commit SHA at `.nexus/cache/github.com/org/repo/sha/`. Immutable snapshots enable instant rollbacks and safe concurrent operations.
- **Bash + jq + yq**: Zero Python/Node runtime dependencies for the tool itself. Single script distribution.
- **Global Proxy via symlinks**: This repository is the single point-of-truth; IDE global skill directories symlink into it.
- **FINDINGS.md separation**: External/untrusted content is logged to FINDINGS.md (not PLANS.md) to prevent prompt injection via auto-read hooks.
- **Xiaohongshu MCP**: Uses xpzouying/xiaohongshu-mcp Go binary (HTTP on localhost:18060) after headless Playwright approaches proved flaky. Declared as optional SSE MCP in nexus.yml.

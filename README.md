# Agent Nexus

A framework and configuration repository for managing AI agent environments across multiple IDEs. Define your skills, hooks, and MCP servers once in `nexus.yml` — then deploy to **Claude Code**, **Cursor**, **Google Antigravity**, and more from a single command.

## Overview

Agent Nexus is building the best tool for unified AI agent environment management. It auto-discovers all asset types from packages (skills, hooks, commands, agents), declares MCP servers inline, deduplicates hooks, and includes a security review gate before writing to your global IDE configs.

### Why Nexus?

| Feature | APM | Kasetto | **Nexus** |
|---------|-----|---------|-----------|
| Hybrid packages (skills + hooks + MCPs) | No | No | **Yes** |
| Inline MCP declarations | No | No | **Yes** |
| Hook management + deduplication | Buggy (42x duplication) | Not implemented | **Yes** |
| Security review before deploy | No | No ([issue #15](https://github.com/pivoshenko/kasetto/issues/15)) | **Yes** |
| Optional/conditional deps | No | No | **Yes** |
| Project context system | No | No | **Yes** (via context-harness) |
| Auto-discover package assets | No (type classification) | No (manual listing) | **Yes** |

### Supported IDEs
- **Claude Code** — skills, MCP servers, hooks
- **Cursor** — skills, MCP servers, hooks
- **Google Antigravity** — skills, MCP servers

## Getting Started

### Prerequisites
- **Git**
- **Node.js 18+** (for MCP servers that use npx)
- **jq** and **yq** (for the nexus CLI)

### Installation & Deployment

```bash
# 1. Clone the repository
git clone https://github.com/lifan-builds/agent-nexus.git ~/Project/agent-nexus
cd ~/Project/agent-nexus

# 2. Add nexus to your PATH
mkdir -p ~/.local/bin
ln -snf "$(pwd)/nexus.sh" ~/.local/bin/nexus

# 3. Deploy everything
nexus sync
```

`nexus sync` automates the entire setup:
1. Fetches packages from GitHub into `.nexus/cache/` (content-addressed by commit SHA).
2. Auto-discovers all assets in each package (skills, hooks, commands, agents).
3. Compiles and symlinks skills to all target IDE directories.
4. Aggregates and deduplicates hooks across packages.
5. Shows a security review of MCP changes before writing to global configs.
6. Merges MCP server configs into Claude (`~/.claude.json`), Cursor (`~/.cursor/mcp.json`), and Antigravity (`~/.gemini/antigravity/mcp_config.json`).
7. Generates `nexus.lock.yml` tracking what was deployed where.

## Managed Assets

### Skills (15 total)

| Package | Skills | Description |
|---------|--------|-------------|
| `fantasy-cc/context-harness` | 1 | Project docs generation and context management |
| `obra/superpowers` | 14 | TDD, brainstorming, code review, debugging, worktrees, parallel agents, and more |
| `find-skills` (global) | 1 | Agent skill discovery tool |

### MCP Servers

| Name | Transport | Description |
|------|-----------|-------------|
| `sequential-thinking` | stdio | Structured reasoning |
| `playwright` | stdio | Browser automation |
| `context7` | stdio | Library documentation retrieval |
| `nitan-mcp` | stdio | Discourse integration |
| `xiaohongshu-mcp` | sse (optional) | Xiaohongshu content API via local Go server |
| `github-mcp` | stdio (optional) | GitHub API integration |
| `notion-mcp` | stdio (optional) | Notion workspace integration |

## Project Structure

- `nexus.yml` — The manifest. Define packages, MCP servers, and targets here.
- `.nexus/` — Package cache and compiled artifacts (gitignored).
- `bin/` — Pre-built Go binaries for optional services.
- `scripts/` — Helper scripts for optional services.
- `AGENTS.md` — AI agent context for this repository.
- `PLANS.md` — Active development roadmap.
- `FINDINGS.md` — Research logs and discovery notes.
- `EVALUATION.md` — Verification contracts and evaluation log.

## Development

To add a new package:
1. Add a `repo:` entry under `packages:` in `nexus.yml`.
2. Run `nexus sync` to fetch, discover, and deploy.

To add an inline MCP server:
1. Add to the `mcps:` section of `nexus.yml`.
2. Run `nexus sync`.

---
*Maintained by lfan. Powered by nexus.*

# Agent Nexus

A framework and configuration repository for managing AI agent environments across multiple IDEs. Define your skills, hooks, and MCP servers once in a nexus manifest — then deploy to **Claude Code**, **Cursor**, **Google Antigravity**, **Codex**, and more from a single command.

## Quick Start

```bash
git clone https://github.com/lifan-builds/agent-nexus.git ~/Project/agent-nexus
cd ~/Project/agent-nexus
cp nexus.example.yml nexus.personal.yml
pip install pyyaml
python nexus.py sync
```

`nexus sync` fetches declared packages, discovers skills/hooks/MCPs, previews
security-sensitive MCP changes, and deploys approved assets to your configured
agent IDEs.

## What Nexus Manages

- **Skills** - reusable agent workflows discovered from package contents
- **Hooks** - deduplicated automation deployed to supported IDEs
- **MCP servers** - merged into existing global configs without overwriting local secrets
- **Package cache** - content-addressed snapshots under `.nexus/cache/`
- **Context systems** - project context via packages such as `context-harness`

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
- **Codex** — skills, MCP servers

## Getting Started

### Prerequisites
- **Python 3.10+** with **PyYAML** (`pip install pyyaml`)
- **Git**
- **Node.js 18+** (for MCP servers that use npx)

### Installation & Deployment

```bash
# 1. Clone the repository
git clone https://github.com/lifan-builds/agent-nexus.git ~/Project/agent-nexus
cd ~/Project/agent-nexus

# 2. Copy the example config and customize for your machine
cp nexus.example.yml nexus.personal.yml

# 3. Add nexus to your PATH
mkdir -p ~/.local/bin
ln -snf "$(pwd)/nexus.py" ~/.local/bin/nexus

# 4. Deploy everything
nexus sync
```

`nexus sync` automates the entire setup:
1. Fetches packages from GitHub into `.nexus/cache/` (content-addressed by commit SHA).
2. Auto-discovers all assets in each package (skills, hooks, commands, agents).
3. Prunes stale skill symlinks and MCP entries removed since last sync.
4. Symlinks skills to all target IDE directories.
5. Aggregates and deduplicates hooks across packages.
6. Shows a security review of MCP changes before writing to global configs.
7. Merges MCP server configs into Claude (`~/.claude.json`), Cursor (`~/.cursor/mcp.json`), and Antigravity (`~/.gemini/antigravity/mcp_config.json`).
8. Generates a lockfile tracking what was deployed where.

## Managed Assets

### Skills (example config)

| Package | Skills | Description |
|---------|--------|-------------|
| `fantasy-cc/context-harness` | 1 | Project docs generation and context management |
| `obra/superpowers` | 14 | TDD, brainstorming, code review, debugging, worktrees, parallel agents, and more |

### MCP Servers

| Name | Transport | Description |
|------|-----------|-------------|
| `sequential-thinking` | stdio | Structured reasoning |
| `playwright` | stdio | Browser automation |
| `context7` | stdio | Library documentation retrieval |
| `nitan-mcp` | stdio | Discourse integration |
| `github-mcp` | stdio (optional) | GitHub API integration |

## Project Structure

- `nexus.example.yml` — Public template manifest for the repo.
- `nexus.personal.yml` — Gitignored personal manifest. Define your packages, MCP servers, and targets here.
- `.nexus/` — Package cache and compiled artifacts (gitignored).
- `AGENTS.md` — AI agent context for this repository.
- `PLANS.md` — Active development roadmap.
- `FINDINGS.md` — Research logs and discovery notes.
- `EVALUATION.md` — Verification contracts and evaluation log.

## Development

To add a new package:
1. Add a `repo:` entry under `packages:` in `nexus.personal.yml`.
2. Run `nexus sync` to fetch, discover, and deploy.

To add an inline MCP server:
1. Add to the `mcps:` section of `nexus.personal.yml`.
2. Run `nexus sync`.

---
*Maintained by lfan. Powered by nexus.*

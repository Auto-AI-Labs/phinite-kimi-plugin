# Phinite plugin for Kimi Code

Build, ship, run and debug **Phinite agent graphs** from Kimi Code CLI. The plugin connects Kimi Code to the hosted
**Phinite MCP server** (`https://mcp.dev.phinite.ai/mcp`) and adds the Phinite playbook as skills, commands and the `phinite-builder`
subagent. It is generated from the Claude plugin, so both behave the same, with the same `/phinite:*` commands.

## Install

```text
/plugins install https://github.com/Auto-AI-Labs/phinite-kimi-plugin
/reload
```

Or from a local folder: `/plugins install <path to this folder>` (Kimi copies it, so reinstall after changes).

You need Python 3.9+ for the project script. No Phinite token is stored in the repo.

To use another server (production or a local one), change `url` under `mcpServers` in `kimi.plugin.json`.

## Sign in

Run `/mcp-config login phinite`, or `/phinite:login`. A browser opens on the Phinite login (Google, GitHub or
email); pick the org and the workspace. To switch workspace, sign in again and pick another one. `/mcp` shows the
server's status.

## Update

Kimi Code has no automatic updates for plugins. Run `/plugins`, press `Tab` to **Installed**, select **phinite**
and press `Enter` (it updates when a newer version exists), then `/reload`. A plugin installed from a local folder
is a copy: install it again after `git pull`.

## Use

| Command | What it does |
|---|---|
| `/phinite:login [workspace]` | Sign in, check the sign-in, or switch workspace |
| `/phinite:new <folder> [schema] [idea]` | Scaffold a use-case project and design its graphs with you |
| `/phinite:deploy <folder>` | Dry run, then deploy: tools upserted (published when changed), graphs created or updated |
| `/phinite:test <graph> [message \| user_variables]` | Chat or run the graph autonomously, read the log, diagnose |
| `/phinite:ship <graph> [env]` | Publish → build → environment → trigger or Chat API → API key → call it from outside |
| `/phinite:logs <workflow_id>` | Explain a run: tool calls, inputs, outputs, variables, errors |
| `/phinite:eval <graph> [focus]` | Evaluate with LLM judges: cases, metrics, failures and fixes |

Skills are `/skill:phinite:<skill>` (for example `/skill:phinite:phinite-design`), and Kimi also loads them by
itself when they fit. The `phinite-builder` subagent takes a whole use case end to end.

## What's inside

- **Skills** (the same as the Claude plugin's, from the Phinite playbook): `phinite-design`, `phinite-ship`,
  `phinite-debug`, `phinite-api` and `phinite-project`, plus `phinite-quality` for evaluations (LLM judges,
  custom metrics, production scoring).
- **A local project script**, `skills/phinite-project/scripts/phinite_project.py` (Python 3.9+, standard library
  only). It scaffolds use-case folders from `templates/`, runs `graph_spec.py` on your machine, bundles the project
  and sends files through the server's upload links. The hosted server can't read your disk.
- `tests/`: the offline tests for that script: `python -m unittest tests/test_offline.py`.

The MCP tools and the rules they apply are listed in the
[phinite-mcp README](https://github.com/Auto-AI-Labs/phinite-mcp#tools).

## Security notes

- **No secrets on disk or in the chat.** The client stores and refreshes the OAuth token; the assistant never sees
  it. Env values go from your `.env` straight to an upload link and never pass through the chat.
- **Upload links** are valid for 15 minutes and for one purpose. The project script accepts only https links on
  `*.phinite.ai` (or the `PHINITE_MCP_URL` host, or localhost) with the `/u/<ticket>` path, so a link from a file,
  web page or message is refused.
- **Hidden files** (`.env`, `.git`…) in folders are never sent, and `KNOWLEDGE` files must be inside the project.
- **`.env` values never reach the chat.** The ground rules tell the assistant never to open or print a
  `.env` in any task, only to list its names (`cut -d= -f1 .env`). This client's plugins can't ship
  hooks, so unlike the other Phinite plugins nothing enforces it.
- **Env variables are workspace-wide**: prefix a use case's own secrets (`MYUC_SLACK_TOKEN`), and the assistant asks
  before overwriting a name that already exists.
- **`graph_spec.py` is code**: `deploy` runs it on your machine. Deploy only projects you wrote or trust.

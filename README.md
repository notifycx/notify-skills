# Notify Skills

Official [Agent Skills](https://agentskills.io) and [Agent Plugins](https://agent-plugins.org) package for [Notify](https://notify.cx) — a transactional email API for developers and AI agents. Send password resets, magic links, receipts, and product notifications with one HTTPS call, the `notifycx` SDK, or the `notifycx-mcp` MCP server. This repo bundles the **notify-email** skill, domain setup guidance, and MCP wiring for Claude Code, Cursor, Codex, and any skills-compatible client.

## Install

### Claude Code (marketplace)

Add this repository as a marketplace, then install the plugin:

```text
/plugin marketplace add notifycx/notify-skills
/plugin install notifycx@notify-skills
```

When prompted, set your **Notify API key** (`notify_api_key`). It is stored securely and passed to `notifycx-mcp` as `NOTIFY_API_KEY`.

Alternatively, clone the repo and load it locally:

```bash
claude --plugin-dir .
```

### Cursor

Install from the Cursor plugin marketplace entry in `.cursor-plugin/`, or add MCP manually (see below). Enable the **notify-email** skill from this repo via:

```bash
npx skills add notifycx/notify-skills
```

### Codex (OpenAI)

Install the portable plugin (`plugin.json`, `skills/`, `mcp.json`) from this repository using your Codex plugin workflow, or use the compatibility manifest in `.codex-plugin/plugin.json`. Onboarding runs the **notify-email** skill.

### skills.sh

```bash
npx skills add notifycx/notify-skills
```

Select **notify-email** when prompted.

### Manual MCP setup

If you only need MCP tools (no skill bundle):

```json
{
  "mcpServers": {
    "notify": {
      "command": "npx",
      "args": ["-y", "notifycx-mcp@1.2.3"],
      "env": {
        "NOTIFY_API_KEY": "nk_..."
      }
    }
  }
}
```

Without a key yet, omit `NOTIFY_API_KEY` and use MCP `bootstrap_start` / `bootstrap_confirm`. See https://notify.cx/docs/agent-bootstrap-no-dashboard

Never commit real API keys.

## Skill

| Skill | Description |
|-------|-------------|
| [`notify-email`](./skills/notify-email) | Transactional email with Notify — setup, send via SDK/HTTP/MCP, framework snippets, limits, and errors |

## MCP server

This plugin runs the official [`notifycx-mcp`](https://www.npmjs.com/package/notifycx-mcp) package (v1.2.3) via `npx`. It reads **`NOTIFY_API_KEY`** from the environment. Claude Code maps your configured `notify_api_key` into that variable automatically.

Docs: https://notify.cx/mcp · https://notify.cx/docs/how-to-use-notifycx-mcp

## Prerequisites

- A [Notify](https://notify.cx/sign-up) account
- A verified domain for production sends from your own `from` address
- `NOTIFY_API_KEY` in server-side env or plugin config

## Plugin layout

| Path | Purpose |
|------|---------|
| `.claude-plugin/` | Claude Code manifest + marketplace |
| `.cursor-plugin/` | Cursor plugin metadata |
| `.codex-plugin/` | Codex compatibility overlay |
| `plugin.json` / `mcp.json` | Portable [Agent Plugins](https://agent-plugins.org) manifest |
| `.mcp.json` | Claude MCP config with `${user_config.notify_api_key}` |
| `skills/notify-email/` | Agent skill |
| `evals/` | Example skill eval prompts |

## Links

- Docs: https://notify.cx/docs
- Pricing: https://notify.cx/pricing
- SDK: https://github.com/notifycx/notifycx

## License

MIT — see [LICENSE](./LICENSE).

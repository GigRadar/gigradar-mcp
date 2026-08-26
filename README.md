# GigRadar MCP

Connect your [GigRadar](https://gigradar.io) account to Claude, Cursor, ChatGPT, Codex or any MCP-compatible AI agent.

GigRadar monitors Upwork for jobs matching your saved searches and can send proposals automatically. This server lets an AI agent configure and operate that for you — in your own words.

> "Set me up a scanner for senior React jobs from US clients paying at least $60/hour, but skip anything WordPress."

The agent drafts the query, checks how many jobs a month it would actually match, shows you the number, and only then saves it.

## Install

The server URL is:

```
https://api.gigradar.io/mcp/v1/mcp
```

Add it to your client, then sign in with your normal GigRadar login when the browser opens. Nothing to copy, no API key to generate.

**Claude Code**

```bash
claude mcp add --transport http gigradar https://api.gigradar.io/mcp/v1/mcp
```

**Claude Desktop** — Settings → Connectors → Add custom connector, paste the URL.

**Cursor** — Settings → MCP → Add new MCP server, choose HTTP, paste the URL.

**Codex / other clients** — add an HTTP (streamable) MCP server pointing at the URL. Authentication is standard OAuth 2.1 with PKCE and dynamic client registration, so any spec-compliant client handles the login for you.

## The skill

The server ships tools; the skill ships the know-how — how to scope a scanner that neither starves nor floods, the search syntax, how to diagnose one that stopped finding work.

Ask your agent to run `gigradar_init` and it will offer to install it, or do it yourself:

```bash
mkdir -p .agents/skills
git clone --depth 1 https://github.com/GigRadar/gigradar-mcp /tmp/gigradar-mcp
cp -r /tmp/gigradar-mcp/skill .agents/skills/gigradar
rm -rf /tmp/gigradar-mcp
```

`.agents/skills/` is deliberately client-agnostic — the same files work in Claude, Codex, OpenClaw and anything else following that convention.

## What it can do

**Scanners** — list, read, create, update, duplicate, delete, reorder. Every save reports how many jobs per month the query matches, so a broken scanner cannot go live unnoticed.

**Job search** — search the live Upwork index with GigRadar's filters, and pull market aggregates (volume over time, budget distribution, client mix).

**Opportunities** — read the jobs your scanners matched, with the match reasoning.

**Teams** — one login reaches every team you belong to. Switch between them mid-conversation; the switch sticks.

**Ask GigRadar** — put a question to GigRadar's built-in assistant, answered from GigRadar's official documentation. Your agent uses this instead of guessing at a product that updates often.

## Safety

The server acts as you, with exactly the access you already have — it can reach the same teams your dashboard can, and nothing else. Team membership is re-checked on every switch, so a team you have been removed from becomes unreachable immediately.

Destructive actions (deleting a scanner, sending a proposal) are marked as such, and the tool descriptions instruct agents to confirm with you first.

Disconnecting the server in your MCP client revokes that connection alone and leaves your dashboard session untouched.

## Support

- Docs: [help.gigradar.io](https://help.gigradar.io)
- Email: support@gigradar.io

# GigRadar MCP tools

Every tool acts on the connection's ACTIVE team. The team is never a tool argument — change it with `switch_team`.

## Orientation

| Tool | Purpose |
| --- | --- |
| `gigradar_init` | Call once per session. Account, team, working practices, skill install. |
| `whoami` | Signed-in account and active team. |
| `list_teams` | Every team this account can act on. |
| `switch_team` | Change the active team. Persists across conversations. |

## Scanners

A scanner is a saved search that runs continuously and feeds the autobidder.

| Tool | Purpose |
| --- | --- |
| `list_scanners` | All scanners on the team. Start here — you need the id to read or edit one. |
| `get_scanner` | One scanner's full configuration. Call before every update. |
| `preview_scanner_matches` | Validate a query and estimate monthly volume WITHOUT saving. |
| `create_scanner` | Create. Returns `avgMonthlyMatches`. |
| `update_scanner` | Update. Query fields MERGE into the existing query. |
| `duplicate_scanner` | Clone for A/B testing. Fully independent of the source. |
| `delete_scanner` | Delete. Immediate — confirm first. |
| `reorder_scanner` | Change priority. Position 1 is highest. |

Priority decides which scanner claims a job when several match — relevant when scanners overlap and each has a different cover-letter template.

## Job search

| Tool | Purpose |
| --- | --- |
| `search_gigs` | Search the live Upwork index. Read-only — the safe way to test a query. |
| `get_gigs_insights` | Aggregates: volume over time, budget distribution, client mix. |

`search_gigs` shows you WHICH jobs a query returns; `preview_scanner_matches` tells you HOW MANY per month. Use both before saving a scanner.

## Opportunities

An opportunity is a job one of the team's scanners matched.

| Tool | Purpose |
| --- | --- |
| `get_opportunity` | Full detail: job, match reasoning, application state. |

## Help

| Tool | Purpose |
| --- | --- |
| `ask_gigradar` | Ask GigRadar's built-in assistant, answered from GigRadar's documentation. |
| `submit_feedback` | Report a bug or request a feature, straight to the GigRadar team. |

Stateless — each call is independent, so include context in the question.

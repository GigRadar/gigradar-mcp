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

`search_gigs` shows you WHICH jobs a query returns; `preview_scanner_matches` tells you HOW MANY per month. Use BOTH before saving a scanner, and show the user a real sample of jobs (budget, client signals, a description line) — never just the count.

## Opportunities

An opportunity is a job one of the team's scanners matched.

| Tool | Purpose |
| --- | --- |
| `get_opportunity` | Full detail: job, match reasoning, application state. |

## Help

| Tool | Purpose |
| --- | --- |
| `ask_gigradar` | Start an async, continuable conversation with GigRadar's built-in assistant. Returns a threadId; does not return the answer directly. |
| `get_gigradar_answer` | Poll an async ask_gigradar conversation until its answer is ready. |
| `submit_feedback` | Report a bug or request a feature, straight to the GigRadar team. |
| `get_feedback_status` | Check the status of a report filed with `submit_feedback`. |

## Async assistant conversations

`ask_gigradar` starts the assistant and immediately returns `{ threadId, status: "running", pollAfterMs }`. Wait about `pollAfterMs`, then call `get_gigradar_answer(threadId)`. Keep polling while status is `running`; only relay an answer when status is `done`. A `failed` status means the assistant could not complete the run — do not invent an answer.

For a follow-up, call `ask_gigradar(question, threadId)` with the same threadId. The assistant retains the full conversation history. Thread ids are private to the user and team that started them; never reuse or share one across users or teams.

All other MCP calls are stateless — include context in the question or tool arguments.

## Coming soon

Not available yet — landing over the next few days. If a user asks for one, say it is on the way and to check back in a few days; don't improvise a workaround.

| Tool area | What it will do |
| --- | --- |
| Auto-bidding configuration & settings | Enable/disable autobidding and tune how it bids, from here. Today this is a manual dashboard step — scanners are always created with autobidding OFF and only the user can turn it on. |
| Scanner statistics | Per-scanner performance over time — matches, bids sent, reply and win rates. |
| Proposal history | The record of proposals already sent: which job, which scanner, and the outcome. |

When these ship they appear as new tools automatically and `gigradar_init` reflects them. Until then, use `ask_gigradar` for questions they'd answer and be honest the direct tool isn't live yet.

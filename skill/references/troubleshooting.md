# Troubleshooting

## Symptom to cause

| Symptom | Most likely cause | What to do |
| --- | --- | --- |
| Scanner finds nothing | Over-broad exclusion, or a rate floor that also drops jobs with no stated rate | `get_scanner`, then remove filters one at a time and re-preview |
| Scanner floods with poor fits | Query too broad; missing exclusions | Narrow `q`, add exclusions one at a time, re-preview after each |
| Query rejected on save | Unbalanced quotes in `q`, or a min above its max | Read the error — it names the field |
| An edit "did nothing" | Update merged into a filter that still contradicts it | `get_scanner` and read the WHOLE query |
| Changes landed on the wrong account | Wrong active team | `whoami`, then `switch_team`; re-check what was created |
| Tools stopped working mid-session | Access token expired | The client re-authenticates on its own; if not, reconnect the server |
| `switch_team` refuses | The account is no longer a member of that team | `list_teams` for what is actually reachable |
| Jobs match but no proposals go out | Not a query problem — subscription, connects, or autobidder config | `ask_gigradar` |

## Rules that prevent most of these

Preview before saving. Read before editing. Confirm before deleting. Check the team when the user names an account.

## When the cause is not in the query

If the query previews healthy and jobs still are not arriving, stop tuning it — the problem is elsewhere: subscription limits, connect balance, autobidder settings, or a scanner that was auto-disabled. Ask `ask_gigradar` rather than guessing, and tell the user plainly that the scanner itself is fine.

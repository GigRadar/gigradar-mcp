---
name: gigradar
description: Operate a GigRadar account through the GigRadar MCP server — configure Upwork job scanners, search the job market, manage applications and CRM conversations, and switch between teams. Use whenever the user mentions GigRadar, Upwork lead generation, job scanners, autobidding, proposals, or connects; or asks to find Upwork work, set up job alerts, or see why they are not receiving jobs.
---

# GigRadar

GigRadar monitors Upwork for jobs matching saved searches ("scanners") and can send proposals automatically ("autobidding"). This skill covers operating a real customer account through the GigRadar MCP server.

## Start here

Call `gigradar_init` once per session. It reports which account and team you are on, and gives you the current working practices.

Then `whoami` any time the user's wording implies a particular account ("on my agency", "for the client team"). Every tool acts on ONE active team, and acting on the wrong one is the most expensive mistake available here.

## The two rules

**1. Preview before you save.** Always call `preview_scanner_matches` before `create_scanner` or `update_scanner`, show the user the number, and only then save.

A scanner is not a search box — it runs continuously, and if autobidding is on it spends the customer's connects (Upwork's paid application currency). The two failure modes are silent and both cost real money:

- **Matches 0 jobs/month.** Nobody notices for a week. The customer thinks GigRadar is broken.
- **Matches thousands.** The autobidder applies to poor-fit jobs and burns connects.

Reading `avgMonthlyMatches`:

| Volume | Reading |
| --- | --- |
| 0 | Broken. Do not leave it live. |
| 1–20 | Very narrow. Fine for a real niche — confirm it is intended. |
| 20–300 | Healthy for most teams. |
| 300–2000 | Broad. Needs a well-tuned autobidder. |
| 2000+ | Too broad. Narrow it before enabling bidding. |

**2. Read before you edit.** `get_scanner` before `update_scanner`. Query updates MERGE field by field — editing blind can contradict a filter that is already there (adding a country filter when `notInCountry` already excludes it) and silently match nothing.

## Query syntax

`query.q` is an Elasticsearch `simple_query_string`:

| Pattern | Meaning |
| --- | --- |
| `react` | one term |
| `"react native"` | exact phrase — always quote multi-word terms |
| `react + typescript` | AND |
| `react \| vue` | OR |
| `react + -wordpress` | AND NOT |
| `(react \| vue) + senior` | grouping |
| `develop*` | prefix wildcard |

Every `"` must be closed — unbalanced quotes are rejected before saving.

Prefer `q` with operators over the `anyKeywords` / `excludedKeywords` fields for anything non-trivial; those are plain comma lists with no grouping.

## Building a scanner

The loop that works:

1. **Understand the intent.** What work does the user actually want? Ask about exclusions explicitly — they are the most common source of a starved scanner and users rarely volunteer them.
2. **Draft the query.** Start with the positive terms only.
3. **`search_gigs`** to see what actually comes back. Read a few titles: are these the jobs the user meant?
4. **`preview_scanner_matches`** for the monthly volume.
5. **Add exclusions ONE AT A TIME**, re-previewing after each. A single over-broad exclusion can zero out a healthy query.
6. **Tell the user the number** and what it implies.
7. **`create_scanner`.** Report the returned `avgMonthlyMatches`.

Worked example — "React jobs but not WordPress, US clients, at least $50/hr":

```
q:              react + -wordpress
country:        United States
minHourlyRate:  50
```

Preview it. If the volume is 0, drop the rate floor first (it excludes jobs with no stated rate), then the country filter, then the exclusion — in that order, because that is the order of how often each is the culprit.

## Diagnosing a scanner that finds nothing

In the order worth checking:

1. `get_scanner` and read the WHOLE query, not just `q`.
2. `preview_scanner_matches` with the same query — confirm it really is zero.
3. Remove filters one at a time and re-preview. The usual culprits, most common first: an over-broad exclusion, a rate floor (which also drops jobs with no stated rate), a country filter, an over-specific phrase.
4. If the query is fine but no jobs arrive, it is not a query problem — check subscription and connect balance with `ask_gigradar`.

## Teams

One login can reach several teams. `list_teams` shows them, `switch_team` changes the active one, and the switch persists across conversations.

Announce every switch. A scanner created on the wrong team can start sending real proposals from the wrong profile.

## Asking GigRadar itself

`ask_gigradar` puts a question to GigRadar's own assistant, answered from the official knowledge base.

Use it instead of guessing. GigRadar ships weekly, so training data goes stale — anything about how a feature behaves, why something happened, or what a term means should come from here.

Good: "How does the autobidder choose which jobs to bid on?", "Why would a scanner be auto-disabled?", "How are connects consumed?"

If it says it has no answer, tell the user that. Do not fill the gap yourself.

## Destructive actions

`delete_scanner` takes effect immediately. Confirm first, and name the scanner you are about to delete — never infer it from a vague "clean up my scanners".

`create_application` sends a real proposal and spends real connects. Show the user what will be sent before you call it.

## Reference

- `references/tools.md` — every tool, grouped by domain
- `references/scanner-filters.md` — the full scanner filter list
- `references/troubleshooting.md` — symptom-to-cause table

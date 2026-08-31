---
name: gigradar
description: Operate a GigRadar account through the GigRadar MCP server — configure Upwork job scanners, search the job market, manage applications and CRM conversations, and switch between teams. Use whenever the user mentions GigRadar, Upwork lead generation, job scanners, autobidding, proposals, or connects; or asks to find Upwork work, set up job alerts, or see why they are not receiving jobs.
---

# GigRadar

GigRadar monitors Upwork for jobs matching saved searches ("scanners") and can send proposals automatically ("autobidding"). This skill covers operating a real customer account through the GigRadar MCP server.

## Start here

Call `gigradar_init` once per session. It reports which account and team you are on, and gives you the current working practices.

Then `whoami` any time the user's wording implies a particular account ("on my agency", "for the client team"). Every tool acts on ONE active team, and acting on the wrong one is the most expensive mistake available here.

## What a scanner is, and why care is required

A scanner is not a search box. It runs continuously, and when autobidding is on it sends a real Upwork proposal — spending the customer's connects (Upwork's paid application currency) — for **every** new job it matches. A careless scanner spends real money on jobs the freelancer would never have applied to by hand.

So the standard is higher than "does the query run". The standard is: **the user has seen the actual jobs this scanner will bid on, and agreed they are the right jobs, on the right team, before it is saved.** A scanner you saved without the user looking at its real matches is a scanner you have not finished configuring — even if the volume number looked fine.

## Creating a scanner — the required sequence

Every step below is mandatory. Do not skip a step because the request seems obvious or the user seems in a hurry. A count is never a substitute for looking at the jobs, and "it matches ~40/month" is never approval.

### 1. Confirm the team first

Before creating anything, confirm WHICH team this scanner belongs on and get the user's OK for that specific team. Use `whoami` to see the active team; `list_teams` and `switch_team` if it belongs elsewhere. A scanner created on the wrong team can start sending real proposals from the wrong freelancer's profile. Never assume the active team is the intended one — name it and confirm.

### 2. Read the existing scanners and their order

Call `list_scanners` and actually read the lineup **before** drafting anything new. You need three things from it:

- **What is already covered**, so you do not create a near-duplicate. If the new request overlaps an existing scanner, surface that scanner and ask whether to edit it, duplicate it, or scope the new one more narrowly.
- **The strategy** the existing scanners express (broad catch-all? several tight niches?).
- **The ORDER**, because order changes which jobs the new scanner will actually win.

**Why order matters (the pyramid rule).** When two scanners share the same freelancer profile, only the highest-priority scanner that matches a given job sends the bid — the lower ones stay silent on that job. So a broad scanner sitting near the top "claims" jobs that a more specialized scanner below it was built to win, and the specialized scanner's sharper, tailored proposal never goes out. The rule: **most specialized at the top, broadest catch-all at the bottom.** Volume is the easy proxy — a scanner matching 50 jobs/month is usually more specialized than one matching 400.

When each scanner has its **own** freelancer profile (deliberate cross-bidding), order does not matter — every matching scanner bids. You cannot see profiles from here, so when order looks like it matters, ask: "do these scanners share one freelancer profile?"

Decide the new scanner's position deliberately, and tell the user the consequence in plain words: e.g. "I'll place this above your `All React` catch-all so it wins the PropTech jobs with its own proposal — otherwise the catch-all would bid on them first." New scanners append at the bottom by default; use the `index` argument on `create_scanner` (1 = top) to position it in one step, or `reorder_scanner` afterwards.

### 3. Draft the query — including client-quality filters

**Write `query.q` yourself in the query syntax — read "Query syntax" below before drafting it.** `q` is an Elasticsearch `simple_query_string` that AND-s bare words together, so passing the user's phrasing straight through ("power bi dashboard developer") silently matches a fraction of what they asked for. Every query you save is quoted phrases, `|` alternatives, `()` groups and `*` prefixes — never a plain sentence.

A good query is also not just the right keywords — a query with **no client-quality or budget floor feeds the autobidder low-quality jobs**: unverified clients who never pay, $10 budgets, one-line postings. Propose these deliberately and explain the trade-off:

- `paymentVerified` — the client has confirmed a billing method with Upwork. Correlates with clients who actually pay. Almost always worth setting.
- `minFixedBudget` / `minHourlyRate` — a floor that keeps out the $5 jobs. (Note: a rate floor also drops jobs that state *no* rate — see troubleshooting.)
- `minRating`, `totalSpent` — clients with a track record.

Do not silently pile these on — each one narrows volume, and several stacked together is the most common way to build a scanner that looks reasonable and matches almost nothing. Add them, then check volume, then check the jobs.

### 4. Preview the real jobs — and SHOW them

This is the step that separates a configured scanner from a guess. Two different tools, both needed:

- **`search_gigs`** with the draft query returns the ACTUAL jobs — title, description, budget, client, country. This tells you WHICH jobs.
- **`preview_scanner_matches`** returns `avgMonthlyMatches` and validates the syntax WITHOUT saving. This tells you HOW MANY.

Read the returned jobs yourself and decide, per job: relevant / borderline / off-target. Then show the user a real sample — not a count. Surface the specifics a freelancer or client actually judges on:

- **budget** (and whether it clears their floor),
- **client signals** — payment verified? total spent? rating? country?,
- **what the posting actually asks for** — a line from the description that shows it is (or isn't) the work they want.

Search results carry an excerpt of each description (`descriptionPreview`), centred on what matched — enough to spot an off-target job, not enough to judge a good one. Open the postings that matter with `get_gig` (it takes the ciphertext from the search result) and read the real text: the words worth including, and the words worth excluding, are in there rather than in the title.

If the sample contains off-target jobs, extract the exact word(s) that made each one wrong and add them to `excludedKeywords` — **one at a time, re-previewing after each**, because a single over-broad exclusion can zero out a healthy query. Only exclude words you saw in a real bad job, and check the exclusion does not collide with a wanted term (excluding `lead` would kill `technical lead`).

`avgMonthlyMatches` measures volume, not quality. A query returning 200 jobs/month means nothing until you have read a sample of them.

Reading `avgMonthlyMatches`:

| Volume | Reading |
| --- | --- |
| 0 | Broken. Do not leave it live. |
| 1–20 | Very narrow. Fine for a real niche — confirm it is intended. |
| 20–300 | Healthy for most teams. Aim here, with the sample mostly relevant. |
| 300–2000 | Broad. Needs a well-tuned autobidder or more exclusions. |
| 2000+ | Too broad. Narrow it before enabling bidding. |

### 5. Get explicit approval, then save

Show the user the final query, the exclusions, the measured volume, and the real sample. Get an explicit OK on **those jobs and that query** before calling `create_scanner`. "Looks good" on a sample of real postings is approval; a healthy-looking number is not. Never save on the assumption that the matches are relevant.

Report the returned `avgMonthlyMatches`. If it comes back 0, the scanner is broken — fix it rather than leaving it live.

### 6. After creating — link, and recommend the autobidder

Once the scanner is saved:

- **Give the user a dashboard link** so they can review the scanner and its incoming matches / insights: `https://beta.gigradar.io/scanner/<id>` (the `<id>` is on the created scanner). Always include this.
- **Recommend reviewing the autobidder** — that is what turns matches into sent proposals. It is a real-money action, so frame it as a recommendation, not something you do for them.

## One scanner = one target

Resist the mega-scanner. A single broad "Full-Stack" or "Web Dev" scanner with loose keywords kills reply rate, because it bids on everything with one generic proposal. The better shape is several tight scanners, each one technology + optional industry (`React + PropTech`, `React + Healthcare`, `React only`), ordered by the pyramid rule above. If the user asks for something broad, propose splitting it and explain why.

## Query syntax

**`query.q` is an Elasticsearch `simple_query_string`, with the default operator set to AND.** It is a query language, not a search box. Never pass the user's sentence through as-is: write the query yourself, in this syntax, every time.

Reference: https://www.elastic.co/docs/reference/query-languages/query-dsl/query-dsl-simple-query-string-query

| Pattern | Meaning |
| --- | --- |
| `react` | one term |
| `"react native"` | exact phrase — always quote multi-word terms |
| `react + typescript` | AND |
| `react \| vue` | OR |
| `react + -wordpress` | AND NOT |
| `(react \| vue) + senior` | grouping |
| `react*` | prefix wildcard — matches react, reactjs, reacts |
| `"react developer"~3` | phrase with slop (words up to 3 apart) |

**The AND default is the trap.** Because unquoted words are AND-ed, `power bi dashboard developer` means *all four tokens must appear* — it does not error, it just quietly matches a fraction of what the user wanted:

| Query | Matches/month |
| --- | --- |
| `power bi dashboard developer` | 24 |
| `"power bi" (dashboard \| developer)` | 103 |

Nothing tells you this happened. A bare multi-word `q` is a bug even when it returns results.

Craft notes that matter:

- **Quote every multi-word phrase.** `machine learning engineer` demands all three tokens anywhere in the posting — write `"machine learning engineer"`, or break it into groups.
- **Reach for the wildcard before enumerating.** `react*` beats `(react | reactjs | "react.js" | "react js")` — shorter, and it matched *more* jobs (1,710 vs 1,685) with about 6 noise hits. Enumerate only genuinely distinct phrasings a prefix cannot reach: `("front end" | "front-end" | frontend)`. Hyphenated terms tokenise oddly, so include the spaced, joined, and hyphenated forms.
- **The usual shape is an AND of ORs**: one group per concept, alternatives inside each group — `(react* | vue*) + (developer | engineer*) + -wordpress`.
- **Prefer `q` with operators** over the `anyKeywords` / `excludedKeywords` fields for anything non-trivial; those are plain comma lists with no grouping.
- **`onlySearchOnTitle`** is the sharpest fix for a keyword that only shows up in passing in the body — narrower than piling on exclusions.
- Every `"` must be closed — unbalanced quotes are rejected before saving. Use straight ASCII quotes, never curly `“ ”`.

**Compare shapes before you save.** Since a too-narrow query looks identical to a good one, run `preview_scanner_matches` on two or three candidate shapes, show the user the counts side by side, and save the one that wins. That is the only way the AND trap becomes visible.

A single scanner cannot cleanly express "hourly ≥ $70 OR fixed ≥ $4000" as one rule. If you set both floors and are unsure how they combine, check with `ask_gigradar` before promising behaviour — or propose two scanners, one per budget type, which is unambiguous.

## Editing a scanner

`get_scanner` before `update_scanner`, every time. Query updates MERGE field by field — editing blind can contradict a filter that is already there (adding a country filter when `notInCountry` already excludes it) and silently match nothing. Send only the field(s) the user asked to change.

After an edit that changes matching, preview or `search_gigs` the result and confirm the jobs still look right before telling the user it is done — the same "look at the actual jobs" standard as a create.

## Diagnosing a scanner that finds nothing

In the order worth checking:

1. `get_scanner` and read the WHOLE query, not just `q`.
2. `preview_scanner_matches` with the same query — confirm it really is zero.
3. Remove filters one at a time and re-preview. The usual culprits, most common first: an over-broad exclusion, a rate floor (which also drops jobs with no stated rate), a country filter, an over-specific phrase.
4. If the query previews healthy but no jobs arrive, it is not a query problem — check subscription and connect balance with `ask_gigradar`.

## Diagnosing low reply rate — check order first

When a specialized scanner "used to do well" and now gets few replies, check the **order** before touching the query. If a broader scanner sits above it and they share a freelancer profile, the broad one is bidding on those jobs first with its generic proposal — the niche scanner never gets to send its tailored one. That is the pyramid inversion; fix it with `reorder_scanner` (after confirming the shared-profile caveat) rather than rewriting a query that was fine.

## Diagnosing a job that should not have matched

When the user points at a specific irrelevant job: read the actual query with `get_scanner` first (do not reason from the scanner's name), and pull the real posting via `search_gigs` / `get_opportunity` so you know which keyword it actually hit. The most common and most missed cause is a grouping bug — a bracket left one term OR'd at the top level, so a job matching just that one term comes through. Fix the grouping and/or add a targeted exclusion drawn from the real bad job. Make one `update_scanner`, then preview to confirm the noise is gone and the wanted jobs remain.

## Teams

One login can reach several teams. `list_teams` shows them, `switch_team` changes the active one, and the switch persists across conversations.

Announce every switch. A scanner created on the wrong team can start sending real proposals from the wrong profile.

## Asking GigRadar itself

`ask_gigradar` starts a conversation with GigRadar's built-in assistant. It can use the full history of that conversation and work through a thoughtful answer, so it runs asynchronously rather than making you wait in one tool call.

Use it instead of guessing. GigRadar ships weekly, so training data goes stale — anything about how a feature behaves, why something happened, what a term means, or the account's own setup should come from here.

**Required async sequence**

1. Call `ask_gigradar` with the question. It returns a `threadId`, `status: "running"`, and `pollAfterMs`.
2. Wait about `pollAfterMs`, then call `get_gigradar_answer` with that `threadId`.
3. If it still says `running`, wait and poll again. Do not tell the user it failed just because the first poll is still running.
4. Once status is `done`, relay the answer. If status is `failed`, say it could not complete the answer — do not invent one.
5. For a follow-up, call `ask_gigradar` again with the SAME `threadId`. The assistant keeps the conversation history, so the follow-up can be brief.

Wait for `done` or `failed` before asking the next question on that conversation. A follow-up sent while the previous one is still running is refused with "that conversation is still answering" — poll instead of retrying it. A question that needs the account looked at can take a couple of minutes; that is the assistant working, not a stall.

Never share a `threadId` between users or teams. The server enforces this too, but a conversation is private to the account that started it.

Good: "How does the autobidder choose which jobs to bid on?", "Why would a scanner be auto-disabled?", "How are connects consumed?"

If it says it has no answer, tell the user that. Do not fill the gap yourself.

## Proposals, CRM, webhooks, and destructive actions

The MCP can now create/schedule applications, manage CRM work, configure webhooks, and queue Upwork-visible messages, meetings, or participants.

**Explicit confirmation is mandatory immediately before any external or destructive action.** Present the exact active team, target job or client/room, sending account, final content/attachments, cost or schedule/timezone, and what happens next. A general earlier "yes" never authorizes a new send, application, schedule, webhook change, deletion, or meeting action.

- `create_application` and `schedule_application` can spend credits/connects or lead to a real proposal. Confirm the exact job, freelancer profile, final cover letter/answers, bid, and timing.
- CRM sends, meeting proposals, participant additions, and scheduled-message actions are visible on Upwork. Say **accepted/pending delivery** when GigRadar queues work; never claim it has already sent.
- `delete_scanner`, `delete_crm_message`, `delete_crm_files`, and `delete_crm_scheduled_message` take effect immediately or cancel future delivery. Confirm first and name the exact object.
- Webhooks send account events to an external URL. Show the destination and scopes; never invent an endpoint or repeat a password returned nowhere by the tools.

CRM file bytes never travel through the tool call: get a signed upload URL, upload directly, then reference its file id.

## Destructive actions

`delete_scanner` takes effect immediately. Confirm first, and name the scanner you are about to delete — never infer it from a vague "clean up my scanners".

## Coming soon

More tools are landing over the next few days. If a user asks for one of these, tell them it is on the way and to check back in a few days — do not improvise a workaround or claim it cannot be done:

- **Auto-bidding configuration and settings** — turning autobidding on or off, and tuning how it bids, directly from here. Today this stays a manual step in the GigRadar dashboard: a scanner is always created with autobidding OFF, and only the user can enable it in the dashboard. That is deliberate, not a limitation to route around.
- **Scanner statistics** — per-scanner performance (matches, bids sent, reply and win rates over time), so you can diagnose a scanner from its numbers instead of inferring from the query alone.
- **Proposal history** — the record of proposals already sent (which job, which scanner, the outcome), so you can answer "what have we bid on" and spot what is working.

When any of these arrive they will show up as new tools automatically; `gigradar_init` will reflect them. Until then, for questions they would answer, use `ask_gigradar` and be honest that the direct tool is not live yet.

## Reference

- `references/tools.md` — every tool, grouped by domain
- `references/scanner-filters.md` — the full scanner filter list
- `references/troubleshooting.md` — symptom-to-cause table

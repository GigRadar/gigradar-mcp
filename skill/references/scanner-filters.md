# Scanner filters

Every field lives under `query`. All are optional. Omitting a filter means "do not filter on this" — it never means "exclude".

## Search terms

| Field | Notes |
| --- | --- |
| `q` | The main query. GigRadar search syntax — see SKILL.md. |
| `anyKeywords` | Comma-separated; any may match. No grouping. |
| `excludedKeywords` | Comma-separated exclusions. No grouping. |
| `onlySearchOnTitle` | Match the title only, ignoring the body. Sharply narrowing. |
| `category` | Upwork category name(s). |

Prefer `q` with operators for anything non-trivial; the keyword lists cannot express grouping or precedence.

## Budget

| Field | Notes |
| --- | --- |
| `minHourlyRate` / `maxHourlyRate` | USD per hour. |
| `minFixedBudget` / `maxFixedBudget` | USD total. |
| `excludeJobsUnspecifiedBudget` | Drop jobs with no stated budget. |
| `includeBudgetPlaceholders` | Include jobs whose budget is a placeholder. |
| `estimateHourlyRate` / `estimateFixedBudget` | Let GigRadar infer a missing budget. |
| `budgetProjectType` | Restrict to hourly or fixed-price. |

A rate floor silently drops jobs that state no rate at all. If a scanner starves, this is the first filter to relax — and the reason is usually not that the rates are too low.

## Client quality

| Field | Notes |
| --- | --- |
| `paymentVerified` | Only clients with a verified payment method. |
| `minRating` | 0–5. |
| `noFeedbackClients` | Include clients with no feedback history. |
| `totalSpent` / `totalSpentMax` | Lifetime client spend, USD. |
| `avgHourlyRatePaidMin` / `avgHourlyRatePaidMax` | What this client has historically paid. |
| `includeClientsWithoutRateHistory` | Keep clients with no rate history. |
| `clientHireRate` | Share of postings that result in a hire. |
| `clientIndustry` | Client industry. |
| `enterpriseClients` | Enterprise accounts. |
| `companySize` | Client company size. |

Stacking several quality filters is the most common way to build a scanner that looks reasonable and matches almost nothing. Add them one at a time and re-preview.

## Location

| Field | Notes |
| --- | --- |
| `country` | Client country name(s) to include. |
| `notInCountry` | Client country name(s) to exclude. |
| `freelancerLocations` | Locations the job is open to. |
| `freelancerLocationMandatory` | Only jobs that require those locations. |
| `freeLancerLanguage` | Required freelancer language. |

Countries are NAMES ("United States"), not codes.

## Job shape

| Field | Notes |
| --- | --- |
| `experienceLevel` | 1 entry, 2 intermediate, 3 expert. |
| `workload` | Full-time or part-time. |
| `duration` | Expected project length. |
| `talentPreference` | Upwork's freelancer-type preference. |
| `descriptionSize` | Job description length bucket. |
| `jobPostingQuestions` | Whether the posting asks screening questions. |
| `connectsPriceMin` / `connectsPriceMax` | Connects Upwork charges to apply. |
| `ignoreSkills` | Ignore the skills taxonomy when matching. |
| `time` | Posting-time window. |

## Consistency

Any `min`/`max` pair must satisfy min ≤ max, or the save is rejected. `q` must have balanced quotes.

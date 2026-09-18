---
name: trackiq-amazon-share-of-shelf
description: Measures how much of Amazon page one a brand owns for the keywords that carry its revenue — organic slots, sponsored slots and Amazon's Choice badges per keyword, rolled up by product category, set against the competitor brands holding the rest of the shelf, with week-over-week movement and the revenue sitting behind every keyword the brand is absent from. Use when the user asks for share of shelf, share of search, page-one visibility, SERP tracking, who owns the first page, competitor shelf analysis, keyword visibility, or where a brand is losing search real estate.
---

# TrackIQ: Amazon Share of Shelf

Answers one question a week: **for the keywords that actually make this
brand money, how much of page one does it own, and who has the rest?**

Output is a branded HTML dashboard — a keyword table weighted by revenue, a
category rollup, the competitor board, and the list of keywords the brand is
absent from with the revenue behind them.

Live Amazon search results are the only source that shows the shelf as a
shopper sees it. Rank tracking tells you where one ASIN sits; this tells you
how much of the page the brand occupies against everyone else on it.

## Requires

- The **Oxylabs scraper**, for `search_keyword` (1 credit per keyword per
  run) and `get_product` (1 credit per ASIN, for brand resolution).
- The **TrackIQ MCP**, for `list_marketplaces` and `get_product_performance`
  — the brand's own ASIN list, which is what makes own placements exact.
- A **keyword basket**, ideally the output of
  `trackiq-category-priority-keywords` so every row carries a revenue figure.
- **A stored prior run** for week-over-week. Without one the comparison
  column reads "first run".
- **Without the scraper:** there is no report. Live SERP data cannot be
  substituted from first-party sources — say so rather than approximating it
  from rank tracking.

## First run

Fill in a copy of `assets/account.example.md` saved as account.md beside the
skill. Every TrackIQ skill reads the same file.

1. **Brand and marketplace** — which account, and the brand name to match on
   the shelf
2. **Keyword basket** — ideally the output of the priority-keywords skill, so
   every row carries a revenue figure
3. **Delivery** — in-chat, file, Slack, n8n or email

Tell the user the credit cost before the first run on a new brand: one credit
per keyword per run, plus a one-off resolution pass of one credit per ASIN.

## Read first

- `assets/pulls.md` — the call sequence, the credit model, and the caching
  design that keeps a weekly run cheap
- `assets/method.md` — the shelf definition, brand attribution, the metrics,
  and what this cannot measure
- `assets/checks.md` — data, attribution, honesty and render checks
- `assets/account.example.md` — the first-run answers, filled in once

Copy `assets/dashboard-template.html` and replace every `{{TOKEN}}`,
repeating the row patterns. Do not restyle it — it shares a design system
with the other TrackIQ reports.

## Non-negotiables

1. **Match the brand's own placements by ASIN, never by title.** Measured on
   a live 6-keyword basket: the brand held 18 placements and only 5 carried
   the brand name in the title. Title matching would have reported a third of
   the real shelf. Pull the ASIN list from `get_product_performance` first;
   if that fails, stop rather than falling back.
2. **Unresolved competitors are shown as `Unresolved`, never guessed and
   never hidden.** Amazon returns no brand on search results. Own ASINs are
   exact, titles resolve about half the rest, and `get_product` resolves the
   remainder at a credit each. A board that admits its gap is useful; one
   that quietly folds unknowns into "other" is not.
3. **Fix the shelf depth at the top 20 organic slots and never change it.**
   The scrape returns ~48. If the denominator moves between runs the trend
   is fiction, and the trend is the entire point.
4. **Organic and sponsored are counted separately and never summed.** One is
   earned, the other is bought. A blended number tells a client their SEO
   worked when they bought the slot.
5. **Never claim pixel share, screen share or above-the-fold position.**
   Sponsored results arrive as their own sequence, not interleaved, so their
   page position is unknown. Count slots out of slots.
6. **A single scrape is a sample, not an average.** Amazon personalises and
   rotates. Only report a change of two or more slots, or a keyword crossing
   between present and absent. Never imply a trend from one run.
7. **Sort by revenue, not volume.** The basket's job is to put the expensive
   gap first. If the basket carries no revenue figures, say so and sort by
   volume — never invent a revenue number to fill the column.
8. **State the credit cost in the output,** and get the user's approval
   before the one-off brand-resolution pass on a new brand. Budget cannot be
   checked programmatically: `get_account_usage` reports only this process's
   spend, and Oxylabs exposes no balance endpoint.
9. **An empty scrape is a failed call, not a zero.** Retry it; if it still
   fails, mark the keyword "not scraped" rather than reporting the brand as
   absent.
10. **Store the run record every time.** Per keyword: slots held, best rank,
    sponsored count, badge. It is the only part of this skill that compounds,
    and next week's report is worthless without it.
11. **Never print `account_id`.**

## Delivery

The dashboard is produced in the chat first. Delivery is the last step and the
method comes from the Delivery block in account.md — never ask per run.

| Method | What to do | Needs |
|---|---|---|
| `in-chat` | Return the HTML. The default, and the fallback for every other method. | nothing |
| `file` | Write it beside the skill as `share-of-shelf-<YYYY-MM-DD>.html`. | a filesystem |
| `slack` | Post the movement and the biggest absent keyword as text, then upload the HTML. | a connected Slack tool |
| `n8n` | POST the HTML to the configured webhook, `Content-Type: text/html`. | network access |
| `email` | Hand it to the connected mail tool. | a connected mail tool |

Confirm before the first outward send of a session, fall back to in-chat
loudly when a method is unavailable, and never substitute a different
outward channel. The rendered dashboard names competitor brands, because
that is the report's subject — it names no tooling vendor.

## What it pairs with

`trackiq-category-priority-keywords` chooses the basket and prices each term;
this skill measures the shelf for it. Run that one quarterly and this one
weekly. `rank-readiness` explains why a listing does or does not deserve the
slot — this one only shows whether the brand holds it.

## Version

`trackiq-amazon-share-of-shelf` v1.0.0 (2026-09-18).

If the user asks whether this skill is current, fetch
`https://trackiq.com/skills/registry.json`, compare the `version` field for
`trackiq-amazon-share-of-shelf`, and if it is newer, give them the download link and
the one-line changelog. Do not fetch at any other time.

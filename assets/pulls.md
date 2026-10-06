# The pull sequence

## 0. Account and catalogue

`list_marketplaces` to identify the account — more than one TrackIQ MCP can
be connected with identical tool names. Then `get_product_performance`
(`group_by='product'`, the trailing month, `limit=200`) for **the brand's own
ASIN list**. That list is what makes own placements exact; without it this
report is guesswork. Never print `account_id`.

## 1. The keyword basket

The basket is the input, and a bad basket makes every number meaningless.
In order of preference:

1. **The priority-keyword list** from `trackiq-category-priority-keywords` —
   already filtered for relevance, grouped by category, and carrying each
   term's monthly revenue. This is the intended pairing: the shelf gets
   weighted by what each keyword is worth rather than by volume alone.
2. A list the client supplies.
3. Built here from `get_search_query_performance` — but then apply that
   skill's relevance gate, or the basket fills with "fresh produce".

Keep it to **20–30 keywords per brand**. Every keyword is a credit a week
and a row nobody reads past thirty.

## 2. The scrape

One call per keyword, per run:

```
search_keyword(keyword=<term>, country='us', max_pages=1)
```

**One page costs one credit.** A 25-keyword basket is 25 credits a week.
`max_pages` above 1 buys depth nobody ranks for — page one is the shelf.

What comes back, per keyword:

| Field | Note |
|---|---|
| `organic` | ~48 items, each with `organic_rank` starting at 1 |
| `sponsored` | 5–8 items, ranked 1..n **in their own sequence** |
| `amazons_choices` | the badge holder, if any |
| `refinements.brands` | **Amazon's own brand filter — the competitor set, free** |
| `credits_consumed_this_call` | always check it |

Each result carries `asin`, `title`, `price`, `rating`, `reviews_count`,
`organic_rank`, `is_amazons_choice`, `best_seller`, `is_prime` — and **no
brand**.

Results run to ~48 organic, which is more than page one. Fix the shelf depth
at the **top 20 organic slots** and hold it constant across runs, or the
share moves because the denominator moved.

## 3. Brand resolution

Three passes, cheapest first. See `assets/method.md` for why the order
matters.

1. **Own ASINs** — exact match against the catalogue from step 0. Free.
2. **Title contains a brand** from `refinements.brands`. Free, and resolves
   roughly half of the rest.
3. **`get_product(asin)`** returns a real `brand` field. **One credit per
   ASIN.** Only for what passes 1 and 2 leave unresolved.

**Cache pass 3 forever.** ASINs recur week to week, so the first run carries
the cost and later runs pay only for new entrants. Measured on a 6-keyword
basket: 92 distinct ASINs in the top 20, 46 resolved free, 46 needing a
lookup. Scale that to 25 keywords and budget roughly 150 lookups once, then
a handful a week.

## 4. Credits

`get_account_usage` reports **only what this process has spent since boot** —
Oxylabs exposes no balance endpoint. The account total lives at
`dashboard.oxylabs.io` and cannot be checked from here. So:

- State the run's credit cost in the output.
- Before a first run on a new brand, tell the user the one-off resolution
  cost in credits and let them approve it.
- Never loop `get_product` over an unbounded ASIN list.

## 5. The prior week

Share of shelf is worthless as a single number and valuable as a trend.
Store each run as a small record per (keyword, run date): slots held, best
rank, sponsored count, badge. On the next run, diff against the most recent
stored run.

With no filesystem, ask the user for the previous run's file or say plainly
that the week-over-week column is unavailable. **Never imply a trend from a
single scrape.**

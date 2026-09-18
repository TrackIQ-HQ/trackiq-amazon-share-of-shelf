# Method

## 1. Define the shelf once, then never move it

**The shelf is the top 20 organic slots plus every sponsored slot returned.**

The scrape returns ~48 organic results, well past page one. Twenty is a
defensible page-one proxy and, more importantly, it is a fixed denominator.
If the depth changes between runs the share moves for a reason that has
nothing to do with the brand, and the trend — the only thing this report is
actually for — becomes fiction.

Sponsored slots are counted separately and never mixed into the organic
share. They are bought; organic is earned, and a client reading one number
that blends them will draw the wrong conclusion about what their SEO work
achieved.

## 2. Brand attribution — own by ASIN, never by title

**This is the rule that makes the report right or wrong.**

Amazon returns no brand field on search results. The obvious shortcut is to
look for the brand name in the title. Measured on a real 6-keyword basket:
the brand held **18 organic and sponsored placements**, and only **5 of them
had the brand name in the title**. Title matching would have reported
roughly a third of the brand's actual shelf.

The reason is ordinary: strong listings lead with the product, not the
label — "Outdoor String Lights 48ft Weatherproof, Shatterproof LED Bulbs" is
one of the brand's own bestsellers and never says who makes it.

So:

- **Own placements: match ASIN against the catalogue.** Exact, free, and the
  only acceptable method.
- **Competitors: title match against `refinements.brands`,** then
  `get_product` for what remains, cached.
- **Anything still unresolved is labelled `Unresolved` and shown as its own
  row.** Never fold it into "other", never guess from the title, never omit
  it — an incomplete competitor board that admits it is useful; one that
  hides the gap is not.

## 3. The metrics

Per keyword:

```
organic_share  = own slots in top 20 / 20
best_rank      = lowest organic_rank the brand holds (null if absent)
sponsored_held = own slots / sponsored slots returned
badge          = does the brand hold Amazon's Choice
```

Per category, roll up slots held over slots available — **sum the slots, do
not average the per-keyword percentages.** Averaging gives a one-result
keyword the same weight as the category's head term.

## 4. Weight by revenue, not by volume

A basket sorted by search volume puts the loudest term first. A basket
sorted by the revenue each term generates puts the *expensive* gap first,
which is the one worth acting on.

When the basket comes from `trackiq-category-priority-keywords`, each term
already carries an estimated monthly revenue. Carry it through into every
table and sort by it. Then the headline metric becomes the one that matters:

```
revenue_with_no_organic_slot = sum of monthly revenue over keywords
                               where the brand holds zero top-20 slots
```

That figure is the report's argument. On the prototype basket it was $4,058
a month sitting behind a single term the brand was entirely absent from.

If the basket has no revenue attached, say so and sort by volume instead —
do not invent a revenue figure to fill the column.

## 5. Week over week

Diff against the most recent stored run:

- **slots lost** — flag any keyword where the count dropped, loudest first
- **slots gained**
- **rank moves** — best_rank worsening by more than three positions
- **new entrants** — brands on the shelf this week that were not there last
- **competitor buying the brand's own name** — a competitor holding a
  sponsored slot on a branded query

Two disciplines:

1. **A single scrape is a sample, not an average.** Amazon personalises and
   rotates. A one-slot move is noise. Only call a change when it is two or
   more slots, or when a keyword crosses between present and absent.
2. **First run has no comparison.** The column reads "first run" and the
   report says so. Never imply a trend from one observation.

## 6. What this cannot measure

- **Pixel share or above-the-fold position.** Sponsored results come back as
  their own sequence, not interleaved, so their position on the page is
  unknown. Count slots; never claim share of screen.
- **Personalised results.** One ZIP, one moment, no purchase history.
- **Why a slot moved.** The report shows the shelf changed, not the cause.
  Pair it with rank tracking and listing changes for that.

# Before you send it

## 1. The data checks

- **Own ASINs came from the catalogue**, not from reading titles. If the
  catalogue pull failed, stop — do not fall back to title matching.
- **Shelf depth is the same as last run.** Twenty organic slots, every run.
- **Every keyword returned results.** A scrape that came back empty is a
  failed call, not a zero — retry it, and if it still fails, mark the
  keyword "not scraped" rather than "absent".
- **Slot counts never exceed slots available.** `own <= 20` per keyword.
- **Credits reported.** The run states what it consumed, and a first run on
  a new brand states the one-off brand-resolution cost.
- **The prior run actually loaded.** If it did not, the week-over-week column
  says "first run" — it never silently shows no change.

## 2. The attribution checks

- The competitor board carries an `Unresolved` row whenever anything is
  unresolved, with its real count.
- No brand was inferred from a title that merely mentions it — "string
  lights" in a title is the product, not the brand String Light Co.
- Spot-check three own placements against the catalogue by ASIN.
- If unresolved exceeds about a third of the shelf, either run the lookups
  or drop the competitor board from this run and say why. A board that is
  half unknown is not worth a client's attention.

## 3. The honesty checks

- Nothing claims pixel share, screen share, or above-the-fold position.
- Organic and sponsored are reported separately and never summed into one
  "share of shelf" number.
- The single-scrape caveat appears in the method section.
- No trend is implied from one run.
- Moves of one slot are not written up as changes.

## 4. The render check

Open the HTML and run:

```js
({ overflows: document.documentElement.scrollWidth > window.innerWidth,
   tables: document.querySelectorAll('table').length,
   rows: [...document.querySelectorAll('table')].map(t => t.querySelectorAll('tbody tr').length),
   logos: [...document.images].map(i => i.naturalWidth > 0),
   unresolved: document.body.innerText.includes('Unresolved') })
```

- `overflows` false, `logos` all true.
- `rows` matches the keyword and category counts you computed.
- `unresolved` true whenever the board has unresolved slots.

Then look at it. If the page will not paint — a hidden or backgrounded
window — say the check was structural, not visual. Do not claim to have
inspected what you did not see.

## 5. Ship

Save as `<client>-share-of-shelf-<YYYY-MM-DD>.html`, dated by scrape day
rather than by week number so two runs never collide.

**Store the run record** alongside it: per keyword, the slots held, best
rank, sponsored count and badge. Next week's report depends on it, and it is
the only part of this skill that compounds.

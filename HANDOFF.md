# 4for4 IDP Position vs Team — HANDOFF

One static page: **IDP Position vs Team** — what each NFL offense gives up per game to
each defensive position group (LB / S / CB / DT / DE): tackles + assists and sacks, per
game or per 100 plays, each group's share of the team's tackles, an index vs the league
average and a rank. 2025 regular season is the base, the current season is layered in
(Blend / 2025 / 2026 toggles), and both seasons' values are printed under every headline
number. Same publishing pattern as `4for4-snap-tools`: this repo is the record, `site/`
is what Cloudflare Pages serves, a push republishes, one iframe on 4for4.com shows it.

## Where the page comes from

This repo does **not** render the page. It is built by the internal projections pipeline
(`projects4for4-idp/idp_dvp/build_idp_dvp.py`) from nflverse play-by-play-derived weekly
stat lines plus raw pbp play counts, with position groups classified by the IDP Truth
workbook (OLB = DE in this model). That pipeline is not portable, so the rendered page
is committed here as-is.

```
site/index.html    the whole page, self-contained (fonts embedded, no external requests)
site/robots.txt    noindex — the page is meant to be seen inside 4for4.com, not found by search
data/              the per-week CSV behind the page (blended per-game values + index table)
```

## Weekly update

From the `projects4for4-idp` repo, after the week's data is in (Tuesday, once nflverse has
posted the Monday game):

```
python idp_dvp/build_idp_dvp.py --week N --site
cp idp_dvp/idp_dvp_2026_wkN.csv idp_dvp/idp_dvp_current.csv <this repo>/data/
git add site/ data/ && git commit -m "week N" && git push     # Cloudflare republishes in ~1 min
```

`week_turnover.py` runs the build; the copy + push is the one manual step, on purpose
(no scheduled job — the data only moves when the pipeline runs).

## Embedding

```html
<iframe src="https://<project>.pages.dev/" style="width:100%;border:0" scrolling="no"></iframe>
```

The page posts `{type:'tool-height', height}` to its parent on load/resize, the same
message the snap tools use, so the 4for4 side can size the iframe and never scroll it
internally.

## Reading it

- Rows are the offense FACED. Green = that offense allows more than the league average to
  that group, red = less; index = team ÷ league average.
- The share columns (blue = above-average slice, amber = below) show how a team's tackles
  split across groups regardless of volume.
- Per-100-plays separates pace from the actual matchup signal.
- It is noisy in the middle and useful at the tails. Nothing in the projection model reads
  it; it is a judgement view.

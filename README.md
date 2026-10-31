# Daily Ranking Static

A single-file, dependency-free HTML dashboard of daily app-store rankings for Appier BDs.
Each run covers **9 markets × 2 charts = 18 sheets** and roughly **4,600 ranked rows**.

**Live:**
- https://tengyuan-chang-appier.github.io/daily-ranking-static/ (GitHub Pages, `main` → `/docs`)
- Render static site `daily-ranking-top-app-ad-cloud` (see `render.yaml`)

**Current data snapshot:** 2026-08-23 (shown in the page `<h1>`).

## Markets and charts

| Market | Charts |
| --- | --- |
| KR, US, DE, GB, TR, JP, TW, HK, RU | `Overall_<MARKET>`, `Games_<MARKET>` |

`Overall` is the top-free-style overall chart for the market; `Games` aggregates the games
category charts, so it can be larger than `Overall` in some markets (e.g. US, RU).

## Repo layout

```
docs/index.html                          # the dashboard (served at /)
docs/top_apps_combined_2026-08-23.html   # dated snapshot copy, byte-identical to index.html
docs/.nojekyll                           # keeps GitHub Pages from running Jekyll
render.yaml                              # Render static-site blueprint (publishes ./docs)
```

There is **no build step, no package manager, no CI**. `index.html` is a fully self-contained
artifact: inline CSS, inline data, ~60 lines of vanilla JS.

> `index.html` and the dated file are kept byte-identical. Any edit must be applied to both —
> verify with `md5sum docs/*.html` before committing.

## How the page works

- Each sheet is a `<section class="sheet-panel" data-sheet="<CHART>_<MARKET>">` containing one table.
- JS toggles `.active` on sections; the tab bar (desktop) and `<select id="sheet-select">` (mobile)
  stay in sync.
- Two filters run entirely client-side, reading row attributes:
  - **Market segment** — `data-market="apple-store" | "google-play"`.
  - **Company HQ** — options are built from the active sheet's `data-hq` / `data-hq-group` values,
    plus pseudo-values `highlight-blue`, `highlight-green`, `highlight-red`.
- Every `<tr>` carries the filter state: `data-market`, `data-hq`, `data-hq-group`,
  `data-highlight` (`blue`/`red`/`none`), `data-emea`, `data-cash-cow` (`purple`/`yellow`/`grey`/`none`).

### Columns

Rank · Rank Change (`+n` / `-n` / `=` / `NEW`) · Market · App Name · Company Name ·
Country/Region · Company HQ · App Category Name · MMP · Daily downloads · DAU · DSPs.

### Colour coding

| Layer | Meaning |
| --- | --- |
| Row background blue | HQ matches the sheet's campaign region |
| Row background green (`tr.emea-hq`) | EMEA HQ |
| Row background red | CN / HK / SG HQ |
| App-name chip purple | Cash Cow match, priority 1–6, CRM status not Running/Pitching |
| App-name chip yellow | Cash Cow match, priority 1–6, CRM status Running or Pitching |
| App-name chip grey | Cash Cow match outside priority 1–6 (ignore) |

## Data caveats

- **Daily downloads are market-level** (one country, one chart). **DAU is a global app-level figure**
  from the data source and is not scoped to the market in that row, so the two columns are not
  directly comparable. This is stated in the page header; keep it if you regenerate the page.
- `NA` means "missing", not zero — it is not the same as "no data provider / no DSP".
- Title collisions are real, not bugs: the same app name can appear twice with different publishers.
- Column coverage in the 2026-08-23 snapshot: MMP ~68%, DAU ~67%, DSPs ~55%, Cash Cow match ~52%.

## Updating the data

1. Regenerate the dashboard HTML from the upstream export, keeping the structure above
   (18 `sheet-panel` sections, the row `data-*` attributes, and the `<caption class="sr-only">` /
   `<th scope="col">` markup).
2. Update the date in the `<h1>` and the `18 tabs · N rows` line in the header.
3. Update the canonical / Open Graph / Twitter URLs if the hosting path changes — they point at the
   GitHub Pages URL.
4. Write the new file to both `docs/index.html` and `docs/top_apps_combined_<YYYY-MM-DD>.html`.
5. Commit to `main`; Pages redeploys on push and Render redeploys from the same branch.

## Accessibility / quality notes

- Tables have `<caption class="sr-only">` and `scope="col"` headers; the colour legend is text-based.
- With JavaScript disabled, `<noscript>` CSS reveals all 18 sheets and a notice explains the
  filters are unavailable.
- HQ tiers are additionally encoded in the "Company HQ" text, but the blue/green/red tier itself is
  colour-only — avoid relying on it alone when quoting results.

## Relevant Files

- `Stock Recovery Signal Identifier_v_10.6.html` - Main single-file app containing chart rendering, backtest metrics, UI, and CSV export handlers.

### Notes

- Keep layout stable: existing KPIs remain in place; add a second KPI row.
- Recompute on pan/zoom and on sequential toggle changes.
- Respect relative-to-index mode for visible range return.
- Append a summary row to the equity CSV export with the specified fields.

## Tasks

- [ ] 1.0 Add second KPI row in Backtest card with four metrics and tooltips
  - [x] 1.1 Add four KPI blocks beneath existing grid in Backtest card with ids: `btInvestedDays`, `btInvReturnPerDay`, `btRangeReturn`, `btRangeReturnPerDay` and a muted span `btVisibleDates` adjacent to Range return.
  - [x] 1.2 Use existing compact styles and add concise tooltips for each metric
  - [ ] 1.2 Use existing compact styles (`.grid4`, `.muted`) to match typography/spacing; ensure responsive behavior stays consistent.
  - [ ] 1.3 Add short tooltips (title attributes or small help icons) explaining each metric per PRD language.

- [ ] 2.0 Wire metrics to main chart visible window and sequential toggle updates
  - [x] 2.1 Implement `getVisibleDateRange()` that maps `chart.visibleRange()` indices → `[Vs, Ve]` dates from `STATE.rows`, guarding empty/short windows.
  - [x] 2.2 Implement `updateVisibleRangeKPIs()` and call it from `chart.onrender` and `#sequentialTrades` change
  - [x] 2.3 Ensure idempotency: cache last-signature to avoid redundant recomputation on re-render

- [x] 3.0 Implement Invested days (calendar, inclusive) over visible window
  - [x] 3.1 Get trades: `const trades = sequentialOn ? sequentialize(STATE.signals) : STATE.signals;` using `triggerDate/endDate` bounds.
  - [x] 3.2 Intersect each `[triggerDate, endDate]` with `[Vs, Ve]`; sum inclusive calendar days via dense calendar iteration.
  - [x] 3.3 Render integer result in `#btInvestedDays` (show `0` when none).

- [x] 4.0 Implement Return/day (invested) as geometric mean over invested calendar days
  - [x] 4.1 Build dense calendar `[Vs..Ve]`; forward-fill equity from `STATE.equity` into a calendar-daily series (carry last known value on non-trading days).
  - [x] 4.2 Build invested-day Set from intersected trades (inclusive). Skip first visible day for return calc.
  - [x] 4.3 Compute `g_inv = (∏(1+r[d]))^(1/N)−1` over invested days; handle `N==0` → `—`.
  - [x] 4.4 Render with 3 decimals and green/red coloring based on sign.

- [x] 5.0 Implement Range return (visible) with absolute/relative modes per toggle
  - [x] 5.1 Add `priceOnOrBefore(rows, date)` helper choosing nearest prior point (inclusive).
  - [x] 5.2 Compute `R_stock = P[Ve]/P[Vs]−1`; when relative mode is ON, compute `R_bench` on index rows and return `(1+R_stock)/(1+R_bench)−1`.
  - [x] 5.3 Render colored percentage in `#btRangeReturn` and show `Vs → Ve` in muted `#btVisibleDates`.

- [x] 6.0 Implement Return/day (visible) from visible range total and calendar days
  - [x] 6.1 Compute `D_visible = 1 + (Ve − Vs)` in calendar days.
  - [x] 6.2 Compute `(1+R_vis)^(1/D_visible)−1` with 3-decimal formatting; show `—` if window < 2 points.

- [x] 7.0 Apply value coloring (green/red) and muted visible date range next to Range return
  - [x] 7.1 Reuse existing color helpers or add a `colorPct3(x)` for 3-decimal formatting; apply to numeric values only.
  - [x] 7.2 Keep labels neutral; place dates in `.muted` near Range return block.

- [x] 8.0 Update Export equity CSV to append the summary row with all fields
  - [x] 8.1 Locate/add `#exportEquity` click handler; ensure existing equity export remains intact.
  - [x] 8.2 Recompute the four metrics at export time using the same helpers (and current `[Vs, Ve]`).
  - [x] 8.3 Append a single CSV row with columns: `visible_start_date, visible_end_date, invested_days_visible, return_per_day_invested, range_return_visible, return_per_day_visible`.
  - [x] 8.4 Trigger download; verify rounding matches UI.

- [x] 9.0 Add lightweight caching/guards to recomputations to keep pan/zoom snappy
  - [x] 9.1 Cache the last `[Vs, Ve]` and `sequentialOn` inputs; skip compute when unchanged.
  - [x] 9.2 Memoize dense calendar for `[Vs, Ve]` and map of equity-by-date; reuse across metric calculations.

- [x] 10.0 QA against acceptance criteria and edge cases (short ranges, no trades, missing benchmark)
  - [x] 10.1 Verify layout stability and ordering of KPIs (existing first row unchanged).
  - [x] 10.2 Validate metric values on: (a) no trades, (b) invested days = 0, (c) short windows, (d) missing index, (e) zero/flat equity.
  - [x] 10.3 Check live updates on pan/zoom and sequential toggle; confirm CSV summary aligns with UI at time of export.



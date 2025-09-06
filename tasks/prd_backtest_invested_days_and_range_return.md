# PRD — Backtest “Invested Days” & Visible-Range Return

## 1) Introduction / Overview
Enhance the Backtest header of the existing HTML/JS app to add two practical analytics tied to the **currently visible date range** of the main chart:

1) **Invested days** — the count of calendar days (inclusive) when capital was actually in a position, plus its **per-invested-day return**.
2) **Visible-range stock return** — the stock’s return from the **first visible** date to the **last visible** date (absolute or relative to benchmark depending on the UI toggle), plus its **per-day return** over that full visible range.

These metrics should update live on pan/zoom and when the “Sequential (no overlaps)” toggle changes, without disturbing the current layout where **Total return** stays top-left and **Trades/Signals** stays top-right.

## 2) Goals
- Provide an at-a-glance measure of how long capital was employed and how efficiently it worked on a daily basis.
- Provide a quick read of the underlying stock’s performance over the visible range (matching what the user is looking at), including a per-day rate.
- Keep the summary header compact, readable, and consistent with the current design; green when non-negative, red when negative.
- Include these values in the **Export equity CSV** for downstream analysis.

## 3) User Stories
- **As a trader**, I want to know **how many days I was actually in the market** within the current view so I can compare strategy efficiency across symbols/ranges.
- **As a trader**, I want to see the **average daily gain/loss while invested** so I can benchmark trade quality irrespective of time out of market.
- **As an analyst**, I want the **stock’s visible-range return** and its **per-day rate** to contextualize strategy results versus simply buying-and-holding over the same window.
- **As a user**, I want these metrics to **follow the visible chart window** and the **sequential (no overlaps) setting** so they always reflect what I’m seeing.

## 4) Functional Requirements

### 4.1 Data Scope & Toggles
1. **Visible window only**: All new metrics are computed using the main chart’s **visible start date (Vₛ)** and **visible end date (Vₑ)**.
2. **Sequentialization**: When the **“Sequential (no overlaps)”** toggle is ON, invested-day calculations must use the **post-sequential trade list** (no overlaps). When OFF, use the raw trades/signals as the app currently does.
3. **Live updates**: Recompute and re-render on each pan/zoom and on toggle change.

### 4.2 Metrics to Add (labels approved)
Left → Right on the second summary row:
- **Invested days**
- **Return/day (invested)**
- **Range return (visible)**
- **Return/day (visible)**

### 4.3 Definitions & Formulas

#### 4.3.1 Invested days (calendar, inclusive)
- For each trade **T = [entry_date, exit_date]** after applying sequentialization, compute the intersection with the visible window **I = [max(entry, Vₛ), min(exit, Vₑ)]**.
- If **I** is empty, contribute 0. Otherwise the day count for that trade is:
  - **days_T = 1 + (I.end − I.start)** measured in **calendar days**.
- **Invested days = Σ days_T**.

#### 4.3.2 Return/day (invested)
- Build an **in-position calendar-day mask** over the visible window: a set of dates **D_invested** containing each calendar day where at least one trade is open (after sequentialization and intersection with the visible window).
- Create a **calendar-daily equity series** **E[d]** over the visible window by forward-filling from the backtest equity (non-trading days carry the last close; i.e., return 0 on weekends/holidays).
- Let **r[d] = E[d] / E[d−1] − 1** for **d ∈ D_invested** (skip the very first visible day).
- If **N = |D_invested\{first}| = invested calendar days with a prior day**, then:
  - **Geometric mean daily return** (time-weighted):
    - **g_inv = (∏(1 + r[d]))^(1/N) − 1**.
- Display **Return/day (invested) = g_inv** formatted as a percentage with **3 decimals** (e.g., `+0.123%`). If **Invested days = 0** or **N = 0**, display `—`.

> Rationale: This produces a per-calendar-day rate that is time-weighted and robust to gaps (weekends/holidays carry 0% for held positions).

#### 4.3.3 Range return (visible)
- Define **stock return over visible window** using the first and last **visible** data points:
  - **R_stock = P[Vₑ] / P[Vₛ] − 1** where **P[d]** is the stock’s close aligned to the chart.
- When **Relative to index** mode is ON, compute **relative return** multiplicatively:
  - Let **R_bench = B[Vₑ] / B[Vₛ] − 1** where **B[d]** is the benchmark close.
  - **Range return (visible)** = **R_rel = (1 + R_stock) / (1 + R_bench) − 1**.
- When the mode is OFF, display **R_stock**.
- Color: green if ≥ 0, red if < 0.

#### 4.3.4 Return/day (visible)
- Let **D_visible = 1 + (Vₑ − Vₛ)** in **calendar days**.
- Let **R_vis** be the value chosen above (absolute or relative based on mode).
- **Return/day (visible) = (1 + R_vis)^(1 / D_visible) − 1**.
- Format to **3 decimals** as a percentage, green if ≥ 0, red otherwise.

### 4.4 Display & Layout
1. Preserve existing header layout: **Total return** remains **upper left**; **Trades/Signals** remains **upper right**.
2. Add a **second row** beneath the current KPIs with four compact KPI blocks, left → right:
   - **Invested days** (integer)
   - **Return/day (invested)** (±X.XXX%)
   - **Range return (visible)** (±XX.XX%)
   - **Return/day (visible)** (±X.XXX%)
3. **Colors**: green for values ≥ 0, red for < 0. (Numbers only; labels remain neutral.)
4. **Visible dates shown**: Append the actual **Vₛ → Vₑ** dates next to **Range return (visible)** in small muted text (e.g., `2024‑03‑01 → 2024‑09‑01`).
5. **Tooltips**: For each metric, show a brief formula/explanation on hover.
6. **Rounding**: Per-day metrics show **3 decimals**; other percentages retain existing app rounding; day counts are integers.
7. **Empty states**:
   - If **Invested days = 0** → show `0` for days and `—` for per-day.
   - If the visible range yields < 2 price points → show `—` for range and per-day.

### 4.5 CSV Export
- Include a one-row **“Summary”** block appended to the exported CSV (below equity rows) with these columns:
  - `visible_start_date`, `visible_end_date`
  - `invested_days_visible`
  - `return_per_day_invested`
  - `range_return_visible` (absolute or relative depending on current toggle)
  - `return_per_day_visible`
- Values use the same rounding as the UI.

## 5) Non-Goals
- No changes to the backtest engine, signal generation, or chart drawing beyond what’s required to compute/display these metrics.
- No portfolio aggregation or multi-symbol summaries.
- No additional statistics (e.g., alpha/beta) beyond the items listed.

## 6) Design Considerations (UI/UX)
- Reuse existing KPI card styling: typography, spacing, and compact cards with clear labels and values.
- Keep the second row height the same as the first to avoid layout jitter.
- Use subtle muted text for the visible date range near **Range return (visible)**.
- Ensure tooltips are consistent with existing tooltip components (delay, placement).

## 7) Technical Considerations

### 7.1 Inputs expected from current app
- **Trades**: array of `{ entryDate, exitDate, side }` (post-sequentialization when toggle is ON).
- **Equity series**: `{ date, equity }` at trading-day frequency.
- **Price series**: `{ date, close }` for the stock; optionally benchmark `{ date, close }` for relative mode.
- **Chart viewport**: callbacks/events to read **Vₛ**/**Vₑ** on pan/zoom.

### 7.2 Calendar-day handling
- Build a **dense calendar** `[Vₛ … Vₑ]` (local or UTC consistent with chart dates).
- **Forward-fill** equity and prices for non-trading days to maintain calendar alignment (yields 0% return on closed markets).
- Use this dense calendar both for **invested mask** and **visible-range** computations.

### 7.3 Algorithms (pseudocode)
```js
function computeVisibleWindow() {
  const { start: Vs, end: Ve } = chart.getVisibleExtents();
  return [floorToDate(Vs), floorToDate(Ve)]; // inclusive dates
}

function sequentializeTradesIfNeeded(trades, sequentialOn) {
  return sequentialOn ? removeOverlaps(trades) : trades;
}

function intersectDays(aStart, aEnd, bStart, bEnd) {
  const s = maxDate(aStart, bStart);
  const e = minDate(aEnd, bEnd);
  return (s <= e) ? [s, e] : null; // inclusive
}

function buildInvestedMask(trades, Vs, Ve) {
  const days = denseCalendar(Vs, Ve); // array of Dates inclusive
  const invested = new Set();
  for (const t of trades) {
    const I = intersectDays(t.entryDate, t.exitDate, Vs, Ve);
    if (!I) continue;
    for (const d of eachDate(I[0], I[1])) invested.add(+d);
  }
  return { days, invested };
}

function calendarEquity(equitySeries, Vs, Ve) {
  const days = denseCalendar(Vs, Ve);
  let idx = 0, last = equitySeries[0].equity;
  const map = new Map(equitySeries.map(r => [+toDate(r.date), r.equity]));
  return days.map(d => {
    const key = +d;
    if (map.has(key)) last = map.get(key);
    return { date: d, equity: last };
  });
}

function gMeanDailyReturnInvested(equityCal, investedSet) {
  let prod = 1, n = 0;
  for (let i = 1; i < equityCal.length; i++) {
    const d = +equityCal[i].date;
    if (!investedSet.has(d)) continue;
    const r = equityCal[i].equity / equityCal[i-1].equity - 1;
    prod *= (1 + r);
    n++;
  }
  return n > 0 ? Math.pow(prod, 1/n) - 1 : null; // null → "—"
}

function visibleRangeReturn(prices, bench, Vs, Ve, relativeMode) {
  const P0 = priceOnOrBefore(prices, Vs); // aligned to visible first
  const P1 = priceOnOrBefore(prices, Ve);
  const Rstock = P1 / P0 - 1;
  if (!relativeMode) return Rstock;
  const B0 = priceOnOrBefore(bench, Vs);
  const B1 = priceOnOrBefore(bench, Ve);
  const Rbench = B1 / B0 - 1;
  return (1 + Rstock) / (1 + Rbench) - 1; // multiplicative relative
}

function perDayFromTotal(R, days) {
  return days > 0 ? Math.pow(1 + R, 1/days) - 1 : null;
}
```

### 7.4 Performance
- All computations are **O(D)** where **D** is number of calendar days in the visible window (usually a few thousand max) — fast enough to recompute on every view change.
- Avoid allocations in hot paths; reuse arrays where possible.

### 7.5 Numeric & Date Details
- Use a consistent timezone (UTC recommended) for date comparisons and calendar generation.
- Guard against `equity == 0` (skip that return).
- When visible range endpoints fall between data points, choose the **nearest prior available** data point (as the chart typically does) for `P[Vₛ]` and `P[Vₑ]`.

## 8) Success Metrics
- Users can read invested days and both per-day rates at a glance without scrolling.
- Values update correctly and instantly as the user pans/zooms or toggles sequentialization.
- CSV exports include the summary row with correct values in ≥ 99% of test cases.

## 9) Acceptance Criteria (QA Checklist)
1. **Layout**: Total return remains top-left; Trades/Signals remains top-right. Four new KPIs appear on the second row in the specified order.
2. **Invested days** equals the count of inclusive calendar days covered by visible, sequentialized trades (partial overlaps counted proportionally by days).
3. **Return/day (invested)** shows `—` when invested days = 0; otherwise shows a 3-decimal percentage. Color is green when ≥ 0, red when < 0.
4. **Range return (visible)** uses absolute return when Relative mode is OFF; multiplicative relative return when ON. Colors applied as above.
5. **Return/day (visible)** is the geometric per-day from the selected range total. 3-decimal percentage with color coding.
6. **Visible dates** rendered next to Range return in muted text.
7. **Tooltips**: Each metric shows a short explanation including its formula.
8. **Live updates**: All values change immediately when the visible window changes or the sequential toggle toggles.
9. **CSV**: Export includes one summary row with the fields listed in §4.5 and uses the same rounding and values as the UI at export time.
10. **Edge cases**: If the visible window has < 2 points for the stock (or benchmark when in relative mode), show `—` for the affected values. If equity series missing for the first visible day, forward-fill from the nearest prior.

## 10) Open Questions
_None — all options confirmed by requester._


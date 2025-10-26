# Product Requirements Document (PRD) for Stock Recovery Signal Identifier Tool

## 1. Overview
### 1.1 Purpose
This PRD defines the requirements for a single, self-contained HTML file (with embedded CSS and JavaScript) that implements a "Stock Recovery Signal Identifier" tool. The tool analyzes stock price data to detect "dip → recovery" and "breakout" signals based on customizable parameters. It visualizes price/relative performance charts, computes signal metrics, performs a simple backtest (1 unit per signal), and displays an equity curve.

The file must use modern JavaScript practices for readability and maintainability, such as modular functions and classes, ES6+ features (e.g., async/await, template literals), and appropriate error handling. No external dependencies are allowed.

The tool fetches data from Börsdata API (requires user-provided API key) and supports relative performance to a benchmark index.

### 1.2 Key Features
- **Instrument Search and Selection**: Search and select stock/index via loaded JSON file.
- **Data Fetching**: Retrieve historical prices from Börsdata API.
- **Signal Detection**: Identify recovery (after dip) and breakout signals with filters (e.g., volume, SMA, cliff exits).
- **Charting**: Interactive canvas-based line chart with zoom/pan, tooltips, HUD, dots for signals.
- **Metrics**: Signal counts, average/median/max/min returns, hit rate.
- **Backtest**: Sequential trades (1 unit per signal), equity curve, metrics (CAGR, drawdown, Sharpe, etc.).
- **Exports**: CSV for signals/equity, PNG for chart.
- **UI Controls**: Parameters for signals, indicators, relative mode, auto-scaling.

### 1.3 Assumptions and Constraints
- The file is a single HTML document with inline `<style>` and `<script>`.
- No external CSS/JS files or libraries.
- Browser compatibility: Modern browsers (Chrome, Firefox) with canvas support and ES6+ features.
- Data sources: Börsdata API for prices; user-uploaded JSON for instruments.
- Dates are handled as JavaScript Date objects; prices as numbers.
- The tool assumes daily data (one bar per trading day).
- Use modern JS for performance and readability: Organize code into logical sections/functions/classes; avoid global pollution where possible.

### 1.4 Version
This specifies version 9.0 with features for panned HUD, relative triggers, real P&L, and backtest.

### 1.5 Data sources and Börsdata API calls and format from Börsdata
When fetching data for a given instrument, we must first get the instrument-id (insid) for the selected instrument. This is done via loading a user selectable local json file (default name is 'nordic_stocks.json')
The format is the json data in this file is:
{
  "instruments": [
    {
      "insId": 2,
      "name": "AAK",
      "urlName": "aarhuskarlshamn",
      "instrument": 0,
      "isin": "SE0011337708",
      "ticker": "AAK",
      "yahoo": "AAK.ST",
      "sectorId": 2,
      "marketId": 1,
      "branchId": 63,
      "countryId": 1,
      "listingDate": "2005-09-29T00:00:00",
      "stockPriceCurrency": "SEK",
      "reportCurrency": "SEK"
    },
    {
      "insId": 3,
      "name": "ABB",
      "urlName": "abb",
      "instrument": 0,
      "isin": "CH0012221716",
      "ticker": "ABB",
      "yahoo": "ABB.ST",
      "sectorId": 5,
      "marketId": 1,
      "branchId": 23,
      "countryId": 1,
      "listingDate": "1999-06-22T00:00:00",
      "stockPriceCurrency": "SEK",
      "reportCurrency": "USD"
    },...

When you have identified the <insid> then we can fetch the price (high, low, close, open, volume) for a given date for this instrument via the Borsdata API the following API needs to be called:
https://apiservice.borsdata.se/v1/instruments/<insid>/stockprices?authKey=<API_key>&from=<date>&maxCount=20
The data returned from such a call is on the following json format:
{
  "instrument": 3,
  "stockPricesList": [
    {
      "d": "2025-01-30",
      "h": 642.6,
      "l": 603.4,
      "c": 606.6,
      "o": 628.4,
      "v": 1483530
    },
    {
      "d": "2025-01-31",
      "h": 614.6,
      "l": 602.8,
      "c": 607.2,
      "o": 608.8,
      "v": 702749
    },...


## 2. Visual Design
### 2.1 Color Palette
Define in `:root` CSS variables using similar hex codes:
- `--bg: #0b0f14` (background, dark blue-gray).
- `--panel: #121821` (panels/cards, darker gray).
- `--ink: #e6eef7` (text, light blue-white).
- `--muted: #9fb3c8` (muted text/labels, medium gray-blue).
- `--accent: #4da3ff` (buttons, blue).
- `--line: #64d2ff` (price line, bright blue).
- `--grid: #223043` (borders/grid lines, dark gray-blue).
- `--red: #ff4d4d` (negative returns, trigger dots, red).
- `--green: #39e29d` (positive returns, exit dots, green).
- `--yellow: #ffd700` (SMA line, yellow).

Additional colors used inline:
- Tooltip background: `rgba(20,26,36,0.97)` (semi-transparent dark).
- HUD background: `rgba(0,0,0,0.5)` (semi-transparent black).
- Crosshair: `rgba(255,255,255,0.25)` (faint white line).
- Dot highlights: `#ffffffcc` (white stroke).

### 2.2 Typography
- Font family: `ui-sans-serif, system-ui, -apple-system, Segoe UI, Roboto, Helvetica, Arial` (system sans-serif).
- Base text color: `--ink`.
- Sizes:
  - Labels: 12px.
  - Buttons: 600 weight (bold).
  - Metrics (large): 18px, 700 weight.
  - Table: 12px.
  - HUD/Tooltip: 12px.
  - Titles: h2 (default size, ~16-18px).
- Muted text: `--muted` color.
- Bold: 700 weight (e.g., summaries, metrics).

### 2.3 Layout
- Full-page grid: `.wrap` as `display: grid; grid-template-columns: 400px 1fr; gap: 16px; height: 100%;`.
  - Left column (aside.left): Fixed 400px width, scrollable (`overflow: auto`), background `--panel`, padding 16px, right border `--grid`.
  - Right column (main.right): Flexible, grid with rows `1fr auto`, gap 8px, padding 8px.
- Cards: `.card` - `--panel` background, 1px `--grid` border, 12px radius, 12px padding.
- Grids:
  - `.grid2`: 2 columns, gap 8px.
  - `.grid3`: 3 columns, gap 8px.
  - `.grid4`: 4 columns, gap 10px.
- Rows: `.row` - flex, gap 8px, align center, wrap.
- Flex: `.flex` - flex, gap 8px, wrap, align center.
- List: `.list` - max-height 160px, scrollable, `--grid` border, 8px radius.
  - Items: Padding 6px 8px, cursor pointer, hover `#0e141d` background, bottom dashed border `#1f2a3a`.
- Tables: Full width, collapse, 1px bottom borders `--grid`, 6px/8px padding, right-aligned except first column left.
- Pills: `.pill` - Inline block, 2px/8px padding, 999px radius (rounded), 1px `--grid` border, 12px font.
- HUD: Absolute, top 10px left 12px, `rgba(0,0,0,0.5)` background, 6px/8px padding, 8px radius, `--grid` border, 12px font.
- Tooltip: Absolute, pointer-events none, `rgba(20,26,36,0.97)` background, `--grid` border, 8px/10px padding, 8px radius, 12px font, translate -50% -120%, nowrap.
- Help bubbles: `.help` - 16x16 circle, 1px `--ink` border, 11px font, cursor help. Hover shows tooltip (min-width 200px, max 320px, opacity transition).
- Details: `.details.card` - No padding, overflow hidden. Summary: Cursor pointer, 12px padding, 700 weight, flex with chevron (10x10 arrow, rotate on open).
  - Chevron: 10x10, border-right/bottom 2px `--ink`, rotate 0/45deg on open/close.
  - Body: 12px padding, top 1px `--grid` border.
- Inputs: Full width, 8px padding, 8px radius, 1px `--grid` border, `#0e141d` background, `--ink` color.
- Buttons: `--accent` background, white text, 10px/12px padding, 10px radius, 600 weight, cursor pointer.
  - Ghost: Transparent background, 1px `--grid` border, `--ink` color.
- Disabled: 0.6 opacity.
- Chart: `#chartBox` relative, min-height 420px. Canvas: Full size, `#0b0f14` background, 12px radius.

### 2.4 Responsiveness
- Viewport: `width=device-width, initial-scale=1`.
- Height: 100% for html/body/wrap.
- Overflow: Left column scrollable; signals list max 36vh scrollable.
- Canvas resizes on window resize (using `devicePixelRatio` for high-DPI).

## 3. HTML Structure
### 3.1 Head
- `<!DOCTYPE html><html lang="en">`
- Meta: charset utf-8, viewport.
- Title: "Stock Recovery Signal Identifier — v9.0 (rel-panned HUD + relative triggers w/ real P&L + backtest)"
- `<style>`: Include all CSS definitions as described.

### 3.2 Body
- `<div class="wrap">`
  - `<aside class="left">`
    - h2: "Stock Recovery Signal Identifier"
    - Muted div: Description ("Analyze dip → recovery and breakout signals with customizable triggers and metrics.").
    - Card: Instrument search (input#search, div#searchResults.list), selected pills (grid3 with #selName, #selTicker, #selInsId), index search (similar with #indexSelName, etc.), buttons (#fetchBtn, #clearSel, #clearIndexSel), checkbox#relativeToIndex with help bubble.
    - Details#dataSrc (open by default): Summary with chev and subtitle, body with grid2 for API key (password#apiKey), from date (date#fromDate with default "2024-01-01"), instruments file (file#instrFile accept JSON, status#instrStatus, load button).
    - Details: Signal Parameters - Grid3/grid2 for inputs (#window min1 max60 value5, #delay min0 max60 value0, #dipPct min0.1 max10 step0.1 value1.5, #horizon min3 max60 value10, #highestSince min0 max250 value0), checkboxes (#requireDip, #onlyBreakout, #shiftExit), cliff card with checkbox#cliffOn, grid2#cliffCfg (#cliffDays min1 max60 value4, #cliffPct min0.1 max50 step0.1 value3).
    - Details: Additional Indicators - Grid2 for #smaPeriod min0 max200 value20, #volumeThreshold min0 max500 step10 value100, checkboxes (#aboveSMA, #belowSMA).
    - Card: Auto-scale checkbox#autoY (checked), buttons (#lockY, #resetY, #resetZoom), run (#runBtn) and export buttons (#exportSignals, #exportPNG).
    - Card: Metrics grid3 (signals#mSignals, avg#mAvg with label#labelAvg, hit#mHit, med#mMed with #labelMed, max#mMax with #labelMax, min#mMin with #labelMin).
    - Card: Signals list with title, summary#sigSummary, table#sigTable (thead: Trigger Date, Trigger, +H/Exit#thH, Return, Cliff?, Type; tbody dynamic).
  - `<main class="right">`
    - Div#chartBox.card: Canvas#chart, tip#tip.tooltip (initially hidden), hud#hud.hud (hidden).
    - Card: Flex pills for legend (Line=close/rel, Yellow=SMA, • Red=trigger, • Green=exit, Hover dot to highlight, Scroll to zoom • Drag to pan).
    - Card#btCard: Backtest title, export button (#exportEquity), metrics grid4 (#btTotal, #btCAGR, #btDD, #btTrades, #btWinRate, #btAvgWinLoss, #btPF, #btSharpe), canvas#equityChart (height 180px).

### 3.3 Script Placement
- Main script at end: All JS code, organized modularly (e.g., utilities, data structures, functions, classes, app logic).

## 4. JavaScript Functionality
### 4.1 Globals and Data Structures
- INSTRUMENTS array: Objects with {insId: number, name: string, ticker: string, urlName: string, yahoo: string}.
- SELECTED and INDEX_SELECTED: Selected instrument objects or null.
- STATE object: { rows: [], indexRows: [], signals: [], name: '—', sma: null, relativeToIndex: false, equity: [] }.
  - rows: Array of {date: Date, close: number, volume: number|null}.
  - indexRows: Similar for index.
  - signals: Array of {kind: 'recovery'|'breakout', baseIndex: number, triggerIndex: number, triggerDate: Date, triggerClose: number, endIndex: number, endDate: Date, endClose: number, fwdRet: number, cliff: boolean, cliffReason: string, cliffType: 'low'|'drop', cliffValue: number}.
  - sma: Array of numbers or null (SMA values).
  - equity: Array of {date: Date, close: number} (equity curve).

### 4.2 Utility Functions
- toCSV(rows): Convert objects array to CSV string, escaping quotes.
- parseDate(s): Parse string/number to Date or null.
- computeSMA(rows, period, key='close'): Compute simple moving average array (nulls for incomplete periods).
- computeRelativePerformance(stockRows, indexRows, mode='rebased', baseDate=null): Compute relative series (excess or rebased %).

### 4.3 Data Parsing
- autoDetect(json): Recursively find largest array of objects, auto-detect date/close/volume keys, clean and sort data.

### 4.4 Signal Detection
- detectSignals(rows, windowDays=5, dipPct=1.5, horizon=10, opts={}): Core function to detect signals.
  - opts: stopOnCliff, cliffLookback, cliffDropPct, delayDays, shiftExit, highestSince, requireDip, volumeThreshold, aboveSMA, belowSMA, onlyBreakout, sma.
  - Precompute dip flags (dip >= threshold, volume check with 50-day avg).
  - Detect recovery signals within window and breakout (highest since, optional require dip).
  - Apply cliff exits (low break or % drop), SMA filters at trigger.
  - Return sorted signals array.

### 4.4.1 Check boxes and data input fields
-Input fields
--Recover window (days) - number of days over which you check if the price now is above the price before the Dip threshold.
-- Dip threshold (%) - how much does the price need to go down to be a candidate for recovery
-- Delay trigger (days) - if the trigger happens based on the above two parameters, then user can specify to delay the actual trigger with these # of days
-- Shift exit date with Delay - also shift the exit date in case user have specified a Delay trigger
-- Horizon (days) - the number of days when you normally exit the position (unless other features makes it earlier or later)
-- Breakout high (days) - the minimum number of days that has passed since current date/bar is highest.
-- Above cross-SMA trigger (%) - used together with checkbox 'Trigger on SMA crossing' and defines how much above the SMA the price needs to go after a SMA crossing event to trigger a purchase (if checkbox is set).
-- Below cross-SMA exit (%) - used together with checkbox 'Trigger on SMA crossing' and defines how much below the SMA the price needs to go after a SMA crossing event to trigger an exit (if checkbox is set)

--Checkboxes
- Each checkbox and input data fields should have a (?) beside the information text and when the user hovers over that (?) it should show a small pop-up explaining more details what it is used for or how it works
- The following check boxes (as a minimum) should be available somewhere in the left pane, grouped together with relevant input data. Potentially multiple areas with different input data and check boxes that can be collapsed to make the left pane more clean.
- Relative to index. If checked the graph should show the graph rebased, relative to index and have % on Y axis. User should be able to drag and zoom the chart. The first date should always be in the middle (y-axis) and be 0 %
-- Only trigger on breakout. If checked only make triggers based on breakout. Decision point for this is from the input data 'Breakout high (days)'
-- Require dip before breakout - as the name suggest. there is a input field 'Dip threshold (%) that defines the threshold. This threshold is also used for triggering a breakout.
-- Close trigger on cliff - there are two separate input fields for this feature - 'Lookback days for low' and 'Max drop (%)'. if the drop is > 'Max drop (%)' over the past 'Lookback days for low' (after the trigger date), then you exit
-- Only trigger if above SMA
-- Only trigger if below SMA
---For the SMA triggers, there are two associated input fields; 'Moving average  (days)' and 'Volume Threshold (% of avg). The Moving average should also be visible in the graph, in yellow.
-- Trigger on SMA crossing - if checked a trigger and exit takes place based on the values defined in the input fields 'Above cross-SMA trigger (%)' and ' Below cross-SMA exit (%)'
-- Use Momentum based exit - see 4.4.2 below for detailed behaviour if this is checked

### 4.4.2 - Momentum based exit
A solid momentum-based trailing exit, which extends a fixed holding period by monitoring short-term price momentum. It prevents selling during upward trends but triggers an exit on signs of reversal, specifically a recent drawdown.
Key concepts:

- Minimum Hold Period (eval_days): This is the same as 'Horizon (days) above. No need for additional input field, use Horizon. Enforces a baseline hold time post-trigger to avoid knee-jerk reactions to early volatility.
- Momentum Check Window (last_couple_of_days): Defines the lookback period for detecting a drop. This acts like a short-term trend filter—e.g., if set to 3, it checks if the price has fallen sharply over the past 3 trading days.
- Drop Threshold (drop_percentage): The percentage decline (e.g., 5% for -0.05) that signals a potential trend reversal. This is calculated as (current_price - price_n_days_ago) / price_n_days_ago <= -drop_percentage.
- Daily Evaluation: Starting after eval_days, evaluate at the close of each trading day. If the drop condition isn't met, continue holding to "ride the slope." This implicitly assumes positive or neutral trend if no significant drop occurs.
Assumptions:

Prices are daily closing prices.

Note, that if the price after the 'Horizon (days)' day is lower than the trigger price, then exit and don't follow the momentum based exit. If however the price is higher than the trigger price, do use the Momentum based exit strategy.

We decide to exit at the end of the day if the condition triggers, selling at the next open or same close (depending on your system).
No other exits like overall stop-loss or profit targets are included here—focus is purely on your spec.
Edge cases: If last_couple_of_days > current hold days, skip check until enough data. If no drop ever occurs, hold indefinitely (in practice, pair with a max hold or other rules).

This strategy works well for trending stocks (e.g., growth tech in bull markets) but may lag in choppy or reversing markets. Backtest on historical data to tune parameters—start with eval_days=10, last_couple_of_days=3, drop_percentage=5.
Implementation in Python
I'll provide a complete, configurable Python function to simulate or integrate this into a trading system. It takes a list of daily closing prices (starting from the trigger day as index 0) and returns the exit day index (or None if no exit within the data). The example below is in python to clarify the mechanism, but here we should obviously use JS instead.
<python example>
def find_exit_day(prices, eval_days, drop_percentage, last_couple_of_days):
    """
    Determines the exit day for a position based on the described algorithm.

    Args:
    - prices (list[float]): List of daily closing prices, index 0 = trigger day.
    - eval_days (int): Minimum days to hold after trigger.
    - drop_percentage (float): Percentage drop threshold (e.g., 5 for 5%).
    - last_couple_of_days (int): Lookback days for drop check.

    Returns:
    - int or None: Exit day index (relative to trigger), or None if no exit.
    """
    if len(prices) <= eval_days:
        return None  # Not enough data to evaluate

    drop_threshold = -drop_percentage / 100.0  # Convert to decimal, negative

    for day in range(eval_days + 1, len(prices)):
        lookback_day = day - last_couple_of_days
        if lookback_day < 0:
            continue  # Skip if not enough history for lookback

        price_n_days_ago = prices[lookback_day]
        current_price = prices[day]

        if price_n_days_ago == 0:
            continue  # Avoid division by zero (rare, but handle)

        return_rate = (current_price - price_n_days_ago) / price_n_days_ago
        if return_rate <= drop_threshold:
            return day  # Exit at this day's close

    return None  # No exit triggered within available data
</python example>
<sample price data>
# Example: Trigger on day 0 at $100, then prices over 15 days
prices = [100.0, 102.5, 105.0, 103.0, 107.0, 110.0, 112.0, 115.0, 113.0, 116.0,  # Days 0-9
          118.0, 120.0, 117.0, 114.0, 111.0]  # Days 10-14

eval_days = 10
drop_percentage = 5  # 5%
last_couple_of_days = 3

exit_day = find_exit_day(prices, eval_days, drop_percentage, last_couple_of_days)
if exit_day is not None:
    print(f"Exit on day {exit_day} at price {prices[exit_day]}")
else:
    print("No exit triggered")
</sample price data>
Output Explanation:

Hold through day 10 (eval_days=10).
On day 11: Check days 8-11 (115→113→116→118→120), but for day 11 (price=120), lookback to day 8 (115): +4.3% (hold).
On day 12 (117): Lookback to day 9 (116): +0.9% (hold).
On day 13 (114): Lookback to day 10 (118): -3.4% (hold, since >-5%).
On day 14 (111): Lookback to day 11 (120): -7.5% (<-5%, exit).

This exits on day 14 due to the 7.5% drop over the last 3 days.

### 4.5 Instruments Handling
- loadInstrumentsFile(file): Parse JSON, extract array, map to standard fields, filter valid insId, update status.
- filterInstruments(q): Score and sort matches in name/ticker/etc., return top 40.

### 4.6 Chart Implementation
- class LineChart(canvas, getDataFn, opts={}): Handles charting.
  - Properties: ctx, padding {l:56, r:16, t:16, b:28}, xmin/xmax for zoom, pointer, drag, autoY=true, ypad=0.1, fixedY=null, cross=null.
  - Methods: resize (handle DPR), fitXDomain, visibleRange, computeYDomain (min/max with padding, symmetric for relative), lockYToCurrent, resetYToFull, x2px/y2px/px2x transforms, onMove (update pointer/cross/HUD), onWheel (zoom), onDown/Up/Drag (pan), focusIndex, resetZoom, render (grid, title, axes, lines, dots, highlights, tooltip).
  - HUD: Date • value (customizable decimals, mode like pctFromOne, colorize).
  - Tooltip: Detailed info on hover (date, values, volume, SMA, returns with color).
  - NOTE: graph should be possible to pan and zoom
  - SMA should have yellow color
  - Trigger and exit evemts/dates should be marked with red dot (trigger) and green dot (exit)
  - When user moves mouse over the main chart, apart from the HUD in upper left corner, a square popup at the cursor should show: date, closing price, volume, sma. When there is not already a trigger (red) or exit (green) dot, a very small white dot should show on the graph on as you move the mouse. Note that if the date has a trigger or exit, these should still show red and green dots respectively. ALso if the mouse is hovered on one of these, the matching pair should both be highligthed and the following info should be shown: Date (bold), Trigger: <price>, +<days>d from trigger: <exit price>, Return: <% return> (with color), Type: <recovery|breakout|sma-cross>
- When user moves mouse over the Backtest chart, it should have a HUD showing the date and % change with 2 decimals. (color coded, green for plus, red for minus)
- when hovering over an entry or exit point, the square popup at the cursor should have 6 rows: Dates, Trigger, Exit, Return, Type, EType.
-- If hovering over one of those points:
---Dates: you should show both the date for the entry point as well as the date for the exit point; hovering over an entry point it should show this format: '<entry-date> - (<exit-date>)', whereas if hovering an exit point it should have this format: '(<entry-date>) - <exit-date>', i.e. with parathesis around the "paired dot".
entry point example '2025-02-03 - (2025-02-15)',  exit point example:  '(2025-02-03) - 2025-02-15'
---Trigger: show the price on trigger date, e.g. 'Trigger <entry price>', example: 'Trigger: 255.00'
----'Exit: Show both the exit price and how many days (bars) between entry and exit, e.g. 'Exit: <exit price> (<number of bars between entry and exit> bars)', Example: 'Exit: 254.00 (10 bars)'
---Return: show the price on exit date, e.g. 'Return: <percentage gain or loss>', color coded green if gain, red if loss. Example: 'Return: 2.45%'
--- Type: <reason for trigger>, Example 'Type: sma-cross'
---EType (short for Exit type): shows the exit type, e.g. 'Etype: <horizon>|<cliff>|<sma-cross>|<momentum>', Example: 'Etype: momentum' if exit was triggered by momentum.


### 4.7 Backtest Implementation
- computeEquitySeries(rows, signals, sequential=true): Build equity array (flat between, mark-to-market during trades).
- updateBacktestMetrics(): Compute total return, CAGR, max drawdown, trades, win rate, avg win/loss, profit factor, Sharpe (daily, annualized with sqrt(252)).

### 4.8 App Logic
- updateMetricLabels(): Update labels with current horizon (e.g., "Avg 10d").
- compute(): Read inputs, filter rows from visible start, compute SMA, detect signals (on relative if enabled, P&L on real prices), update metrics/table, equity, render charts.
- Event listeners: Input changes trigger compute; file change loads instruments; search input/click filters/selects; fetch uses async/await for API, parses, updates STATE; exports create blobs; checkboxes toggle features (e.g., cliff enables inputs, above/below SMA mutually exclusive).

## 5. Interactions and Behavior
- Search: Live filtering, selection updates pills.
- Fetch: Require key/insId, fetch prices (stock and optional index) using async/await.
- Run: Compute signals/metrics/equity based on parameters and visible range.
- Chart: Hover shows crosshair/tooltip/HUD; wheel zooms; drag pans; reset buttons for zoom/Y.
- Details: Expand/collapse with chevron animation.
- Help: Hover displays tooltip.
- Exports: Signals CSV (dates/closes/return/cliff/type); Equity CSV (date/equity); Chart PNG (data URL).
- Relative: Checkbox enables rebasing to visible start for charts/signals.
- Cliff: Checkbox enables related inputs.
- SMA Filters: Mutually exclusive (checking one unchecks the other).
- AutoY: Checkbox toggles visible vs full Y scaling.

## 6. Edge Cases and Error Handling
- No data: Display dashes in metrics, empty chart/table.
- Invalid inputs: Enforce min/max/step on number inputs.
- Relative without index: Effectively disable.
- Signals overlap: Use Set to prevent duplicates.
- Volume null: Skip threshold check.
- Dates: Sort ascending, use ISO in outputs.
- Errors: Alerts for fetch/parse failures; use try/catch in code.

This PRD provides a complete specification for building the tool. The implementation must ensure all described functionality is present, with a layout and color scheme closely matching the descriptions.
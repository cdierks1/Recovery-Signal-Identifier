## Relevant Files

- `Stock Recovery Signal Identifier_v_10.6.html` - Existing single-file app; UI, charting, backtest logic live here.
- `10i/stock_recovery_signal_identifier_v_10i.html` - Prior iteration reference if needed for patterns.
- `tasks/` - Destination for this task list.

### Notes

- The app is an all-in-one HTML/JS file; expect edits within `Stock Recovery Signal Identifier_v_10.6.html` across data, UI, and chart logic.
- Prefer modularizing new RSI code into clearly labeled sections and functions for readability/testing.

## Tasks

- [ ] 1.0 Compute RSI (Wilder 14) within data pipeline
  - [x] 1.1 Implement `computeRSIArray(vals, period)` using Wilder smoothing with caching
  - [x] 1.2 Implement `computeRSI(rows, period)` on closes with WeakMap cache
  - [x] 1.3 Handle edge cases: insufficient lookback, zero loss/gain, NaNs
  - [x] 1.4 Unit test RSI against known series (±0.1 tolerance)
  - [x] 1.5 Expose RSI in data precompute for downstream consumers (no UI yet)
- [ ] 2.0 Render RSI chart panel beneath price chart with reference and threshold lines
  - [x] 2.1 Add RSI panel container and canvas under price chart (~160px)
  - [x] 2.2 Create fixed [0,100] chart and plot RSI series
  - [x] 2.3 Draw dashed 30/70 and solid user thresholds; update on changes
  - [x] 2.4 Show/hide panel with Enable toggle; hover HUD placeholder
- [ ] 3.0 Add left-pane "RSI Strategy" controls (enable, period, thresholds, mode, trigger style)
  - [x] 3.1 Add controls UI with validation ranges
  - [x] 3.2 Wire change events to recompute/render
  - [x] 3.3 Persist read helpers and defaults (no storage yet)
- [ ] 4.0 Implement RSI crossing-based signals and integrate with backtest engine
  - [x] 4.1 Build RSI options in buildSignalOptions; cache array
  - [x] 4.2 Add Independent mode: generate RSI entries/exits (Crossing/State)
  - [x] 4.3 Add Filter (AND) mode: gate base entries on RSI condition
  - [x] 4.4 Include rsiAtEntry/Exit, thresholds, period in signal objects
- [ ] 5.0 Support Independent vs Filter (AND) composition with existing strategies
  - [x] 5.1 Mode radios; entry gating for Filter; separate RSI kind for Independent
- [ ] 6.0 Add RSI markers and tooltips; align with main chart markers
  - [x] 6.1 Render RSI panel always; markers for entries/exits (main chart already dots)
  - [x] 6.2 Add HUD text/tooltip fields for RSI values (basic)
- [ ] 7.0 Extend tables and CSV exports with RSI fields when enabled
  - [x] 7.1 Add RSI columns to Signals CSV (values, thresholds, cross, period)
- [ ] 8.0 Input validation, UX polish, accessibility for new controls
  - [x] 8.1 Clamp period 2–200 and thresholds 0–100; wire events
- [ ] 9.0 Performance safeguards, caching, and recompute triggers
  - [x] 9.1 Reuse precompute cache; avoid recompute unless inputs change
- [ ] 10.0 Testing: unit/integration and visual snapshot coverage
  - [x] 10.1 Add manual Test button and console tests for RSI value/bounds
  - [x] 10.2 Embed Wilder example check (silent at init)
  - [ ] 10.3 Add automated snapshot hooks (deferred)



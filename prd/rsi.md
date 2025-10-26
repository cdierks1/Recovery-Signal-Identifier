# RSI — Product Requirements Document (PRD)

**Version:** 1.0  
**Date:** 2025-09-18  
**Source code base:** *Stock Recovery Signal Identifier v10.6* (existing)

---

## 1) Introduction / Overview
Add a **Relative Strength Index (RSI)** indicator to the application with:
1) A new **RSI chart panel** displayed **directly under** the existing equity/price chart.  
2) **Left‑pane controls** to enable an RSI‑based strategy, with user‑configurable thresholds:  
   - “**Purchase on RSI above <percentage>**” (entry threshold)  
   - “**Sale on lower than <percentage>**” (exit threshold)

This feature enables users to visualize momentum and optionally auto‑generate backtest signals from RSI either as an **independent strategy** or as a **filter** on top of existing strategies.

---

## 2) Goals
- G1 — Display a **correct RSI(14, Wilder)** series for the active symbol over the visible date range.  
- G2 — Provide left‑pane controls to **enable/disable** RSI strategy and set thresholds/period.  
- G3 — Support **two modes** for RSI signals: *Independent* and *Filter (AND logic)*.  
- G4 — Implement **crossing‑based** entries/exits (entry when RSI crosses **up** through the entry threshold; exit when RSI crosses **down** through the exit threshold).  
- G5 — Respect the existing **Sequential (no overlaps)** setting and current backtest conventions.  
- G6 — Add RSI fields to **CSV exports** when RSI strategy is enabled.  
- G7 — Performance: no noticeable lag when toggling RSI or scrubbing the chart (within current app performance envelope).

---

## 3) User Stories
- US1 — *As a swing trader*, I want to **see RSI** under the price chart so I can quickly judge momentum extremes.  
- US2 — *As a tester of strategies*, I want to **turn on an RSI rule** with thresholds so I can backtest without writing code.  
- US3 — *As an advanced user*, I want RSI to work as a **filter** in addition to my existing rules so I can narrow entries to strong momentum conditions.  
- US4 — *As a data‑driven user*, I want to **export** signals with RSI context so I can audit entries and exits offline.  
- US5 — *As a learner*, I want clear **tooltips and reference lines** so I understand what values (e.g., 30/70) mean.

---

## 4) Functional Requirements

### 4.1 RSI Calculation & Data
1. The system **must compute RSI using a 14‑period Welles Wilder’s smoothing** by default.  
2. RSI value domain is **0–100**. NaNs/insufficient lookback must be handled gracefully (no spikes).  
3. **Period input** is user‑configurable (min 2, max 200) with default **14**.  
4. When the global **Relative to Index** mode is ON, RSI is **still computed on raw price** (not on the relative series).  
5. RSI must update consistently with the app’s existing data cadence (e.g., daily bars).  
6. Edge case: If there are < *period* data points, show partial RSI after enough data exists; no strategy signals should trigger until the first valid RSI is available.

### 4.2 RSI Chart Panel
7. Add a new panel **directly below** the equity chart, height ~**160px**.  
8. Show a **line plot** of RSI (0–100 y‑axis).  
9. Draw faint **reference lines** at **30** and **70** by default (dashed), and draw **solid lines at user thresholds** (see 4.3).  
10. Show **hover tooltip** with date and RSI value.  
11. Optionally render **entry/exit markers** on the RSI line aligned to trade dates (small dots/triangles) when RSI strategy is enabled.  
12. Provide a small **“eye” toggle** within the panel header to show/hide markers.

### 4.3 Left‑Pane Controls (RSI Strategy)
13. Add a new card titled **“RSI Strategy”** in the left pane.  
14. Card contains:  
    a. **Enable RSI Strategy** (checkbox).  
    b. **RSI Period** (numeric input, default **14**).  
    c. **Purchase on RSI above** (numeric input, default **60**).  
    d. **Sale on lower than** (numeric input, default **50**).  
    e. **Signal Mode** (radio): **Independent** | **Filter (AND)**.  
    f. **Trigger Style** (radio): **Crossing** *(default)* | State. *(Crossing is active by default; see 4.4)*  
    g. Tooltips explaining each control (e.g., “Crossing up = yesterday < threshold and today ≥ threshold”).

15. **Input validation**: thresholds must be in [0, 100]. If invalid, show inline error and disable Apply.

### 4.4 Signal Semantics (Crossing‑based, default)
16. **Entry rule (Cross Up)**: Generate a **Buy** signal when `RSI[t-1] < EntryThreshold` **and** `RSI[t] ≥ EntryThreshold`.  
17. **Exit rule (Cross Down)**: Generate a **Sell** signal when `RSI[t-1] > ExitThreshold` **and** `RSI[t] ≤ ExitThreshold`.  
18. **Equality** counts as being **on** the threshold (≥ for entry, ≤ for exit).  
19. If both a buy and sell would occur the **same day** due to gaps, resolve using existing engine precedence; if none exists, prefer **exit first**, then entry (avoids accidental instant flips).  
20. **Sequential/no‑overlap** behavior must follow the current global setting (no special exceptions for RSI).  
21. Apply existing **Delay** parameter to RSI‑based entries if Delay is enabled globally; **Cliff** and **Momentum exit** rules may close RSI positions as they do for other strategies.

### 4.5 Strategy Composition
22. **Independent mode**: RSI generates trades on its own, ignoring other entry conditions. Exits from other global rules may still close positions per current engine design.  
23. **Filter (AND) mode**: RSI condition must be **true at entry time** in addition to the primary strategy’s conditions. RSI **does not** block exits unless the base strategy requires it.  
24. UI must clearly display which mode is active in the card header subtitle.

### 4.6 Backtest & Visualization
25. When RSI strategy is enabled, reflect entries/exits on the main equity chart consistent with existing marker styles; RSI panel markers should align vertically in date.  
26. Trades table should include a column “**RSI@Entry**” and “**RSI@Exit**”.  
27. If RSI is disabled, no RSI signals are computed and no RSI columns appear in the trades table.

### 4.7 Exports
28. **Signals CSV**: add columns — `RSI`, `RSI_Threshold_Entry`, `RSI_Threshold_Exit`, `RSI_CrossDirection` (UP/DOWN), `RSI_Period`.  
29. **Equity/Prices CSV**: add a column `RSI` on each date when RSI is enabled.  
30. Respect existing CSV date/time formats and column orders where applicable.

### 4.8 Accessibility & UX
31. Controls are keyboard accessible and labeled.  
32. Threshold inputs use **step=1** with support for direct typing; clamp to 0–100 on blur.  
33. Tooltips and helper text avoid jargon and explain *Crossing* vs *State* clearly.

---

## 5) Non‑Goals (Out of Scope)
- Implementing other oscillators (e.g., Stochastic, MFI) — out of scope.  
- Multi‑timeframe RSI or intraday RSI — out of scope.  
- Alerts/notifications outside the app (email/push) — out of scope.  
- Optimizers or auto‑threshold search — out of scope.

---

## 6) Design Considerations (Optional)
- **Visual style:** Match existing chart theming. Reference lines: dashed grey at 30/70; solid lines for active thresholds (60/50 by default).  
- **Panel header:** “RSI (period: 14)” with mini legend and an eye icon to toggle markers.  
- **Consistency:** Reuse existing input components and validation patterns from other strategy cards.

---

## 7) Technical Considerations (Optional)
- **Algorithm:** Use Wilder’s smoothing for gains/losses (EMA‑like recursive method). Confirm with unit tests on known series.  
- **Data flow:** Compute RSI within the same pipeline that feeds the equity chart to ensure alignment — cache per symbol/period.  
- **Performance:** Avoid recomputing RSI on every hover; recompute only on symbol/period/price updates.  
- **Edge cases:** Handle flat periods (zero average loss) to avoid division by zero (RSI = 100); likewise zero average gain ⇒ RSI = 0.  
- **Composition:** Filter mode should hook at the *final* entry check stage to avoid generating partial trades.  
- **Testing:**  
  - Unit tests for: RSI values on synthetic data; crossing detection; equality at thresholds; delayed entries.  
  - Integration tests for: Independent vs Filter behavior; Sequential mode; CSV export columns.  
  - Snapshot tests for chart rendering with threshold lines visible.

---

## 8) Success Metrics
- SM1 — RSI values match a reference implementation within **±0.1** across a 1‑year sample for 5 test tickers.  
- SM2 — Enabling RSI strategy and backtesting **adds < 5%** to computation time on a 5‑year dataset.  
- SM3 — No new console errors; Lighthouse/Perf budget unchanged.  
- SM4 — CSV exports include correct RSI fields when enabled and none when disabled.

---

## 9) Open Questions
- OQ1 — Do you want the **reference lines** at 30/70 **always visible**, even if thresholds are 60/50? *(Current PRD: yes, dashed at 30/70 + solid at active thresholds.)*  
- OQ2 — Should the **RSI panel** be **collapsible** by default? *(Current PRD: visible when RSI is enabled; hidden otherwise.)*  
- OQ3 — Any need for a **State‑based** trigger option in the UI? *(Current PRD: present but Crossing is default.)*

---

## 10) Defaults Selected (per request)
- Trigger style: **Crossing** (entry = cross up through *above*; exit = cross down through *lower than*).  
- Strategy composition: **Toggleable** Independent / Filter (AND).  
- Threshold defaults: **Entry 60**, **Exit 50**.  
- RSI period & method: **14, Wilder**.  
- Relative‑to‑Index interaction: Compute on **raw price**.  
- Left‑pane UX: **New “RSI Strategy” card** with controls.  
- Panel design: **~160px**, y‑axis 0–100, reference lines at 30/70 + active thresholds; markers optional.  
- Trade coordination: **Respect Sequential** setting.  
- Delay/Momentum/Cliff: **Apply** as with other strategies.  
- Exports: **Add RSI fields** to Signals/Equity CSVs.  
- Filename: **RSI.md**.

---

### Implementation Checklist (non‑normative aide)
- [ ] Compute RSI (14, Wilder) & cache per symbol/period.  
- [ ] Render RSI panel under equity chart; tooltips; reference/threshold lines.  
- [ ] Add left‑pane card with controls (enable, period, thresholds, mode, trigger style).  
- [ ] Wire independent/filter logic into entry evaluation.  
- [ ] Implement crossing detection; respect sequential/delay/cliff/momentum.  
- [ ] Add RSI columns to tables and CSV exports.  
- [ ] Unit/integration tests; doc tooltips; QA with 5 tickers.


# Import Stocks CSV & Keyboard Scrolling for Instruments

## 1) Introduction / Overview
Add the ability to import a CSV containing a list of stocks (from Nordnet export style) and present them in a collapsible, scrollable list within the **Instruments search** area. The user can select a stock by mouse or by Up/Down arrow keys and the selected instrument loads immediately (no need to press **Fetch prices**). The CSV columns include `source, watchlist, stock_name, insId` (header row present, to be ignored). While the list is visible, the standard **Search name / ticker…** field remains fully functional.

**Goal:** Make it fast to step through a user-provided watchlist, with minimal clicks and instant instrument loading.

---

## 2) Goals
- Allow users to **load a CSV** of instruments with the specified header format.
- Display a **collapsible, scrollable** stock list showing **only the stock name** (ticker optional, muted).
- Enable **selection by mouse** and **keyboard navigation** (Up/Down, PageUp/PageDown, Home/End).
- **Immediately load** the selected instrument on selection changes (no extra button).
- **Keep search fully usable** while the list is shown.
- Provide clear **error feedback** on invalid CSV and handle duplicates/missing data predictably.

---

## 3) User Stories
1. **As a user**, I can click **Load Stocks CSV** to import my Nordnet CSV so I can review my list inside the app.
2. **As a user**, I can **click** a stock in the list to immediately load its instrument view.
3. **As a user**, I can use **Up/Down arrows** to move through the list and have the instrument load instantly for each step.
4. **As a user**, I can use **PageUp/PageDown** to jump ~10 items and **Home/End** to jump to first/last, with the selection loading each time.
5. **As a user**, I can still **type** any instrument into **Search name / ticker…** and load it, independent of the CSV list.
6. **As a user**, I can **hide** the list if it clutters the interface, and show it again later.
7. **As a user**, I see a **brief spinner** on the selected row while the instrument fetch is in flight.

---

## 4) Functional Requirements

### 4.1 CSV Import
1. Accept CSV file with headers `source,watchlist,stock_name,insId` (case-insensitive).
2. Ignore header row, import data from row 2 onwards.
3. Parse as comma-separated values.
4. Import all rows.
5. De-duplicate by `insId` (keep first occurrence) and drop rows with missing/invalid `insId`.
6. Expect up to 2,000 rows.
7. On parse/validation errors, show a toast/banner.

### 4.2 UI Placement & Layout
8. Add **Load Stocks CSV** button to **Instruments search**, right of **Search name / ticker…** input.
9. Loaded list appears in a **collapsible panel** below the search section with a **fixed height** and **scrollbar**.

### 4.3 List Content & Interaction
10. Each row shows **stock name** and optionally ticker (muted).
11. Single-select rows with clear highlight.
12. Clicking or navigating loads the instrument immediately.
13. Show spinner on selected row during load.

### 4.4 Keyboard Behavior
14. Focus remains in list after click.
15. Navigation:
    - Up/Down: move by 1.
    - PageUp/PageDown: jump ~10.
    - Home/End: jump to first/last.
    - Wrap-around enabled.
16. Auto-scroll selected row into view.

### 4.5 List Controls
17. Include both a **Hide list** button and a **×** icon.
18. Include a **Clear list** link to reset the list.

### 4.6 Sorting
19. Default: CSV order.
20. Click header to sort alphabetically (A-Z, Z-A, CSV order cycle).
21. Preserve current selection after sorting.

### 4.7 Search Interactions
22. Search input remains usable.
23. If a searched stock isn't in CSV, load it normally and clear selection highlight.
24. List remains visible.

### 4.8 Data Matching & Missing IDs
25. Match by `insId`.
26. If not found, attempt by `stock_name`.
27. Skip if still unmatched.

### 4.9 Feedback & Diagnostics
28. Optional toast summary: Imported N, removed D duplicates, K invalid.
29. On errors, show toast/banner.

### 4.10 Persistence
30. In-memory only, clears on refresh.

---

## 5) Non-Goals
- No CSV export/editing.
- No multi-select.
- No backend persistence.
- No advanced filtering.
- No virtualization.

---

## 6) Design Considerations
- Maintain layout balance.
- Card with shadow, fixed height (~320px).
- Highlight selected row with background and left-edge accent.
- Spinner on right of selected row.
- Header includes title, sort label, Hide list, and × icon.

---

## 7) Technical Considerations
- Use CSV parser (e.g., PapaParse) with comma delimiter.
- Validate header case-insensitively.
- De-duplicate by `insId`.
- Use `insId` to trigger instrument load (reuse Fetch prices logic).
- Accessibility: ARIA listbox/option roles.
- Auto-scroll with `scrollIntoView`.
- Toasts for errors.
- Sorting: CSV -> A-Z -> Z-A cycle.

---

## 8) Success Metrics
- Load first item ≤ 2s.
- Arrow key latency ≤ 100ms.
- Parse error rate < 2%.
- 50% of import users navigate ≥5 items via arrows.

---

## 9) Other
- Include just error toasts
- Final list height (proposed 320px).



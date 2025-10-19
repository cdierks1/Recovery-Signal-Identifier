## Tasks

- ## Relevant Files

- `Stock Recovery Signal Identifier_v_10.6.html` - Main single-file app; CSV parser, UI, list, sorting, keyboard, toasts.

- [ ] 1.0 Implement CSV import and schema validation (headers, dedupe, limits, errors)
  - [x] 1.1 Add CSV parsing and header validation utilities in main script
  - [x] 1.2 Implement row validation and normalization (insId, stock_name)
  - [x] 1.3 De-duplicate by insId and drop invalid/missing insId
  - [x] 1.4 Enforce row count limits (<= 2000) and error messages
  - [x] 1.5 Wire error/success reporting to toast/banner API
- [ ] 2.0 Add UI button: "Load Stocks CSV" in Instruments search
  - [x] 2.1 Add button and hidden file input next to search
  - [x] 2.2 Wire click/change to parse CSV, store in memory, show toast
- [x] 3.0 Build collapsible, scrollable stock list panel (fixed height, scrollbar)
  - [x] 3.1 Add CSV Watchlist panel markup/CSS below search
  - [x] 3.2 Render list items with selection highlight and spinner
  - [x] 3.3 Show panel on import; hide/close/clear controls
- [x] 4.0 Render rows: stock name (+ optional ticker), selection highlight, spinner on selected
  - [x] 4.1 Render name; ticker optional (muted) — currently name only
  - [x] 4.2 Selected row shows left accent and spinner during load
  - [x] 4.3 Lookup and render muted ticker by insId when available
- [x] 5.0 Mouse interactions: select row and immediately load instrument
  - [x] 5.1 Click row selects and loads instrument
- [x] 6.0 Keyboard navigation: Up/Down, PageUp/PageDown (~10), Home/End, wrap-around, auto-scroll
  - [x] 6.1 Implement key handlers with wrap-around
  - [x] 6.2 Auto-scroll selected into view
- [x] 7.0 Sorting cycle: CSV order ↔ A-Z ↔ Z-A; preserve selection on sort
  - [x] 7.1 Sort cycle and label; maintain selection index
- [x] 8.0 Search interplay: keep search usable; load searched items; clear selection if not in CSV
  - [x] 8.1 CSV list remains visible; search unaffected
  - [x] 8.2 Selecting non-CSV via search clears row selection
- [x] 9.0 Data matching: prefer insId, fallback by stock_name; skip unmatched
  - [x] 9.1 Use insId direct; fallback selection by stock_name when needed
- [x] 10.0 Feedback: import summary toast (N imported, D duplicates, K invalid) and error toasts
  - [x] 10.1 Implement toast host + summary & error messages
- [x] 11.0 State & controls: in-memory only; Clear list; Hide list; × close
  - [x] 11.1 In-memory only; controls wired
  - [x] 11.2 Add "Show list" toggle when hidden
- [x] 12.0 Accessibility & focus: ARIA roles (listbox/option), focus management
  - [x] 12.1 `role=listbox/option`, focusable list, keyboard supported


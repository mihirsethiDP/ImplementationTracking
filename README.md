# Implementation Tracking — Digital Paani Feature Adoption Console

A static dashboard that shows feature adoption across all Digital Paani plants.
It reads live from a Google Sheet (via an Apps Script web app). No build step,
no dependencies — pure HTML/CSS/JS.

## Two links (same repo, GitHub Pages)

| Link | File | Purpose |
|---|---|---|
| **Broadcast** (share widely) | `index.html` → https://mihirsethidp.github.io/ImplementationTracking/ | **View-only.** No admin password in source, no Edit Mode, no write path. |
| **Admin** (keep private) | `admin.html` → https://mihirsethidp.github.io/ImplementationTracking/admin.html | **Edit-enabled** (Edit Mode + password). Owner keeps this URL private / restricted. |

> Both pages share the same UI. **When changing shared behaviour, edit BOTH files.**
> The only difference is `admin.html` adds the auth modal, the Edit Mode toggle,
> and the write path (`saveField` → POST).

## Data source

The dashboard does **not** read the sheet directly — it calls an Apps Script
web app, which reads/writes the sheet.

**Dashboard → Apps Script → Google Sheet**

| Thing | Value |
|---|---|
| Apps Script endpoint (`const API` in both HTML files) | `https://script.google.com/a/macros/digitalpaani.com/s/AKfycbxFSHNmPjCEtZ8NLNNhGjQKQWJUCjzObmGXwiza8TPR88vfHGwUwstV6gU0lBaAV8elfA/exec` |
| **Google Sheet** | https://docs.google.com/spreadsheets/d/1TS1EfvtoI7d3M2uyDCEx9LbkU1jkghzRaNdXowdJDiM/edit |
| **Spreadsheet ID** | `1TS1EfvtoI7d3M2uyDCEx9LbkU1jkghzRaNdXowdJDiM` |
| **Tab read** | `Sheet1` — one row per plant |
| **ChangeLog tab** | auto-created by the Apps Script; logs every edit (timestamp, row, field, value, editor email) |

- **GET** (no params) → all plant records as JSON.
- **POST** `{rowIndex, field, value}` → updates one cell (admin build only).
- The spreadsheet ID lives **inside the Apps Script**, not in this repo. To view/
  edit the script: open the sheet → **Extensions → Apps Script**.
- See [docs/context.md](docs/context.md) for the full column mapping and redeploy steps.

## Dynamic by design (data-driven)

The tool builds itself from the sheet — nothing about features/plants/workspaces
is hard-coded in the dashboard:

- **New column on the sheet → new feature.** Feature columns are discovered at load
  (`Object.keys(records[0])` minus the meta fields `rowIndex, workspace, plant,
  reportsType`). They appear automatically in the stat strip, feature grid, table,
  and drill-downs, and count toward every coverage/implementation %.
- **New row → new plant**, and **new workspace value → new workspace** — both flow
  through with no code change.
- **Dependency:** this only works end-to-end if the **Apps Script returns the new
  column** in its JSON. The script is header-driven (it preserves exact column
  names), so new boolean columns should flow through automatically — but confirm by
  adding a test column and hitting ↻ Refresh. `Reports` is treated specially
  (cadence: Daily/Weekly/Monthly/All/None) and `Maintenance` is excluded by the
  script.

## What it shows

- **Overall implementation %** = features implemented ÷ total features, computed at
  **plant** and **workspace** level (a workspace aggregates its plants).
- Per-feature adoption cards, a per-workspace breakdown, a full plant-by-plant table,
  and workspace/plant drill-downs.
- **Search** is selection-based: typing only suggests; the dashboard filters when you
  **select** a plant/workspace. Selecting a workspace scopes the plant suggestions to
  that workspace; the two searches otherwise work independently.

## Running locally

```bash
npx serve -l 8000 .
# then open http://localhost:8000  (broadcast)  or  /admin.html  (admin)
```

## Roadmap / ideas

- PostHog event tracking to measure actual in-tool usage (see discussion notes).
- Optional: restrict the Apps Script to `@digitalpaani.com` accounts if the data
  should not be publicly readable.

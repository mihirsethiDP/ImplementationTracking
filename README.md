# Implementation Tracking — Digital Paani Feature Adoption Console

A single-file HTML dashboard that shows feature adoption across all Digital Paani
plants. It reads live from a Google Sheet (via Apps Script) and lets admins edit
feature flags directly from the site.

## Files

| File | Purpose |
|---|---|
| `index.html` | The entire website — single self-contained file (HTML + CSS + JS) |
| `docs/context.md` | Full project context: data model, API, auth, design system |

## Running

Open `index.html` in a browser — no build step, no dependencies. It fetches data
from the deployed Google Apps Script endpoint at load time.

To serve locally:

```bash
python -m http.server 8000
# then open http://localhost:8000
```

## Data source

- **Google Sheet** — one row per plant, feature flags in columns C–Q.
- **Apps Script** — GET returns all records as JSON; POST updates a single field.
  Endpoint URL is set in `index.html` (`const API = ...`).
- See [docs/context.md](docs/context.md) for the full column mapping, feature
  categories (client / operator / common), and redeploy instructions.

## Access

- **Viewer mode** — anyone who opens the page can browse all analytics.
- **Admin mode** — click *Edit Mode*, enter the admin password to enable inline
  editing of feature flags (writes back to the sheet).

## Feature groups

- **Client-side** — Reports (frequency), Insight Digest (WhatsApp), Dashboard Summary (WhatsApp)
- **Operator-side** — OCR Data Input, Data Input, Task List
- **Common** — Visualisation, Insights, Dashboard, Inventory, Tickets, Remote Control, Floc Detector, Events

## Roadmap / ideas

- PostHog event tracking to measure actual in-tool usage (see discussion notes).

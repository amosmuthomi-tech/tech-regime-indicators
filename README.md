# Tech Regime Indicators (TRI)
### Macroeconomic Data Pipeline — Automated FRED API Ingestion + Looker Studio Dashboard

![Status](https://img.shields.io/badge/Status-v1%20Delivered-0D7377?style=flat-square)
![Stack](https://img.shields.io/badge/Stack-Apps%20Script%20%7C%20FRED%20API%20%7C%20Looker%20Studio-1B3A6B?style=flat-square)
![Client](https://img.shields.io/badge/Client-US%20Quant%20Research%20Firm-223D5F?style=flat-square)

---

## Overview

TRI is a config-driven data pipeline that ingests a configurable set of macroeconomic FRED series — Industrial Production, PPI spreads, Manufacturing indices, Construction spend — and feeds a real-time Looker Studio dashboard tracking tech-sector economic regimes.

Built for a US-based quantitative research firm as part of a broader effort to understand how macro indicators predict tech industry performance across different economic regimes.

---

## Architecture

```
FRED REST API
     │
     ▼
Google Apps Script (config-driven puller)
     │  reads Master Config Tab
     │  ┌────────────────────────────────────────────┐
     │  │ series_key | source | series_id | agg_rule │
     │  │ is_enabled | destination_sheet | dest_tab  │
     │  └────────────────────────────────────────────┘
     │
     ├──→ Raw Data Tab (wide format: date × series)
     ├──→ Long Format Tab (date | series_key | value)
     └──→ calc_ Derived Metrics (YoY, spreads, burn ratios)
                │
                ▼
        Looker Studio Dashboard
        (time-series | overlays | heatmaps)
```

---

## Key Features

- **Config-driven:** Add or disable any FRED series by editing one row in the Config tab — no code changes
- **Dynamic routing:** `destination_sheet` column holds the URL — pipeline writes to any target sheet automatically
- **Secure:** API key stored in Google Script Properties, never exposed in code or error logs
- **Aggregation rules:** Converts daily/weekly series to monthly using `avg` or `eom` per series
- **Error handling:** Per-series success/failure logging with last-refresh timestamp
- **Derived metrics:** `calc_` prefix series (YoY growth, spreads, burn ratios) computed as a second layer, cleanly separated from live FRED fetches

---

## Config Schema

| Column | Description |
|--------|-------------|
| `series_key` | Short name with prefix: `ip_`, `ppi_`, `mfg_`, `calc_` |
| `source` | `FRED` (more sources in Phase 2) |
| `series_id` | Official FRED series ID |
| `start_date` | Earliest date to pull |
| `target_frequency` | `monthly` |
| `agg_rule` | `avg` or `eom` |
| `is_enabled` | `Y` / `N` |
| `destination_sheet` | Full Google Sheets URL |
| `destination_tab` | Tab name in destination sheet |

---

## Example config

[`sample_config.csv`](sample_config.csv) shows the Config tab layout with a few public FRED series (`(yoy)` = year-over-year transform, `calc_` rows are derived). The real client destination sheets are not included.

---

## Series Covered (Phase 1 — Hardware Layer)

| Prefix | Category | Examples |
|--------|----------|---------|
| `ip_` | Industrial Production | `ip_semiconductor_raw`, `ip_computer_raw` |
| `ppi_` | Producer Price Index | `ppi_semiconductor_raw`, `ppi_computer_raw` |
| `mfg_` | Manufacturing | `mfg_orders_raw`, `mfg_shipments_raw` |
| `calc_` | Derived / Computed | `calc_ppi_spread_yoy`, `calc_ip_mfg_ratio` |

---

## Setup

1. Copy the Config Sheet template
2. Add your FRED API key to Google Apps Script → Project Settings → Script Properties → `FRED_API_KEY`
3. Fill in the Config tab with the series you want
4. Run `Tech Indicators → Refresh FRED Data` from the custom menu
5. Connect the destination sheets to Looker Studio

---

## Possible next steps (not built)

- Additional sources (BLS, BEA, Census) alongside FRED
- Scheduled triggers
- A warehouse backend (e.g. BigQuery) and a Python port

Client configuration and the Apps Script source are confidential and not included; this README documents the design.

---

## Stack

`Google Apps Script` · `FRED REST API` · `Google Sheets` · `Looker Studio`

---

*Part of a broader quantitative research platform — [Valuation Anchor Engine](https://github.com/amosmuthomi-tech/valuation-anchor-engine) for company-level financial analytics runs in parallel.*

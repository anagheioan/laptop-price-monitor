# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Repository Is

A data repository for automated daily price tracking of three AMD Ryzen AI 7 laptops (32GB RAM) sold in Romania. There is no application code — the monitoring logic runs as a [Claude Code cloud routine](https://claude.ai/code/routines/trig_01HiZPB71GPn4qcYz4i1BFhX) that appends rows to `price_history.csv` and commits the result.

## Monitored Products and Price Sources

| Laptop | Retailer | Baseline (lei) | How to Fetch Price |
|--------|----------|---------------:|-------------------|
| Lenovo IdeaPad Slim 5 16AKP10 | Altex | 4,699.90 | WebFetch direct from altex.ro product page |
| ASUS Vivobook S16 M3607GA | eMAG | 5,899.99 | compari.ro aggregator (eMAG returns HTTP 403 on direct fetch) |
| Lenovo Yoga Slim 7 14AGP11 | eMAG | 6,999.99 | compari.ro / price.ro (currently not indexed — write `UNAVAILABLE`) |

eMAG.ro blocks direct HTTP requests. Always source eMAG prices via **compari.ro** first, with **price.ro** as fallback. Validate that the fetched listing matches the 32GB variant and the correct product ID before recording the price — reject reseller or wrong-spec results.

## Routine Behaviour

The cloud routine runs **daily at 09:00 Romania time (06:00 UTC)**, from 2026-06-27 to 2026-09-25 (90 days). Each run:

1. Fetches current prices using the sources above.
2. Compares against baseline prices (established 2026-06-27).
3. Appends one row per laptop to `price_history.csv`.
4. Commits and pushes the updated CSV.

## price_history.csv Schema

Columns in order: `date`, `product`, `retailer`, `url`, `baseline_lei`, `current_lei`, `change_lei`, `change_pct`, `direction`, `in_stock`, `note`

- `direction`: `DOWN` / `UP` / `NO_CHANGE` / `UNAVAILABLE`
- `in_stock`: `YES` / `NO` / `UNKNOWN`
- `change_lei`: negative means cheaper than baseline
- `change_pct`: rounded to 2 decimal places
- `note`: record the price source (e.g. `compari.ro`) and any observed promotion

When a price cannot be found, write `UNAVAILABLE` for `direction`, leave `current_lei` / `change_lei` / `change_pct` empty, and explain in `note`.

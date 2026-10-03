# NEXT STEP

Stage F, Step 2 - IBKR historical downloader.

Goal: a small offline script in scripts/ that connects to Trader Workstation
via the IBKR API, requests 1-minute regular-hours bars for AAPL on
2026-09-08, and writes them to data/raw/aapl-2026-09-08-1m.csv using the
existing write_csv from app/data/historical.py.

This is a script, not runtime code. It is not imported by the package. It
requires TWS running and the IBKR API enabled.

Awaiting: instructor to issue the step contract.

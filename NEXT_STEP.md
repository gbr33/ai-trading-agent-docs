# NEXT STEP

Stage F, Step 1 - Historical data preparation.

Before any F work can begin, the following must be decided:
1. Data vendor (Polygon, Alpaca, Databento, IEX Cloud, or a CSV dump).
2. Universe (single symbol, small basket, or index).
3. Timeframe (1-minute recommended; 1-second if available).
4. Date range for the first test (start small: one day).
5. Where the data lives (data/raw/<dataset_id>/).

F1 is not code. F1 is: pick the source, download the data, and run the
existing dataset validator against it. Only after F1 passes do we proceed
to F2 (one-day qualification).

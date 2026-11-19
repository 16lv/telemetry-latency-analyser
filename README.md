# telemetry-latency-analyser

Telemetry ingestion, anomaly detection, and SLA chart pipeline built with **Python 3.12**, **SQLite**, **pandas**, **NumPy**, and **Matplotlib**.

## Overview

`telemetry-latency-analyser` ingests CSV latency logs into SQLite while discarding malformed rows, summarises per-service `p50` / `p95` / `p99` latencies with NumPy, flags Z-score outliers, and exports SVG/PNG comparison charts.

## Features

- **Strict CSV ingestion**: Validates required columns up front and rejects negative latencies, non-finite values, or out-of-range error percentages cleanly.
- **Per-service latency distribution**: Computes sample counts, mean, median (`p50`), tail (`p95`, `p99`), population standard deviation, error rates, and total throughput.
- **Zero-variance-safe anomaly detection**: Computes Z-score outliers safely without dividing by zero when latency samples are uniform.
- **Headless chart export & CLI**: Writes SVG/PNG bar charts via Matplotlib `Agg` backend and ships an installable `telemetry-analyser` CLI entrypoint.

## Technology Stack

- Python 3.11+ (tested on Python 3.12)
- pandas, NumPy, Matplotlib (`Agg`)
- SQLite3 (`sqlite3` standard library)
- pytest, Ruff

## Installation

```bash
pip install -e ".[dev]"
```

## Usage Examples

Run analysis and chart generation from a CSV log via CLI:

```bash
telemetry-analyser telemetry.csv --chart reports/latency.svg --z-threshold 2.0
```

Use the pipeline classes from Python:

```python
from pathlib import Path
from data_workbench.pipeline import AnalyticsWorkbench

with AnalyticsWorkbench(":memory:") as wb:
    wb.ingest_csv(Path("telemetry.csv"))
    print(wb.summarize_by_service())
    wb.export_latency_chart(Path("latency.svg"))
```

## Technical Implementation Details

- Matplotlib's `Agg` backend is configured before importing `pyplot`, preventing GUI window allocation in headless environments, and figure disposal sits inside `try ... finally` to avoid leaking memory on export errors.
- `ingest_csv()` inserts valid rows with `executemany` inside a single SQLite transaction, returning both `inserted` and `rejected` counts.

## Project Structure

```text
telemetry-latency-analyser/
├── src/data_workbench/
│   ├── __init__.py
│   └── pipeline.py
├── tests/
│   └── test_pipeline.py
├── pyproject.toml
└── requirements.txt
```

## Testing

```bash
ruff check .
pytest -v
```

## Known Limitations & Future Improvements

- `to_dataframe()` loads the full `service_telemetry` table into memory for grouping; streaming time-windowed aggregations would scale better on multi-gigabyte logs.
- Latency anomalies currently use single-pass global Z-scores rather than rolling time windows.

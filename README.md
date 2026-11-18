# telemetry-latency-analyser

Telemetry ingestion, statistical summary, anomaly detection, and visualisation pipeline written in **Python 3.12** using **SQLite**, **pandas**, **NumPy**, and **Matplotlib**.

## Pipeline Stages

1. **CSV validation & batch load**: `AnalyticsWorkbench.ingest_csv()` checks required headers (`record_id`, `service`, `region`, `latency_ms`, `error_rate_pct`, `requests_per_sec`), filters invalid or non-finite rows, and loads clean records via `executemany()`.
2. **Per-service summary**: `summarize_by_service()` calculates count, mean, `p50`, `p95`, and `p99` latencies, standard deviation, mean error rate, and total requests per second.
3. **Outlier flagging**: `detect_latency_anomalies()` spots records exceeding a configurable population z-score threshold (`|z| >= z_threshold`), returning an empty list safely on zero-variance series.
4. **Correlation & charts**: `pearson_correlation()` computes `r` across finite numeric series, and `export_latency_chart()` writes non-interactive SVG or PNG latency charts with Matplotlib's `Agg` backend.

## Structure

```text
telemetry-latency-analyser/
├── src/data_workbench/
│   └── pipeline.py     # AnalyticsWorkbench, validation, summary stats, SVG chart, CLI
├── tests/
│   └── test_pipeline.py
├── pyproject.toml
└── requirements.txt
```

## Quick Start

```bash
pip install -e ".[dev]"
ruff check .
pytest -v
```

Run directly on a telemetry CSV log:

```bash
PYTHONPATH=src python -m data_workbench.pipeline path/to/telemetry.csv --chart out/latency.svg
```

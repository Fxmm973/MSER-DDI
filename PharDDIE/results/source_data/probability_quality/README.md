# PharDDIE probability-quality source data

This directory contains the compact source package for the rare-event 1-shot and 5-shot discrimination and probability-quality table.

- `table3_pharddie_rows.csv`: formal five-seed aggregate results, including AUROC, AUPRC, ACC, event-macro F1, ECE, Brier score, NLL, HCE, confidence intervals, and HCE coverage.
- `table3_complete_detail.csv`: per-seed probability-quality metrics.
- `table_rows_probability_quality.csv`: full-precision rows used in the LaTeX table.
- `SHA256SUMS.csv`: SHA256 checksums for the three data files.

Training seeds: `19940419`, `20230801`, `20240115`, `20240520`, and `20240910`.

Values in the paper are rounded to four decimals; the CSV files retain full precision.

# EviDDIE component-ablation source data

This directory contains the compact source package for the BSA and EVI ablation tables.

- `ablation_summary_eviddie_new.csv`: five-seed mean and SD for AUROC, AUPR, ACC, and pooled binary F1.
- `ablation_sigtest.csv`: paired tests between the complete EviDDIE configuration and each ablated configuration.
- `table_rows_bsa_evi.csv`: full-precision rows used in the BSA and EVI LaTeX tables.
- `SHA256SUMS.csv`: SHA256 checksums for the three data files.

Training seeds: `19940419`, `20230801`, `20240115`, `20240520`, and `20240910`.

`diff = variant_mean - full_mean`; negative values indicate degradation after component removal. F1 is pooled binary F1. Paired p-values are nominal and are not adjusted for multiple comparisons.

# RESULTS_MAP.md — Latest Manuscript-to-Code Provenance

This file maps the latest manuscript, **“MSER-DDI: Molecular and Semantic Event Representations for Few-Shot and Unseen-Event Drug–Drug Interaction Prediction,”** to the repository artifacts used for its analyses. Numbering follows `MSER_DDI_Final_Language_QA_Revised_Deduplicated.tex` dated 2026-09-15.

## Paper-to-code summary

| Manuscript item | Analysis | Primary code | Primary source/output |
|---|---|---|---|
| Table 1 | Dataset summary | Dataset metadata and task files | `PharDDIE/dataset1/`, `EviDDIE/dataset1/`, `EviDDIE/dataset2/` |
| Figure 1 | MSER-DDI framework | Architecture modules below | Manuscript schematic |
| Table 2 | Rare-event 1-shot/5-shot performance | `PharDDIE/pharddie_table2.py`, RareDDIE evaluation, `shared/paired_diff_rareddie.py` | PharDDIE prediction CSV, RareDDIE seed results, paired-difference CSV |
| Table 3 | PharDDIE probability quality and HCE | `PharDDIE/pharddie_table3_complete.py` | `PharDDIE/results/predictions/predictions_dataset1_PharDDIE.csv` |
| Figure 2 | PharDDIE component removal | `PharDDIE/pharddie_ablation_figure.py` | `PharDDIE/results/validation/ablation_results.csv` and provenance record |
| Table 4 | EviDDIE discrimination and probability quality | `shared/calibration_table.py`, `EviDDIE/eviddie_train_ablation.py` | EviDDIE prediction and calibration CSVs |
| Figure 3 | EviDDIE reliability | Five-seed prediction export and 10-bin calculation | `EviDDIE/results/predictions/predictions_eviddie_new_ablation.csv` |
| Table 5 | BSA component removal | `EviDDIE/eviddie_ablation_sigtest.py` | `EviDDIE/results/ablation_sigtest.csv` |
| Table 6 | EVI component removal | `EviDDIE/eviddie_ablation_sigtest.py` | `EviDDIE/results/ablation_sigtest.csv` |
| Supplementary Figures S1–S2 | Single-seed cross-method comparisons | Archived evaluation records | Seed 19940419 records |
| Supplementary Figures S3–S4 | SHCR coefficient sensitivity | `PharDDIE/pharddie_weight_figure.py` | `PharDDIE/results/validation/weight_sweep.csv` |
| Supplementary Table S7 | Dataset 2 plausibility | `external/case_study_per_event.py`, evidence and leakage scripts | Candidate, evidence, and audit outputs |

## Common protocol

Dataset 1 contains 58 common training events, 5 common validation events, 13 fewer held-out events, and 10 rare held-out events. Its results use training seeds `19940419`, `20230801`, `20240115`, `20240520`, and `20240910`. Runs within an evaluation condition use the same held-out examples and fixed episode sequence.

Positive queries receive tail-corrupted negatives at a 1:1 ratio after excluding known positive tails for the same head-event combination. Directional DDI triples remain ordered. `shared/verify_manifests.py` checks fixed negative manifests and hashes; `shared/audit_leakage.py` implements the reported overlap checks.

- AUROC and AUPRC are pooled over held-out examples.
- Accuracy uses threshold 0.5.
- Event-macro F1 averages event-specific F1 scores.
- Brier score, NLL, and ECE are computed per seed before five-run aggregation.
- ECE uses 10 equal-width probability bins.
- HCE is classification error among predictions with `max(p,1−p)≥0.9` and is reported with coverage.
- Paired tests use seed-level differences (`n=5`, `df=4`) and nominal two-sided p values.

## Table 1 — Dataset characteristics

| Dataset | Drugs | Events | Records | Partition and role |
|---|---:|---:|---:|---|
| Dataset 1 | 1,706 | 86 | 191,808 | 58 train, 5 validation, 13 fewer, 10 rare; primary evaluation |
| Dataset 2 | 1,258 | 80 | 320,108 | 50 train, 5 validation, 25 test; exploratory plausibility and supplementary transfer analysis |

Dataset 1 is mirrored under the PharDDIE and EviDDIE paths. Dataset 2 is used by the `external/` workflow.

## Figure 1 — Framework mapping

| Component | Implementation | Role |
|---|---|---|
| PharDDIE molecular encoder and SHCR | `PharDDIE/pharddie_models.py`, `PharDDIE/pharddie_layers.py` | Molecular representation and hidden-channel reweighting |
| PharDDIE ACI and SRAE | `PharDDIE/pharddie_matcher.py` | Pair-conditioned DRKG context and stochastic latent DDI encoding |
| PharDDIE matching | `PharDDIE/pharddie_matcher.py`, `PharDDIE/pharddie_trainer.py` | Mean support prototype and absolute-difference matching |
| EviDDIE molecular/context encoder and SRAE | `EviDDIE/eviddie_models.py`, `EviDDIE/eviddie_matcher.py` | Independently trained molecular DDI representation without SHCR |
| EviDDIE BSA | `EviDDIE/eviddie_matcher.py` | BioSentVec mapping and auxiliary distribution alignment |
| EviDDIE EVI | `EviDDIE/eviddie_matcher.py`, `EviDDIE/eviddie_trainer.py` | Dirichlet evidence and evidential objective |

The pathways share no learned parameters.

## Table 2 — Rare-event few-shot prediction

**PharDDIE source:** `PharDDIE/results/predictions/predictions_dataset1_PharDDIE.csv`; aggregation by `PharDDIE/pharddie_table2.py`. Training uses five seeds, batch size 256, learning rate `1e-3`, up to 40,000 batches, and validation-AUROC checkpoint selection.

**Retrained RareDDIE source:** `PharDDIE/results/rareddie_seed_{seed}.txt`; aggregation by `PharDDIE/aggregate_rareddie.py`. It uses the same five seeds, budget, checkpoint criterion, and evaluation samples. `shared/paired_diff_rareddie.py` produces `PharDDIE/results/paired_diff_PharDDIE_RareDDIE.csv`.

PharDDIE has the highest displayed 1-shot point estimates; retrained RareDDIE has the highest displayed 5-shot point estimates. All six paired 95% confidence intervals include zero. Seven other baseline rows are published summaries and were not retrained.

## Table 3 — PharDDIE probability quality

Source predictions are in `PharDDIE/results/predictions/predictions_dataset1_PharDDIE.csv`; `PharDDIE/pharddie_table3_complete.py` implements the paper-facing calculation using the same definitions as the shared calibration utility.

Brier score, NLL, and ECE are calculated per seed and reported as mean ± sample SD. HCE pools seed-specific high-confidence predictions and reports errors/total with coverage. Each shot condition uses the queries remaining after support selection, so 1-shot and 5-shot results are descriptive rather than a paired test on identical query sets.

The manuscript reports lower mean Brier score, NLL, ECE, and HCE with broader coverage at 5 shots across common, fewer, and rare event groups.

## Figure 2 — PharDDIE component removal

The figure compares complete PharDDIE with variants removing SHCR, ACI, or SRAE under 1-shot and 5-shot supervision. Archived source rows are in `PharDDIE/results/validation/ablation_results.csv`; provenance and protocol scope are documented in `PharDDIE/results/validation/provenance.md`. The repository plotting entry is `PharDDIE/pharddie_ablation_figure.py`.

This is descriptive development-stage evidence. ACI removal changes both compound–gene context and its weighting; SRAE removal changes perturbation and reconstruction.

## Table 4 — EviDDIE unseen-event evaluation

Table 4 contains two analyses that use different training procedures:

1. complete EviDDIE evaluation, including rare-event discrimination and probability quality on fewer and rare events;
2. fixed-representation comparisons with molecular encoders, DRKG neighbor encoders, and SRAE held fixed; BSA is also fixed except in the linear-projection condition.

Primary sources are:

- `EviDDIE/results/predictions/predictions_eviddie_new_ablation.csv`;
- `EviDDIE/results/predictions/predictions_evi_full_frozen.csv` for the separately fitted complete EDL head;
- `EviDDIE/results/calibration_table_variants.csv` and its detail files.

The fixed-representation strategies in the manuscript are a complete two-class EDL classifier, softmax with cross-entropy, an evidential classifier without KL regularization, and a trainable linear semantic projection with the complete EDL loss. Each uses five seed-specific frozen representations, 5,000 fitting iterations, the same held-out samples, and the same validation procedure.

Complete EviDDIE reaches rare-event AUROC `0.7238±0.0503`. Complete-model and fixed-representation rows are interpreted separately.

## Figure 3 — EviDDIE reliability

The paper figure uses complete-EviDDIE records from `EviDDIE/results/predictions/predictions_eviddie_new_ablation.csv`. Within the fewer and rare groups:

1. positive-class probabilities and binary labels are pooled across five independently trained models;
2. pooled records are assigned to 10 equal-width probability bins;
3. each non-empty bin is plotted at its mean predicted probability and observed positive fraction;
4. marker annotations give pooled seed-example counts.

The totals are 4,080 records for fewer events and 1,080 for rare events. `shared/calibration_table.py --fig` and `EviDDIE/eviddie_reliability_figure.py` are repository diagnostic plotting paths from the same prediction export. The manuscript figure reports the raw complete-model reliability curve and does not represent between-seed uncertainty.

## Table 5 — BSA component removal

`EviDDIE/eviddie_ablation_sigtest.py` summarizes the five seed-level comparisons in `EviDDIE/results/ablation_sigtest.csv`.

| Group | Metric | Complete | Without BSA | Difference | Paired p |
|---|---|---:|---:|---:|---:|
| Common | Accuracy | 0.6075 | 0.5652 | −0.0423 | 0.0480 |
| Common | Pooled F1 | 0.6259 | 0.3456 | −0.2803 | 0.0060 |
| Fewer | Accuracy | 0.5821 | 0.5517 | −0.0304 | 0.0282 |
| Fewer | Pooled F1 | 0.5994 | 0.3806 | −0.2188 | 0.0013 |

Differences are `without BSA − complete`. Tests are nominal and unadjusted for multiplicity.

## Table 6 — EVI component removal

The same seed-matched export and significance script provide the EVI comparisons.

| Group | Metric | Complete | Without EVI | Difference | Paired p |
|---|---|---:|---:|---:|---:|
| Common | Pooled F1 | 0.6259 | 0.5854 | −0.0405 | 0.0560 |
| Fewer | Pooled F1 | 0.5994 | 0.5219 | −0.0775 | 0.0833 |
| Rare | Pooled F1 | 0.7036 | 0.6623 | −0.0413 | 0.0310 |

Complete EviDDIE has higher mean pooled F1 in all groups; only the rare-event result reaches the nominal 0.05 threshold.

## Supplementary figures and Dataset 2 analyses

- **Figures S1–S2:** descriptive common/fewer/rare comparisons for seed 19940419. Their error bars follow the archived single-seed analysis and do not measure training-seed variability.
- **Figures S3–S4:** SHCR coefficient analysis from `PharDDIE/results/validation/weight_sweep.csv` and `PharDDIE/pharddie_weight_figure.py`. The selected-coordinate weight 0.3 was chosen on common-event validation; metric-specific optima vary.
- **Dataset 2:** direct transfer, Dataset 2-specific training, collapse reporting, overlap sensitivity, and self-pair exclusion are supplementary analyses documented by `external/REPRODUCE_CASE_STUDY.md`.
- **Table S7:** `external/case_study_per_event.py` ranks one positive benchmark example per held-out event by `mean(p) × [1−mean(u_EDL)]` after overlap exclusions. `external/case_evidence_upgrade.py` records literature evidence and `external/audit_case_leakage.py` verifies task-data exclusions. The manuscript reports 14/24 examples with directionally consistent drug- or class-level evidence, 10 with none identified, and zero satisfying the pair-specific criterion.


## Interpretation boundaries

- PharDDIE and EviDDIE are independently trained, supervision-dependent pathways.
- Main claims concern unseen DDI events, not novel drugs; drug identities may recur across event partitions.
- BSA aligns marginal distributions and does not enforce event-wise molecular-text correspondence.
- `u_EDL=2/S` is an inverse-total-evidence descriptor, not a calibrated uncertainty probability.
- Complete-model and fixed-representation EviDDIE results use different training procedures.
- Paired component tests are exploratory, nominal, and unadjusted for multiplicity.
- PharDDIE component removal and SHCR coefficient sensitivity are descriptive development-stage analyses.
- Dataset 2 findings provide drug- or class-level plausibility, not pair-specific or clinical validation.

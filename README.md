# PRISM-GI: prediction and single-cell analysis

Research software accompanying the PRISM-GI study of colorectal, gastric and pancreatic cancer prognosis.

**Web predictor:** https://prism-gi.cpmlab.cn/

This release includes only frozen-model inference, optional clinical survival estimates, and selected single-cell analysis scripts. It does **not** include prognostic model training, feature selection, cross-validation, hyperparameter optimization, or model-search code. No patient-level expression, clinical records, credentials, or raw single-cell datasets are distributed here.

## Cohort prediction

Download and extract `PRISM-GI-source.zip`, then run the commands from the extracted directory. The archive preserves the original analysis directory layout. The repository README and verification report are also available separately.

Requires Node.js 20 or newer; no npm dependencies.

```bash
node predict.mjs expression.csv colorectal predictions.csv
node predict.mjs expression_with_clinical.csv gastric predictions.csv --nomogram
node test.mjs
```

Cancer labels: `colorectal`, `gastric`, `pancreatic`. Input: samples in rows, first column `sample_id`, remaining gene columns named with symbols. Use at least the model-specified minimum number of genes from the 158-gene background, including **all 15 model genes**. Provide non-missing numeric expression for measured genes. At least two samples from the same source cohort are required; individual-patient scoring is not supported. Do not rank only the 15 model genes. `examples/synthetic_input.csv` is artificial demonstration data, not study patients.

For `--nomogram`, add `age` (18–100) and `stage` (I–IV or 1–4). Output includes estimated 1-, 3- and 5-year overall survival. These are research estimates, not clinical recommendations. The pancreatic clinical model lacks external GEO evaluation because required clinical information was unavailable.

## What this inference release computes

The JavaScript engine and JSON parameters are copied from the existing deployed predictor. It computes within-sample percentile ranks, cohort-mean adjustment with frozen λ=0.5, frozen training standardization and a frozen Tanh DeepSurv network. Because cohort centering is used, predictions depend on the supplied cohort. The raw score alone is not an absolute survival probability; optional probabilities are supplied by a separate frozen cancer-stratified clinical Cox model.

The study's formal framework is described by the investigator as partial three-domain subspace projection. The released local inference artifact operationally uses cohort-mean adjustment; it does not contain or fit an SVD projection matrix. Investigator-reported cross-computer numerical equivalence is not independently demonstrated by this code release. The computational implementation is retained faithfully rather than relabeled or silently replaced. This release is not a complete model-development reproducibility package.

## Single-cell analyses

Original analysis scripts are in `work/rebuild/python/`, preserving their relative workspace paths and original provenance comments. Public sources: GSE132465 (CRC), GSE183904 (gastric), GSE155698 (pancreatic). Obtain the original count matrices and author/sample annotations from GEO before running. Scripts require the study's staged source files and intermediate metadata at the paths specified in each script; these data are intentionally not uploaded. Read `SINGLE_CELL.md` before running.

The scripts cover QC, broad cell annotations, gene localization, UMAP/t-SNE display, rank-based expression signatures, frozen network scoring and patient-by-cell-type pseudobulk summaries. No prognostic network is trained. PCA/neighbor graph/embedding estimation and doublet detection are single-cell preprocessing, not prognostic model fitting. UCell enrichment is distinct from the parameterized model score. Cell scores are descriptive and are not calibrated cell-specific survival probabilities.

## Checks and scope

`test.mjs` checks synthetic-input inference, gene coverage and invalid-input rejection, and clinical probability ranges/order. Local release verification also compares the unchanged engine with saved website references; see `verification.json`. The full single-cell workflow was not rerun for publication of this repository.

`outputs/final_model_deepsurv_v1/model.pt` is the existing frozen inference checkpoint needed by the original single-cell scripts. It contains parameters and training-derived reference summaries, not individual patient records. Use `torch.load(..., weights_only=True)` as in the code.

No open-source license is assigned by this upload; contact the repository owner for reuse terms. The code is provided for research and reproducibility inspection, not diagnosis or treatment decisions.

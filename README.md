# PRISM-GI: prediction, single-cell analysis and CrossDomainAdjust

Research software accompanying the PRISM-GI study of colorectal, gastric and pancreatic cancer prognosis.

**Web predictor:** https://prism-gi.cpmlab.cn/

This repository contains frozen-model inference, optional clinical survival estimates, selected single-cell analysis scripts, and the standalone CrossDomainAdjust R package.

## CrossDomainAdjust R package

[CrossDomainAdjust 0.1.1: installation and usage](https://github.com/weikaixi/PRISM-GI/blob/main/CrossDomainAdjust.md) provides reusable centroid/SVD domain-projector fitting and lambda-controlled partial projection. Download the complete [R source package](https://github.com/weikaixi/PRISM-GI/blob/main/CrossDomainAdjust_0.1.1.tar.gz), including R code, help pages, examples and tests. It is separate from the frozen prediction archive below. Requires R >= 4.1.0; no compilation is needed.

```r
install.packages("remotes", repos = "https://cloud.r-project.org")
remotes::install_url(
  "https://raw.githubusercontent.com/weikaixi/PRISM-GI/main/CrossDomainAdjust_0.1.1.tar.gz",
  upgrade = "never", build_vignettes = FALSE)
library(CrossDomainAdjust)
packageVersion("CrossDomainAdjust")
```

The repository root is not an R package: use `install_url()`, not `install_github()`. Lambda selection, feature construction, and survival modeling are separate from the package's projection functions.

## Prediction from processed features

Download and extract `PRISM-GI-source.zip`, then run the commands from the extracted directory. Requires Node.js 20 or newer; no npm dependencies.

```bash
node predict.mjs examples/preprocessed_template.csv colorectal predictions.csv
node predict.mjs examples/preprocessed_template.csv gastric predictions_clinical.csv --nomogram
node test.mjs
```

Supply CSV or TSV with `sample_id` as the first column, `input_stage` set to `domain_adjusted_rank` on every row, and all 15 gene columns listed in `prediction/model.json`. Gene columns may be reordered because names determine their mapping. Values must be finite, domain-adjusted ranked features **before** frozen training standardization, prepared with preprocessing compatible with the checkpoint. Raw expression, counts, unadjusted ranks and already standardized z scores are not valid inputs. The stage label identifies the required format; it does not check the correctness of upstream processing. The two-row template contains artificial example values, not patient data.

The prediction engine applies the stored training standardization and frozen Tanh network to each supplied row. Upstream feature preparation is performed separately. A given processed row receives the same score whether submitted alone or with other rows. Comparative groups use the submitted cohort's median; a single row has no assigned high/low group.

Cancer labels are `colorectal`, `gastric` and `pancreatic`. For `--nomogram`, also provide `age` (18–100) and `stage` (I–IV or 1–4). The separate clinical model provides estimated 1-, 3- and 5-year overall survival. The raw network score alone is not a survival probability. These estimates are for research, not clinical recommendations. The pancreatic clinical model lacks external GEO evaluation because required clinical information was unavailable.

## Single-cell analyses

Scripts are in `work/rebuild/python/`. Public sources are GSE132465 (CRC), GSE183904 (gastric) and GSE155698 (pancreatic). Obtain the original count matrices and author/sample annotations from GEO, and arrange the inputs/intermediates at the paths specified by each script. Read `SINGLE_CELL.md` before running.

The scripts cover QC, broad cell annotations, gene localization, UMAP/t-SNE display, rank-based expression signatures, frozen network scoring and patient-by-cell-type pseudobulk summaries. The model-scoring wrapper accepts separately preprocessed cell or pseudobulk feature tables using the prediction format above. UCell enrichment is distinct from the parameterized model score.

## Tests and licensing

`node test.mjs` runs functional checks of the frozen inference interface, input validation, batch-independent scores and clinical probability outputs.

The CrossDomainAdjust source package is MIT-licensed, as specified in its DESCRIPTION and LICENSE files. No open-source license is assigned to the separate prediction and single-cell archive by this upload; contact the repository owner for its reuse terms. All resources are provided for research, not diagnosis or treatment decisions.

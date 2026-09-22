# CrossDomainAdjust

Version 0.1.1 implements lambda-controlled partial domain-subspace projection for numeric features. It supports two, three, or more training domains and reuses the fitted transformation for new samples.

## Installation

Requires **R >= 4.1.0**. The package uses base R and requires no compilation. This is a GitHub distribution; installation does not depend on CRAN acceptance of CrossDomainAdjust.

Run in R or RStudio:

```r
install.packages("remotes", repos = "https://cloud.r-project.org")
remotes::install_url(
  "https://raw.githubusercontent.com/weikaixi/PRISM-GI/main/CrossDomainAdjust_0.1.1.tar.gz",
  upgrade = "never",
  build_vignettes = FALSE
)
library(CrossDomainAdjust)
packageVersion("CrossDomainAdjust")  # 0.1.1
```

The repository distributes a standard R source package (`.tar.gz`), containing all package source code, help pages, examples and tests. Use `install_url()`, not `install_github()`: the repository root is not an R package. To pin a reproducible version, replace `main` in the download URL with the desired commit SHA from the repository history.

Alternatively, download `CrossDomainAdjust_0.1.1.tar.gz` from the repository, leave it compressed, and install the local file without remotes:

```r
install.packages("path/to/CrossDomainAdjust_0.1.1.tar.gz",
                 repos = NULL, type = "source")
```

## Quick start: three training domains

Rows are samples; columns are named numeric features. Use the same feature construction and column names for training and new data. Missing values are not supported.

```r
library(CrossDomainAdjust)
set.seed(20260923)
domain <- rep(c("CRC", "GC", "PAAD"), each = 20)
x <- matrix(rnorm(60 * 15), nrow = 60, ncol = 15)
colnames(x) <- paste0("feature", seq_len(ncol(x)))
x[domain == "GC", 1:4] <- x[domain == "GC", 1:4] + 1
x[domain == "PAAD", 5:8] <- x[domain == "PAAD", 5:8] - 1

projector <- fit_domain_projector(x, domain, anchor = "domain_mean")
projector
unchanged <- project_features(projector, x, lambda = 0)
partial <- project_features(projector, x, lambda = 0.5)
full <- project_features(projector, x, lambda = 1)
stopifnot(identical(unchanged, x), projector$rank <= 2L)
stopifnot(isTRUE(all.equal(partial, (x + full) / 2)))

# Artificial new sample, using the already fitted projector.
new_sample <- setNames(rnorm(ncol(x)), colnames(x))
adjusted_sample <- project_new_sample(projector, new_sample, lambda = 0.5)

# A new dataset uses the same fitted transformation.
new_x <- matrix(rnorm(5 * ncol(x)), nrow = 5,
                dimnames = list(NULL, colnames(x)))
adjusted_new_x <- project_features(projector, new_x, lambda = 0.5)
```

These are synthetic examples, not study patient data. The value 0.5 illustrates partial correction; the package does not select lambda automatically.

## Algorithm

`fit_domain_projector()` computes a centroid for each training domain. SVD of the centroid offsets from a common anchor yields an orthonormal feature-space basis `B` and projection matrix `P = B %*% t(B)`.

```r
x_corrected = x - lambda * ((x - anchor) %*% P)
```

- `lambda = 0`: preserve input features.
- `lambda = 1`: remove the entire component in the learned domain-shift subspace.
- `0 < lambda < 1`: partially remove that component.

For K domains, the rank is at most K - 1 with either built-in anchor. Three domains therefore define at most two independent centroid-difference directions. A custom anchor outside the centroids' affine hull can increase the rank to K, subject to the feature dimension.

Available anchors are `"domain_mean"` (equal-weight domain center), `"sample_mean"` (sample-size-weighted center), or a named numeric vector. The optional `domain` argument of `project_new_sample()` accepts only training-domain labels and does not change the global projection; omit it for a new source.

## Scope and validation workflow

This package provides projection, not a fitted survival predictor. Gene ranking, feature selection, survival-model fitting, calibration and cross-validation are separate steps. Fit the projector inside each training fold, apply it unchanged to the validation fold, and choose lambda using the training cross-validation criterion. Freeze the selected pipeline before external evaluation; do not fit the projector on external test data.

For PRISM-GI feature construction, compute the specified within-sample ranks against the 158-gene candidate background before extracting the 15 model genes; do not substitute ranks computed only among those 15 genes. The separate frozen prediction archive and website are documented in the repository root.

## Help and plotting examples

```r
help("fit_domain_projector", package = "CrossDomainAdjust")
help("project_features", package = "CrossDomainAdjust")
help("project_new_sample", package = "CrossDomainAdjust")
example_dir <- system.file("examples", package = "CrossDomainAdjust")
source(file.path(example_dir, "01_pca_lambda_overlap.R"))
source(file.path(example_dir, "02_two_domain_new_sample_projection.R"))
source(file.path(example_dir, "03_three_domain_new_sample_projection.R"))
```

## License

MIT, copyright 2026 Weikaixin Kong. This license applies to the CrossDomainAdjust source package, not to other repository contents. Numerical projection code is unchanged from the CRAN-submitted version 0.1.1.

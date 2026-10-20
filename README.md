# PRISM-GI: CrossDomainAdjust R package

Research software accompanying the PRISM-GI study of colorectal, gastric and pancreatic cancer prognosis.

**Web predictor:** https://prism-gi.cpmlab.cn/

This repository distributes the standalone CrossDomainAdjust R source package and its installation and usage guide. The separately hosted website, its source code and frozen prediction models are not included in this distribution.

## CrossDomainAdjust R package

[CrossDomainAdjust 0.1.1: installation and usage](CrossDomainAdjust.md) provides reusable centroid/SVD domain-projector fitting and lambda-controlled partial projection. Download the complete [R source package](CrossDomainAdjust_0.1.1.tar.gz), including R code, help pages, examples and tests. Requires R >= 4.1.0; no compilation is needed.

```r
install.packages("remotes", repos = "https://cloud.r-project.org")
remotes::install_url(
  "https://raw.githubusercontent.com/weikaixi/PRISM-GI/main/CrossDomainAdjust_0.1.1.tar.gz",
  upgrade = "never", build_vignettes = FALSE)
library(CrossDomainAdjust)
packageVersion("CrossDomainAdjust")
```

The repository root is not an R package: use `install_url()`, not `install_github()`. The package provides projection functions, not a fitted survival predictor. Lambda selection, feature construction, and survival modeling are separate steps.

## License

The CrossDomainAdjust source package is MIT-licensed, as specified in its DESCRIPTION and LICENSE files. The software is provided for research, not diagnosis or treatment decisions.

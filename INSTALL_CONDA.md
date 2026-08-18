# Conda environment for LineageMap

These commands create the dedicated R 4.3 environment used on this server.

## 1. Create the core environment

```bash
CONDA=/localscratch/ychen4013/miniconda3/bin/conda

$CONDA create -y -n lineagemap_env \
  --strict-channel-priority -c conda-forge -c bioconda \
  'r-base=4.3' r-treetools r-ape r-castor r-data.table r-igraph \
  r-phangorn r-mvtnorm r-dplyr r-magrittr
```

## 2. Install the local checkout

```bash
ENV=/localscratch/ychen4013/miniconda3/envs/lineagemap_env
SRC=/localscratch/ychen4013/LineageMap_datasets/LineageMap

$ENV/bin/R CMD INSTALL --no-multiarch --library="$ENV/lib/R/library" "$SRC"
```

Use the environment's absolute executable for reproducible runs:

```bash
$ENV/bin/Rscript --vanilla analysis.R
```

## 3. Verify the core installation

```bash
$ENV/bin/Rscript --vanilla -e '
  library(LineageMap)
  x <- matrix(c("0", "1", "0", "0", "1", "1"), nrow = 2, byrow = TRUE)
  stopifnot(is.matrix(DropoutDistMatrix(x)))
  cat("LineageMap", as.character(packageVersion("LineageMap")), "is ready\n")
'
```

## 4. Optional: packages used by the vignettes

```bash
$CONDA install -y -n lineagemap_env \
  --strict-channel-priority -c conda-forge -c bioconda \
  r-devtools r-treedist r-tidyverse r-cluster r-mclust r-fnn \
  r-ggplot2 r-plotly r-rcolorbrewer r-ggnewscale r-viridis r-gt \
  r-knitr r-rmarkdown bioconductor-ggtree r-drimpute r-rtsne \
  bioconductor-summarizedexperiment r-deldir r-doparallel r-foreach \
  r-phytools r-plyr r-reshape2 r-spatstat r-spatstat.geom r-umap
```

SpaTedSim's published source repository is currently unavailable. On this
server, the validated pure-R builds can be added to a fresh environment with:

```bash
cp -a /localscratch/ychen4013/Lineagemap_project/Rlib/TedSim "$ENV/lib/R/library/"
cp -a /localscratch/ychen4013/Lineagemap_project/Rlib/SpaTedSim "$ENV/lib/R/library/"
```

Verify them with:

```bash
$ENV/bin/Rscript --vanilla -e '
  stopifnot(requireNamespace("TedSim", quietly = TRUE))
  stopifnot(requireNamespace("SpaTedSim", quietly = TRUE))
  stopifnot(requireNamespace("ggtree", quietly = TRUE))
'
```

## Current cautions

- Pass `state_lineages` as a list of ordered state vectors, not an `ape::phylo`.
- After reconstruction, always check `setequal(tree$tip.label, rownames(muts))`;
  the current parallel tree assembly can corrupt tip labels.
- Do not prepend old project R libraries or clear `LD_LIBRARY_PATH` when using
  this Conda R environment.
- Installing dependencies does not repair the existing vignette syntax, path,
  or input-semantic problems.

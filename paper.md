---
title: 'scPassport: A Bioconductor package for persistent metadata provenance in single-cell objects'
tags:
  - R
  - single-cell RNA-seq
  - metadata
  - provenance
  - Bioconductor
  - Seurat
  - SingleCellExperiment
authors:
  - name: Sedat Kacar
    orcid: 0000-0002-0671-8529
    affiliation: 1
affiliations:
  - name: Indiana University School of Medicine, Indianapolis, Indiana, USA
    index: 1
date: 11 April 2026
bibliography: paper.bib
---

# Summary

`scPassport` is an R/Bioconductor package that embeds a persistent metadata
passport directly inside single-cell objects. It supports `Seurat`
[@hao2021integrated], `SingleCellExperiment` [@amezquita2020orchestrating],
and `SummarizedExperiment` [@huber2015orchestrating] objects. The passport
travels with the object inside the `.rds` file — no external spreadsheets or
sidecar files are needed. `scPassport` provides an interactive Shiny gadget to
fill and update passport fields, a function to print the full passport and
processing log to the console, and a function to append timestamped processing
steps to an auditable log. Core passport and log operations are implemented in
C++ via Rcpp [@eddelbuettel2011rcpp] for performance.

# Statement of need

Single-cell RNA-seq analysis typically produces dozens of R objects representing
different subsets, timepoints, or conditions. A common and underappreciated
problem is that these objects quickly lose their context: after saving and
sharing an `.rds` file, the recipient — or even the original analyst weeks
later — cannot easily tell what animal the data came from, what processing steps
were applied, or how this object relates to others in the same project. This
information is usually tracked in separate spreadsheets, lab notebooks, or
README files that become decoupled from the data over time [@wilkinson2016fair].

Existing tools address related but distinct problems. `SummarizedExperiment`
metadata slots provide a flexible storage mechanism but no standardized schema
or user interface for filling them [@huber2015orchestrating]. Workflow
documentation tools such as `targets` [@landau2021targets] and `drake`
[@landau2018drake] track computational pipelines but are not embedded in the
object itself and require adoption of a specific pipeline framework.
`scRNAseq` [@cole2019performance] and similar data packages provide curated
metadata for published datasets but offer no mechanism for analysts to annotate
their own objects during active research.

`scPassport` fills this gap by providing a lightweight, opinionated schema for
the most common fields needed during single-cell analysis — animal identity,
experimental condition, tissue, project details, lineage relationships between
parent and child objects, and arbitrary custom fields — and storing everything
inside the object itself. Because the passport is part of the object, it is
automatically preserved when the object is saved, shared, or transferred to a
collaborator. The interactive Shiny gadget [@chang2022shiny] lowers the barrier
to adoption: analysts do not need to write any code to stamp an object with
provenance information.

# Functionality

`scPassport` exports three functions. Figure 1 illustrates the complete
workflow: stamping an object with the interactive popup, logging processing
steps, and reading the passport back to the console.

![**Figure 1.** The interactive Shiny popup opened by `scPassport()`, showing
passport fields filled in for a Seurat object.](paper_figures/figure1.png)

![**Figure 2.** Example `read_passport()` console output displaying the full
passport and processing log for a stamped Seurat object.](paper_figures/figure2.png)



## `scPassport()`

Opens an interactive Shiny popup that allows the analyst to fill in passport
fields for any supported object class. If a passport already exists in the
object, all previously entered fields are pre-loaded so the analyst only needs
to update what changed. The popup is organized into four sections:

- **Identity** — object name and RDS registry number
- **Animal Info** — animal ID, species, sex, age, condition, and tissue
- **Experiment Info** — project name, researcher, date, and free-text notes
- **Lineage** — parent object ID, parent RDS number, full ancestry chain, and
  child objects subset from this object

A **Custom Fields** section allows any additional key-value pairs to be added
and persisted alongside the standard fields.

When a `parent` object is supplied, the `parent_id` and `lineage` fields are
automatically populated from the parent's passport, so lineage chains are built
without manual typing. Cancelling the popup or any runtime error returns the
original object unchanged — the function never returns `NULL`.

```r
# Stamp a root object
WTHeme <- scPassport(WTHeme)

# Stamp a child subset, linking lineage automatically
EndofrHeme <- subset(WTHeme, idents = "Endothelial")
EndofrHeme <- scPassport(EndofrHeme, parent = WTHeme)
```

For `SingleCellExperiment` and `SummarizedExperiment` objects, the passport is
stored in `metadata(obj)$passport`; for `Seurat` objects it is stored in
`@misc$passport`.

## `read_passport()`

Prints the full passport and processing log to the console in a structured,
human-readable format. Works with all supported object classes.

```r
read_passport(WTHeme)
# ========== PASSPORT ==========
# Object ID  : WTHeme
# RDS Self   : 224
# Created    : 2026-04-11 09:47:07
# -------- Animal --------
# Animal ID  : M01
# Species    : rat
# Sex        : male
# Age        : P60
# Condition  : control/Heme-treated
# Tissue     : lung
# -------- Experiment --------
# Project    : Heme project
# Researcher : Sedat Kacar
# Date       : 2026-03-09
# Notes      : Form big parse.
# -------- Lineage --------
# Parent     : root
# RDS Parent : root
# Chain      : root
# Children   : WTHeme_downsampled
# RDS Children: none
# -------- Custom Fields --------
# integration_type: rpca
# ======= PROCESSING LOG =======
# No processing steps logged yet.
# ==============================
```

Any number of custom fields can be added through the Shiny popup — for example
`integration_type`, `genome_build`, `batch`, `sequencing_platform`, or any
other key-value pair relevant to the experiment. All custom fields are stored
alongside the standard fields and printed by `read_passport()`.

## `log_step()`

Appends a timestamped entry to the object's processing log, recording the step
name, cell count, gene count, and any parameters supplied by the analyst. This
creates an auditable record of what was done to each object.

```r
WTHeme <- NormalizeData(WTHeme)
WTHeme <- log_step(WTHeme, "NormalizeData",
                   params = list(method = "LogNormalize", scale_factor = 10000))

WTHeme <- RunPCA(WTHeme, npcs = 30)
WTHeme <- log_step(WTHeme, "RunPCA", params = list(npcs = 30))
```

## Implementation

The core passport assembly and log operations (`build_passport`,
`validate_passport`, `build_log_entry`, `append_log_entry`) are implemented in
C++ via Rcpp. The R layer consists of thin wrappers that handle object-class
routing through internal helper functions (`.get_passport()`, `.set_passport()`,
`.get_processing_log()`, `.set_processing_log()`), making it straightforward
to extend `scPassport` to additional S4 object classes in the future.

The package passes `R CMD check` with 0 errors, 0 warnings, and 0 notes on
Bioconductor's Linux build servers, and 0 errors and 0 warnings under
`BiocCheck`. A full test suite covers C++ functions via `testthat`
[@wickham2011testthat]. GitHub Actions runs automated Linux checks on every
push to the development branch.

# Demonstration

A video demonstration of `scPassport` is available at
https://doi.org/10.5281/zenodo.19512801. The video shows the interactive Shiny
passport popup being used with a Seurat object, followed by `read_passport()`
console output displaying the full passport and processing log.

# Acknowledgements

The author thanks Allah (swt) for inspiring and granting the acceptance of the first R package to Bioconductor.

# References

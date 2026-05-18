---
name: omics-workspace-inspection
description: Use when a user asks to inspect bioinformatics, omics, sequencing, spatial transcriptomics, MERFISH, single-cell, RNA-seq, proteomics, or metabolomics project folders, especially when files, scripts, QC reports, outputs, progress, or reproducibility are unclear.
---

# Omics Workspace Inspection

## Overview

Inspect an omics project as a provenance graph: raw inputs flow through scripts, notebooks, workflow files, logs, and intermediate outputs into final results. Report evidence, assumptions, confidence, and missing links in upstream-to-downstream order.

## When to Use

Use for local folders containing data analysis artifacts such as `fastq`, `bam`, `h5ad`, `loom`, `rds`, `csv`, `parquet`, `zarr`, images, notebooks, Snakemake, Nextflow, R scripts, Python scripts, logs, or figures.

Do not use for biological interpretation alone. First reconstruct the computational workflow, then summarize findings.

## Required Output

Write a Markdown report in the inspected project, defaulting to:

```text
project_inspection_report.md
```

Ask only if overwriting an existing report would be risky. Otherwise create a timestamped variant such as `project_inspection_report_YYYYMMDD.md`.

If QC files exist, write a separate Markdown report, defaulting to:

```text
qc_metrics_report.md
```

Do not put detailed QC metric interpretation in `project_inspection_report.md`. The project inspection report may link to the QC report and summarize whether QC evidence exists.

## Inspection Workflow

1. Map the workspace without reading huge binaries:
   - Use `rg --files`, `find`, `du`, `stat`, and `tree` if available.
   - Classify paths as raw input, reference, script, workflow, notebook, config, log, intermediate, result, figure, report, or environment.
2. Identify executable provenance:
   - Read shell scripts, Snakefiles, Nextflow files, CWL/WDL, Makefiles, notebooks, R, Python, Julia, MATLAB, and config files.
   - Search for file reads/writes, command-line arguments, output directories, package imports, and tool invocations.
3. Pair inputs, scripts, and outputs:
   - Match by explicit paths in code first.
   - Then match by filename stems, sample IDs, directory conventions, timestamps, and log messages.
   - Mark each link as `confirmed`, `likely`, or `uncertain`.
4. Infer progress:
   - Order stages from upstream to downstream: acquisition/raw data, preprocessing, QC, alignment/registration, quantification, normalization, integration, clustering, annotation, differential analysis, visualization, export.
   - Use completion evidence such as final files, logs with success messages, newer output timestamps, non-empty result directories, and generated figures.
   - Report stale, partial, failed, or rerunnable stages explicitly.
5. Inspect QC and benchmark files when present:
   - Search for FastQC, MultiQC, `samtools flagstat/stats`, Picard, STAR, Bowtie2, Cell Ranger, Space Ranger, Salmon, kallisto, STARsolo, Scanpy, Seurat, Squidpy, MERFISH decoding, image registration, segmentation, spot/cell calling, and other tool-specific QC outputs.
   - Extract the metrics actually used in the workspace. Do not invent a generic QC checklist when the files show a narrower benchmark set.
   - Explain what each metric measures, which data aspect it evaluates, and what usually indicates good or bad quality.
   - Separate sequencing quality, mapping/alignment quality, quantification quality, sample/cell quality, spatial or imaging quality, and downstream analysis benchmarks when applicable.
   - Include caveats when thresholds are assay-, organism-, chemistry-, aligner-, or project-specific.
   - Write these details to `qc_metrics_report.md`, not to the main project inspection report.
6. Infer reproducibility requirements:
   - Prefer existing `environment.yml`, `conda-lock`, `renv.lock`, `requirements.txt`, `pyproject.toml`, `Dockerfile`, `Singularity`, `module load`, workflow configs, and notebook metadata.
   - If no environment exists, list observed tools and packages and propose a minimal conda environment with channels and package names.

## QC Metric Reporting

If QC files exist, generate a separate `qc_metrics_report.md`. Use the project's own QC outputs and tool logs as the source of truth.

| Metric Group | Examples | Explain |
| --- | --- | --- |
| Sequencing quality | per-base quality, per-sequence quality, adapter content, duplication, GC content, Q30, read length | Whether raw reads are accurate, contaminated, biased, duplicated, or truncated. Higher base quality and Q30 are usually better; high adapter content, severe quality drop-off, or unexpected GC shifts are warnings. |
| Mapping quality | alignment rate, uniquely mapped reads, multimapping, MAPQ, mismatch rate, insert size, chimeric reads, rRNA/mitochondrial fraction | Whether reads map confidently to the expected reference. Higher unique alignment and MAPQ are usually better; high multimapping, mismatch, chimeric, rRNA, or mitochondrial fractions can indicate contamination, poor reference fit, damaged RNA, or low-quality samples. |
| Quantification quality | assigned reads, genes/features detected, UMI counts, saturation, transcript compatibility, empty droplets/background | Whether molecules or features were counted reliably. Higher assigned read fraction and sensible feature/UMI distributions are usually better; excessive background, low detected features, or strong saturation can limit interpretation. |
| Cell/sample quality | cells retained, reads per cell, genes per cell, percent mitochondrial, doublet score, batch metrics | Whether samples or cells are usable. Good/bad cutoffs depend on tissue and protocol; outliers, high mitochondrial fraction, low complexity, and high doublet scores often mark poor quality. |
| Spatial/imaging quality | registration error, segmentation quality, spot density, decoded molecule count, blank/barcode rate, field-of-view coverage | Whether spatial coordinates, images, cells, or molecules are trustworthy. Lower registration error and blank rate are usually better; poor segmentation, uneven coverage, or low molecule counts weaken spatial conclusions. |
| Downstream benchmarks | clustering stability, marker coherence, batch mixing, annotation confidence, replicate concordance | Whether analysis outputs are biologically and statistically coherent. Interpret these as validation evidence, not raw data quality alone. |

Report the observed value when available, the expected direction of quality, and the evidence path. When thresholds are not present in the workspace, describe directionality instead of imposing universal pass/fail cutoffs.

## Evidence Rules

Never present inferred workflow order as fact without evidence. Use these labels:

| Label | Meaning |
| --- | --- |
| `confirmed` | Explicit path, command, workflow rule, log entry, or metadata links the files. |
| `likely` | Filename, sample ID, timestamp, and directory conventions agree. |
| `uncertain` | Plausible but missing direct evidence. |
| `missing` | Expected file, script, log, or environment artifact was not found. |

Prefer file paths and concrete timestamps over general claims. Mention when large binary files were intentionally not opened.

## Report Template

```markdown
# Project Inspection Report

Generated: YYYY-MM-DD HH:MM TZ
Project root: /path/to/project

## Executive Summary
- Apparent project type:
- Apparent current stage:
- QC report:
- Main reproducibility gaps:

## Workspace Map
| Category | Paths | Notes |
| --- | --- | --- |

## Upstream-to-Downstream Workflow
| Stage | Inputs | Scripts / Commands | Outputs | Status | Confidence | Evidence |
| --- | --- | --- | --- | --- | --- | --- |

## Input-Output Pairing
| Input | Script / Rule / Notebook | Output | Link | Evidence |
| --- | --- | --- | --- | --- |

## Progress Assessment
List completed, partial, failed, stale, and missing stages. Explain assumptions.

## Tools and Packages
| Source | Tools / Packages | Evidence |
| --- | --- | --- |

## Suggested Conda Environment
Only include if no better environment file already exists.

~~~yaml
name: inferred-omics-env
channels:
  - conda-forge
  - bioconda
  - defaults
dependencies:
  - python
  - r-base
  # Add only packages observed in scripts, notebooks, logs, or workflow files.
~~~

## Open Questions
- Items that need human confirmation.
```

## QC Report Template

```markdown
# QC Metrics Report

Generated: YYYY-MM-DD HH:MM TZ
Project root: /path/to/project
Related inspection report: project_inspection_report.md

## QC Sources
| Source File | Tool / Workflow | Samples or Scope | Notes |
| --- | --- | --- | --- |

## Metric Interpretation
| Metric | Observed Value | Measures | Good / Bad Direction | Affected Aspect | Evidence |
| --- | --- | --- | --- | --- | --- |

## Sequencing Quality
Explain raw-read benchmarks found in the workspace.

## Mapping Quality
Explain alignment, mapping, and reference-fit benchmarks found in the workspace.

## Quantification and Sample Quality
Explain count, UMI, gene/feature, saturation, cell/sample filtering, and related benchmarks.

## Spatial, Imaging, or Assay-Specific Benchmarks
Explain registration, segmentation, decoding, spot/cell calling, blank rate, and related metrics when present.

## Caveats and Missing QC
List missing QC files, absent thresholds, assay-specific uncertainty, and items needing human confirmation.
```

## Common Mistakes

- Do not treat directory order as workflow order without script, log, or timestamp evidence.
- Do not ignore notebooks; they often contain the only write path for figures and `.h5ad` or `.rds` outputs.
- Do not put detailed QC metric interpretation in `project_inspection_report.md`; write the separate QC report and link to it.
- Do not summarize QC as "passed" or "failed" without explaining the metric, observed value, expected direction, and affected data aspect.
- Do not apply one universal QC threshold across assays. Report project/tool thresholds when present; otherwise state directionality and caveats.
- Do not generate an environment from memory alone. Derive it from imports, commands, lockfiles, and workflow configs.
- Do not hide uncertainty. A useful report distinguishes known provenance from educated guesses.

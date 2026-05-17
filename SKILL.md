---
name: omics-workspace-inspection
description: Use when a user asks to inspect bioinformatics, omics, sequencing, spatial transcriptomics, MERFISH, single-cell, RNA-seq, proteomics, or metabolomics project folders, especially when files, scripts, outputs, progress, or reproducibility are unclear.
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
5. Infer reproducibility requirements:
   - Prefer existing `environment.yml`, `conda-lock`, `renv.lock`, `requirements.txt`, `pyproject.toml`, `Dockerfile`, `Singularity`, `module load`, workflow configs, and notebook metadata.
   - If no environment exists, list observed tools and packages and propose a minimal conda environment with channels and package names.

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

## Common Mistakes

- Do not treat directory order as workflow order without script, log, or timestamp evidence.
- Do not ignore notebooks; they often contain the only write path for figures and `.h5ad` or `.rds` outputs.
- Do not generate an environment from memory alone. Derive it from imports, commands, lockfiles, and workflow configs.
- Do not hide uncertainty. A useful report distinguishes known provenance from educated guesses.

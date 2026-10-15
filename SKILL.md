---
name: pipeline-methods
description: Write or update METHODS.md for a bioinformatics project by scanning its pipeline scripts (bash, R, Python, Snakemake, Nextflow, SLURM/PBS job files) and listing, step by step in pipeline order, every tool, its version, a short description, the parameters used and their exact values. Use this whenever the user asks to document a pipeline, write methods, record "what did I run / which parameters did I use", summarize tools of an analysis, prepare a methods section for a thesis/paper/report, or wants a reproducibility record of a project — even if they don't say "METHODS.md" or "skill".
---

# Pipeline methods documenter

Bioinformatics projects pile up many tools across bash, R and Python. After weeks the user can no
longer remember which tool ran with which settings. This skill produces one plain, scannable
record — `METHODS.md` at the project root — built from the scripts that actually exist, so it can
be trusted and regenerated any time.

## Workflow

1. **Find the project root.** Default: current working directory. If the user names a folder, use it.
2. **Collect scripts.** Search recursively for `*.sh *.bash *.R *.Rmd *.qmd *.py *.ipynb *.smk
   Snakefile *.nf *.slurm *.pbs *.sbatch *.job`. Skip `.git`, `.snakemake`, `work/`, conda env
   dirs, `node_modules`, `renv/library`, `site-packages`, and obvious copies/backups. If two
   folders hold near-identical scripts (e.g. `analysis/` and `server_bundle/`), document the
   pipeline once and note in the header which script is the source and which copies exist,
   flagging any parameter that differs between the copies.
3. **Read each script fully** and extract tool calls in execution order. Respect step order
   given by comments/numbering, `if`/step-guards and driver scripts that call other scripts.
4. **Resolve values.** Replace shell variables with their defaults when the script defines them
   (`THREADS=${THREADS:-4}` → `4`). If a value is only known at runtime (e.g. `${DEPTH:?...}`),
   write `<runtime: $DEPTH>` and list it under "Unresolved" so the user can fill it in. Never
   invent a value — a wrong number in a methods record is worse than an honest gap.
5. **Find versions** (most reliable first): version stated in the script or its comments; the
   conda env the script activates (`conda list -p <env> <pkg>` or `conda list -n <env>`);
   `<tool> --version` inside that env; for R `packageVersion("pkg")`; for Python
   `pip show pkg`. If none works, write `version unknown`. Don't spend more than a couple of
   commands per tool on this.
6. **Catch unscripted steps.** People often run QC or one-off commands by hand, so scripts miss
   them. Glance at output folders for artifacts of tools no script calls (e.g. `*_fastqc.zip`,
   `multiqc_report.html`, `*.bam.bai`, Trimmomatic/fastp reports, `.qzv` from commands not in
   scripts). Many embed their version (`fastqc_data.txt` first line, MultiQC HTML "MultiQC vX").
   Put these into the pipeline at the position they logically ran (e.g. raw-read QC first),
   marked `[not scripted — parameters unknown]`, and don't guess their parameters. Also read
   logs the scripts themselves write (`tee logs/x.txt`) to resolve runtime values from step 4.
7. **Group into steps.** A step is one logical stage of the analysis (primer removal, denoising,
   taxonomy, diversity …). Use the script's own section headers/comments when they exist. A step
   may contain several tools or actions (e.g. denoise, then tabulate stats, then filter rare
   features).
8. **Write METHODS.md** in the format below. If it already exists, read it first, keep any
   section the user wrote by hand (anything outside the generated block), and replace only the
   generated block. Show the user a short summary of what changed.

## What counts as a tool / action / parameter

- **CLI tool with subcommands** (qiime, samtools, bcftools, gatk, bedtools, seqkit, plink2 …):
  tool = top-level program, or `program plugin` for QIIME 2 (`qiime dada2`); action = subcommand.
- **Plain CLI tool** (fastqc, fastp, bwa mem, minimap2, cutadapt, multiqc …): tool = program;
  action = the command name itself (or the mode, e.g. `bwa mem`).
- **R**: tool = package (`vegan`, `DESeq2`, `phyloseq`); action = function call; parameters =
  named arguments plus important positional ones (formula, method). Base R/stats functions that
  do analysis (e.g. `wilcox.test`, `p.adjust`) go under tool `R stats`. Skip plotting/data
  wrangling calls (ggplot2, dplyr, read.csv) unless they change the result (filtering thresholds,
  rarefying, normalisation) — those are methods.
- **Python**: same idea — package (`scikit-bio`, `scipy.stats`, `pandas` only for result-changing
  filters) and function. For a custom script whose logic is driven by constants
  (`MIN_ABUND = 0.001`), treat the constants as its parameters.
- **Parameters to include**: every option/argument that affects the result or the run
  (thresholds, lengths, methods, databases/reference names, models, threads, seeds, formulas,
  metadata columns, permutations). Name the reference database/classifier file — it's part of
  the method.
- **Parameters to leave out**: pure input/output path options (`--i-*`, `--o-*`, `--output-dir`,
  `-o out.bam`, `--input-path`) and `--verbose`/logging flags. They describe file plumbing, not
  the method, and double the length. Keep a path only if it identifies a reference (genome,
  classifier, database).
- **Actions with no method parameters** (e.g. `demux summarize`, `metadata tabulate`): keep them
  — they show the step happened — and write `(run as default)` as the value line.
- **Loops**: an action called in a loop with one varying argument is one action; write the
  varying values as a list, e.g. `--i-alpha-diversity {observed_features, shannon, faith_pd, evenness}`.

## Output format

Readability is the whole point, so the layout is strictly vertical: one parameter per line, with
the descriptions aligned into a column. The body follows pipeline order, numbered; each step is
self-contained (describe a tool and its parameters again in a later step if it reappears there,
but only the parameters used in that step), so the user can read any single step without
scrolling elsewhere.

Indentation levels are 4 spaces each:
- level 0: `N. Step name`
- level 1: `tool (version)` and, on the next line, its one-line description
- level 2: `Parameters:` block — one `--param   description` per line, descriptions aligned
- level 2: action name
- level 3: one `--param value` per line (or `(run as default)`)

Parameter descriptions are short (≤ ~8 words), in English, and say what the parameter *does*,
including what special values mean (e.g. `0 = no truncation`). Flags without a value
(e.g. `--p-discard-untrimmed`) appear as the bare flag. Put a blank line between tools and
between steps.

````markdown
# Methods — <project name>

<!-- pipeline-methods:start (generated; edits inside this block are overwritten) -->
Generated: <YYYY-MM-DD> from <script paths>
Environment: <conda env / module / container, if known>

Pipeline order:
1. <step name> — <tool action>
2. ...

```text
1. <Step name>
    <tool_name> (v<version>)
    <one-line description of what the tool does>
        Parameters:
            --param1    description
            --param2    description
        <action1>
            --param1 value
            --param2 value

    <tool_name2> (v<version>)
    ...

2. <Step name>
    ...
```

Unresolved:
- <param> in <script>:<line> — value set at runtime; fill in.
<!-- pipeline-methods:end -->
````

## Example (QIIME 2 excerpt)

```text
1. Raw read QC
    FastQC (v0.12.1)  [not scripted — parameters unknown]
    Per-sample read quality report
        fastqc
            (parameters unknown)

3. Primer removal
    qiime cutadapt (q2-cutadapt v2024.10.0; cutadapt v4.9)
    Removes primer/adapter sequences from demultiplexed reads
        Parameters:
            --p-front-f              5' forward primer to remove
            --p-front-r              5' reverse primer to remove
            --p-discard-untrimmed    drop read pairs where primer not found
            --p-cores                CPU cores
        trim-paired
            --p-front-f CCTAYGGGRBGCASCAG
            --p-front-r GGACTACNNGGGTATCTAAT
            --p-discard-untrimmed
            --p-cores 4

    qiime demux (q2-demux v2024.10.0)
    Summarizes reads per sample and per-position quality
        summarize
            (run as default)
```

## Finish

Tell the user, briefly: where the file is, how many steps/tools were documented, which
versions are unknown, which steps were not scripted, and the list of unresolved values. Don't
paste the whole file back into chat — they can open it.

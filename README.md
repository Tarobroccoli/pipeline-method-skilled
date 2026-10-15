# pipeline-methods — Claude Code skill

A [Claude Code](https://claude.com/claude-code) skill that writes `METHODS.md` for a
bioinformatics project. It scans the project's pipeline scripts (bash, R, Python, Snakemake,
Nextflow, SLURM/PBS) and records, step by step in pipeline order:

- every tool and its version (from the script, conda env or `--version`)
- a one-line description of the tool
- each parameter used, with a short description
- the exact value of each parameter per action

It also flags tools that left outputs but were never scripted (e.g. FastQC run by hand) and
values only known at runtime, instead of guessing them.

## Install

```bash
git clone https://github.com/Tarobroccoli/pipeline-method-skilled.git ~/.claude/skills/pipeline-methods
```

## Use

Inside a project folder, run `claude` and ask e.g. "write the methods for this pipeline", or
type `/pipeline-methods`. Re-running updates only the generated block; hand-written text outside
`<!-- pipeline-methods:start/end -->` is kept.

## Example output

```text
3. Primer removal
    qiime cutadapt (q2-cutadapt v2024.10.0; cutadapt v4.9)
    Removes primer/adapter sequences from demultiplexed reads
        Parameters:
            --p-front-f              5' forward primer to remove (341F)
            --p-front-r              5' reverse primer to remove (806R)
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

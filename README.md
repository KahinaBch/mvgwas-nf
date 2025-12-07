# mvgwas-nf

[![nextflow](https://img.shields.io/badge/nextflow-%E2%89%A522.04.0%2B-blue.svg)](http://nextflow.io)
[![CI-checks](https://github.com/dgarrimar/mvgwas-nf/actions/workflows/ci.yaml/badge.svg)](https://github.com/dgarrimar/mvgwas-nf/actions/workflows/ci.yaml)

A pipeline for multi-trait genome-wide association studies (GWAS) using [MANTA](https://github.com/dgarrimar/manta).

> **Note**: This is a fork of [dgarrimar/mvgwas-nf](https://github.com/dgarrimar/mvgwas-nf) with the following enhancements:
> - **DSL2 conversion**: Updated from legacy DSL1 to modern DSL2 syntax for compatibility with Nextflow ≥22.04.0
> - **MANOVA p-value**: Added classical MANOVA p-value computation alongside MANTA statistics for comparison
> - **Java 8 compatibility**: Tested and documented for HPC clusters running Java 8

The pipeline performs the following analysis steps:

* Split genotype file 
* Preprocess phenotype and covariate data
* Test for association between phenotypes and genetic variants using **both MANTA and MANOVA**
* Collect summary statistics

The pipeline uses [Nextflow](http://www.nextflow.io) as the execution backend. Please check [Nextflow documentation](http://www.nextflow.io/docs/latest/index.html) for more information.

## Requirements

- Unix-like operating system (Linux, MacOS, etc.)
- Java 8 or later 
- [Docker](https://www.docker.com/) (v1.10.0 or later) or [Singularity](http://singularity.lbl.gov) (v2.5.0 or later)

## Quickstart (~2 min)

1. Install Nextflow:
    ```
    curl -fsSL get.nextflow.io | bash
    ```

2. Make a test run:
    ```
    nextflow run KahinaBch/mvgwas-nf -with-docker
    ```

**Notes**: move the `nextflow` executable to a directory in your `$PATH`. Set `-with-singularity` to use Singularity instead of Docker.

(*) Alternatively you can clone this repository:
```
git clone https://github.com/KahinaBch/mvgwas-nf
cd mvgwas-nf
nextflow run mvgwas.nf -with-docker
```

**Important**: This pipeline has been updated to DSL2 syntax and is compatible with newer Nextflow versions. However, if you are using Java 8, you need to use Nextflow version 22.04.0 or compatible versions that support Java 8.
This can be done using `NXF_VER` before Nextflow commands, e.g. `NXF_VER=22.04.0 nextflow run mvgwas.nf -with-docker`.

## Pipeline usage

Launching the pipeline with the `--help` parameter shows the help message:

```
nextflow run mvgwas.nf --help
```

```
N E X T F L O W  ~  version 20.04.1
Launching `mvgwas.nf` [amazing_roentgen] - revision: 56125073b7

mvgwas-nf: A pipeline for multivariate Genome-Wide Association Studies
==============================================================================================
Performs multi-trait GWAS using using MANTA (https://github.com/KahinaBch/manta)

Usage:
nextflow run mvgwas.nf [options]

Parameters:
--pheno PHENOTYPES          phenotype file
--geno GENOTYPES            indexed genotype VCF file
--cov COVARIATES            covariate file
--l VARIANTS/CHUNK          variants tested per chunk (default: 10000)
--t TRANSFOMATION           phenotype transformation: none, sqrt, log (default: none)
--i INTERACTION             test for interaction with a covariate: none, <covariate> (default: none)
--ng INDIVIDUALS/GENOTYPE   minimum number of individuals per genotype group (default: 10)
--dir DIRECTORY             output directory (default: result)
--out OUTPUT                output file (default: mvgwas.tsv)
```

## Input files and format

`mvgwas-nf` requires the following input files:

* **Genotypes.** 
[bgzip](http://www.htslib.org/doc/bgzip.html)-compressed and indexed [VCF](https://samtools.github.io/hts-specs/VCFv4.3.pdf) genotype file.

* **Phenotypes.**
Tab-separated file with phenotype measurements (quantitative) for each sample (i.e. *n* samples x *q* phenotypes).
The first column should contain sample IDs. Columns should be named.

* **Covariates.**
Tab-separated file with covariate measurements (quantitative or categorical) for each sample (i.e. *n* samples x *k* covariates). 
The first column should contain sample IDs. Columns should be named. 

Example [data](data) is available for the test run.

## Pipeline results

An output text file containing the multi-trait GWAS summary statistics (default: `./result/mvgwas.tsv`), with the following information:

* `CHR`: chromosome
* `POS`: position
* `ID`: variant ID
* `REF`: reference allele
* `ALT`: alternative allele
* `F_manta`: pseudo-F statistic from MANTA
* `R2_manta`: fraction of variance explained by the variant (MANTA)
* `P_manta`: P-value from MANTA (non-parametric, permutation-based)
* `P_manova`: P-value from classical MANOVA (Pillai's trace)

When using the interaction option (`--i`), additional columns are provided for the covariate, genotype, and interaction effects.

The output folder and file names can be modified with the `--dir` and `--out` parameters, respectively.

## Differences from Original Pipeline

This fork extends the original [dgarrimar/mvgwas-nf](https://github.com/dgarrimar/mvgwas-nf) with:

| Feature | Original | This Fork |
|---------|----------|-----------|
| Nextflow syntax | DSL1 (deprecated) | DSL2 (modern) |
| Statistical tests | MANTA only | MANTA + MANOVA |
| Output columns | F, R2, P | F_manta, R2_manta, P_manta, P_manova |
| Java 8 support | Limited docs | Fully documented |

### Why add MANOVA?

MANTA uses a fast non-parametric permutation-based approach, while classical MANOVA uses Pillai's trace statistic with parametric assumptions. Including both allows:
- Comparison of parametric vs non-parametric results
- Validation of significant associations
- Assessment of the impact of distributional assumptions

## Cite mvgwas-nf

If you find `mvgwas-nf` useful in your research please cite the related publication:

Garrido-Martín, D., Calvo, M., Reverter, F., Guigó, R. A fast non-parametric test of association for multiple traits. *Genome Biol* **24**, 230 (2023). [https://doi.org/10.1186/s13059-023-03076-8](https://doi.org/10.1186/s13059-023-03076-8)


# Reproducibility Document

**Project Title**: *Past Populations and Future Threats: Genome-Inferred Demography of Vertebrates*  
**Author**: Minoli Daigavane  
**Institution**: University of California, Berkeley  
**Date**: May 2025  

---

## Repository Structure

```
population-inference/
├── README.md               # This reproducibility document
├── code/                   # All executable code and notebooks
│   ├── iucn_data.ipynb
│   ├── location_data.ipynb
│   ├── psmc_preprocessing.ipynb
│   ├── gbif_df.rds
│   ├── run_pipe.sh
│   └── Snakefile
├── metadata/               # Input metadata files
├── plots/                  # Final figures
```

---

## 1. Project Overview

This study reconstructs historical effective population sizes (Ne) using whole-genome assemblies for 499 vertebrate species. Using the Pairwise 
Sequentially Markovian Coalescent (PSMC) model, we estimate Ne trajectories and evaluate whether genome-based demographic history aligns with IUCN 
conservation status. The pipeline includes:

- Geographic and taxonomic metadata resolution  
- IUCN Red List status retrieval and integration  
- Phylogenetic estimation of missing mutation/gen parameters  
- Automated PSMC modeling on an HPC cluster  
- Interactive Shiny browser for demographic exploration

This document provides a full walkthrough of how to reproduce all results.

---

## 2. ️ Environment Setup

The pipeline is built for Linux/macOS systems with `conda`, `snakemake`, and `R` ≥ 4.2. Set up the environment using:

```bash
conda env create -f code/envs/environment.yml
conda activate psmc_env
```

To use R notebooks, install IRkernel in R:

```r
install.packages("IRkernel")
IRkernel::installspec()
```

---

## 3. Step 1: Geographic Metadata & Sample Locations

**Notebook**: `code/location_data.ipynb`  
**Input**: `assembly_metadata_long.tsv`, NCBI `.jsonl` files  
**Output**: `location_data_cleaned.csv`, maps in `plots/`

Processes location data using:

- `tidygeocoder` to geocode sample descriptions  
- `sf` + `ggplot2` for global + regional map visualizations  
- Manual fixes for samples without coordinates

Final figures include:

- Goode homolosine world map  
- Insets for Europe and North America  
- Jittered points for dense locations

---

## 4.  Step 2: IUCN Red List Retrieval

**Notebook**: `code/iucn_data.ipynb`  
**Input**: `assembly_metadata_wide.tsv`  
**Output**:  
- `historic_iucn_df.tsv` — all assessments  
- `latest_iucn_df.tsv` — latest status per species

Retrieves IUCN status using:

- `rredlist` API  
- GBIF synonym fallback via `taxadb`  
- Manual overrides for known unmatched species  
- Assignment of "NE" to species with no assessment

Includes a stacked bar plot of status distributions by taxonomic group.

---

## 5.  Step 3: Parameter Estimation for PSMC

**Notebook**: `code/psmc_preprocessing.ipynb`  
**Input**:  
- `my_species.txt`, `wang_species.txt`  
- `mutation_rate_literature_updating3.xlsx`  
- `assembly_metadata_wide.tsv`, `downloaded_assemblies.tsv`  
**Output**: `final_species_list.tsv`

### A. R Section
- Uses `rotl` and Open Tree of Life API  
- Matches each study species to its closest Wang et al. (2023) relative

### B. Python Section
- Merges mutation rate (`mu`) and generation time (`gen`)  
- Fills gaps with clade-level averages

---

## 6.  Step 4: Running the Snakemake Workflow

**Location**: `code/Snakefile`  
**Execution:**

```bash
cd code/
snakemake --use-conda --jobs 100 --profile slurm_profile
```

Uses SLURM cluster with `run_pipe.sh` and `slurm_profile`. Resources are defined in `cluster_config.yaml`.

---

## 7.  Output Structure

After execution, expect:

- `outputs/psmc/` — raw `.psmc` outputs  
- `scaled_outputs/` — scaled Ne/time curves  
- `plots/` — all figures  
- `location_data_cleaned.csv` and `latest_iucn_df.tsv` — for plotting

---

## 8.  Final Table Exports

All key tables can be exported from notebooks:

- `final_species_list.tsv` — input species list with parameters  
- `historic_iucn_df.tsv`, `latest_iucn_df.tsv` — for conservation comparisons

These live in `code/` or `plots/`.

---

## 9.  Figures Generated

- Global and regional maps of sample locations  
- IUCN category distribution per clade  
- Individual and group-level PSMC curves  
- Summary plots of median Ne across time

---

## 10.  Reproducibility Notes

- All `.ipynb` notebooks run top-to-bottom  
- IUCN and GBIF lookups are cached to avoid API throttling  
- Manual corrections are documented in code  
- Conda and IRkernel environments guarantee version control

---



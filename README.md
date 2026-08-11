# sandes-etal-antibiotic-dosing-errors-vigimed-pds
# Reproducible Processing and Analysis of VigiMed Reports for the Article Entitled “Reported Dosing-Related Medication Errors Involving Systemic Antibiotics in the Brazilian Pharmacovigilance Database (2018–2025): Distribution Across the WHO AWaRe Classification”

## Overview

This repository contains the Python code used to prepare and analyse data for the associated scientific article. Its purpose is to support reproducibility and enable transparent review of the record linkage, hierarchical deduplication, and harmonisation procedures, as well as the statistical modelling and figures presented in the study.

The code is organised in three notebooks that must be run in order:

1. `Pipeline_Sandes_et_al_Antibiotic_dosing-related_errors_in_Brazil.ipynb` — data preparation (cleaning, deduplication, linkage, harmonisation, and selection), which generates the analytical datasets.
2. `Analise_1_so1grupo_Sandes_et_al.ipynb` — medication-error group comparison.
3. `Analise_2_aware_Sandes_et_al.ipynb` — WHO AWaRe classification and dosing-related errors.

The two analysis notebooks read files produced by the pipeline and therefore cannot be run before it.

The source data are open pharmacovigilance data from VigiMed, the Brazilian spontaneous reporting system maintained by the Brazilian Health Regulatory Agency (Anvisa).

## Expected input files

Place the following files in the input directory specified in the code:

- `harmonizacao_2.csv` — active-substance harmonisation table provided in this repository;
- `Mapeamento_aware.xlsx` — lookup table mapping each harmonised antibiotic name to its WHO AWaRe class, provided in this repository and used by the AWaRe analysis; and
- `SMQ Medication Error.xlsx` — MedDRA Standardised MedDRA Query (SMQ) terms used to identify medication errors (Brazilian Portuguese, version 28.1). The file is available from MedDRA after logging in at https://tools.meddra.org/wbb/. Users without access may request it by contacting MedDRA at https://www.meddra.org/contact.

The MedDRA/SMQ file is not redistributed in this repository because it is subject to licensing restrictions. Users must obtain an appropriately licensed copy and provide it locally before running the relevant stages of the pipeline.

Release contains the VigiMed source data extracts used in the study. The files were downloaded from the Brazilian Open Data Portal in November 2025 and are provided to support the exact reproduction of the data-processing and analytical procedures.

Included files:

- VigiMed_Notificacoes.csv
- VigiMed_Medicamentos.csv
- VigiMed_Reacoes.csv

Download the three files and place them in the same directory as the analysis notebooks before running the pipeline.

The current versions of the VigiMed datasets remain available from the Brazilian Open Data Portal. However, they may differ from the archived extracts used in this study.

The MedDRA/SMQ file is not included because it is subject to MedDRA licensing conditions.
## Computing environment

The analysis was developed using Python 3.13.5. Package versions required to reproduce the environment are listed in `requirements.txt` (pandas, numpy, python-docx, openpyxl, statsmodels, matplotlib, and seaborn).

## Installation

Create and activate a virtual environment, then install the dependencies from the repository root:

```bash
python -m venv .venv
```

On Windows:

```bash
.venv\Scripts\activate
pip install -r requirements.txt
```

On macOS or Linux:

```bash
source .venv/bin/activate
pip install -r requirements.txt
```

## Pipeline

The processing stages should be run in the order implemented in the analysis code:

1. Initial cleaning of the VigiMed source tables.
2. Parsing and processing of date variables.
3. Normalisation of relevant identifiers and text fields.
4. Two-level hierarchical deduplication of reports.
5. Removal of duplicate records within the medicines and reactions tables.
6. Record linkage and merging of report-, medicine-, and reaction-level data.
7. Exclusion of records with missing Preferred Terms (PTs) or active substances, and exclusion of fetal records.
8. Selection of suspected medicines and medicines reported as not administered.
9. Harmonisation of active-substance names using `harmonizacao_2.csv`.
10. Hierarchical derivation of age from the available age-related variables.
11. Classification into age groups.
12. Restriction to ATC group J01, followed by the exclusion of items that do not represent antibacterials for systemic use.
13. Application of the Medication Error SMQ.
14. Subselection of dose-error reports.

The order is important because later selections rely on variables and linked records produced during earlier stages. Running the pipeline generates the analytical datasets used by the analysis notebooks, including `filtered_j01_suspeito_nao_adm_y_analise_1225.csv` and `arquivo_filtrado_smq_erro_de_medicacao_y.csv`.

## Analysis notebooks

After the pipeline has produced the analytical subsets, two notebooks run the study analyses. They read files generated by the pipeline and must be run after it, top to bottom in a single kernel session.

### `Analise_1_so1grupo_Sandes_et_al.ipynb` — Medication-error group comparison

Input: `filtered_j01_suspeito_nao_adm_y_analise_1225.csv` (produced by the pipeline).

This notebook classifies J01 medication–event pairs into analytic groups (no medication error; medication error without a dosing component; dosing-related error) and compares any medication error versus no error. Crude and adjusted prevalence ratios (PR) are estimated using Poisson regression with robust variance clustered by notification.

Outputs: a descriptive table and the prevalence-ratio table (PR with 95% confidence intervals) as Word documents, and forest-plot figures (PNG/PDF).

### `Analise_2_aware_Sandes_et_al.ipynb` — WHO AWaRe classification and dosing-related errors

Inputs: `arquivo_filtrado_smq_erro_de_medicacao_y.csv` (produced by the pipeline) and `Mapeamento_aware.xlsx` (provided in this repository).

This notebook classifies antibiotics according to the WHO AWaRe framework and models the proportion of dosing-related errors among medication-error drug–event pairs by AWaRe class, adjusted for age group, sex, and reporting source (Poisson regression with robust variance clustered by notification, reference AWaRe class = Access). It includes two sensitivity analyses: one restricted to drug–event pairs in which the antibiotic was reported as a suspect medication, and one additionally adjusted for calendar year.

Outputs: the AWaRe prevalence-ratio table and the sensitivity and temporal tables as Word documents; Figure 3 (a heatmap of the proportion of dosing-related errors by antibiotic and age group, with the stratum denominator shown in each cell) and a supplementary figure of drug–event pairs by year (PNG/PDF).

## Running the analysis

1. Obtain a licensed copy of the required MedDRA/SMQ file.
2. Add all expected input files to the input location configured in the code.
3. Create the Python environment and install `requirements.txt` as described above.
4. Run the pipeline notebook from data cleaning through dose-error subselection; this generates the intermediate and analytical datasets, including the two files consumed by the analysis notebooks.
5. Run `Analise_1_so1grupo_Sandes_et_al.ipynb` from top to bottom.
6. Run `Analise_2_aware_Sandes_et_al.ipynb` from top to bottom.

## Main outputs

The pipeline produces the principal analytical datasets used in the article, including:

- cleaned report-, medicine-, and reaction-level data;
- the hierarchically deduplicated report dataset;
- the linked report–medicine–reaction dataset;
- data with harmonised active substances and hierarchically derived age variables;
- the analytical subset of suspected or non-administered systemic antibacterials;
- reports matching the Medication Error SMQ; and
- the final dose-error subset used in the study analysis.

The analysis notebooks produce the tables and figures reported in the article, including the medication-error and AWaRe prevalence-ratio tables (Word), the sensitivity and temporal tables (Word), the forest-plot figures, the AWaRe heatmap (Figure 3), and the supplementary by-year figure (PNG/PDF).

Output filenames and locations follow the settings defined in the analysis code. Intermediate files are retained where required to audit the linkage, deduplication, and harmonisation steps.

## Reproducibility scope

The code is provided to document the transformations applied to the source data and to facilitate scientific review. In particular, it allows reviewers and other researchers to inspect:

- the linkage of VigiMed report, medicine, and reaction tables;
- the rules and hierarchy used for deduplication;
- the mapping and harmonisation of active-substance names; and
- the classification and statistical modelling underlying the reported tables and figures.

Results may depend on the versions of the VigiMed extracts, MedDRA/SMQ terminology, harmonisation table, and Python packages used.

## Citation

If you use this repository, please cite the associated article and repository:

> [ARTICLE CITATION TO BE ADDED]
>
> DOI: [DOI TO BE ADDED]



# CS598-DL4H-Reproducibility-Project

# About

This project is a reproducibility study of [MIMIC-Extract:A Data Extraction, Preprocessing, and Representation Pipeline for MIMIC-III](https://github.com/mz4987/MIMIC_Extract). The citation of the original paper is listed below. We have re-used the codes provide by this study for this reproduction.

```
Shirly Wang, Matthew B. A. McDermott, Geeticka Chauhan, Michael C. Hughes, Tristan Naumann, 
and Marzyeh Ghassemi. MIMIC-Extract: A Data Extraction, Preprocessing, and Representation 
Pipeline for MIMIC-III. arXiv:1907.08322. 
```
The ouptut of this data pipeline is provided by the authors [here].(https://console.cloud.google.com/storage/browser/mimic_extract).

# Pre-processed Output
Vist [MIMIC-III doc](https://mimic.mit.edu/docs/gettingstarted/) for more information and downoad of the data.

## Step 0: Required software and prereqs

To complete the reproduction, we need the following preparation. 

* conda
* psql (PostgreSQL 9.4 or higher)
* git
* MIMIC-iii psql relational database (Refer to [MIT-LCP Repo](https://github.com/MIT-LCP/mimic-code))
* [MIT-LCP Repo](https://github.com/MIT-LCP/mimic-code) code assumed to store in the environment variable `$MIMIC_CODE_DIR`

## Step 1: Create conda environment

Next, make a new conda environment using [mimic_extract_env_py36.yml](../mimic_extract_env_py36.yml).

```
conda env create --force -f ../mimic_extract_env_py36.yml
```
If this environment fails, simply install the missing package with `pip install <package name>`and re-run the command 
Aactivate your environment for following steps.

```
conda activate mimic_data_extraction
```

## Step 3: Build MIMIC-III Concepts Table Views for Feature Extraction

Before working on this step, make sure you have the MIMIC PostgreSQL database generated. This includes all concept tables in [MIT-LCP Repo](https://github.com/MIT-LCP/mimic-code) 

```
cd $MIMIC_CODE_DIR/concepts
psql -d mimic -f postgres-functions.sql
bash postgres_make_concepts.sh
```

## Step 4: Build Additional Tables for Feature Extraction 

Navigate to `utils` folder
```
bash postgres_make_extended_concepts.sh
psql -d mimic -f niv-durations.sql
```
This command will build 3 additional materialized views necessary for this pipeline. 

## Step 5: Set Cohort Selection and Extraction Criteria

Navigate to the root directory, activate your conda environment with
```
conda activate mimic_data_extraction
```
Run`python mimic_direct_extract.py ...` with your args as desired to start data extraction

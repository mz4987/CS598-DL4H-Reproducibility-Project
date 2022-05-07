# CS598-DL4H-Reproducibility-Project

# About

This project is a reproducibility study of [MIMIC-Extract:A Data Extraction, Preprocessing, and Representation Pipeline for MIMIC-III](https://github.com/mz4987/MIMIC_Extract). The citation of the original paper is listed below. We have re-used the codes provide by this study for this reproduction.

```
Shirly Wang, Matthew B. A. McDermott, Geeticka Chauhan, Michael C. Hughes, Tristan Naumann, 
and Marzyeh Ghassemi. MIMIC-Extract: A Data Extraction, Preprocessing, and Representation 
Pipeline for MIMIC-III. arXiv:1907.08322. 
```
The ouptut of this data pipeline is provided by the authors [here](https://console.cloud.google.com/storage/browser/mimic_extract).

# Data Download Instruction
Vist [MIMIC-III doc](https://mimic.mit.edu/docs/gettingstarted/) for more information and downoad the data. The MIMIC-III can be downloaded by following steps.

* Complete CITI training course. Select “Massachusetts Institute of Technology Affiliates” as your organization affiliation and “Data or Specimens Only Research” course.
* Create a PhysioNet account using Illinois email.
* File an application to be a credentialed user.
* Upload training report.
* Sign data use agreement.
* Download MIMIC III database.


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

## Step 2: Build MIMIC-III Concepts Table Views for Feature Extraction

Before working on this step, make sure you have the MIMIC PostgreSQL database generated. This includes all concept tables in [MIT-LCP Repo](https://github.com/MIT-LCP/mimic-code) 

```
cd $MIMIC_CODE_DIR/concepts
psql -d mimic -f postgres-functions.sql
bash postgres_make_concepts.sh
```

## Step 3: Build Additional Tables for Feature Extraction 

Navigate to `utils` folder
```
bash postgres_make_extended_concepts.sh
psql -d mimic -f niv-durations.sql
```
This command will build 3 additional materialized views necessary for this pipeline. 

## Step 4: Set Cohort Selection and Extraction Criteria

Navigate to the root directory, activate your conda environment with
```
conda activate mimic_data_extraction
```
Run`python mimic_direct_extract.py ...` with your args as desired to start data extraction.

This process may take over 15 hours depending on your machine. The default setting will create an hdf5 file inside MIMIC_EXTRACT_OUTPUT_DIR with four tables.
Use `mimic_direct_extract.ipynb` for test your output extracted by MIMIC-Extract.

# Step 5: Test Extraction
`Testing mimic_direct_extract.ipynb` contains tests for different data processing funcations in **MIMIC-Extract**.
Results of this test are shown in `Result.xlsx`.

|           |            | Male  | Female | Total |
|-----------|------------|-------|--------|-------|
| Ethnicity | Asian      | 370   | 472    | 842   |
|           | Hispanic   | 448   | 689    | 1137  |
|           | Black      | 1448  | 219    | 2667  |
|           | Other      | 2061  | 3122   | 5183  |
|           | White      | 10651 | 13992  | 24643 |
| Age       | $<$30      | 748   | 1084   | 1832  |
|           | 30-50      | 2212  | 3277   | 5489  |
|           | 50-70      | 4888  | 8050   | 12983 |
|           | $>$70      | 7130  | 7083   | 14213 |
| Insurance | Self       | 125   | 352    | 477   |
|           | Government | 402   | 648    | 1050  |
|           | Medicaid   | 1186  | 1596   | 2782  |
|           | Private    | 4415  | 7431   | 1186  |
|           | Medicare   | 8850  | 9467   | 18317 |
| Admission Type | Urgent     | 409   | 528    | 937   |
|           | Elective   | 2282  | 3423   | 5705  |
|           | Emergency  | 12287 | 15543  | 27830 |
| First Careunit    | TSICU      | 1777  | 2725   | 4502  |
|           | CCU        | 2185  | 3008   | 5193  |
|           | SICU       | 2678  | 2842   | 5520  |
|           | CSRU       | 2326  | 4724   | 7050  |
|           | MICU       | 6012  | 6195   | 12207 |
| Total     |            | 14978 | 19494  | 34472 |

Our pipeline outputs exactly same result as shown in the table 5 of the original study, indicating that we have successfully reproduce the MIMIC-III pipeline which is available for prediction tasks. 

# Step 6: Train Models and Evaluate Results  

Models are provided in notebooks. 
Results of this test are shown in `Result.xlsx`.

Mortality and Length-of-stay (LOS) Predictions models are demonstrated as `Baselines for Mortality and LOS prediction - Sklearn.ipynb` and `Baselines for Mortality and LOS prediction - GRU-D.ipynb`.
| Task                  | Model |            | AUROC | AUPRC  | Accuracy | F1     |
|-----------------------|-------|------------|-------|--------|----------|--------|
| In-ICU Mortality      | LR    | Original   | 88.7  | 46.4   | 93.4     | 38.4   |
|                       |       | Reproduced | 86.0  | 43.0   | 93.2     | 37.9   |
|                       |       | Diff (%)   | -3.0% | -7.3%  | -0.2%    | -1.3%  |
|                       | RF    | Original   | 89.7  | 49.8   | 93.3     | 12.6   |
|                       |       | Reproduced | 89.0  | 50.3   | 93.3     | 12.1   |
|                       |       | Diff (%)   | -0.7% | 1.1%   | 0.0%     | -4.4%  |
|                       | GRU-D | Original   | 89.1  | 50.9   | 94.0     | 43.1   |
|                       |       | Reproduced | 86.9  | 46.1   | 93.7     | 44.0   |
|                       |       | Diff (%)   | -2.4% | -9.5%  | -0.3%    | 2.0%   |
| In-Hospital Mortality | LR    | Original   | 85.6  | 49.1   | 91.1     | 42.1   |
|                       |       | Reproduced | 84.8  | 47.1   | 90.7     | 40.3   |
|                       |       | Diff (%)   | -0.9% | -4.1%  | -0.4%    | -4.3%  |
|                       | RF    | Original   | 86.7  | 53.1   | 90.7     | 19.6   |
|                       |       | Reproduced | 86.2  | 51.5   | 90.4     | 13.5   |
|                       |       | Diff (%)   | -0.6% | -3.1%  | -0.3%    | -31.0% |
|                       | GRU-D | Original   | 87.6  | 53.2   | 91.7     | 44.8   |
|                       |       | Reproduced | 85.9  | 50.5   | 91.1     | 45.3   |
|                       |       | Diff (%)   | -1.9% | -5.1%  | -0.7%    | 1.2%   |
| LOS > 3 Days          | LR    | Original   | 71.6  | 65.1   | 68.6     | 59.4   |
|                       |       | Reproduced | 70.0  | 63.8   | 66.8     | 55.6   |
|                       |       | Diff (%)   | -2.2% | -2.0%  | -2.6%    | -6.4%  |
|                       | RF    | Original   | 73.6  | 68.5   | 69.5     | 59.5   |
|                       |       | Reproduced | 73.1  | 67.7   | 69.6     | 58.8   |
|                       |       | Diff (%)   | -0.6% | -1.2%  | 0.2%     | -1.2%  |
|                       | GRU-D | Original   | 73.3  | 68.5   | 68.3     | 62.2   |
|                       |       | Reproduced | 72.9  | 67.3   | 69.3     | 60.1   |
|                       |       | Diff (%)   | -0.5% | -1.8%  | 1.4%     | -3.3%  |
| LOS > 7 Days          | LR    | Original   | 72.4  | 18.5   | 91.9     | 7.2    |
|                       |       | Reproduced | 67.2  | 13.8   | 91.6     | 2.9    |
|                       |       | Diff (%)   | -7.2% | -25.4% | -0.3%    | -59.7% |
|                       | RF    | Original   | 76.4  | 19.5   | 92.3     | 0.0    |
|                       |       | Reproduced | 75.9  | 19.4   | 92.3     | 0.0    |
|                       |       | Diff (%)   | -0.7% | -0.4%  | 0.0%     | 0.0%   |
|                       | GRU-D | Original   | 71.0  | 17.9   | 91.2     | 10.7   |
|                       |       | Reproduced | 73.1  | 17.2   | 91.4     | 7.2    |
|                       |       | Diff (%)   | 2.9%  | -3.9%  | 0.2%     | -32.6% |


Accuracy is within 5% compared with reported values in the original study for all of our reproduced models. Especially for LOS>7, In-ICU Mortality and  In-Hospital Mortality predictions, the differences are below 1%. Our results confirm that RF and GRU-D models have higher performance compare with LR models. 

Clinical Intervention Prediction models are demonstrated as `Baselines for Intervention Prediction - Mechanical Ventilation.ipynb` and `Baselines for Intervention Prediction - Vasopressor.ipynb`.

|                           | RF     | RF     | LR      | LR     | CNN    | CNN     | LSTM    | LSTM   |
|---------------------------|--------|--------|---------|--------|--------|---------|---------|--------|
|                           | Vent.  | Vaso.  | Vent.   | Vaso.  | Vent.  | Vaso.   | Vent.   | Vaso.  |
| Onset AUROC Original      | 87.1   | 71.6   | 71.9    | 68.4   | 72.2   | 69.4    | 70.1    | 71.9   |
| Onset AUROC Reproduced    | 84.9   | 69.9   | 71.8    | 68.6   | 71     | 64.2    | 69.7    | 72.1   |
| Diff (\%)                 | -2.5% | -2.4% | -0.1%  | 0.3%  | -1.7% | -7.5%  | -0.6%  | 0.3%  |
| Wean AUROC Original       | 94     | 94.2   | 93.2    | 93.9   | 93.9   | 94      | 93.1    | 93.9   |
| Wean AUROC Reproduced     | 98.9   | 98.1   | 98.2    | 98.2   | 98.3   | 98      | 98.3    | 98     |
| Diff (\%)                 | 5.2%  | 4.1%  | 5.4%   | 4.6%  | 4.7%  | 4.3%   | 5.6%   | 4.4%  |
| Stay On AUROC Original    | 98.5   | 98.5   | 98.4    | 98.2   | 98.6   | 98.4    | 98.3    | 98.3   |
| Stay On AUROC Reproduced  | 98.5   | 98.4   | 98.4    | 98.4   | 98.5   | 98.3    | 98.3    | 98.3   |
| Diff (\%)                 | 0.0%  | -0.1% | 0.0%   | 0.2%  | -0.1% | -0.1%  | 0.0%   | 0.0%  |
| Stay Off AUROC Original   | 99     | 98.3   | 98.3    | 98.5   | 98.4   | 98.1    | 98.4    | 98.1   |
| Stay Off AUROC Reproduced | 93.9   | 94     | 92.9    | 93.9   | 93.7   | 93.6    | 93.2    | 94     |
| Diff (\%)                 | -5.2% | -4.5% | -5.5%  | -4.7% | -4.8% | -4.6%  | -5.3%  | -4.2% |
| Macro AUROC Original.     | 94.6   | 90.7   | 90.4    | 89.8   | 90.8   | 90      | 90      | 90.1   |
| Macro AUROC Reproduced    | 94     | 90.1   | 90.3    | 89.8   | 90.4   | 88.5    | 89.9    | 90.6   |
| Diff (\%)                 | -0.6% | -0.7% | -0.1%  | 0.0%  | -0.4% | -1.7%  | -0.1%  | 0.6%  |
| Accuracy Original.        | 79.7   | 83.8   | 78.5    | 72.9   | 61.8   | 77.6    | 84.3    | 82.6   |
| Accuracy Reproduced       | 81.6   | 83.9   | 69.6    | 76.5   | 58.5   | 56.3    | 73      | 80.1   |
| Diff (\%)                 | 2.4%  | 0.1%  | -11.3% | 4.9%  | -5.3% | -27.4% | -13.4% | -3.0% |
| Macro F1 Original         | 48.1   | 48.9   | 47.7    | 45.1   | 44.4   | 44.4    | 50.1    | 48.1   |
| Macro F1 Reproduced       | 48.9   | 48.3   | 44.8    | 45.8   | 41.6   | 42.1    | 47.5    | 46.3   |
| Diff (\%)                 | 1.7%  | -1.2% | -6.1%  | 1.6%  | -6.3% | -5.2%  | -5.2%  | -3.7% |
| Macro AUPRC Original.     | 42.7   | 42     | 43.1    | 40.2   | 42.4   | 38.9    | 44.4    | 41.7   |
| Macro AUPRC Reproduced    | 43.1   | 41.3   | 41.5    | 40.3   | 40.4   | 40.5    | 43.5    | 40.2   |
| Diff (\%)                 | 0.9%  | -1.7% | -3.7%  | 0.2%  | -4.7% | 4.1%   | -2.0%  | -3.6% |

The accuracy differences of RF Vent. prediction model, RF Vaso. prediction model, LR Vaso. prediction model, and LSTM Vaso. prediction model are within 5% range compared with original results. While the results of CNN models are significantly different from original results.

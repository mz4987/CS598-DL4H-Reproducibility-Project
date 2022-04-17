# **CS598-DL4H-Reproducibility-Project

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


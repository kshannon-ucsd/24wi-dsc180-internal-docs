---
title: "Data Team Documentation"
date: 2025-02-09
---

# Subsetting and Preprocessing Data for First Model

## Overview
The document guides you through how to run the SQL and Python script to obtain the data needed for the CNN X-Ray Classifier model

---

## Prerequisites

1. Download the MIMIC 4 Database and concept tables by following the instructions in `Obtaining Mimic 4 Database` file
2. Have postgreSQL installed on your system
3. Install the `pandas` package using `pip install pandas`
4. Download `mimic-cxr-2.0.0-metadata.csv.gz` from [Mimic-CXR-JPG](https://physionet.org/content/mimic-cxr-jpg/2.1.0/) dataset on Physionet
    - Extract the csv file and move it into `data/external` folder in `24-wi-dsc180-project` repo
5. Download `CXLSeg-segmented.csv` from [Chest X-ray Dataset with Lung Segmentation](https://physionet.org/content/chest-x-ray-segmentation/1.0.0/) dataset on Physionet
    - Move csv file into `data/external` folder in `24-wi-dsc180-project` repo

---

## Key Files
1. `data_subset.sql` - Performs the queries to create the subset and saves output to `data/interim` folder
2. `subset_preproc.py` - Performs the queries to create the subset and saves output to `data/processed` folder

---

## Running the Scripts

1. Navigate to `scripts` folder in `24-wi-dsc180-project` repo through terminal
2. Connect to postgreSQL through postgres user `psql -U postgres`
3. Connect to mimic database through `\c mimiciv`
4. Run `\i data_subset.sql`
5. Disconnect from postgreSQL through `\q`
2. Run `py subset_preproc.py` in terminal 
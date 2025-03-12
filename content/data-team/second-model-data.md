---
title: "Data Team Documentation"
date: 2025-03-12
---

# Subsetting Data for Second Model

## Overview
The document guides you through how to run the SQL and Python script to obtain the data needed for the CatBoost Sepsis Predictor model.

---

## Prerequisites

1. Download the MIMIC 4 Database and concept tables by following the instructions in `Obtaining Mimic 4 Database` file
2. Have postgreSQL installed on your system

---

## Key Files
1. `patients_metadata.sql` - Performs the queries to create the subset of patient metadata and saves output to `data/sql-data` folder

---

## Running the Scripts

1. Navigate to `etl/sql` folder in `24-wi-dsc180-project` repo through terminal
2. Connect to postgreSQL through postgres user `psql -U postgres`
3. Connect to mimic database through `\c mimiciv`
4. Run `\i patients_metadata.sql`
5. Disconnect from postgreSQL through `\q`

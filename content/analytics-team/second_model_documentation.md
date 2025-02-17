---
title: "Second Model Baseline Documentation"
date: 2025-02-16
---




## Overview
This script is designed for preprocessing MIMIC dataset files, training a machine learning model, and evaluating its performance. It processes patient metadata, extracts features, and applies a RandomForestClassifier for classification tasks related to medical conditions.

## Data Preprocessing
The preprocessing function `preprocess_data` takes four CSV files as input:
- **mimic**: Contains patient admission data from the mimiciv database.
- **feats**: Contains selected features from the mimiciv database (SQL script used to collect features)
- **segm**: Segmentation info on xrays that includes unique dicom id
- **metadata**: Metadata on the xrays matched with the segmentation data that gives abnormality value.

### Steps in Preprocessing:
1. **Load Data**: Reads the provided CSV files into Pandas DataFrames.
2. **Date and Time Formatting**: Functions `date_format`, `time_format`, and `convert_datetime` ensure that date and time fields are consistently formatted.
3. **Filtering Data**: Removes records where `suspected_infection_time` is before `admittime`.
4. **Feature Engineering**:
   - Calculates the number of days between suspected infection and admission. This will be the target variable we are predicting.
   - Merges X-ray metadata with patient records based on subject_id, hadm_id, and stay_id.
5. **Feature Selection**: Retains a subset of features including:
   - **Vitals & Lab Results**: Heart rate, blood pressure, respiration rate, bilirubin, creatinine, platelet count, etc.
    - Features were chosen based on tests/samples doctors/nurses at UCSD health collect when checking for sepsis.
    - Temperature, sbp_ni, mbp_ni, and lactate features were completely dropped due to very high null values.
   - **Clinical Outcomes**: Abnormal X-ray findings, sepsis indicators.
6. **Handling Missing Data**:
   - Uses imputation by randomly sampling from existing values in the same column.
7. **Feature Scaling**:
   - Applies `StandardScaler` to standardize numerical features except categorical and time-based features.
8. **Target Encoding**:
   - Categorizes `days` into binary labels (`0`, `1`, or `2+` days).
   - No sepsis prediction represented by '-1'.

## Model Training
The function `build_model`:
1. **Data Balancing**:
   - Applies **SMOTE** to handle class imbalances.
2. **Random Forest Model**:
   - Trains a `RandomForestClassifier` with hyperparameters:
     - `max_depth=10`
     - `n_estimators=100`
     - `criterion='entropy'`
     - `class_weight='balanced'`
3. **Model Evaluation**:
   - Computes accuracy on train and test sets.
   - Generates confusion matrices and classification reports.

## Model Evaluation & Output
1. **Performance Metrics**:
   - Train/Test accuracy scores
   - Precision, recall, F1-score from classification reports
   - Confusion matrix
2. **ROC Curve Analysis**:
   - Plots ROC curves for each class label.
   - Saves the plot to the output directory.

## Execution Flow
The `main` function:
1. Defines input/output paths.
2. Calls `preprocess_data` to process the dataset.
3. Trains and evaluates the model using `build_model`.
4. Saves results in JSON format.
5. Generates and saves an ROC curve plot.

## Output Files
- `model2_baseline_results.json`: Contains model performance metrics.
- `ROC Curve Image`: Saved for visualizing classifier performance.



# Metadata Preprocessing Script

## Overview
This script processes metadata related to patient records by cleaning and aggregating lab results. It extracts relevant features, calculates mean values, and merges the processed data with a sepsis dataset.

## Key References
- Uses **Pandas** for data manipulation.
- Uses **NumPy** for numerical operations.

## Logic Summary
1. Filters out records with missing `subject_id`.
2. Selects relevant columns from the metadata.
3. Identifies the most recent lab result for each patient per admission.
4. Computes the mean of key lab values for each patient admission.
5. Merges the most recent lab results with the computed mean values.
6. Integrates the processed metadata with the sepsis dataset.
7. Converts the `sepsis3` column into a binary integer format.
8. Saves the final processed dataset as `full_metadata.csv`.


## **Feature Descriptions**

### **Patient Identifiers**
| Feature Name | Description |
|-------------|------------|
| `subject_id` | Unique identifier for each patient. |
| `hadm_id` | Unique identifier for each hospital admission. |
| `stay_id` | Unique identifier for each ICU stay. |

---

### **Vital Signs**
| Feature Name | Description |
|-------------|------------|
| `heart_rate` | Heart rate of the patient (beats per minute). |
| `sbp` | Systolic blood pressure (mmHg). |
| `sbp_ni` | Non-invasive systolic blood pressure (mmHg). |
| `mbp` | Mean blood pressure (mmHg). |
| `mbp_ni` | Non-invasive mean blood pressure (mmHg). |
| `resp_rate` | Respiratory rate (breaths per minute). |
| `temperature` | Body temperature (°C or °F depending on source). |

---

### **Laboratory Results**
| Feature Name | Description |
|-------------|------------|
| `platelet` | Platelet count (10³/µL). |
| `wbc` | White blood cell count (10³/µL). |
| `bands` | Percentage of band neutrophils (immature white blood cells). |
| `lactate` | Lactate levels (mmol/L), an indicator of tissue oxygenation and sepsis severity. |
| `inr` | International Normalized Ratio, a measure of blood clotting function. |
| `ptt` | Partial thromboplastin time (seconds), another blood clotting measure. |
| `creatinine` | Serum creatinine level (mg/dL), an indicator of kidney function. |
| `bilirubin` | Bilirubin level (mg/dL), a measure of liver function. |

---

### **Conditions & Outcomes**
| Feature Name | Description |
|-------------|------------|
| `pneumonia` | Binary indicator (1 = Yes, 0 = No) for pneumonia diagnosis. |
| `sepsis` | Binary indicator (1 = Sepsis, 0 = No Sepsis), derived from `sepsis3`. |

---

## **Processing Notes**
- **Vital signs and lab values** are taken from the most recent recorded measurement per patient stay.
- When missing, features are **averaged** over the admission before merging with the sepsis dataset.
- The `sepsis3` column is converted to `sepsis` (1 for sepsis, 0 for no sepsis).

---

## Why calculate the most recent features and mean features for each patient and admission?
One of the biggest obstacles we encountered when dealing with this data was null values. Specifically, the features/lab results collected are built using "chart
event" data (data that occurs at several timestamps throughout a patient's admission). For example, a patient at a specific timestamp may have just heartrate
and blood pressure data collected, nothing else. This naturally leads to a higher level of null values. For the sake of simplicity for our sepsis
prediction model, we "squeezed" these lab results into one row of features for each patient admission. We take the most recent set of lab results for 
any single admission, then fill any null values within that most recent timestamp with the mean values across admissions.

## Why these features?
When deciding upon features to use for the model, we aimed to find features that would best mirror real-life practices to detect sepsis. AFter consulting with UCSD Health faculty, we were able to identify the features above as the lab results and data most used to assess risk for sepsis in real time. We believe that when combined with pneumonia detection, these features can be a good predictor for sepsis. We collected all of these features from the mimiciv dataset.



## Process Steps
1. **Data Cleaning**  
   - Removes rows where `subject_id` is missing.
   - Selects relevant clinical features for analysis.
   
2. **Feature Aggregation**  
   - Extracts the most recent lab results per patient admission.
   - Computes the mean of selected lab values for each patient admission.

3. **Data Merging**  
   - Merges the most recent lab results with the computed mean values.
   - Joins the processed metadata with the sepsis dataset.

4. **Final Processing**  
   - Converts `sepsis3` from `'t'`/`'f'` to binary (1 or 0).
   - Saves the processed dataset as a CSV file.

## Output
- The final dataset is saved as **`data/processed/full_metadata.csv`**.
- The dataset includes:
  - **Clinical features**: `heart_rate`, `sbp`, `mbp`, `resp_rate`, `temperature`, `platelet`, `wbc`, `bands`, `lactate`, `inr`, `ptt`, `creatinine`, `bilirubin`, `pneumonia`
  - **Sepsis label**: `sepsis`




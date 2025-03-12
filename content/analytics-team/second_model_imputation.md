# **Probabilistic Imputation Documentation**

## **Overview**
This script performs **probabilistic imputation** to fill in missing values based on the distribution of observed data. The imputation is stratified by **sepsis status**, ensuring that missing values are filled in a way that maintains the statistical properties of each subgroup. This approach prevents introducing bias that could arise from a naïve imputation method.

---

## **Methodology**
### **1. Handling Missing Values Based on Sepsis Status**
- The dataset contains a `sepsis` column (`1 = Sepsis`, `0 = No Sepsis`).
- Missing values are imputed **separately for sepsis and non-sepsis patients**, ensuring that the imputation reflects the true underlying distributions of each group.

### **2. Imputation Strategy**
- **Binary Features (e.g., Pneumonia)**
  - The probability of `pneumonia = 1` is calculated separately for **sepsis** and **non-sepsis** groups.
  - If sufficient data exists:
    - Sepsis patients' missing values are imputed using the **observed probability** of pneumonia within the sepsis group.
    - Non-sepsis patients' missing values are imputed using the **observed probability** within the non-sepsis group.
  - If no observed values are available:
    - A **default probability** is used (`0.5` for sepsis, `0.3` for non-sepsis).
  - Missing values are filled using a **Bernoulli distribution** with the computed probability.

- **Continuous Features (e.g., Lab Results, Vitals)**
  - Observed values are extracted separately for **sepsis** and **non-sepsis** groups.
  - Missing values are imputed by **randomly sampling** from the observed values within the corresponding group.
  - This ensures that the imputed values maintain realistic distributions and avoid bias.

---

## **Processing Steps**
### **1. Data Preparation**
- Load the input dataset and create a **copy** to avoid modifying the original.
- Identify missing values **for each column** except `sepsis`.
- Separate data into **sepsis** and **non-sepsis** groups.

### **2. Imputation Process**
- **Binary Feature Imputation (Pneumonia)**
  - Compute the **mean probability** of pneumonia (`pneumonia = 1`) for both **sepsis** and **non-sepsis** groups.
  - If missing values exist:
    - Use a **Bernoulli distribution** to sample missing values based on the observed probabilities.

- **Continuous Feature Imputation**
  - Extract observed values for sepsis and non-sepsis groups.
  - For each missing value, **randomly sample** from observed values within the same group.

### **3. Data Storage**
- The processed dataset is saved as: data/processed/imputed_metadata.csv
  

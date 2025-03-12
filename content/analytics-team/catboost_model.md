# **CatBoost Classifier Model Documentation**

## **Overview**
This script defines a **CatBoost-based classification model** using a `Pipeline` for streamlined training. The model is designed for **binary classification** tasks, optimizing for **AUC (Area Under the Curve)**. The chosen hyperparameters balance **training speed, generalization, and regularization**, making this model particularly suited for structured tabular data with categorical and numerical features.

---

## **Why Use CatBoost?**
CatBoost (Categorical Boosting) is a gradient boosting algorithm specifically designed to handle **categorical features efficiently** and reduce **overfitting**. It is well-suited for structured tabular datasets where interactions between variables are complex but essential.

### **Advantages of CatBoost for This Use Case**
1. **Handles Categorical Features Efficiently**  
   - Unlike XGBoost or LightGBM, CatBoost **does not require one-hot encoding** or label encoding, preventing **loss of ordinal relationships** in categorical data.
   - Uses **ordered boosting** to prevent **target leakage** when encoding categorical features.

2. **Prevents Overfitting**  
   - **Ordered boosting** ensures better generalization by handling categorical splits dynamically.
   - **L2 regularization (`l2_leaf_reg`)** and **early stopping** help prevent overfitting on small datasets.

3. **Optimized for Speed and Performance**  
   - CatBoost has **built-in GPU acceleration**, allowing faster training on large datasets.
   - **Efficient default hyperparameters** make it robust to different data distributions.

---

## **Hyperparameter Selection and Justification**
Each hyperparameter in the model is carefully chosen to **balance performance and computational efficiency**:

| **Hyperparameter**       | **Value**  | **Reasoning** |
|-------------------------|------------|------------------------------------------------------------------|
| `iterations`            | 3000       | Provides enough rounds for convergence, ensuring thorough training. |
| `learning_rate`         | 0.4        | Higher learning rate speeds up convergence, compensating for a shallow tree depth. |
| `depth`                | 2          | Limits tree depth to prevent overfitting while still capturing key interactions. |
| `min_data_in_leaf`     | 4          | Controls minimum samples per leaf, ensuring stable splits and preventing overfitting. |
| `subsample`            | 0.8        | Introduces randomness in data selection per iteration, improving generalization. |
| `colsample_bylevel`    | 0.9        | Selects 90% of features at each split, maintaining diversity while avoiding redundancy. |
| `l2_leaf_reg`         | 0.3        | L2 regularization to prevent overfitting, making the model more stable. |
| `random_seed`         | 42         | Ensures reproducibility of results. |
| `loss_function`       | `Logloss`  | Suitable for binary classification, equivalent to logistic regression loss. |
| `eval_metric`         | `AUC`      | Prioritizes ranking quality over accuracy, making it useful for imbalanced datasets. |
| `bootstrap_type`      | `Bernoulli` | Randomly selects a subset of data per iteration, enhancing variance reduction. |
| `verbose`            | `False`     | Suppresses detailed output to avoid excessive logging. |
| `early_stopping_rounds` | 20       | Stops training if validation performance plateaus, preventing unnecessary iterations. |

### **Key Trade-offs**
- **Higher `learning_rate` + Lower `depth`**  
  - Encourages faster convergence but requires careful tuning of `iterations` and `early_stopping_rounds`.
- **Subsample and Feature Sampling**  
  - Introduces controlled randomness to avoid overfitting while maintaining predictive power.
- **Regularization (`l2_leaf_reg`)**  
  - Adds a small penalty to prevent excessive complexity in the tree structure.

---

## **Process Steps**
### **1. Model Definition**
- A **`Pipeline`** is created to encapsulate the **CatBoostClassifier**, ensuring a structured workflow.
- The model is configured with pre-defined hyperparameters optimized for performance.

### **2. Model Training**
- The model is **fitted on the training data (`X_train`, `y_train`)**, allowing CatBoost to determine optimal feature splits and decision boundaries.

### **3. Output**
- The function **returns a trained CatBoost model**, which can be used for predictions or further fine-tuning.

---

## Model Evaluation & Output
1. **Performance Metrics**:
   - Train/Test accuracy scores
   - Precision, recall, F1-score from classification reports
2. **ROC Curve Analysis**:
   - Plots ROC curves for each class label.
   - Saves the plot to the output directory.
  
### Why These Metrics?  

#### 1. Performance Metrics  
- **Accuracy**: Basic measure of correctness but insufficient for imbalanced medical data.  
- **Precision & Recall**: Essential for medical diagnosis—minimizing false positives (unnecessary treatments) and false negatives (missed diagnoses).  
- **F1-Score**: Balances precision and recall, crucial for handling class imbalances.  

#### 2. ROC Curve Analysis  
- **ROC & AUC**: Measure model's ability to distinguish between conditions.  

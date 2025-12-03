# fraud_detection_project
project for Medicare fraud detection


 1.1 Overview
 Data Orbit has been contracted by the Centers for Medicare & Medicaid Services (CMS) to develop an
 intelligent fraud detection system. Currently, CMS can only investigate a small fraction of suspicious
 cases, allowing many fraudulent activities to go undetected. Existing systems rely on basic rule-based
 methods that capture obvious patterns but fail to identify more sophisticated fraud schemes.
 Your goal is to design a data-driven model that identifies high-risk providers while maintaining inter
pretability and minimizing false positives, which can lead to unnecessary investigations and reputational
 damage.
 Types of Healthcare Fraud:
 • Billing for services never rendered.
 • Upcoding- billing for higher-cost procedures than those performed.
 • Unbundling- billing separately for procedures that should be combined.
 • Submitting claims for deceased patients.
 • Prescribing unnecessary treatments for financial gain.
 • Engaging in kickback or referral schemes.



TEAM MEMBERS :
OMAR HAZEM
EBRAM HANY
AYA SALAH 
FARIDA ELBANNA







## 1. Data Exploration & Feature Engineering Results

The initial analysis focused on identifying differences between **506 fraudulent providers** and **4,904 non-fraudulent providers**.

| Metric | Fraud Provider Mean | Non-Fraud Provider Mean | Ratio (Fraud:Non-Fraud) | Insight |
| :--- | :--- | :--- | :--- | :--- |
| **Total Reimbursed** | $584,350.04 | $53,193.72 | **10.99:1** | Fraud providers have a significantly higher reimbursement value. |
| **Total Claims** | 420.55 | 70.44 | **5.97:1** | Fraud providers submit far more claims. |
| **Unique Beneficiaries** | 242.02 | 49.11 | **4.93:1** | Fraudulent activity involves a greater number of unique patients. |

**Key Feature Insight:** Claim patterns (Total Claims, Total Reimbursed) were found to be the strongest predictors of fraud.

***

## 2. Modeling and Model Selection

The project addressed a significant class imbalance with a **9.69:1** ratio of non-fraud to fraud cases.

| Criterion | Result/Recommendation |
| :--- | :--- |
| **Best Sampling Strategy** | **Class Weighting+** was used as the optimal strategy, balancing the severe class imbalance during training. |
| **Algorithm Performance** | **Logistic Regression** achieved the highest Precision-Recall AUC (**PR AUC: 0.7588**) among the comparison models when using the Class Weighting+ strategy. |
| **Primary Metric** | **PR AUC** was prioritized as the most reliable metric for heavily imbalanced fraud detection data. |
| **Business Recommendation** | The **Logistic Regression** model was recommended for the business context due to its optimal balance of high detection performance and excellent interpretability (9.0/10), which is crucial for investigators. |

***

## 3. Final Model Evaluation

The final model evaluated was a **Logistic Regression** model.

### Final Performance Metrics

| Metric | Value |
| :--- | :--- |
| **PR AUC** | 0.6756 |
| **ROC AUC** | 0.7131 |
| **F1-Score** | 0.6483 |
| **Precision** | 0.6136 |
| **Recall** | 0.6875 |

### Operational Insights

The model's operational performance was optimized based on a cost analysis:

* **Optimal Threshold:** **0.35** (determined to minimize overall operational costs).
* **Minimum Operational Cost:** **$83,000.00**.
* **Fraud Detection Rate (Recall):** 68.8%.
* **Investigation Efficiency (Precision):** 61.4% (meaning 61.4% of flagged cases are actual fraud).
* **False Positive Rate:** 8.0%.

### Model Robustness & Validation

* **Cross-Validation:** The model showed good stability with a Mean ROC-AUC of 0.9250 (±0.0100) across 5 Time Series Cross-Validation splits.
* **Train-Test Gap:** A notable training-testing performance gap of **0.2119** was observed between the cross-validation mean and the test set ROC-AUC.
* **Key Insight:** The model was confirmed to effectively balance precision and recall, and the cross-validation process confirmed its generalizability.



## Reproduction Instructions: Fraud Detection Model

### 1. Project Setup and Data Loading

The initial step involves importing all necessary libraries and loading the raw datasets.

1.  **Import Libraries:** Import essential Python libraries for data manipulation, machine learning, and imbalance handling. Key libraries include `pandas`, `numpy`, `matplotlib`, `seaborn`, and components from `sklearn` (e.g., `LogisticRegression`, `StandardScaler`, `TimeSeriesSplit`).
2.  **Load Data:** Load the four training data files:
    * `Train-1542865627584.csv` (Provider labels)
    * `Train_Beneficiarydata-1542865627584.csv` (Beneficiary data)
    * `Train_Inpatientdata-1542865627584.csv` (Inpatient claims)
    * `Train_Outpatientdata-1542865627584.csv` (Outpatient claims)
3.  **Standardize Data:** Rename columns in the loaded dataframes for consistency, specifically renaming `Provider` to **`ProviderID`** and `PotentialFraud` to **`is_fraud`**.

***

### 2. Data Cleaning and Feature Engineering

This section involves preparing the raw data and aggregating claims information to the provider level.

1.  **Date Conversion:** Convert relevant claim date columns (`ClaimStartDt`, `ClaimEndDt`, etc.) to `datetime` objects.
2.  **Handle Missing Values:** Fill missing values in chronic condition columns within the beneficiary data with **`0`**.
3.  **Combine Claims:** Concatenate the Inpatient and Outpatient claims dataframes to create a single `all_claims_df`.
4.  **Calculate Claim-Level Features:** Calculate the **`ClaimDuration`** feature for each claim as the difference in days between `ClaimEndDt` and `ClaimStartDt`.
5.  **Aggregate to Provider Level:** Aggregate the claims and beneficiary data to generate features for each unique provider (`ProviderID`). Key features to generate include:
    * **`Total_Reimbursed`** (Total claim amount paid by the healthcare company).
    * **`Total_Claims`** (Total number of claims submitted).
    * **`Unique_Beneficiaries`** (Total number of unique patients associated with the provider).
    * Derived risk indicators like **`Claims_per_Beneficiary`** and **`Amount_Variation_Risk`**.

***

### 3. Modeling and Training

The model training process must account for the severe class imbalance of **9.69:1** (No Fraud:Yes Fraud).

1.  **Split Data:** Split the final provider-level feature set into training and testing partitions (`X_train`, `X_test`, `y_train`, `y_test`).
2.  **Scale Features:** Apply **`StandardScaler`** to the feature set (X).
3.  **Imbalance Handling Strategy:** Use the **Class Weighting+** strategy, which was determined to be the optimal method for handling the imbalance without data manipulation.
4.  **Model Selection:** Select the **Logistic Regression** algorithm, as it demonstrated the best balance of high predictive power (**PR AUC: 0.7588**) and excellent interpretability (**9.0/10**) for the business context.
5.  **Train Final Model:** Train the Logistic Regression model using the following hyperparameters:
    * `random_state=42`
    * `class_weight='balanced'`
    * `C=0.1`
    * `max_iter=1000`
    * `solver='liblinear'`

***

### 4. Evaluation and Operationalization

The final steps involve rigorous performance validation and setting the operational deployment parameter.

1.  **Model Robustness:** Validate the model's stability using **Time Series Cross-Validation** (5 splits), confirming a stable **Mean ROC-AUC of 0.9250 (±0.0100)**.
2.  **Final Metrics:** Calculate the final performance metrics on the test set, prioritizing **PR AUC**:
    * PR AUC: 0.6756
    * Precision: 0.6136
    * Recall: 0.6875
3.  **Optimal Threshold Determination:** Conduct a cost-benefit analysis to find the decision threshold that minimizes total operational costs. The **optimal threshold is 0.35**, which results in the **minimum operational cost of $83,000.00**.
4.  **Deployment Instructions:** Deploy the model using the determined **optimal threshold (0.35)**. The ongoing maintenance plan includes:
    * Implementing monitoring for performance degradation.
    * Establishing a feedback loop with investigators.
    * Scheduling quarterly model retraining.

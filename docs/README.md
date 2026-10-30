# 📊 Telco Customer Churn Prediction

[![Python](https://img.shields.io/badge/Python-3.10+-blue?logo=python&logoColor=white)](https://www.python.org/)
[![Pandas](https://img.shields.io/badge/Pandas-Data%20Analysis-150458?logo=pandas&logoColor=white)](https://pandas.pydata.org/)
[![Scikit-learn](https://img.shields.io/badge/Scikit--learn-Machine%20Learning-F7931E?logo=scikitlearn&logoColor=white)](https://scikit-learn.org/)
[![CatBoost](https://img.shields.io/badge/CatBoost-Gradient%20Boosting-yellow)](https://catboost.ai/)

## 📌 Project Overview

This project focuses on predicting customer churn for a telecommunications company.

The main goal is not only to classify customers as likely to churn or stay, but also to rank them by predicted churn probability. This can help a company with limited retention resources prioritise customers who may require attention first.

The project covers the full machine learning workflow: data analysis, preprocessing, model training, hyperparameter tuning, evaluation, and customer ranking.

**Important:** The model predicts historical churn patterns. It does not prove that a discount or another retention offer will prevent a customer from leaving or generate additional profit.

## 🎯 Business Problem

A telecommunications company wants to reduce customer churn, but its retention resources are limited. Offering discounts to every customer may be inefficient: some customers would stay anyway, while others may leave despite receiving an offer.

The project investigates whether customers can be ranked by churn risk so that a limited group with a higher concentration of actual churn cases can be prioritised.

The business question is:

**How effectively can the model identify and rank customers who are likely to churn?**

The project does not evaluate the causal effect of retention campaigns or calculate their financial return.

## 📂 Dataset

The project uses the [Telco Customer Churn dataset](https://www.kaggle.com/datasets/blastchar/telco-customer-churn).

- **Rows:** 7,043 customers
- **Columns:** 21
- **Target:** `Churn`
- **Identifier:** `customerID`

The `customerID` column was excluded from modelling because it identifies customers but does not describe their characteristics.

The target variable was converted to binary format:

- `No` → 0
- `Yes` → 1

## 🔍 Exploratory Data Analysis

The EDA was performed on the training set.

### Contract Type

| Contract | Churn Rate |
|---|---:|
| Month-to-month | 42.75% |
| One year | 11.08% |
| Two year | 2.87% |

Customers with month-to-month contracts showed a substantially higher churn rate. The rate was approximately 15 times higher than for customers with two-year contracts.

### Customer Tenure

| Tenure | Churn Rate |
|---|---:|
| 0–12 months | 47.32% |
| 13–24 months | 28.61% |
| 25–36 months | 20.93% |
| 37–48 months | 19.48% |
| 49–60 months | 15.35% |
| 61–72 months | 6.81% |

The analysis showed an inverse relationship between customer tenure and churn rate: customers with shorter tenure churned more frequently.

### Monthly Charges

Customers who churned had higher average and median monthly charges:

- Average: 74.86 versus 61.34
- Median: 79.95 versus 64.40

However, the distributions overlapped, so monthly charges alone could not clearly distinguish churned customers from retained ones.

### Financial Scale of Churn

Month-to-month contracts accounted for approximately 87.14% of the total monthly charges associated with churned customers in the training set.

This figure describes monthly charges associated with churned customers. It should not be interpreted as net profit or confirmed financial loss because service costs, margins, and future customer value are not available.

### Outlier Check

Potential outliers in `tenure`, `MonthlyCharges`, and `TotalCharges` were checked using the IQR method. No observations were found beyond the calculated bounds, so all records were retained.

## 🧹 Data Preparation

- The dataset was split into training and test sets using an 80/20 stratified split.
- `random_state=42` was used for reproducibility.
- The training set contains 5,634 observations; the test set contains 1,409.
- Eleven blank values in `TotalCharges` were found among customers with `tenure = 0`. They were filled with zero based on the assumption that these customers had not yet accumulated charges.
- `tenure_group` was created for EDA but was not used as a model feature.

For Logistic Regression, preprocessing was organised with `Pipeline` and `ColumnTransformer`:

- Numerical features: median imputation and `StandardScaler`.
- Categorical features: most-frequent imputation and `OneHotEncoder(handle_unknown="ignore")`.

This setup ensures that preprocessing parameters are fitted only on the training portion of each cross-validation fold.

CatBoost was trained using native categorical features.

## 🤖 Modelling

The following models and approaches were evaluated:

- `DummyClassifier` — a simple baseline that always predicts the majority class.
- Logistic Regression — the first working classification baseline.
- CatBoost — a model using native categorical feature processing.
- `RandomizedSearchCV` — used to tune CatBoost hyperparameters with Average Precision as the search metric.

Stratified five-fold cross-validation was used during model development. The held-out test set was reserved for final evaluation.

### Cross-Validation Results

| Metric | Logistic Regression | Initial CatBoost |
|---|---:|---:|
| ROC-AUC | 0.846 ± 0.013 | 0.845 ± 0.010 |
| Average Precision | 0.662 ± 0.019 | 0.660 ± 0.023 |
| Precision | 0.653 ± 0.026 | 0.665 ± 0.043 |
| Recall | 0.543 ± 0.041 | 0.513 ± 0.020 |
| F1-score | 0.592 ± 0.030 | 0.578 ± 0.024 |

The initial CatBoost configuration did not show a clear advantage over Logistic Regression. Hyperparameter tuning was then performed to improve ranking quality.

After tuning, mean Average Precision increased from 0.660 to 0.672, while mean ROC-AUC increased from 0.845 to 0.851.

## 📈 Customer Ranking

Since the business task involves prioritising a limited number of customers, model quality was also evaluated using out-of-fold predictions and Precision@K.

Each training-set customer received a prediction from a model that had not been trained on that customer's row.

### Out-of-Fold Ranking Results

| K | LR: Churn Customers Found | CatBoost: Churn Customers Found | LR Precision@K | CatBoost Precision@K |
|---:|---:|---:|---:|---:|
| 100 | 89 | 88 | 0.890 | 0.880 |
| 200 | 165 | 174 | 0.825 | 0.870 |
| 300 | 244 | 261 | 0.813 | 0.870 |
| 500 | 382 | 391 | 0.764 | 0.782 |
| 1,000 | 696 | 691 | 0.696 | 0.691 |

Both models concentrated actual churn cases more effectively than random selection.

CatBoost performed better for groups of 200, 300, and 500 customers, while Logistic Regression had a small advantage at K = 100 and K = 1,000. Therefore, the preferred model depends partly on the size of the group the company can handle.

The range of 200–400 customers was considered as a preliminary business scenario, not as a proven optimal group size.

## 🧪 Final Test Results

The selected models were evaluated on the held-out test set.

| Metric | Logistic Regression | Tuned CatBoost |
|---|---:|---:|
| ROC-AUC | 0.842 | 0.846 |
| Average Precision | 0.634 | 0.669 |
| Precision | 0.657 | 0.670 |
| Recall | 0.559 | 0.521 |
| F1-score | 0.604 | 0.586 |

At the standard classification threshold of 0.5, Logistic Regression achieved higher Recall and F1-score. CatBoost achieved higher ROC-AUC, Average Precision, and Precision.

### Confusion Matrix Comparison

| Result | Logistic Regression | Tuned CatBoost |
|---|---:|---:|
| True Negatives | 926 | 939 |
| False Positives | 109 | 96 |
| False Negatives | 165 | 179 |
| True Positives | 209 | 195 |

CatBoost produced fewer false positives but also missed more actual churn cases at the 0.5 threshold.

### Test-Set Ranking Results

| K | LR: Churn Customers Found | CatBoost: Churn Customers Found | LR Precision@K | CatBoost Precision@K |
|---:|---:|---:|---:|---:|
| 100 | 74 | 80 | 0.740 | 0.800 |
| 200 | 145 | 146 | 0.725 | 0.730 |
| 300 | 198 | 199 | 0.660 | 0.663 |
| 400 | 234 | 239 | 0.585 | 0.598 |
| 500 | 269 | 278 | 0.538 | 0.556 |

CatBoost achieved higher Precision@K for all tested group sizes, although the differences were relatively small at K = 200 and K = 300.

## 💡 Key Findings

- Month-to-month customers showed the highest churn rate in the training set.
- Shorter customer tenure was associated with a higher churn rate.
- Customers who churned had higher average and median monthly charges.
- Logistic Regression provided a strong and interpretable baseline.
- The initial CatBoost configuration did not outperform Logistic Regression clearly, but hyperparameter tuning improved its ranking metrics.
- On the held-out test set, tuned CatBoost achieved higher Average Precision and better Precision@K across the tested group sizes.
- Logistic Regression achieved higher Recall and F1-score at the standard classification threshold.

**Tuned CatBoost was selected as the preferred model for the customer-prioritisation scenario**, because ranking customers under a limited resource constraint is the main objective of the project.

## ⚠️ Limitations and Further Improvements

- The dataset contains historical churn outcomes, not the results of retention campaigns.
- A high predicted churn probability does not mean that a customer can be retained with a discount or another offer.
- The project does not establish a causal effect of retention activities.
- Financial ROI cannot be calculated without information about customer value, service margins, campaign costs, and retention effectiveness.
- The preferred size of the priority group should be determined using actual operational and financial constraints.
- A controlled experiment or campaign outcome data would be needed to evaluate whether retention offers change customer behaviour.

Possible next steps:

- Evaluate different classification thresholds.
- Test the model on new data.
- Add campaign outcomes and customer-value information when available.
- Estimate retention costs and potential financial impact.
- Run a controlled experiment to measure the effect of retention offers.

## 🛠️ Technologies

- Python
- Pandas
- NumPy
- Matplotlib / Seaborn
- Scikit-learn
- CatBoost
- Jupyter Notebook

## 🚀 How to Run

1. Clone the repository:

   ```bash
   git clone <YOUR_GITHUB_REPOSITORY_URL>
   cd <PROJECT_FOLDER>
   ```

2. Install the required libraries:

   ```bash
   pip install pandas numpy matplotlib seaborn scikit-learn catboost jupyter
   ```

3. Download the dataset from [Kaggle — Telco Customer Churn](https://www.kaggle.com/datasets/blastchar/telco-customer-churn).

4. Place the dataset files in the project directory.

5. Open and run the notebook:

   ```bash
   jupyter notebook
   ```

## 📁 Project Structure

```text
.
├── data/
│   └── WA_Fn-UseC_-Telco-Customer-Churn.csv
├── notebooks/
│   └── telco_customer_churn.ipynb
└── README.md
```

*The structure above is an example; adjust file and folder names to match the actual repository.*

## 🔗 Links

- [Kaggle Dataset — Telco Customer Churn](https://www.kaggle.com/datasets/blastchar/telco-customer-churn)

---

**Project workflow:** Business Understanding → EDA → Data Preprocessing → Baseline Model → Model Comparison → Hyperparameter Tuning → Customer Ranking → Final Evaluation

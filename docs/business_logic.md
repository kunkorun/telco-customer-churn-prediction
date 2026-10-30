# 📞 Telco Customer Churn Prediction

[![Python](https://img.shields.io/badge/Python-3.x-3776AB?logo=python&logoColor=white)](https://www.python.org/)
[![Scikit-learn](https://img.shields.io/badge/Scikit--learn-ML-F7931E?logo=scikitlearn&logoColor=white)](https://scikit-learn.org/)
[![CatBoost](https://img.shields.io/badge/CatBoost-Classifier-FFCC00)](https://catboost.ai/)

A machine-learning project focused on predicting customer churn in the telecommunications industry and prioritising customers with higher historical churn risk.

The project combines exploratory data analysis, classification models, and business-oriented ranking evaluation to investigate how predictive models could support retention planning when resources are limited.

---

## 🎯 Project Overview

The main objectives are:

* Explore customer churn patterns in a telecommunications dataset.
* Identify customer characteristics associated with churn.
* Build machine-learning models to estimate historical churn risk.
* Compare Logistic Regression and CatBoost.
* Evaluate both classification quality and ranking quality.
* Investigate how effectively models identify churned customers within different top-K customer groups.

**Important:** The models predict historical churn risk. This project does not prove that discounts or outreach campaigns prevent churn or generate a positive financial return.

---

## 💼 Business Problem

A telecommunications company wants to identify customers who are more likely to leave.

When retention resources are limited, contacting every customer may not be practical. A risk ranking can help prioritise customers for further retention activities.

This project addresses two questions:

1. How accurately can a model predict customer churn?
2. If the company can focus on only a limited number of customers, how effectively can the model rank customers who later appear as churned in the data?

The business scenario is educational. Actual campaign costs, customer value, and retention effectiveness are not available in the dataset.

---

## 📊 Dataset

The project uses the **Telco Customer Churn** dataset from Kaggle.

* Number of records: **7,043**
* Number of columns: **21**
* Target variable: `Churn`

The dataset contains information about:
* Customer demographics.
* Subscribed services.
* Contract types.
* Billing information.
* Customer churn status.

**Target variable:**
* `Yes` — the customer left the company.
* `No` — the customer remained with the company.

The `customerID` column was excluded from model features because it is an identifier rather than a meaningful customer characteristic.

**Dataset source:** [Kaggle — Telco Customer Churn](<ADD_KAGGLE_DATASET_URL>)

---

## 🧹 Data Preparation

The data preparation stage included:

* Auditing data types, missing values, duplicate records, target distribution, and categorical values.
* Identifying 11 blank strings in `TotalCharges`.
* Checking the corresponding records and finding that all had `tenure = 0`.
* Converting blank strings to numeric missing values and replacing them with `0`, based on the assumption that no total charges had accumulated yet.

This replacement is a data-based assumption, not a verified original billing amount.

### Train-Test Split

* Training set: **5,634 records**
* Test set: **1,409 records**
* Split: 80/20, stratified by target, `random_state=42`

The training set was used for exploratory analysis and model development. The test set was reserved for final evaluation.

I also created `tenure_group` for exploratory analysis only and excluded it from model training.

---

## 🔍 Exploratory Data Analysis (EDA)

The exploratory analysis focused on identifying customer characteristics associated with churn.

### Contract Type and Churn

| Contract type | Churn rate |
|---|---:|
| Month-to-month | 42.75% |
| One year | 11.08% |
| Two year | 2.87% |

* Month-to-month customers had a substantially higher observed churn rate.
* The churn rate in this group was approximately 15 times higher than among customers with two-year contracts.
* Contract type was retained as a potentially important feature for modelling.

### Customer Tenure and Churn

| Tenure | Churn rate |
|---|---:|
| 0–12 months | 47.32% |
| 13–24 months | 28.61% |
| 25–36 months | 20.93% |
| 37–48 months | 19.48% |
| 49–60 months | 15.35% |
| 61–72 months | 6.81% |

* Churn rates generally decreased as customer tenure increased.
* Customers in their first year showed the highest observed churn rate.
* The original `tenure` feature was retained for modelling. Grouping was used for EDA only.

### Monthly Charges and Churn

| Metric | Did not churn | Churned |
|---|---:|---:|
| Number of customers | 4,139 | 1,495 |
| Mean monthly charges | 61.34 | 74.86 |
| Median monthly charges | 64.40 | 79.95 |
| Standard deviation | 31.14 | 24.60 |

* Churned customers had higher average and median monthly charges.
* The distributions overlapped substantially, so monthly charges alone could not clearly separate the two groups.
* The observed association does not prove that higher charges cause churn.

### Financial Scale of Churn by Contract Type

| Contract type | Churned customers | Churn rate | Sum of monthly charges |
|---|---:|---:|---:|
| Month-to-month | 1,326 | 42.75% | 97,528.85 |
| One year | 130 | 11.08% | 11,075.00 |
| Two year | 39 | 2.87% | 3,312.10 |

* Month-to-month customers accounted for approximately 87.14% of the total monthly charges associated with churned customers in the training set.
* This category had both the highest churn rate and the largest number of churn cases.
* Average monthly charges per churned customer were higher in the one-year and two-year contract groups.

**Important:** These sums are not net profit or proven financial losses. The dataset does not provide service costs, customer margins, or future customer value.

### Data Quality and Outliers

* Checked `tenure`, `MonthlyCharges`, and `TotalCharges` using boxplots and the IQR rule.
* No observations were found outside the calculated IQR bounds.
* No additional outlier removal or capping was applied.

---

## ⚙️ Workflow & Modelling

The project followed several stages:

1. **Data audit and cleaning** — reviewed data quality, types, blanks, duplicates, and target distribution.
2. **Exploratory data analysis** — investigated churn patterns by contract type, tenure, monthly charges, and other customer characteristics.
3. **Baseline model** — trained Logistic Regression in a preprocessing pipeline.
4. **Preprocessing** — applied median imputation and scaling to numerical features, and imputation and one-hot encoding to categorical features.
5. **Baseline comparison** — compared Logistic Regression with `DummyClassifier` to check whether the model learned a useful signal beyond always predicting the majority class.
6. **CatBoost** — trained a classifier using native categorical-feature support.
7. **Hyperparameter tuning** — used `RandomizedSearchCV` with Average Precision as the selection metric.
8. **Cross-validation** — re-evaluated the selected CatBoost configuration using five-fold stratified cross-validation.
9. **Business-oriented ranking analysis** — compared out-of-fold customer risk rankings using `Precision@K` and `Recall@K`.
10. **Final evaluation** — trained the selected configurations on the full training set and evaluated them once on the held-out test set.

### Preprocessing

For Logistic Regression, preprocessing was organised using `Pipeline` and `ColumnTransformer`.

* Numerical features: median imputation and `StandardScaler`.
* Categorical features: most-frequent imputation and `OneHotEncoder`.
* `handle_unknown="ignore"` was used for previously unseen categories.

This approach ensures that preprocessing parameters are learned within the training part of each cross-validation fold.

CatBoost was trained using native categorical-feature support without preliminary One-Hot Encoding.

---

## 🤖 Models & Results

Two models were compared:

* Logistic Regression — baseline model.
* Tuned CatBoost — model with native categorical-feature support and hyperparameter tuning.

### Test-Set Classification Metrics

Precision, Recall, and F1-score use the default classification threshold of 0.5.

| **Metric** | **Logistic Regression** | **Tuned CatBoost** |
|---|---:|---:|
| ROC-AUC | 0.842 | **0.846** |
| Average Precision | 0.634 | **0.669** |
| Precision | 0.657 | **0.670** |
| Recall | **0.559** | 0.521 |
| F1-score | **0.604** | 0.586 |

### Main Observations

* CatBoost achieved higher ROC-AUC, Average Precision, and Precision.
* Logistic Regression achieved higher Recall and F1-score at the 0.5 threshold.
* Neither model performed best across every metric.
* The preferred model depends on how predictions will be used.

---

## 📉 Confusion Matrix Analysis

Rows represent actual labels and columns represent predicted labels.

### Logistic Regression

| | **Predicted: No** | **Predicted: Yes** |
|---|---:|---:|
| Actual: No | 926 | 109 |
| Actual: Yes | 165 | 209 |

### Tuned CatBoost

| | **Predicted: No** | **Predicted: Yes** |
|---|---:|---:|
| Actual: No | 939 | 96 |
| Actual: Yes | 179 | 195 |

* CatBoost produced fewer false positives at the 0.5 threshold.
* Logistic Regression identified more actual churn cases.
* This illustrates why threshold-based classification metrics and risk-ranking analysis should be considered separately.

---

## 🎯 Business-Oriented Evaluation: Ranking High-Risk Customers

A classification model can be evaluated not only by its predicted labels, but also by how it ranks customers according to their predicted churn probability.

To simulate limited retention capacity, customers were ranked by predicted churn probability and the quality of selecting the top-K groups was evaluated.

### Out-of-Fold Ranking on the Training Set

Five-fold cross-validation was used to generate out-of-fold predictions. Each customer's prediction was made by a model that was not trained on that observation.

| **K** | **LR churn cases found** | **CatBoost churn cases found** | **LR Precision@K** | **CatBoost Precision@K** |
|---:|---:|---:|---:|---:|
| 100 | 89 | 88 | 0.890 | 0.880 |
| 200 | 165 | 174 | 0.825 | 0.870 |
| 300 | 244 | 261 | 0.813 | 0.870 |
| 500 | 382 | 391 | 0.764 | 0.782 |
| 1,000 | 696 | 691 | 0.696 | 0.691 |

* Both models concentrated actual churn cases near the top of the ranking more effectively than random selection.
* CatBoost performed better for K = 200, 300, and 500.
* Logistic Regression had a small advantage at K = 100 and K = 1,000.
* The model advantage depends on the selected group size.

This comparison was performed on the training set using out-of-fold predictions. Since CatBoost tuning was also performed on the training set, it is not a fully independent estimate of model performance.

### Final Ranking Evaluation on the Test Set

| **K customers selected** | **LR churn cases found** | **CatBoost churn cases found** | **LR Precision@K** | **CatBoost Precision@K** |
|---:|---:|---:|---:|---:|
| 100 | 74 | 80 | 0.740 | **0.800** |
| 200 | 145 | 146 | 0.725 | **0.730** |
| 300 | 198 | 199 | 0.660 | **0.663** |
| 400 | 234 | 239 | 0.585 | **0.598** |
| 500 | 269 | 278 | 0.538 | **0.556** |

### Interpretation

* CatBoost achieved higher Precision@K for all tested group sizes.
* The differences at K = 200 and K = 300 were relatively small.
* Among the top 100 customers selected by CatBoost, 80 had an actual churn label in the test set.
* For the simulated limited-capacity prioritisation scenario, CatBoost was selected as the preferred model.

**Important:** These results measure historical churn identification. They do not show that contacting these customers or offering discounts would prevent churn.

---

## 💡 Key Findings

* Month-to-month customers had a substantially higher observed churn rate than customers on longer contracts.
* Churn rates generally decreased as customer tenure increased.
* Churned customers had higher average monthly charges, although the distributions overlapped.
* Month-to-month customers accounted for the largest number of churn cases and the largest sum of monthly charges associated with churned customers.
* Logistic Regression provided a useful baseline and outperformed the majority-class `DummyClassifier`.
* Hyperparameter tuning improved the cross-validation results of the initial CatBoost configuration.
* Tuned CatBoost achieved higher ROC-AUC, Average Precision, and Precision on the held-out test set.
* Logistic Regression achieved higher Recall and F1-score at the default 0.5 threshold.
* CatBoost achieved higher Precision@K across all tested top-K group sizes on the held-out test set.
* For the simulated limited-capacity prioritisation scenario, CatBoost was selected as the preferred model.

---

## ⚠️ Limitations & Next Steps

The dataset contains customer characteristics and billing information, but it does not provide all the economic information needed to estimate the profitability of retention actions.

Missing information includes:
* Contact or campaign costs.
* Discount and offer costs.
* Customer-level contribution margin or lifetime value.
* Actual effectiveness of a retention offer.

Therefore, this project does not calculate ROI and does not establish that a retention campaign would be profitable.

Possible next steps:
* Combine churn-risk rankings with customer economics.
* Determine an operational cutoff based on company capacity, campaign costs, and expected customer value.
* Evaluate retention actions through a controlled experiment, such as an A/B test.
* Investigate whether different customer segments respond differently to retention offers.

---

## 📁 Project Structure

```text
Telco-Customer-Churn/
├── data/        # Dataset files
├── docs/        # Business logic and project documentation
├── notebooks/   # Analysis and modelling notebook
└── README.md    # Project overview and reproduction instructions
```

Individual notebook and documentation filenames can be added as the project structure becomes more specific.

---

## ▶️ How to Run

### 1. Clone the repository

```bash
git clone <YOUR_GITHUB_REPOSITORY_URL>
cd Telco-Customer-Churn
```

### 2. Create and activate a virtual environment

```bash
python -m venv venv
```

Windows:

```bash
venv\Scripts\activate
```

macOS / Linux:

```bash
source venv/bin/activate
```

### 3. Install dependencies

```bash
pip install pandas numpy matplotlib seaborn scikit-learn catboost jupyter
```

### 4. Download the dataset

Download the Telco Customer Churn dataset from the [Kaggle source page](<ADD_KAGGLE_DATASET_URL>).

Place `WA_Fn-UseC_-Telco-Customer-Churn.csv` in the `data/` directory.

### 5. Run the notebook

```bash
jupyter notebook
```

Open the notebook in `notebooks/` and run the cells from top to bottom.

**Note:** A `requirements.txt` file has not been added yet. The installation command above lists the main packages used in the project.

---

## 🔗 Links & Resources
* [Scikit-learn Documentation](https://scikit-learn.org/stable/)
* [CatBoost Documentation](https://catboost.ai/docs/)
* [Pandas Documentation](https://pandas.pydata.org/docs/)
* [Jupyter Documentation](https://docs.jupyter.org/)

---

**Project workflow:**

Business Problem → Data Audit → EDA → Preprocessing → Baseline Model → CatBoost → Hyperparameter Tuning → Ranking Evaluation → Final Test Evaluation
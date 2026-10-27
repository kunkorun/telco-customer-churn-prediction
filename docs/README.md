# Telco Customer Churn Prediction

## Project Overview

- I explore customer churn in a telecommunications dataset and build machine-learning models to identify customers with higher historical churn risk
- The business motivation is limited retention capacity, so the project checks both classification quality and ranking quality for different target-group sizes
- The models predict historical churn risk
- This project does not prove that a discount or outreach campaign prevents churn or creates a positive financial return

---

## Business Problem

- A telecommunications company wants to identify customers with a higher risk of leaving
- Instead of treating all customers equally, the company can use a risk ranking to prioritize a limited number of customers for retention actions
- I address two related questions:
  1 How well a model can predict customer churn
  2 When the company can focus on only a limited number of customers, how effectively the model can rank customers who later appear as churned in the data

---

## Dataset

- Source: Telco Customer Churn from Kaggle
- The dataset contains 7,043 customer records and 21 columns
- The columns describe customer demographics, subscribed services, contract types, billing information, and churn status
- Target: `Churn`
  - `Yes` means the customer left the company
  - `No` means the customer remained with the company
- I excluded `customerID` from model features because it is an identifier rather than a meaningful customer characteristic

---

## Data preparation

- I audited data types, missing values, duplicate records, target distribution, and categorical values
- I found 11 blank strings in `TotalCharges`
- All corresponding records had `tenure = 0`
- I converted the blank values to numeric missing values and then replaced them with `0` based on the assumption that no total charges had accumulated yet
- This is a data-based assumption, not a verified original billing amount
- I split the data into training and test sets using an 80/20 stratified split with `random_state=42`
- The training set contains 5,634 records and the test set contains 1,409 records
- I used the training data for exploratory analysis and model development
- I reserved the test set for final evaluation
- I created `tenure_group` for exploratory analysis only and excluded it from model training

---

## Workflow

- Data audit and cleaning: I reviewed data quality, types, blanks, duplicates, and target distribution
- Exploratory data analysis: I explored churn patterns by contract type, customer tenure, monthly charges, and related financial summaries
- Baseline model: I trained Logistic Regression in a preprocessing pipeline
- Numeric preprocessing: median imputation and scaling
- Categorical preprocessing: imputation and one-hot encoding
- Baseline comparison: I compared Logistic Regression with `DummyClassifier` to check whether the model learned a useful signal beyond always predicting the majority class
- CatBoost: I trained a CatBoost classifier using native categorical-feature support
- Hyperparameter tuning: I used `RandomizedSearchCV` with Average Precision as the selection metric and then re-evaluated the selected configuration with five-fold stratified cross-validation
- Business-oriented ranking analysis: I compared out-of-fold customer risk rankings using `Precision@K` and `Recall@K` at several target-group sizes
- Final evaluation: I trained the selected configurations on the full training set and evaluated them once on the held-out test set

---

## Models and Results

### Test-set classification metrics

- Precision, Recall, and F1-score below use the default classification threshold 0.5

| Metric | Logistic Regression | Tuned CatBoost |
|---|---:|---:|
| ROC-AUC | 0.842 | 0.846 |
| Average Precision | 0.634 | 0.669 |
| Precision | 0.657 | 0.670 |
| Recall | 0.559 | 0.521 |
| F1-score | 0.604 | 0.586 |

- CatBoost achieved higher ROC-AUC, Average Precision, and Precision on the test set
- Logistic Regression achieved higher Recall and F1-score at the 0.5 threshold
- Therefore neither model is best on every metric
- The preferred model depends on how predictions will be used

---

### Confusion matrices on the test set

- Rows are actual labels and columns are predicted labels

#### Logistic Regression

| | Predicted: No | Predicted: Yes |
|---|---:|---:|
| Actual: No | 926 | 109 |
| Actual: Yes | 165 | 209 |

#### Tuned CatBoost

| | Predicted: No | Predicted: Yes |
|---|---:|---:|
| Actual: No | 939 | 96 |
| Actual: Yes | 179 | 195 |

- At the 0.5 threshold, CatBoost produced fewer false positives
- Logistic Regression identified more actual churn cases
- This shows why threshold-based classification metrics and risk-ranking analysis should be considered separately

---

## Business-Oriented Evaluation: Ranking the Highest-Risk Customers

- To simulate limited retention capacity, I ranked customers by predicted churn probability
- The table below shows the number of customers with an actual `Churn = Yes` label among the top K ranked test customers

| K customers selected | Logistic Regression churn cases found | Tuned CatBoost churn cases found | Precision@K Logistic Regression | Precision@K CatBoost |
|---:|---:|---:|---:|---:|
| 100 | 74 | 80 | 0.740 | 0.800 |
| 200 | 145 | 146 | 0.725 | 0.730 |
| 300 | 198 | 199 | 0.660 | 0.663 |
| 400 | 234 | 239 | 0.585 | 0.598 |
| 500 | 269 | 278 | 0.538 | 0.556 |

- CatBoost had higher Precision@K for all tested group sizes
- The differences at K = 200 and K = 300 were small
- Among the top 100 customers selected by CatBoost, 80 had an actual churn label in the test set
- These results support using CatBoost as the preferred model for the customer-prioritization scenario in this project
- They do not mean the model would retain those customers if the company contacted them
- The analysis measures historical churn identification, not the causal effect of a retention action

---

## Key Findings

- In the exploratory analysis, churn was associated with customer contract type and tenure
- Month-to-month customers had a substantially higher observed churn rate than customers on longer contracts
- Logistic Regression provided a useful baseline and outperformed the majority-class `DummyClassifier` on the selected ranking and classification metrics
- Hyperparameter tuning improved the cross-validation results of the initial CatBoost configuration
- On the held-out test set, tuned CatBoost achieved higher ROC-AUC, Average Precision, and Precision than Logistic Regression
- Logistic Regression achieved higher Recall and F1-score at threshold 0.5
- CatBoost ranked customers more effectively by Precision@K across the tested top-K group sizes
- For the simulated limited-capacity prioritization use case, I treat CatBoost as the preferred model

---

## Limitations and Next Steps

- The dataset contains customer characteristics and billing fields
- It does not provide the economic information needed to estimate the profitability of retention actions, such as:
  - contact or campaign costs
  - discount and offer costs
  - customer-level contribution margin or lifetime value
  - actual effectiveness of a retention offer
- Therefore this project does not calculate ROI and does not establish that a retention campaign would be profitable
- Further work could combine the risk ranking with customer economics
- Further work could evaluate retention actions through a controlled experiment, such as an A/B test
- The ranking results depend on the chosen group size
- The appropriate operational cutoff should ultimately be determined using company capacity, campaign costs, and expected value of retaining a customer

---

## Project Structure

```text
Telco-Customer-Churn/
├── data/        # Dataset files
├── docs/        # Business logic and project documentation
├── notebooks/   # Analysis and modelling notebook
└── README.md    # Project overview and reproduction instructions
```

- I may add the individual notebook and documentation file names later to make the structure more specific

---

## How to Run

- Clone or download this repository
- Download the dataset from the Kaggle source page and place `WA_Fn-UseC_-Telco-Customer-Churn.csv` in the `data/` directory
- Install the required Python packages in your environment
- The project uses pandas, NumPy, Matplotlib, scikit-learn, CatBoost, and Jupyter Notebook
- Open the notebook in `notebooks/` and run the cells from top to bottom
- I still need to add a `requirements.txt` file so the environment can be recreated consistently
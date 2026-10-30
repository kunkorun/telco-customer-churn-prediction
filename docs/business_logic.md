# Telco Customer Churn — Business Logic

> **Working project journal**
>
> This document records the development of the business idea, research questions, hypotheses, experiments, and decisions made throughout the project.
>
> It is a working document rather than a final report. Its contents are updated as the analysis progresses and new results become available.

---

# Block I. Business Logic

## 1. Business Context

- The project uses an educational telecommunications-company scenario.
- The company wants to reduce customer churn.
- Retention resources are limited, so it may not be practical to offer discounts or promotions to every customer.
- The assumed business task is to identify customers who may be more likely to leave and prioritise them for further attention.
- This is an educational scenario, not a confirmed description of a specific company.
- Actual budget constraints, retention costs, and customer value are not provided in the dataset.

## 2. BL-001. Initial Business Idea

**Initial idea:** instead of allocating retention resources to all customers, focus on those who are more likely to leave.

The reasoning behind this idea:

- Mass discounts may be inefficient.
- Some customers would stay even without receiving an offer.
- A discount for such customers could reduce revenue without changing their behaviour.
- Limited resources may be better allocated to customers who are at greater risk of leaving.

A conditional example was discussed: if 150 out of 300 selected customers remain after a retention campaign, this might appear to be a positive result.

However, the numbers in this example are illustrative only. They are not results of the dataset analysis.

The initial idea suggests evaluating the model not only by its classification quality but also by its ability to prioritise customers under limited resources.

## 3. Critical Review of the Initial Idea

A high predicted churn probability does not necessarily mean that a customer can be retained with a discount, a call, or another offer.

The following situations must be distinguished:

- A customer may stay even without receiving an offer.
- A customer may leave despite receiving an offer.
- A customer may stay because of an offer, but this cannot be established from historical churn data alone.

Therefore, two different questions need to be considered:

1. How likely is a customer to leave?
2. Can a retention offer change that customer's decision?

The available churn dataset mainly allows investigating the first question.

To evaluate the actual effect of retention offers, additional campaign and outcome data or a specially designed experiment would be required.

## 4. Refined Working Hypothesis

**Hypothesis:** if customers are ranked by predicted churn probability in descending order and the top part of the list is selected, the share of actual churn cases among selected customers will be higher than with random selection of a comparable size.

If supported by historical data, such a ranking may help the company prioritise customers for possible retention activities.

**Status:** formulated and subsequently evaluated through out-of-fold and test-set ranking experiments.

**Limitation:** this hypothesis concerns the concentration of historical churn cases in the selected group. It does not prove that discounts or other offers will reduce churn or increase profit.

## 5. Preliminary ML Task Definition

- **Task type:** binary classification.
- **Target:** `Churn`.
- **Observation unit:** one customer.
- **Expected output:** an estimated churn probability for each customer.
- **Intended use:** rank customers by predicted churn risk.

The evaluation metrics, validation scheme, and priority-group size were selected during the subsequent stages of the project.

## 6. Assumptions and Limitations

- Limited retention resources are part of the educational business scenario; the company's actual constraints are unknown.
- Any selected customer-group size is a scenario assumption unless supported by operational or financial data.
- An association between a feature and churn does not establish a causal relationship.
- Churn prediction quality and retention-campaign effectiveness are different outcomes.
- Financial impact and ROI cannot be reliably calculated without information about campaign costs, customer value, and retention effectiveness.

## 7. Documentation Protocol

For each significant business decision, I record:

1. **Initial idea or question** — what was assumed and why.
2. **Check and critical review** — possible risks, alternative explanations, and limitations.
3. **Refined hypothesis or decision** — what was concluded after discussion or analysis.
4. **Evidence** — which data results support the conclusion and what remains to be tested.
5. **Status** — hypothesis, being tested, partially supported, not supported, or accepted decision.

---

# Block II. EDA Research Hypotheses

## EDA-001. Contract Type and Customer Churn

### Research Question and Initial Hypothesis

Does customer contract type relate to churn?

**Hypothesis:** customers with month-to-month contracts (`Month-to-month`) churn more frequently than customers with one-year (`One year`) or two-year (`Two year`) contracts.

If churn rates differ substantially between contract categories, contract type may help identify customer groups with different levels of churn risk.

### Validation Method

- The dataset was split into training and test sets in an 80/20 ratio.
- `train_test_split` was used with `random_state=42` and `stratify=y`.
- The hypothesis was checked on `X_train` and `y_train`.
- For each contract type, the proportion of customers with `Churn = Yes` was calculated.
- Results were examined using a relative-frequency table and a bar chart.

### Results

| Contract Type | Did Not Churn (`No`) | Churned (`Yes`) |
|---|---:|---:|
| Month-to-month | 57.25% | 42.75% |
| One year | 88.92% | 11.08% |
| Two year | 97.13% | 2.87% |

The training set shows substantial differences between contract categories:

- Month-to-month churn rate: **42.75%**.
- Two-year contract churn rate: **2.87%**.
- The first rate is approximately 15 times higher than the second.

### Interpretation

The results preliminarily support the hypothesis: contract type is associated with noticeable differences in churn rate in the training set.

Month-to-month customers appear to be a potentially higher-risk segment for further analysis.

However:

- The observed association does not prove that contract type itself causes churn.
- Other customer characteristics, service conditions, and interactions between features may influence the result.
- The analysis does not prove that moving customers to longer contracts would reduce churn.

### Decision and Status

**Status:** preliminarily supported by the training data.

Decisions:

- Keep `Contract` as a feature for further modelling.
- Consider month-to-month customers as a potentially important segment in the business interpretation.
- Do not interpret the observed association as a proven causal relationship.

Next steps:

1. Check other potential churn factors.
2. Investigate possible feature interactions and alternative explanations.
3. Evaluate the contribution of `Contract` to model performance.
4. Validate final model conclusions on the held-out test set.

These EDA results apply only to the training set. They are not an estimate of model quality and do not establish the effectiveness of retention activities.

---

## EDA-002. Customer Tenure and Churn

### Research Question

Is customer tenure (`tenure`) associated with churn?

### Initial Hypothesis

Customers with shorter tenure may churn more frequently than customers who have used the company's services for a longer period.

Identifying higher churn risk among new customers may help define segments that deserve additional attention when planning retention activities.

### Validation Method

- The `tenure` feature was divided into six 12-month intervals in the training set.
- The proportion of customers with `Churn = Yes` was calculated for each interval.

### Results

| Tenure | Churn Rate |
|---|---:|
| 0–12 months | 47.32% |
| 13–24 months | 28.61% |
| 25–36 months | 20.93% |
| 37–48 months | 19.48% |
| 49–60 months | 15.35% |
| 61–72 months | 6.81% |

### Interpretation

The training set shows an inverse relationship between tenure and churn rate:

- Customers with shorter tenure churn more frequently.
- Churn rate generally decreases as tenure increases.
- This pattern is consistent with the initial hypothesis.

Customers at an early stage of service may represent an important segment for further analysis.

However, the result does not prove that longer tenure itself reduces churn probability. Contract type and other customer characteristics may also explain part of the observed relationship.

### Decision and Status

**Status:** preliminarily supported by the training data.

- Keep the original numerical feature `tenure` for further research and modelling.
- Use `tenure_group` as an EDA tool for now.
- Evaluate separately whether the derived `tenure_group` feature should be included in the model.

---

## EDA-003. Monthly Charges and Churn

### Research Question

Are customer monthly charges (`MonthlyCharges`) associated with churn?

### Initial Hypothesis

Customers with higher monthly charges may churn more frequently.

Higher charges could influence a customer's decision to continue using the company's services. However, this relationship may also be explained by contract type, connected services, or other customer characteristics.

### Validation Method

In the training set, the following statistics for `MonthlyCharges` were calculated separately for customers with `Churn = Yes` and `Churn = No`:

- Number of customers.
- Mean.
- Median.
- Standard deviation.

A boxplot was also used to compare the distributions.

### Results

| Metric | Did Not Churn (`No`) | Churned (`Yes`) |
|---|---:|---:|
| Number of customers | 4,139 | 1,495 |
| Mean charges | 61.34 | 74.86 |
| Median charges | 64.40 | 79.95 |
| Standard deviation | 31.14 | 24.60 |

The differences:

- Mean monthly charges are higher by **13.52** among churned customers.
- Median monthly charges are higher by **15.55**.
- The boxplot also shows higher central values for the churned group.
- At the same time, the distributions overlap substantially.

There are customers with high monthly charges who stayed and customers with relatively low charges who churned.

### Interpretation

The descriptive statistics preliminarily support the initial hypothesis.

In the training set, churned customers had higher average and median monthly charges. However, `MonthlyCharges` alone cannot clearly separate churned and retained customers.

The observed association does not prove that higher charges cause churn. Contract type, connected services, and other customer characteristics may influence the result.

### Decision and Status

**Status:** preliminarily supported by the training data.

- Keep `MonthlyCharges` for further modelling.
- No additional feature processing is required at this stage.

---

## EDA-004. Financial Scale of Churn by Contract Type

### Research Question

Which contract types account for the largest total monthly charges associated with customers who churned?

### Idea and Rationale

The previous analysis showed noticeable differences in churn rates across contract types.

The next step was to examine the same pattern from a financial perspective.

For the business task, it may be useful to consider not only the proportion of churned customers but also:

- The absolute number of churn cases.
- The sum of monthly charges associated with those customers.
- The average monthly charges per churned customer.

This helps compare churn prevalence with its potential financial scale.

### Validation Method

In the training set, customers with `Churn = Yes` were grouped by `Contract`.

For each group, the following metrics were calculated:

- Number of churned customers.
- Sum of their `MonthlyCharges`.
- Average monthly charges per churned customer.

These values were then combined with the total number of customers and the churn rate for each contract category.

### Results

| Contract Type | Total Customers | Churned Customers | Churn Rate | Sum of `MonthlyCharges` for Churned Customers | Average Charges per Churned Customer |
|---|---:|---:|---:|---:|---:|
| Month-to-month | 3,102 | 1,326 | 42.75% | 97,528.85 | 73.55 |
| One year | 1,173 | 130 | 11.08% | 11,075.00 | 85.19 |
| Two year | 1,359 | 39 | 2.87% | 3,312.10 | 84.93 |
| **Total** | **5,634** | **1,495** | — | **111,915.95** | — |

The `Month-to-month` category accounts for approximately **87.14%** of the total monthly charges associated with churned customers in the training set.

At the same time, average monthly charges per churned customer are higher in the one-year and two-year contract groups.

### Interpretation

The contract category with the highest churn rate also has the largest number of churned customers and the largest total sum of their monthly charges.

The main reason is the substantially larger number of churned customers with month-to-month contracts.

For the business scenario, this suggests that month-to-month customers may be an important segment for further investigation: they combine a high churn rate with the largest absolute number of churn cases.

However, the calculated sum is **not net profit or confirmed financial loss**:

- `MonthlyCharges` describes monthly charges associated with churned customers.
- It does not account for service costs or profit margins.
- It does not include possible future revenue.
- The analysis does not establish how much revenue the company actually lost.

It is also not possible to conclude that month-to-month customers overall bring the company more money. The analysis concerns only customers who churned.

### Decision and Status

**Status:** the financial scale of churn by contract type has been described using the training set.

Recorded observations:

- `Month-to-month` has the highest churn rate.
- This category also has the largest number of churned customers.
- It has the largest total sum of monthly charges associated with churned customers.
- Average charges per churned customer are higher in the one-year and two-year contract groups.
- The available data is insufficient to calculate net financial losses or evaluate retention-campaign effectiveness.

These results are used for business interpretation, not as proof that a particular retention method will work.

---

## EDA-005. Potential Outlier Check

### Research Question

Do the numerical features `tenure`, `MonthlyCharges`, and `TotalCharges` contain unusual values that may indicate data errors or require additional processing before modelling?

### Validation Method

- Boxplots were built for the three numerical features in the training set.
- The first quartile (Q1), third quartile (Q3), and interquartile range (IQR) were calculated.
- Potential outlier bounds were determined using the 1.5 × IQR rule.

The bounds were calculated as:

- Lower bound: Q1 − 1.5 × IQR.
- Upper bound: Q3 + 1.5 × IQR.

Values below the lower bound or above the upper bound were considered potential outliers under this criterion.

### Results

| Feature | Q1 | Q3 | IQR | Lower Bound | Upper Bound | Potential Outliers |
|---|---:|---:|---:|---:|---:|---:|
| `tenure` | 9.00 | 55.00 | 46.00 | −60.00 | 124.00 | 0 |
| `MonthlyCharges` | 35.66 | 90.00 | 54.34 | −45.84 | 171.51 | 0 |
| `TotalCharges` | 402.98 | 3,835.82 | 3,432.85 | −4,746.30 | 8,985.10 | 0 |

No observations were found beyond the calculated IQR bounds for any of the three features.

The `TotalCharges` distribution has noticeable right skewness. However, skewness alone is not evidence of incorrect data.

### Interpretation

According to the selected statistical criterion, no potential outliers were detected.

This does not prove that the dataset contains no errors or unusual values. The IQR method identifies only observations outside the specified statistical bounds.

### Decision and Status

**Status:** potential-outlier checking using the IQR rule is complete.

- Keep all observations at this stage.
- Do not remove or cap values in `tenure`, `MonthlyCharges`, or `TotalCharges` without additional evidence.
- If later modelling reveals problems related to feature distributions, consider alternative processing methods and justify them through experiments.

---

# Block III. ML Experiments

## ML-001. Baseline Logistic Regression

### Experiment Goal

After the initial data audit and EDA, the first working churn-prediction model was built.

The goal was to establish a baseline classification quality level for comparison with more complex models.

For the business task, the model should not only assign a customer class but also provide a probability estimate that can be used to rank customers by churn risk.

### Model Choice

`LogisticRegression` was selected as the first model because:

- It is a standard algorithm for binary classification.
- It provides a relatively simple baseline.
- Its results can be compared with more complex classifiers.
- It helps assess how well a linear combination of features predicts churn.

Logistic Regression was not assumed to be the final model. Its purpose was to establish a reference point for subsequent experiments.

### Data Preparation

The original dataset contains 7,043 customers and 21 columns.

After excluding the customer identifier and target variable, 19 features were used for modelling.

The data was split into training and test sets in an 80/20 ratio:

| Dataset | Shape |
|---|---:|
| `X_train` | 5,634 × 19 |
| `X_test` | 1,409 × 19 |
| `y_train` | 5,634 |
| `y_test` | 1,409 |

The split used:

- `random_state=42`
- `stratify=y`

The target was converted to binary format:

- `No = 0`
- `Yes = 1`

The `customerID` column was excluded because it is an identifier rather than a customer characteristic.

The `tenure_group` feature created during EDA was not included in the baseline. The original numerical feature `tenure` was retained.

### Preprocessing with Pipeline

Data preparation was organised using `Pipeline` and `ColumnTransformer`.

**Numerical features:**

- `SeniorCitizen`
- `tenure`
- `MonthlyCharges`
- `TotalCharges`

Numerical preprocessing:

- Missing values were filled using `SimpleImputer` with the median strategy.
- Features were scaled using `StandardScaler`.

**Categorical features:**

- `gender`
- `Partner`
- `Dependents`
- `PhoneService`
- `MultipleLines`
- `InternetService`
- `OnlineSecurity`
- `OnlineBackup`
- `DeviceProtection`
- `TechSupport`
- `StreamingTV`
- `StreamingMovies`
- `Contract`
- `PaperlessBilling`
- `PaymentMethod`

Categorical preprocessing:

- Missing values were filled using the most frequent category.
- Categories were encoded using `OneHotEncoder(handle_unknown="ignore")`.

### Why Pipeline Was Used

All preprocessing steps were combined with model training in a single pipeline.

This ensures that the same transformations are applied during cross-validation, model training, and final evaluation.

During cross-validation:

- Preprocessing parameters are learned only from the training portion of each fold.
- The validation portion is used for evaluation.
- Scaling statistics and imputation values are not calculated using the validation fold.

This reduces the risk of data leakage during feature preparation.

No additional outlier processing was performed because the IQR check did not identify observations beyond the calculated bounds.

### Evaluation Scheme

Stratified five-fold cross-validation (`StratifiedKFold`) was used.

Parameters:

- `n_splits=5`
- `shuffle=True`
- `random_state=42`

The following metrics were evaluated:

- **ROC-AUC** — measures the model's ability to rank churn customers above non-churn customers.
- **Average Precision** — evaluates the quality of identifying the positive class across different thresholds.
- **Precision** — the proportion of actual churn customers among customers predicted as churned.
- **Recall** — the proportion of actual churn customers detected by the model.
- **F1-score** — the balance between Precision and Recall.

Precision, Recall, and F1-score were calculated using the standard classification threshold of 0.5.

Threshold tuning was not performed at this stage.

The test set was not used for model training, hyperparameter selection, or cross-validation evaluation.

### Logistic Regression Results

| Metric | Mean | Standard Deviation |
|---|---:|---:|
| ROC-AUC | 0.846 | 0.013 |
| Average Precision | 0.662 | 0.019 |
| Precision | 0.653 | 0.026 |
| Recall | 0.543 | 0.041 |
| F1-score | 0.592 | 0.030 |

The standard deviation describes the variation in results across the five validation folds.

### Interpretation

**ROC-AUC = 0.846**

The model demonstrates an ability to rank customers by churn risk better than random ranking.

**Average Precision = 0.662**

The model shows useful quality in identifying churn customers. This metric should be interpreted in relation to the positive-class prevalence in the dataset.

**Precision = 0.653**

At the standard threshold of 0.5, approximately 65.3% of customers predicted as churned actually belong to the positive class.

**Recall = 0.543**

The model detects approximately 54.3% of all actual churn cases. Around 45.7% remain undetected at this threshold.

**F1-score = 0.592**

This metric reflects the balance between Precision and Recall. It does not directly measure the financial effectiveness of retention activities.

The variation across folds is relatively small, which preliminarily indicates stable results under the selected cross-validation scheme. It does not guarantee the same quality on all future data.

### Initial Model Conclusion

Logistic Regression identifies useful patterns in customer characteristics and provides a working churn-prediction baseline.

However, at the standard threshold of 0.5, it misses a noticeable share of actual churn cases.

For the business task, this means that some customers who eventually churn may not be prioritised by the model.

Further experiments should evaluate how ranking quality, the number of selected customers, and resource constraints relate to one another.

**Status:** the first ML model has been trained and evaluated through cross-validation. Logistic Regression is accepted as the baseline for further experiments.

---

## ML-002. Comparison with DummyClassifier

### Experiment Goal

Check whether using customer characteristics improves churn prediction compared with a simple classifier that does not analyse individual features.

Logistic Regression was compared with `DummyClassifier`.

### Control Model

`DummyClassifier` with the `most_frequent` strategy was used.

It always predicts the most frequent class in the training set: `No`.

The original target distribution is:

- `No`: 5,174 customers, approximately 73.46%.
- `Yes`: 1,869 customers, approximately 26.54%.

The control model does not use `Contract`, `tenure`, `MonthlyCharges`, or other customer features. It serves as a reference point for evaluating the ML model.

### Comparison Method

Both models were evaluated using the same stratified five-fold cross-validation scheme on the training set.

The same metrics were used:

- ROC-AUC
- Average Precision
- Precision
- Recall
- F1-score

Precision, Recall, and F1-score were calculated at the threshold of 0.5.

For `DummyClassifier`, `zero_division=0` was used because the classifier does not predict the positive class.

The test set was not used in this comparison.

### Comparison Results

| Metric | Logistic Regression | DummyClassifier |
|---|---:|---:|
| ROC-AUC | 0.846 | 0.500 |
| Average Precision | 0.662 | 0.265 |
| Precision | 0.653 | 0.000 |
| Recall | 0.543 | 0.000 |
| F1-score | 0.592 | 0.000 |

The Logistic Regression values are mean results across five folds.

For `DummyClassifier`, the standard deviations of the metrics round to 0.000.

### Interpretation

`DummyClassifier` always predicts `No`, so it does not identify any churn customers.

As a result:

- Recall is zero.
- F1-score is zero.
- Precision is zero under the selected `zero_division=0` convention.

At the same time, such a classifier would achieve Accuracy of approximately 73.5% because it correctly predicts the majority class for most customers.

This demonstrates why Accuracy alone is not a sufficient success criterion for this project.

The DummyClassifier ROC-AUC is 0.500, indicating no ability to rank customers by individual churn risk.

Its Average Precision is approximately equal to the positive-class prevalence.

Compared with this control model, Logistic Regression demonstrates:

- Better ranking of customers by churn risk.
- Better identification of the positive class.
- Non-zero Precision, Recall, and F1-score at the standard threshold.

### Decision and Status

**Status:** control comparison completed.

- Logistic Regression remains the working baseline.
- `DummyClassifier` is retained as a reference point for later experiments.
- The comparison confirms that customer features contain useful information for churn prediction.
- It does not establish economic effectiveness.

The next step is to investigate customer ranking under a limited-resource scenario.

---

## ML-003. Baseline CatBoost

### Experiment Goal

Check whether CatBoost can improve on the Logistic Regression baseline.

The first `CatBoostClassifier` configuration used:

- `iterations=500`
- `learning_rate=0.05`
- `depth=6`
- `loss_function='Logloss'`
- `eval_metric='AUC'`
- `random_seed=42`

Categorical features were passed to CatBoost without preliminary One-Hot Encoding.

Evaluation used stratified five-fold cross-validation. The test set was not involved.

### Results

| Metric | Result |
|---|---:|
| ROC-AUC | 0.845 ± 0.010 |
| Average Precision | 0.660 ± 0.023 |
| Precision | 0.665 ± 0.043 |
| Recall | 0.513 ± 0.020 |
| F1-score | 0.578 ± 0.024 |

Precision, Recall, and F1-score were calculated at the standard threshold of 0.5.

### Interpretation

The initial CatBoost configuration did not demonstrate a clear advantage over Logistic Regression.

| Metric | Logistic Regression | Initial CatBoost |
|---|---:|---:|
| ROC-AUC | 0.846 | 0.845 |
| Average Precision | 0.662 | 0.660 |
| Precision | 0.653 | 0.665 |
| Recall | 0.543 | 0.513 |
| F1-score | 0.592 | 0.578 |

CatBoost achieved slightly higher Precision but lower Recall and F1-score.

At the standard threshold, it selected customers somewhat more precisely but detected a smaller share of actual churn cases.

The experiment tested only one CatBoost configuration, so it was too early to make a final conclusion about the algorithm's potential.

### Conclusion and Next Actions

The initial CatBoost configuration did not outperform the Logistic Regression baseline on the main metrics.

The next step was to tune CatBoost hyperparameters using Average Precision as the main search metric.

After tuning, the model would be re-evaluated using five-fold cross-validation and compared with the baseline before final test-set evaluation.

ML metrics alone do not prove the financial effectiveness of a retention campaign. Evaluating this would require additional information about offer economics and actual retention outcomes.

---

## ML-003A. CatBoost Hyperparameter Tuning

### Stage Goal

Check whether hyperparameter tuning can improve CatBoost ranking quality compared with the initial configuration.

### Method

`RandomizedSearchCV` was applied to the training set.

- Number of folds during the search: 3.
- Number of parameter combinations: 6.
- Main search metric: `average_precision`.

The parameter search included:

- `iterations`
- `depth`
- `learning_rate`
- `l2_leaf_reg`

After the search, the best configuration was evaluated again using five-fold cross-validation.

### Results

- Mean Average Precision increased from **0.660 to 0.672**.
- Mean ROC-AUC increased from **0.845 to 0.851**.

The complete metric values are used in the subsequent comparison with Logistic Regression and in the final model evaluation.

### Status

**Status:** tuning completed.

The selected configuration was passed to the customer-ranking experiment and final test evaluation.

---

## ML-004. Customer Ranking Under a Limited Budget

### Experiment Goal

Move beyond standard classification metrics and evaluate how useful the models may be for prioritising customers when retention resources are limited.

**Business hypothesis:** selecting customers by predicted churn risk may concentrate actual churn cases more effectively than random selection of a comparable size.

The models compared were:

- Logistic Regression.
- CatBoost with tuned hyperparameters.

### Method

1. Each customer received an estimated churn probability.
2. Customers were ranked in descending order of predicted risk.
3. The top K customers were selected.
4. The number of actual churn cases in the selected group was calculated.

Out-of-fold (OOF) predictions were used for the training-set ranking experiment.

Each customer received a prediction from a model that had not been trained on that customer's row.

The following group sizes were evaluated:

- K = 100
- K = 200
- K = 300
- K = 500
- K = 1,000

Metrics:

- **Precision@K** — proportion of actual churn customers among the K selected customers.
- **Recall@K** — proportion of all actual churn customers included in the selected group.

### Out-of-Fold Results

| K | Logistic Regression Found | CatBoost Found | Precision@K LR | Precision@K CatBoost |
|---:|---:|---:|---:|---:|
| 100 | 89 | 88 | 0.890 | 0.880 |
| 200 | 165 | 174 | 0.825 | 0.870 |
| 300 | 244 | 261 | 0.813 | 0.870 |
| 500 | 382 | 391 | 0.764 | 0.782 |
| 1,000 | 696 | 691 | 0.696 | 0.691 |

The share of churn customers in the training set is approximately 26.5%.

With random selection of 300 customers, approximately 80 churn cases would be expected. Both models identified substantially more.

### Interpretation

Both models concentrated actual churn cases more effectively than random selection.

CatBoost achieved better results for K = 200, 300, and 500.

At K = 300:

- CatBoost identified 261 churn customers.
- Logistic Regression identified 244.
- Precision@300 was 0.870 for CatBoost and 0.813 for Logistic Regression.

Logistic Regression had a small advantage at K = 100 and K = 1,000.

Therefore, the preferred model depends partly on the size of the group the company can handle.

For a scenario involving several hundred customers, CatBoost appeared more promising in this experiment.

### Business Interpretation

The range of 200–400 customers was selected as a preliminary scenario for further analysis.

It is a working assumption, not a proven optimal group size.

When selecting a priority group, the company would also need to consider:

- Number of employees and available time for customer contacts.
- Cost of handling one customer.
- Cost of a discount or another retention offer.
- Potential marginal customer value.
- Actual effectiveness of retention activities.

Increasing the selected group size may include more customers who are likely to churn, but Precision usually decreases as lower-risk customers enter the group.

The final group size should be determined using operational constraints, budget, and expected financial impact.

### Experiment Limitations

1. Results were obtained on the training set using out-of-fold predictions.
2. CatBoost hyperparameters were tuned on the same training set, so this comparison is not a fully independent estimate.
3. Historical churn does not indicate whether a customer could be retained with a discount or a call.
4. Financial campaign effectiveness has not been calculated.
5. Such an evaluation would require retention costs, customer value, and evidence of the actual effect of offers.

### Conclusion and Next Actions

For a limited-resource scenario, tuned CatBoost showed an advantage over Logistic Regression when selecting groups of 200–500 customers.

However, it cannot be considered the best model for every possible group size.

The next step was to evaluate the selected models on the previously held-out test set.

---

## ML-005. Final Model Evaluation on the Test Set

### Goal

Evaluate the generalisation performance of Logistic Regression and tuned CatBoost on the held-out test set.

The test set was not used for model training or hyperparameter tuning.

### Results

| Metric | Logistic Regression | Tuned CatBoost |
|---|---:|---:|
| ROC-AUC | 0.842 | 0.846 |
| Average Precision | 0.634 | 0.669 |
| Precision | 0.657 | 0.670 |
| Recall | 0.559 | 0.521 |
| F1-score | 0.604 | 0.586 |

Precision, Recall, and F1-score were calculated at the classification threshold of 0.5.

### Confusion Matrix Analysis

- Logistic Regression correctly identified 209 churn customers and missed 165.
- CatBoost correctly identified 195 churn customers and missed 179.
- CatBoost produced fewer false positives: 96 versus 109 for Logistic Regression.

At the 0.5 threshold, CatBoost less frequently classified customers who did not churn as positive. However, it also detected fewer actual churn cases.

### Customer Ranking on the Test Set

The previous experiment evaluated customer ranking using OOF predictions on the training set.

The following results show the separate ranking evaluation on the held-out test set.

| K | Logistic Regression Found | CatBoost Found | Precision@K LR | Precision@K CatBoost |
|---:|---:|---:|---:|---:|
| 100 | 74 | 80 | 0.740 | 0.800 |
| 200 | 145 | 146 | 0.725 | 0.730 |
| 300 | 198 | 199 | 0.660 | 0.663 |
| 400 | 234 | 239 | 0.585 | 0.598 |
| 500 | 269 | 278 | 0.538 | 0.556 |

CatBoost achieved higher Precision@K for all tested group sizes from 100 to 500 customers.

The difference was relatively small at K = 200 and K = 300.

### Link to the Business Task

The main business objective is to prioritise customers under limited retention resources.

On the test set, tuned CatBoost showed better ranking results for all considered group sizes.

At K = 300:

- CatBoost identified 199 customers with actual churn.
- Logistic Regression identified 198.

At K = 500:

- CatBoost identified 278 customers with actual churn.
- Logistic Regression identified 269.

At the standard classification threshold of 0.5, Logistic Regression achieved higher Recall and F1-score.

Therefore, the model choice depends on the intended use:

- For customer ranking and prioritisation, CatBoost showed an advantage in the tested scenarios.
- For classification at the standard threshold, Logistic Regression achieved higher Recall and F1-score.

### Limitations

The test-set evaluation measures the ability to predict historical churn.

It does not establish:

- Whether a discount or a call will change a customer's decision.
- Whether a selected customer can actually be retained.
- Whether a retention campaign will reduce churn.
- Whether the campaign will generate a positive financial return.

Financial evaluation would require information about contact costs, offer costs, customer value, and actual retention effectiveness.

### Conclusion

Both models identify useful patterns and rank customers by churn risk more effectively than random selection.

Tuned CatBoost achieved:

- Higher ROC-AUC.
- Higher Average Precision.
- Higher Precision.
- Better Precision@K for all tested group sizes.

Logistic Regression achieved higher Recall and F1-score at the standard threshold of 0.5.

**Decision:** tuned CatBoost is selected as the preferred model for the customer-prioritisation scenario because ranking customers under limited resources is the main project objective.

This decision is based on the observed model results. It does not demonstrate a proven financial advantage.

Before using the model in a real retention campaign, the company would need to evaluate the economics of the offers and validate their actual effect.

---

# Block IV. Summary

## Current Project Status

### Completed

- Initial data audit and quality check.
- EDA of contract type, tenure, and monthly charges.
- Analysis of the financial scale of churn by contract type.
- Potential-outlier check using the IQR rule.
- Logistic Regression evaluated using stratified five-fold cross-validation.
- Logistic Regression compared with `DummyClassifier`.
- Initial CatBoost model evaluated.
- CatBoost hyperparameter tuning completed.
- Customer-ranking comparison under a limited-resource scenario completed.
- Final evaluation on the held-out test set completed.

### Main Project Decisions

- The task is treated as binary classification.
- The model output is used to rank customers by predicted churn probability.
- Logistic Regression serves as the baseline model.
- Tuned CatBoost is selected as the preferred model for the customer-prioritisation scenario.
- The test set was used only for final evaluation after model selection.

### Important Limitations

- EDA findings refer to the training set.
- EDA describes observed patterns and does not establish causal relationships.
- Model metrics measure historical churn prediction, not the effectiveness of retention offers.
- Financial campaign effectiveness and ROI have not been established.
- Additional data on contact costs, customer value, and retention-offer outcomes is needed for a real business evaluation.

### Next Steps

- Finalise the project documentation.
- Prepare the README.
- Publish a reproducible project version on GitHub.
- If campaign data becomes available, evaluate whether targeted retention activities actually change customer behaviour and produce a positive financial result.

---

**Overall research workflow:**

Business Context → Initial Hypothesis → EDA → Data Preparation → Baseline Model → Model Comparison → Hyperparameter Tuning → Customer Ranking → Final Test Evaluation → Business Interpretation

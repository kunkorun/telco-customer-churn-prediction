# Telco Customer Churn — Business Logic

> Working project journal
> This records the development of the business idea, arguments, refinements, and accepted decisions
> This is not a final report, records will be updated as the research produces actual results

---

## Block I Business logic

### 1 Business context

- I am considering an educational telecom-company case
- The company wants to reduce customer churn
- Retention resources are limited
- Therefore it cannot indiscriminately offer discounts and promotions to all customers
- The assumed business task is to help the company determine which customers should receive attention first
- This is an educational setup, not a confirmed description of a specific company
- Real budget values, retention cost, and customer value are not defined yet in the project

---

### 2 BL-001 Initial idea

- Idea: instead of spending resources on all customers, focus on those who are more likely to leave
- Mass discounts may be unprofitable
- Some customers would have stayed anyway, and a discount would reduce revenue from them
- With a limited budget, this may also leave fewer resources for customers who can really be retained
- As an illustration, a conditional scenario was discussed: if 150 out of 300 selected customers remain after a retention campaign, this may look like a successful result
- The customer count and financial values in this example are conditional and are not results of dataset analysis
- Why the idea looks reasonable: it accounts for limited resources and suggests evaluating the decision not only by prediction quality but also by potential business value

---

### 3 Critical review of the initial idea

- A high churn probability by itself does not mean that the customer can be retained with a discount or another offer
- The fact that a customer stayed after a campaign does not prove that the campaign changed their decision
- Some customers could have stayed without an offer
- In that case, a discount could create extra cost without extra effect
- Therefore two questions must be separated:
  1 How likely is the customer to leave
  2 Whether a retention offer changes customer behavior
- The available churn dataset mainly allows investigating the first question
- To reliably evaluate the effect of retention offers, additional campaign and outcome data or a specially designed experiment would be needed

---

### 4 Refined working hypothesis

- If customers are ranked by predicted churn probability in descending order and the top part of the list is selected, then among the selected customers the share of actual churn cases will be higher than with random selection of a comparable size
- If this is confirmed on historical data, such ranking may help prioritize customers for possible retention activities
- Status: the hypothesis is formulated but not yet tested
- It does not prove that discounts or other offers will actually reduce churn or increase profit

---

### 5 Preliminary ML task definition

- Task type: binary classification
- Assumed target: `Churn`
- Observation unit: customer
- Assumed output: a churn probability estimate for each customer that can be used for ranking
- These points need to be confirmed during the initial audit of the actual data file
- Metrics, validation scheme, and priority-group size have not yet been selected

---

### 6 Assumptions and limitations

- Limited retention resources are used as part of the educational business scenario, while the company's concrete constraints are unknown
- The customer-group size for evaluation will be selected and marked as a scenario assumption if the data or task does not provide a justified value
- Feature association with churn does not by itself confirm a causal relationship
- Churn prediction quality and retention campaign effectiveness are different outcomes and cannot be treated as interchangeable
- Monetary effect and ROI cannot be reliably calculated without justified data on campaign cost, customer value, and retention effectiveness

---

### 7 Documentation protocol

For each significant business decision I record:

1 Initial idea or question — what was assumed and why

2 Check and critique — risks, alternative explanations, and limitations

3 Refined hypothesis or decision — what was concluded after discussion or analysis

4 Evidence — which data results support the conclusion, or what still needs to be tested

5 Status — hypothesis, being tested, partially confirmed, not confirmed, or accepted decision

---

## Block II Research hypotheses EDA

### EDA-001 Relationship between contract type and customer churn

#### Question and initial hypothesis

- Research question: whether customer contract type is associated with churn probability
- Hypothesis: customers with a month-to-month contract `Month-to-month` churn more often than customers with a one-year contract `One year` or a two-year contract `Two year`
- Rationale: if churn rates differ substantially across contract types, this pattern may help identify customer groups with higher churn risk
- For the business, this is a potential opportunity to define segments that should receive more attention when designing retention activities

#### Validation method

- After the initial audit, the data was split into training and test sets in an 80/20 ratio
- `train_test_split`, `random_state=42`, and `stratify=y` were used
- The training set `X_train`, `y_train` was used to test the hypothesis
- For each contract type, the share of customers with `Churn = Yes` was calculated relative to the total number of customers in that category
- The results are presented as a relative-frequency table and a bar chart of churn rate by contract type

#### Validation results

| Contract type | Did not churn `No` | Churned `Yes` |
|---|---:|---:|
| Month-to-month | 57.25% | 42.75% |
| One year | 88.92% | 11.08% |
| Two year | 97.13% | 2.87% |

- A noticeable difference in churn rates across contract categories was found in the training set
- The churn rate among customers with month-to-month contracts is about 42.75%
- The churn rate among customers with two-year contracts is about 2.87%
- The churn rate in the first group is approximately 15 times higher than in the second

#### Interpretation

- The results support the initial hypothesis: in the training set under study, contract type is associated with noticeable differences in churn rate
- From the business-task perspective, this observation may be useful for further study of month-to-month customers as a potentially higher-risk segment
- However, the observed pattern does not prove that contract type itself causes churn
- The differences may be influenced by other customer characteristics, service conditions, and a combination of factors still to be investigated
- It also cannot be claimed that moving customers to longer contracts will reduce churn by itself
- A separate study would be required for such a conclusion

#### Decision based on validation results

- Hypothesis status: preliminarily supported by the training data
- What I record:
  - `Contract` is a potentially important feature for further churn prediction
  - Customers with month-to-month contracts require separate attention when investigating risk factors further
  - The observed association is not interpreted as a proven causal relationship
- Next actions:
  1 Check other potential churn factors
  2 If needed, investigate feature interactions and possible alternative explanations for the observed pattern
  3 Later, evaluate the contribution of `Contract` to model performance and validate final conclusions on the held-out test set
- These analysis results apply only to the training set
- They are not an ML-model quality estimate and do not yet confirm the effectiveness of specific retention activities

---

### EDA-002 Relationship between customer tenure and churn

#### Research question

- Whether customer tenure is associated with churn probability

#### Initial hypothesis

- Customers with short tenure churn more often than customers who have used the company's services for a long time
- Assumed business value: identifying higher churn risk among new customers may help define groups that need extra attention when designing retention activities

#### Validation method

- In the training set, the `tenure` feature was divided into six 12-month intervals
- For each interval, the share of customers with `Churn = Yes` was calculated

#### Results

| Tenure | Churn rate |
|---|---:|
| 0–12 months | 47.32% |
| 13–24 months | 28.61% |
| 25–36 months | 20.93% |
| 37–48 months | 19.48% |
| 49–60 months | 15.35% |
| 61–72 months | 6.81% |

#### Interpretation

- The training set shows a stable inverse relationship between tenure and churn rate
- In groups with longer tenure, churn occurs less often
- This is consistent with the initial hypothesis
- Customers at an early service stage may represent an important segment for further business analysis
- At the same time, the result does not prove that longer tenure by itself reduces churn probability
- The observed association may also be driven by contract type and other customer characteristics

#### Decision and status

- Status: the hypothesis is preliminarily supported by the training data
- I keep the `tenure` feature for further research and later modeling
- Binning is currently used as an EDA tool
- The need to include the derived feature `tenure_group` in the model will be evaluated separately later
- Next step: build a chart of churn rate by tenure group and continue investigating other factors

---

### EDA-003 Relationship between monthly charges and churn

#### Research question

- Whether customer monthly charges `MonthlyCharges` are associated with churn probability

#### Initial hypothesis

- Customers with higher monthly charges may churn more often
- Rationale: higher charges may potentially influence a customer's decision to continue using the company's services
- However, this assumption must be checked on the data
- The possible association may be explained not by charges themselves but by other customer characteristics, such as contract type or the set of connected services

#### Validation method

- In the training set `x_train`, `y_train`, the following `MonthlyCharges` statistics were calculated separately for customers with `Churn = Yes` and `Churn = No`:
  - number of customers
  - mean
  - median
  - standard deviation
- A boxplot was additionally built to compare monthly-charge distributions in the two groups

#### Results

| Metric | Did not churn `No` | Churned `Yes` |
|---|---:|---:|
| Number of customers | 4 139 | 1 495 |
| Mean charges | 61.34 | 74.86 |
| Median charges | 64.40 | 79.95 |
| Standard deviation | 31.14 | 24.60 |

- The mean monthly charges for churned customers are higher by 13.52
- The median is higher by 15.55
- The boxplot also shows that the median and mean are higher for churned customers
- At the same time, the distributions overlap substantially
- There are customers with high charges who stayed and customers with relatively low charges who churned

#### Interpretation

- The descriptive-statistics results preliminarily support the initial hypothesis
- In the training set, churned customers had higher average monthly charges
- However, the single feature `MonthlyCharges` is not enough to clearly separate churned and retained customers
- The observed association does not prove that higher charges cause churn
- Contract type, connected services, and other customer characteristics may influence the result

#### Decision and status

- Status: the hypothesis is preliminarily supported by the training data
- I keep the `MonthlyCharges` feature for further modeling
- Additional feature processing is not required at this stage

---

### EDA-004 Financial scale of churn by contract type

#### Research question

- Which contract types account for the largest total monthly charges of customers who churned

#### Idea and rationale

- Earlier it was established that churn rates differ noticeably across contract types
- The next step is to look at the same problem from a financial angle
- The assumption is that for the business task it is not enough to know only the share of churned customers
- It is also important to understand the absolute number of churn cases and the volume of monthly charges associated with churned customers
- This allows comparing churn prevalence with its potential financial scale

#### Validation method

- In the training set, customers with `Churn = Yes` were grouped by contract type `Contract`
- For each group, the following were calculated:
  - number of churned customers
  - sum of their `MonthlyCharges`
  - average monthly charges per churned customer
- These metrics were then joined with the total number of customers for each contract type and the churn rate inside the corresponding group

#### Results

| Contract type | Total customers | Churned customers | Churn rate | Sum of `MonthlyCharges` for churned customers | Average charges per churned customer |
|---|---:|---:|---:|---:|---:|
| Month-to-month | 3 102 | 1 326 | 42.75% | 97 528.85 | 73.55 |
| One year | 1 173 | 130 | 11.08% | 11 075.00 | 85.19 |
| Two year | 1 359 | 39 | 2.87% | 3 312.10 | 84.93 |
| Total | 5 634 | 1 495 | — | 111 915.95 | — |

- The `Month-to-month` category accounts for approximately 87.14% of the total monthly charges of churned customers in the training set
- At the same time, average monthly charges per churned customer are higher in the `One year` and `Two year` groups than in the `Month-to-month` group

#### Interpretation

- The results show that the contract type with the highest churn rate is also the contract type with the largest total monthly charges among churned customers
- The main reason is the much larger number of churned customers with month-to-month contracts
- At the same time, average charges per churned customer in this category are lower than for customers with one-year and two-year contracts
- For the business, this points to a potentially important segment: month-to-month customers combine a high observed churn rate with the largest absolute number of churn cases
- However, the obtained sum is not net profit or proven financial loss for the company
- `MonthlyCharges` describes monthly charges associated with churned customers but does not account for service cost, margin, or possible future revenue
- It also cannot be claimed that month-to-month customers overall bring the company more money
- The analysis concerns only the group of churned customers

#### Decision and status

- Status: the financial scale of churn by contract type has been described on the training set
- The following observations are recorded:
  - `Month-to-month` has the highest churn rate
  - The same category has the largest number of churned customers and the largest total sum of their monthly charges
  - Average charges per churned customer are higher in the one-year and two-year contract groups
  - Available data is insufficient to calculate net financial losses or evaluate retention-campaign effectiveness
- I use these results as part of the business interpretation of the project
- They do not by themselves prove the effectiveness of specific retention methods

---

### EDA-005 Checking potential outliers

#### Research question

- Whether the numerical features `tenure`, `MonthlyCharges`, and `TotalCharges` contain unusual values that may indicate data errors or require additional processing before modeling

#### Validation method

- Boxplots were built for three numerical features in the training set
- Then for each feature I calculated the first and third quartiles, the interquartile range IQR, and potential outlier bounds using the 1.5 interquartile range rule
- Potential outliers were considered values below the lower bound or above the upper bound
- Lower bound: Q1 − 1.5 × IQR
- Upper bound: Q3 + 1.5 × IQR

#### Results

| Feature | Q1 | Q3 | IQR | Lower bound | Upper bound | Potential outliers |
|---|---:|---:|---:|---:|---:|---:|
| tenure | 9.00 | 55.00 | 46.00 | −60.00 | 124.00 | 0 |
| MonthlyCharges | 35.66 | 90.00 | 54.34 | −45.84 | 171.51 | 0 |
| TotalCharges | 402.98 | 3 835.82 | 3 432.85 | −4 746.30 | 8 985.10 | 0 |

- For all three features, no observations were found beyond the calculated IQR bounds
- The distribution of `TotalCharges` has noticeable right skewness
- However, skewness by itself is not evidence of data errors

#### Interpretation

- According to the selected statistical criterion, potential outliers are absent
- Observed values do not go beyond the bounds calculated from the interquartile range
- This does not prove the complete absence of errors or any unusual values
- The IQR method only detects deviations relative to the specified statistical bounds

#### Decision and status

- Status: potential-outlier checking by the IQR rule is complete
- At this stage I keep all observations without additional processing
- No grounds were found to remove or cap values in `tenure`, `MonthlyCharges`, or `TotalCharges`
- If later modeling reveals problems related to feature distributions, processing methods will be considered separately and justified by experiment results

---

## Block III ML experiments

### ML-001 Baseline Logistic Regression

#### Experiment goal

- After completing the initial audit and main research checks, it is necessary to obtain the first working churn-prediction model
- The goal of this stage is to establish a baseline classification quality level that can later be compared with more complex models
- For the business task, the model should not only assign a customer class but also help rank customers by churn risk
- This may potentially allow prioritizing customers for retention activities under limited company resources

#### Model choice

- Logistic regression `LogisticRegression` was chosen as the first model
- Reasons for choice:
  - it is a standard and relatively simple binary-classification algorithm
  - it provides a baseline reference point before experimenting with more complex algorithms
  - its results are easy to compare with other classifiers
  - it allows assessing how well a linear combination of features helps predict churn
- At this stage, logistic regression is not assumed to necessarily become the final model
- Its task is to provide an initial result against which the value of later improvements can be evaluated

#### Data preparation

- The original dataset contains 7 043 customers and 21 columns
- After excluding the customer identifier and the target variable, 19 features are used for modeling
- Before training, the data was split into training and test sets in an 80/20 ratio
- `X_train`: 5 634 observations and 19 features
- `X_test`: 1 409 observations and 19 features
- `y_train`: 5 634 target values
- `y_test`: 1 409 target values
- `random_state=42` was used for reproducibility
- `stratify=y` preserved approximately the same class proportions in the training and test sets
- The target variable `Churn` was converted to binary format:
  - `No = 0`
  - `Yes = 1`
- The `customerID` column was excluded from model features because it is a customer identifier and does not describe customer characteristics by itself
- The `tenure_group` feature created during EDA for tenure research is not used in the first baseline
- It remained an auxiliary analysis tool, while the original numerical feature `tenure` is kept

#### Preprocessing organization through Pipeline

- A `Pipeline` and `ColumnTransformer` combination is used for data preparation
- Numerical features:
  - `SeniorCitizen`
  - `tenure`
  - `MonthlyCharges`
  - `TotalCharges`
- For numerical features, missing values are filled with the median through `SimpleImputer` and scaling is done through `StandardScaler`
- Categorical features:
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
- For categorical features, missing values are filled with the most frequent category and categories are encoded with `OneHotEncoder` using `handle_unknown="ignore"`

#### Why Pipeline was chosen

- All transformations are combined with model training in a single pipeline
- This allows applying the same transformation sequence during cross-validation, later training, and final evaluation
- During cross-validation, preprocessing parameters are learned only on the training part of the corresponding fold
- The validation part is used for evaluation, not for calculating scaling statistics or filling missing values
- This reduces the risk of data leakage during feature preparation
- Additional processing of potential outliers was not performed
- According to boxplots and the IQR rule check, no values were found beyond the calculated bounds

#### Evaluation scheme

- Stratified five-fold cross-validation `StratifiedKFold` is used for initial evaluation
- Parameters:
  - `n_splits=5`
  - `shuffle=True`
  - `random_state=42`
- Quality was evaluated with five metrics:
  - **ROC-AUC** — the model's ability to rank churn customers higher than non-churn customers
  - **Average Precision** — quality of finding the positive class at different thresholds
  - **Precision** — share of actually churned customers among those predicted as churned
  - **Recall** — share of detected churned customers among all actually churned customers
  - **F1** — balance between Precision and Recall
- For Precision, Recall, and F1, the standard classification threshold 0.5 was used
- Threshold tuning was not performed at this stage
- The test set was not used for model training, hyperparameter selection, or cross-validation result evaluation

#### Logistic Regression results

| Metric | Mean | Standard deviation |
|---|---:|---:|
| ROC-AUC | 0.846 | 0.013 |
| Average Precision | 0.662 | 0.019 |
| Precision | 0.653 | 0.026 |
| Recall | 0.543 | 0.041 |
| F1 | 0.592 | 0.030 |

- Standard deviation describes the spread of results across the five validation folds

#### Interpretation

- **ROC-AUC = 0.846**
  - The model shows an ability to rank customers by churn risk better than random ranking
- **Average Precision = 0.662**
  - The model shows useful quality in finding churn customers
  - For comparison, the prevalence of the positive class in the data must be considered
- **Precision = 0.653**
  - At the standard threshold 0.5, about 65.3% of customers assigned by the model to the `Yes` class actually belong to the positive class
- **Recall = 0.543**
  - At the same threshold, the model detects about 54.3% of all positive-class customers
  - About 45.7% of actual churn cases remain undetected
- **F1 = 0.592**
  - The metric reflects the balance between Precision and Recall
  - It does not directly measure the economic effectiveness of retention activities
- The spread of results across folds is relatively small
- This preliminarily indicates fairly stable results within the selected cross-validation scheme
- It does not guarantee similar quality on any new data

#### Initial model conclusion

- Logistic regression shows an ability to extract useful patterns from customer characteristics and predict churn
- However, at the standard threshold 0.5, the model misses a noticeable share of actual churn cases
- This matters for the business task: missed customers may remain without a timely retention offer
- In the future, it is necessary to evaluate how prediction quality, selected customer count, and budget constraints relate to each other
- High classification quality by itself does not prove that using the model will reduce churn or increase company profit
- Status: the first ML model has been trained and evaluated using cross-validation results
- It is accepted as a working baseline for further experiments

---

### ML-002 Comparison with DummyClassifier

#### Experiment goal

- Check how much using customer characteristics improves churn prediction compared with the simplest classifier that does not look for patterns in features
- For this, logistic regression is compared with `DummyClassifier`

#### Control model choice

- `DummyClassifier` with the `most_frequent` strategy was used
- It always predicts the most frequent class in the training set
- In this case it is `No` — the customer did not churn
- Earlier target-variable analysis found the following distribution:
  - `No`: 5 174 customers, or about 73.46% of the original sample
  - `Yes`: 1 869 customers, or about 26.54% of the original sample
- `DummyClassifier` does not use `Contract`, `tenure`, `MonthlyCharges`, or other feature values for individual prediction
- It serves as a control point for comparison with the ML model

#### Comparison method

- Both models were evaluated with the same stratified five-fold cross-validation scheme on the same training set
- The same metrics were used:
  - ROC-AUC
  - Average Precision
  - Precision
  - Recall
  - F1
- For Precision, Recall, and F1, the standard threshold 0.5 was used
- For `DummyClassifier`, `zero_division=0` was set
- The model does not predict the positive class, so Precision is undefined in that case without this convention
- The test set was not used in the comparison

#### Comparison results

| Metric | Logistic Regression | DummyClassifier |
|---|---:|---:|
| ROC-AUC | 0.846 | 0.500 |
| Average Precision | 0.662 | 0.265 |
| Precision | 0.653 | 0.000 |
| Recall | 0.543 | 0.000 |
| F1 | 0.592 | 0.000 |

- Logistic Regression mean values were obtained across five folds
- For DummyClassifier, standard deviations for all metrics after rounding to three digits are 0.000

#### Result interpretation

- `DummyClassifier` always predicts `No`, so it detects no churn customers
- Its Recall and F1 are zero
- Precision is also taken as zero according to the `zero_division=0` setting
- At the same time, such a classifier would have Accuracy around 73.5%
- It would correctly predict the majority class for most customers
- This shows why Accuracy alone is not suitable as the only success criterion for the project
- DummyClassifier ROC-AUC is 0.500
- The model cannot rank customers by individual risk because it outputs the same class for every observation
- Average Precision is about 0.265
- This corresponds approximately to the positive-class share in the data
- Compared with the control model, logistic regression shows:
  - a higher ability to rank customers by churn risk
  - better positive-class search quality
  - non-zero Precision, Recall, and F1 at the standard classification threshold
- Thus, including features allows extracting useful information for churn prediction compared with a naive rule that ignores individual customer characteristics

#### Decision based on comparison results

- Status: control comparison completed
- Logistic regression is accepted as the working baseline model
- `DummyClassifier` remains a control point for evaluating later improvements
- This comparison confirms that the model uses useful data patterns
- It does not yet prove economic effectiveness
- For that, ranking customers under a limited budget will need to be investigated separately
- With suitable data, retention-campaign outcomes would also need to be evaluated

---

### ML-003 Baseline CatBoost

#### Experiment goal

- Check whether CatBoost can improve the baseline Logistic Regression results on the customer churn prediction task
- The first configuration selected was `CatBoostClassifier` with the following parameters:
  - `iterations=500`
  - `learning_rate=0.05`
  - `depth=6`
  - `loss_function='Logloss'`
  - `eval_metric='AUC'`
  - `random_seed=42`
- Categorical features were passed to the model without preliminary One-Hot Encoding
- 5-fold Stratified Cross-Validation was used for evaluation
- The test set did not participate in the experiment

#### Results

| Metric | Result |
|---|---:|
| ROC-AUC | 0.845 ± 0.010 |
| Average Precision | 0.660 ± 0.023 |
| Precision | 0.665 ± 0.043 |
| Recall | 0.513 ± 0.020 |
| F1-score | 0.578 ± 0.024 |

- Precision, Recall, and F1-score were calculated at the standard classification threshold 0.5

#### Result interpretation

- Compared with Logistic Regression, CatBoost did not improve customer-ranking quality
- Comparison:
  - ROC-AUC: 0.845 versus 0.846 for Logistic Regression
  - Average Precision: 0.660 versus 0.662
  - Precision: 0.665 versus 0.653
  - Recall: 0.513 versus 0.543
  - F1-score: 0.578 versus 0.592
- CatBoost has slightly higher Precision
- However, Recall and F1-score are lower
- Therefore, at the standard threshold, the model selects customers somewhat more precisely but detects a smaller share of actual churn cases
- The obtained results do not show a clear advantage of CatBoost over Logistic Regression
- At the same time, the experiment tests only one CatBoost configuration
- It is too early to make a final conclusion about the algorithm's potential

#### Conclusion and next actions

- The initial CatBoost configuration did not outperform the Logistic Regression baseline on the main metrics
- Therefore the next step selected was CatBoost hyperparameter tuning
- Average Precision is used as the main tuning metric
- The business task assumes ranking customers by churn risk and then selecting a priority retention group under a limited budget
- After tuning, all metrics must be re-evaluated on five folds
- Then the tuned model must be compared with the baseline
- Only after model selection can final evaluation on the test set proceed
- Important: ML metric values alone do not prove the financial effectiveness of a retention campaign
- For that, the economics of the offer and the potential effect of customer retention would need additional evaluation

---

### ML-003A CatBoost hyperparameter tuning

#### Stage goal

- Check whether hyperparameter tuning can improve ranking quality compared with the first CatBoost configuration

#### Method

- `RandomizedSearchCV` was used on the training set
- Three folds were used for the initial search to reduce experiment time
- Six parameter combinations were checked
- Main search metric: `average_precision`
- Parameter ranges were fixed in the code:
  - `iterations`
  - `depth`
  - `learning_rate`
  - `l2_leaf_reg`
- After the search, the best configuration was re-evaluated on 5 folds

#### Results

- Mean Average Precision increased from 0.660 to 0.672
- Mean ROC-AUC increased from 0.845 to 0.851
- Full metric values are used in the later comparison with `Logistic Regression`

#### Status

- Tuning is complete
- The final configuration is passed to ranking experiments and final evaluation

---

### ML-004 Customer ranking under a limited budget

#### Experiment goal

- Move from standard ML-metric comparison to evaluating practical usefulness for the business task
- Business hypothesis: if the company has limited retention resources, selecting customers by predicted churn risk will allocate resources more effectively than random selection
- The models compared were Logistic Regression and CatBoost with tuned hyperparameters

#### Method

- A churn probability estimate was created for each customer
- Customers were then ranked by predicted risk in descending order
- After that, the quality of selecting the top K customers was evaluated
- To obtain predictions on the training set, five-fold cross-validation with out-of-fold predictions was used
- For each customer, the prediction was made by a model that was not trained on that row
- The following K values were considered: 100, 200, 300, 500, and 1000 customers
- Metrics used:
  - **Precision@K** — share of actual churn customers among K selected customers
  - **Recall@K** — share of all actual churn customers that entered the selected group

#### Results

| K | Logistic Regression found | CatBoost found | Precision@K LR | Precision@K CatBoost |
|---:|---:|---:|---:|---:|
| 100 | 89 | 88 | 0.890 | 0.880 |
| 200 | 165 | 174 | 0.825 | 0.870 |
| 300 | 244 | 261 | 0.813 | 0.870 |
| 500 | 382 | 391 | 0.764 | 0.782 |
| 1000 | 696 | 691 | 0.696 | 0.691 |

- The share of churn customers in the training set is approximately 26.5%
- With random selection of 300 customers, about 80 churn customers would be expected
- Both models found significantly more

#### Result interpretation

- Both models substantially outperform random selection in concentrating actual churn customers at the top of the risk ranking
- CatBoost showed better results for K = 200, 300, and 500
- At K = 300 it found 261 churn customers versus 244 for Logistic Regression
- Precision@300 was 87.0% versus 81.3%
- At K = 100 Logistic Regression had a small advantage
- At K = 1000 it found five more churn customers than CatBoost
- Thus, model advantage depends on target-group size
- For a scenario where the company can work with several hundred customers, CatBoost looks more promising based on this experiment

#### Business interpretation

- The range 200–400 customers was selected as a preliminary scenario for further analysis
- It is treated as a working hypothesis, not as a finally established optimal group size
- When choosing the target group, it is necessary to consider not only ranking quality but also operational constraints:
  - number of employees and available time for customer contacts
  - cost of handling one customer
  - cost of a discount or another retention offer
  - potential marginal customer value and retention-campaign effectiveness
- Increasing group size allows covering more potentially churn-prone customers
- This is usually accompanied by lower Precision
- Customers with lower predicted risk enter the group
- The decision about group size should be made with budget and expected financial effect in mind

#### Experiment limitations

1 Results were obtained on the training set using out-of-fold predictions

2 Since CatBoost hyperparameters were tuned on the same training set, the comparison is not a fully independent estimate

3 Historical churn is not equivalent to the possibility of retaining a customer with a discount or a call

4 Financial campaign effectiveness has not yet been calculated

5 It would require estimating retention cost, customer value, and the effect of the offer itself

#### Conclusion and next actions

- For a limited-budget scenario, tuned CatBoost showed an advantage over Logistic Regression when selecting 200–500 customers
- At the same time, there is no basis to consider it best for every possible group size
- The next stage will be final evaluation of the selected models on the previously held-out test set
- After fixing the results, the project code needs to be finalized, a README prepared, and a reproducible project version published on GitHub

---

### ML-005 Final model evaluation on the test set

#### Goal

- Evaluate the generalization ability of Logistic Regression and tuned CatBoost on the held-out test set
- The test set was not used for model training or hyperparameter tuning

#### Results

| Metric | Logistic Regression | CatBoost tuned |
|---|---:|---:|
| ROC-AUC | 0.842 | 0.846 |
| Average Precision | 0.634 | 0.669 |
| Precision | 0.657 | 0.670 |
| Recall | 0.559 | 0.521 |
| F1-score | 0.604 | 0.586 |

- Precision, Recall, and F1-score were calculated at the classification threshold 0.5

#### Confusion-matrix analysis

- Logistic Regression correctly identified 209 churn customers
- It missed 165 cases
- CatBoost correctly identified 195 churn customers
- It missed 179 cases
- At the same time, CatBoost made fewer false positives: 96 versus 109 for Logistic Regression
- This means that at threshold 0.5, CatBoost less often assigned customers without actual churn to the positive class
- At the same time, it detected fewer churn customers

#### Link to the business task

- In the previous experiment, customer-ranking quality was evaluated under a limited target-group size
- On the test set, CatBoost showed better Precision@K results for all checked group sizes from 100 to 500 customers
- At K = 300, CatBoost identified 199 customers with actual churn versus 198 for Logistic Regression
- At K = 500, the difference was 278 versus 269
- Thus, CatBoost keeps an advantage in customer ranking for a limited-budget scenario
- At the standard threshold 0.5, Logistic Regression shows higher Recall and F1-score

#### Limitations

- Test-set evaluation reflects the quality of predicting historical churn
- It does not prove retention-campaign effectiveness
- The model does not determine whether a discount or a call will change the customer's decision to stay
- Financial return from the campaign has not been calculated
- Evaluating it requires data on contact and offer cost, marginal customer value, and actual retention effectiveness

#### Conclusion

- Both models reveal useful patterns and outperform random selection in ranking customers by churn risk
- `CatBoost tuned` shows higher ROC-AUC, Average Precision, and Precision on the test set
- It also shows better Precision@K in the considered limited-budget scenarios
- `Logistic Regression` outperforms CatBoost in Recall and F1-score at threshold 0.5
- Given the business goal of prioritizing customers for retention, CatBoost is selected as the preferred model for further analysis
- This decision is based on the experiment results
- It does not mean proven financial advantage
- Before deploying the model into a real retention campaign, economics evaluation and offer-effect validation will be required

---

## Block IV Summary

### Current project status

- Done: initial data audit and quality check
- Done: EDA for contract type, tenure, and monthly charges
- Done: analysis of financial churn scale by contract type
- Done: potential-outlier check using the IQR rule
- Done: Logistic Regression evaluated with stratified five-fold cross-validation
- Done: results compared with DummyClassifier
- Done: initial CatBoost evaluated and hyperparameter tuning performed
- Done: ranking comparison under a limited budget completed
- Done: final evaluation on the test set completed
- Next: prepare a README and publish a reproducible project version on GitHub
- The test set was used only for final evaluation after model selection
- All results in the EDA sections refer to the training set
- They describe discovered patterns but are not ML-model quality results or proof of retention-activity effectiveness
- Financial campaign effectiveness has not yet been proven
- For business use, additional data is needed on contact cost, customer value, and the actual effect of retention offers
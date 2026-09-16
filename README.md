# Customer prioritization in a banking campaign

A Python machine learning project answering the following question: **using information available before the call, is it possible to prioritize customers based on their likelihood of subscribing to a term deposit?**

[Open the notebook](Priorizacao_Clientes_Bank_Marketing_GitHub.ipynb) · [Data source](https://archive.ics.uci.edu/dataset/222/bank+marketing)

## How to run

1. Open `Priorizacao_Clientes_Bank_Marketing_GitHub.ipynb` in Google Colab.
2. Run the cells in order with internet access. The first code cell downloads the data directly from UCI; there is no need to copy files to Drive.
3. The dependencies used are `pandas`, `numpy`, and `scikit-learn`, which are normally available in Colab. In another environment, install them with `pip install pandas numpy scikit-learn`.

The public notebook was distributed **without execution outputs or personal metadata**. The numbers below were obtained during the original run and can be reproduced by executing all cells.

## Method

- Data: `bank-additional-full.csv`, containing 41,188 records from campaigns run by a Portuguese bank, ordered by date in the original file.
- Target: subscription to the term deposit (`y = yes`).
- Features: customer attributes and campaign history; in the context-aware version, month, day of the week, and planned contact channel are also included.
- `duration` was excluded because it is only known after the call; `campaign` was also excluded because it includes the current contact.
- Chronological split: 60% training, 20% validation, and 20% test. Categorical and numerical preprocessing is fitted only on the training set.
- Models compared on the validation set: logistic regression, random forest, and logistic regression with context variables. The selected model was refitted on the first 80% before the final evaluation.
- Metrics: *average precision*, ROC-AUC, and the subscription rate among the 10% of customers with the highest scores.

## Results

| Metric | Validation (logistic regression with context) | Final test |
|---|---:|---:|
| Subscription rate in the sample | 11.1% | 30.8% |
| Average precision | 0.197 | 0.493 |
| ROC-AUC | 0.643 | 0.696 |
| Subscription rate in the top 10% | 23.7% | 51.3% |
| Subscriptions in the top 10% | 195 | 423 |
| Mean predicted probability | 6.9% | 17.4% |

In the final test set, the 824 records in the top 10% accounted for **423 of the 2,540 subscriptions**. The ranking was useful for prioritizing contacts, but the mean predicted probability (17.4%) was substantially below the observed subscription rate (30.8%).

## Mathematical intuition

### From customer attributes to a score

Logistic regression represents each customer as a vector of features \(x\). Numerical features are standardized, while categorical features are converted into indicator columns. The model assigns a coefficient to each resulting feature and computes:

$$
z = \beta_0 + \sum_{j=1}^{m}\beta_j x_j
$$

The sigmoid function converts this score into a value between zero and one:

$$
p = \sigma(z) = \frac{1}{1+e^{-z}}
$$

Here, \(p\) is the model’s estimated probability of subscription. A larger coefficient increases the score when its associated feature is present, **holding the other features fixed**. Coefficients describe relationships learned from the training data; they do not establish causal effects.

### How the model learns

The coefficients are fitted by minimizing *log-loss* over the training observations:

$$
L(\beta) =
-\frac{1}{n}\sum_{i=1}^{n}
\left[y_i\log(p_i)+(1-y_i)\log(1-p_i)\right]
$$

Here, \(y_i=1\) indicates that customer \(i\) subscribed. Log-loss penalizes confident incorrect predictions particularly strongly. The implementation also uses regularization, which discourages excessively large coefficients. Optimization uses the derivatives of this objective to find coefficients that reduce the loss.

### Ranking customers under a capacity limit

The practical question is not simply whether \(p\) exceeds 0.5. If a team can contact only 10% of customers, it can rank them by \(p\) and select the highest-scoring group. We evaluate that decision using:

$$
\text{Top-10\% subscription rate}
=
\frac{\text{subscriptions in the selected group}}
{\text{customers in the selected group}}
$$

$$
\text{Lift@10\%}
=
\frac{\text{top-10\% subscription rate}}
{\text{overall subscription rate}}
$$

In the final test period, **423 of the 824 selected customers subscribed**. Their subscription rate was **51.3%**, compared with **30.8%** across the entire test set, giving a lift of approximately **1.66**. The selected group contained **423 of all 2,540 subscriptions**, or approximately **16.7%**, while representing 10% of customers.

### Ranking is different from calibration

A model can rank customers usefully while producing probabilities that are too high or too low. In the test period, the mean predicted probability was **17.4%**, whereas the observed subscription rate was **30.8%**. The model therefore underestimated the overall subscription frequency in that period, even though its ranking helped prioritize contacts.

One relevant limitation is the change in subscription rates across the chronological split: **4.8% in training, 11.1% in validation, and 30.8% in testing**. These results support using the model’s scores for **relative prioritization** in this historical exercise; they do not support treating every score as a well-calibrated probability in a future campaign.

## Decisions and limitations

- The data dictionary describes `pdays = 999` as indicating no previous contact, but 4,110 records have this code together with `previous > 0` and `poutcome = failure`. The notebook uses `previous` to preserve information about prior campaign history.
- The subscription rate changes substantially across periods: 4.8% in training, 11.1% in validation, and 30.8% in testing. This limits the extrapolation of the probabilities.
- The test rate and composition were inspected during exploratory data analysis; therefore, the evaluation was not completely blind. Model selection used validation metrics. The final analysis should be interpreted as an honest retrospective exercise, not as a prospective untouched benchmark.
- `month` helped on validation, but the dataset does not provide an explicit year for each row to distinguish seasonality from changes between campaigns. Interpreting the month as a cause of subscription would therefore be unwarranted.
- Using `contact` assumes that the channel has already been defined when preparing the call list. If this information is only available after the call, a prospective model should exclude it and be reevaluated.
- The data are historical (Portugal, 2008–2010). The results do not demonstrate performance in current campaigns or in other populations.

## Credits

Data: Moro, S., Rita, P., & Cortez, P. (2014), [*Bank Marketing*](https://doi.org/10.24432/C5K306), UCI Machine Learning Repository. The dataset is distributed under [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/).

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

In the final test set, the 824 records in the top 10% accounted for **423 of the 2,540 subscriptions**. The ranking was useful for prioritizing contacts, but the mean predicted probability (17.4%) was lower than the observed subscription rate in that group (51.3%), indicating that the probabilities were not well calibrated.

## Decisions and limitations

- The data dictionary describes `pdays = 999` as indicating no previous contact, but 4,110 records have this code together with `previous > 0` and `poutcome = failure`. The notebook uses `previous` to identify historical contact and treats `pdays` as a separate feature.
- The subscription rate changes substantially across periods: 4.8% in training, 11.1% in validation, and 30.8% in testing. This limits the extrapolation of the probabilities.
- The test rate and composition were inspected during exploratory data analysis; therefore, the evaluation was not completely blind. Model selection used validation metrics. The final analysis should be interpreted as an estimate under temporal shift, not as a fully untouched benchmark.
- `month` helped on validation, but the dataset does not provide an explicit year for each row to distinguish seasonality from changes between campaigns. Interpreting the month as a cause of subscription would be incorrect.
- Using `contact` assumes that the channel has already been defined when preparing the call list. If this information is only available after the call, a prospective model should exclude it and be reevaluated.
- The data are historical (Portugal, 2008–2010). The results do not demonstrate performance in current campaigns or in other populations.

## Credits

Data: Moro, S., Rita, P., & Cortez, P. (2014), [*Bank Marketing*](https://doi.org/10.24432/C5K306), UCI Machine Learning Repository. The dataset is distributed under [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/).

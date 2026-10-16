# Bank Marketing — Model Report

**Student:** Gumar Nurtore  
**SEED:** 3242

## Goal and data

The bank can call only 10% of clients in one campaign. The goal is to select clients who are more likely to open a term deposit. The target is `y`: yes means 1 and no means 0.

The main metric is precision at the top 10%. It shows the share of positive answers among selected clients. Accuracy alone is not useful enough here. Always predicting “no” gives 88.73% accuracy on the development training set, but finds no positive clients.

The supplied UCI Bank Marketing data contain 32,940 labelled rows and 8,236 test rows. The data source is Moro, Rita and Cortez (2014), with details and the licence in README.md.

The model uses 19 features. The column `duration` is excluded because the call length is unknown before the call. Using it would cause data leakage. The column `id` is also excluded because it identifies a row.

## Preparation and validation

The labelled data are divided into 80% development training data and 20% hold-out data. Five-fold stratified cross-validation is used on the development part. Stratification keeps a similar share of positive answers in each fold. All models use the same folds and seed 3242.

Preprocessing is inside a Pipeline, so it learns only from the training part of each fold. Numeric missing values are filled with medians, and numeric features are scaled. Categorical features use one-hot encoding.

The value `unknown` stays as a separate category. For `pdays`, 999 means no previous contact. It is replaced with the median of observed days, and an extra indicator keeps the information about no previous contact.

## Model comparison

The following results are from cross-validation:

| Model | Mean precision at top 10% | Fold standard deviation |
|---|---:|---:|
| Majority baseline | 0.1116 | 0.0181 |
| Logistic regression | 0.4994 | 0.0271 |
| Decision tree | 0.4994 | 0.0259 |
| Random forest | 0.5108 | 0.0217 |

For logistic regression, replacing `unknown` with the most common category gives 0.4991. Using balanced class weights gives 0.4987. Neither improves the main metric in this comparison.

The search checks logistic regression with `C` values of 0.01, 0.1, 1 and 10. For random forest, it checks depths of 8, 12 and unlimited, with minimum leaf sizes of 10 and 30.

The selected model is a random forest with 200 trees, depth 12 and minimum leaf size 10. It has the highest mean score and beats logistic regression on all five folds. However, its advantage is small compared with the fold spread, so there is no clear proof that it is better in general. The unlimited-depth forest scores almost the same, at 0.5104, so limited depth is preferred.

## Hold-out results

The model settings are fixed before the hold-out evaluation.

| Metric | Result |
|---|---:|
| ROC-AUC | 0.7989 |
| Average precision | 0.4653 |
| Precision at top 10% | 51.52% |
| Recall at top 10% | 45.69% |
| Lift | 4.57 |

The model selects 658 of 6,588 clients. Among them, 339 said yes and 319 said no. It does not select 403 clients who said yes. Another 5,527 clients are not selected and said no.

Random selection would find about 74 positive clients on average with the same number of calls. The model finds 339. These results describe historical answers; they do not prove an increase in future sales.

## Final files and limitations

After evaluation, the fixed pipeline is trained on all labelled data and saved as `model.joblib`. It accepts the raw test features without `id`. A separate Python process successfully loads it and checks its predictions.

The file `predictions.csv` contains all 8,236 test clients. The 823 highest scores receive `y_pred=1`. Equal scores are resolved using a fixed random order. The cutoff depends on each client list, rather than using a fixed probability of 0.5.

Random validation mixes historical periods. Real campaigns predict future clients, so changes in the economy or client behaviour may reduce performance. Changes in input data and unknown categories may also cause problems.

After deployment, the bank should monitor missing values, new categories and score distributions. When answers become available, it should check precision at 10%, recall and lift. Hidden-test performance is not known.

## Assistance

Codex helped with code, ChatGPT helped with  explanations of tasks .

# Model Card — Bank Marketing

**Student:** Gumar Nurtore  
**Seed:** 3242

## Purpose

The bank can call only 10% of clients in one campaign wave.
The model ranks clients before a call.
The target is term-deposit subscription: yes = 1, no = 0.
Precision at the top 10% is the main selection metric.

This is a teaching project, not a model ready for real banking use.

## Data and validation

The supplied data come from the UCI Bank Marketing dataset
(Moro, Rita and Cortez, 2014), with calls from 2008 to 2010.
The starter contains 32,940 labelled rows and 8,236 hidden-test rows.
The source and licence information remain in README.md.

We exclude id and duration. Duration is known only after a call
and would leak information into a pre-call prediction.

We reserve 20% of labelled rows as a stratified hold-out.
We compare and tune models using the same five stratified folds
on the remaining 80%. All random operations use seed 3242.

## Pipeline

Numeric missing values are filled with training-fold medians.
Numeric features are standardised.
Categorical features are one-hot encoded.
Unknown values remain a separate category.

For pdays, 999 means no previous contact.
We replace it with the median of observed days and add an indicator.

All fitted preprocessing is inside the pipeline.
A notebook check verifies the medians against one training fold.

The selected model is a random forest with 200 trees,
max_depth=12 and min_samples_leaf=10, without class weighting.

## Model selection

The majority baseline has CV accuracy 0.8873 but ROC-AUC 0.5000.
Its mean precision at the top 10% is 0.1116.

Initial mean CV precision at the top 10%:
- Logistic regression: 0.4994.
- Decision tree: 0.4994.
- Random forest: 0.5108.

Replacing unknown with the most frequent category gives 0.4991
for logistic regression. Balanced class weights give 0.4987.
Neither improves its main metric in this comparison.

We tune logistic C over [0.01, 0.1, 1, 10].
We tune forest depth over [8, 12, None] and minimum leaf size
over [10, 30]. The forest keeps its original settings.

The selected forest scores higher than the best logistic model
on all five folds. Its mean advantage is about 0.0114.
The difference is small and does not prove statistical superiority.
The unlimited-depth forest is almost tied; we prefer limited depth.

## Hold-out results

Model choices were fixed before hold-out evaluation.

| Metric | Result |
|---|---:|
| ROC-AUC | 0.7989 |
| Average precision | 0.4653 |
| Precision at top 10% | 0.5152 |
| Recall at top 10% | 0.4569 |
| Lift over random selection | 4.57 |
| Capacity cutoff | 0.326503 |

We select 658 of 6588 clients.
Among selected clients, 339 said yes and 319 said no.
We miss 403 clients who said yes.
Another 5527 clients were not selected and said no.

Random selection would find about 74.1
positive clients on average with the same number of calls.

These are historical outcomes, not measured additional sales
caused by deploying the model.

## Decision rule and saved files

We select floor(0.10 * number of clients) highest scores.
Equal scores are resolved with a fixed seeded random order.
The score cutoff is recalculated for each campaign list.
The hold-out cutoff is not reused as a fixed test threshold.

After evaluation, the fixed pipeline is refitted on all labelled data.
model.joblib accepts the raw test features without id.
predictions.csv contains id, y_pred and y_score for all test rows.

A fresh Python process checks model loading and confirms that
all scores and call decisions match the saved prediction file.
The hidden-test labels are unavailable, so its quality is unknown.

## Limits and monitoring

Random validation mixes historical periods and may give optimistic
results compared with predicting a future campaign.

The model may fail if economic conditions or client behaviour change.
It may also fail if input values, categories or data collection change.
Its scores have not been validated as calibrated probabilities.

Monitor input missingness, unknown categories, score distributions
and the number of selected clients.
When outcomes become available, monitor precision at 10%, recall,
lift and performance across relevant client groups.

## Tools and assistance

Python and scikit-learn were used for analysis.
Codex was used for code drafting and ChatGPT for explanations.

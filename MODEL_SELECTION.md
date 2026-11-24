## The question
This model predicts customer churn risk so retention teams can intervene early; false negatives hurt by missing customers who churn, while false positives waste retention outreach cost.

## Candidates
| Candidate | Why I tried it | AUC | F1 | F1 per slice (lowest) | p95 latency (ms) | Size (KB) | MLflow run ID |
|---|---|---|---|---|---|---|---|
| logreg | Fast linear baseline with standardized numeric features and one-hot categoricals | 0.798 | 0.495 | see MLflow slice metrics | 4.4 | 4 | 96fc7eea4c584112ac59c0d9daa4ac56 |
| random_forest | Non-linear tree ensemble (n_estimators=200, max_depth=8) to capture feature interactions | 0.759 | 0.454 | see MLflow slice metrics | 20.0 | 3160 | df590e7f01f94c768f499f6fb8f177b1 |
| gradient_boosting | Sequential boosting model to test higher-bias/lower-variance tabular performance | 0.748 | 0.417 | see MLflow slice metrics | 4.3 | 142 | 0c841673d8b04b128c4547ab3df7ed44 |

## The champion
I picked `logreg` (MLflow model version 1) because it achieved the highest AUC (`0.798`) and F1 (`0.495`) while keeping p95 latency at `4.41 ms` (`4.4 ms` rounded) and model size at `4 KB`. I accepted trading off non-linear tree capacity from `random_forest` (`20.0 ms`, `3160 KB`) and `gradient_boosting` (`4.3 ms`, `142 KB`), which scored lower on both AUC and F1 on this dataset.

## My promotion rules
- Latency budget (p95): 25.0 ms, because single-row online inference must stay fast and predictable under load.
- Minimum gain to replace the champion (F1 points): 0.005, because replacing a live champion model requires a meaningful F1 improvement rather than noise.

## What I would check next
I would check per-slice F1 across contract segments (especially small slices) and monitor input feature drift and target shift once deployed in production.

# Employee Attrition Classification

This project is a portfolio modernization of a collaborative 2025.2 ICMC Júnior Statistics Trainee project. It applies a reproducible, leakage-aware machine-learning workflow to the classification of the recorded `Attrition` label (`No` / `Yes`) in an HR analytics dataset. The final model is a Logistic Regression selected using training-only validation evidence, then evaluated once on a reserved holdout set.

## Key Results

| Stage | Precision — Yes | Recall — Yes | F1 — Yes |
|---|---:|---:|---:|
| Out-of-fold validation | 0.5722 | 0.6011 | 0.5863 |
| Final holdout | 0.5273 | 0.4915 | 0.5088 |

The holdout evaluation also produced accuracy of **0.8478**, ROC-AUC of **0.8098**, and average precision of **0.5783**.
The selected model is Logistic Regression, with an OOF-selected decision threshold of **0.34**.

## Problem

The technical task is binary classification of the recorded `Attrition` label. The dataset does not document observation dates, a prediction horizon, or whether all features were available before a recorded departure. This work therefore does not claim to predict which employees will leave in the future.

## Dataset

The tracked file [`data/dados.csv`](data/dados.csv) contains 1,470 rows and 35 columns. It was supplied to trainees during the ICMC Júnior Statistics Trainee activity. Original project materials describe the analyzed company as fictional.

The repository does not document the external creator, source URL, redistribution license, or whether the CSV was modified before it was supplied to trainees. The file is included as the version used by this portfolio project.

## Methodology

- A stratified train/test split is created before any learned preprocessing.
- Preprocessing is encapsulated in model pipelines.
- Stratified 5-fold cross-validation is performed on training data only.
- The benchmark includes Dummy, Logistic Regression, Random Forest, Gradient Boosting, and XGBoost classifiers.
- Hyperparameter tuning uses training data only.
- F1 for `Attrition=Yes` is the primary selection metric.
- The decision threshold is optimized using out-of-fold probabilities generated on training data only.
- The final configuration is frozen before the holdout is opened; the holdout is used only for final evaluation.

## Exploratory Analysis

The notebook uses training-only exploratory analysis for target distribution, selected categorical attrition rates, selected numerical distributions, and Pearson/Spearman correlation summaries. These observations are descriptive and unadjusted; they do not establish causal effects or independent predictive value.

## Model Selection and Threshold

The final configuration was selected from training-only evidence:

| Component | Final choice |
|---|---|
| Model | Logistic Regression |
| `C` | 10 |
| `class_weight` | `None` |
| Solver | `lbfgs` |
| Decision threshold | 0.34 |

At the selected threshold, Logistic Regression achieved OOF precision of 0.5722, recall of 0.6011, and F1 of 0.5863 for `Attrition=Yes`.

## Final Holdout Evaluation

The frozen model was fit once on all training observations and evaluated once on the reserved holdout set.

| Metric | Value |
|---|---:|
| Accuracy | 0.8478 |
| Precision — Yes | 0.5273 |
| Recall — Yes | 0.4915 |
| F1 — Yes | 0.5088 |
| ROC-AUC | 0.8098 |
| Average precision | 0.5783 |

|  | Predicted No | Predicted Yes |
|---|---:|---:|
| Actual No | TN = 283 | FP = 26 |
| Actual Yes | FN = 30 | TP = 29 |

The lower holdout F1 is recorded as a generalization difference; it was not used to retune the model, hyperparameters, or threshold.

## Limitations

- This is an internal evaluation on one historical dataset and one train/test split, not external validation.
- The same dataset was analyzed during the earlier trainee project, so it was not entirely unfamiliar to the project author at a human-analysis level; however, the V2 holdout was not used for model selection, hyperparameter tuning, or threshold optimization.
- The number of positive holdout cases is limited.
- The workflow classifies a recorded label and does not establish prospective prediction, causality, or generalization to other organizations or periods.

## Repository Structure

| Path | Purpose |
|---|---|
| [`notebooks/employee_attrition_v2.ipynb`](notebooks/employee_attrition_v2.ipynb) | Current portfolio notebook and validated workflow |
| [`notebooks/employee_attrition_original.ipynb`](notebooks/employee_attrition_original.ipynb) | Archival version of the collaborative trainee project |
| [`data/dados.csv`](data/dados.csv) | Dataset used by the notebooks |
| [`docs/reproducibility.md`](docs/reproducibility.md) | Reference environment and reproduction procedure |
| [`requirements.txt`](requirements.txt) | Pinned validated Python dependencies |

## Reproducibility

The validated reference environment is macOS ARM64 with Python 3.11.14 and Homebrew `libomp` 23.1.0. The notebook was executed from a fresh kernel with all 47 code cells completing in order. See the [reproducibility guide](docs/reproducibility.md) and [requirements](requirements.txt) for the exact procedure and dependency versions.

## Project History and Attribution

The archival notebook preserves a collaborative project developed during the 2025.2 ICMC Júnior Statistics Trainee program by Caio Augusto Marangoni de Mello, Gabriel Cury de Paula, Giovanna Maruyama, Henrique Nardi Farbelow, and Samuel Filipe de Almeida Silva. The current V2 is a later individual portfolio modernization by Gabriel Cury de Paula.

### My Contributions — 2026 Portfolio Modernization

- Redesigned the evaluation protocol to prevent train/test and preprocessing leakage, with learned preprocessing inside pipelines.
- Implemented stratified 5-fold cross-validation and a benchmark covering Dummy, Logistic Regression, Random Forest, Gradient Boosting, and XGBoost.
- Restricted tuning to training data, selected F1 for `Attrition=Yes` as the primary metric, and optimized thresholds with out-of-fold probabilities.
- Froze the model and decision threshold before the final holdout evaluation.
- Revised and simplified EDA; removed redundant or misleading analyses; and strengthened wording around causal and prospective-prediction limits.
- Validated clean-environment reproducibility, pinned the reference dependencies, and documented the reproduction procedure.
- Curated verified notebook outputs and prepared the repository for portfolio publication.

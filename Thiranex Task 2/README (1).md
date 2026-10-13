# Supervised Learning and Model Evaluation

Train Linear Regression, Decision Tree and Random Forest models, then judge them properly with **confusion matrices, ROC curves and AUC**. Built around the topic of the video "145 - Confusion matrix, ROC and AUC in machine learning".

> **Note on the video:** the YouTube page could not be read while building this project (the request was rate-limited and refused), so the project follows the video's title and your brief, not its exact content or dataset. If the video uses a different dataset or technique, swap it into the notebook; the structure stays the same.

## What is in the folder
| Path | What it is |
|---|---|
| `notebook/model_evaluation.ipynb` | The main deliverable: 9 parts, explanations, code and stored outputs |
| `figures/` | The 10 charts as PNG |
| `results/` | `metrics_classification.csv`, `metrics_regression.csv`, `auc_confidence_intervals.csv`, `summary.json` |
| `models/best_classifier.joblib` | The chosen model, its decision threshold and feature names |
| `data/breast_cancer.csv` | The dataset (ships with scikit-learn; 1 = malignant) |
| `scripts/build_notebook.py` | Rebuilds the notebook from source |
| `scripts/run_notebook.py` | Runs it headless and stores outputs (for machines without Jupyter) |

## What the notebook covers
1. **Data and class balance** (37% malignant, so accuracy alone misleads)
2. **Stratified train/test split**, and picking the tree depth with cross-validation on training data only
3. **Four models**: Linear Regression (used as a scorer), Logistic Regression, Decision Tree, Random Forest, compared with 5-fold cross-validation
4. **Confusion matrices**: TP, FP, TN, FN, precision, recall, F1
5. **ROC curves and AUC**, including where the default 0.5 threshold sits on each curve
6. **Threshold tuning** chosen on out-of-fold training predictions, then applied once to the test set
7. **Bootstrap confidence intervals** on AUC: are the gaps between models real?
8. **Checks by hand**: confusion matrix by counting, AUC as the chance a malignant case outscores a benign one (asserted equal to scikit-learn); decision-tree rules; feature importance
9. **Regression** (diabetes dataset): MAE, RMSE, R² against a "predict the mean" baseline

## Results (test set: 114 patients, 42 malignant)
- Logistic Regression had the best cross-validated AUC (0.996) and was the model carried forward; test AUC 0.996.
- Accuracy ranged 93.9% to 97.4%, and missed cancers (false negatives) ranged 3 to 5 across the four models at the default 0.5 threshold.
- Lowering the threshold from 0.50 to 0.21 (chosen on training data) cut missed cancers from 3 to 1 and raised false alarms from 1 to 3.
- The top three models' AUC intervals overlap, so with this little test data their ranking is not settled. The single Decision Tree is clearly behind.
- Regression: Linear Regression was best (R² 0.45), the mean-only baseline scored about 0.

These are results on a small public teaching dataset. This is a learning project, not a diagnostic tool.

## Run it
```bash
pip install -r requirements.txt
jupyter notebook notebook/model_evaluation.ipynb      # then Run All

# or without Jupyter
python scripts/build_notebook.py
python scripts/run_notebook.py
```
Tested with scikit-learn 1.9, pandas 3.0 and Python 3. Everything uses `random_state=42`, so reruns give the same numbers.

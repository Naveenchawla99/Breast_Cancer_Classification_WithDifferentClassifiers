# 

Compares logistic regression, random forest, KNN, Gaussian Naive Bayes and an SVM on
the Breast Cancer Wisconsin (Diagnostic) data, for a second-read triage aid where a
pathologist reviews every flagged case and needs a per-case explanation.
Primary metric: recall on malignant at precision >= 0.90 (decided before any fit).
Decision: logistic regression (see `decision.md`).

## Dataset

Breast Cancer Wisconsin (Diagnostic): 569 samples, 30 numeric features, 212 malignant
(37.3%). Creators: W. H. Wolberg, W. N. Street, O. L. Mangasarian (UCI Machine
Learning Repository). Licence: CC BY 4.0 as listed by UCI (please verify on the UCI
dataset page before submitting).
No download needed: `sklearn.datasets.load\_breast\_cancer(as\_frame=True)`.
Label coding: scikit-learn uses 0 = malignant; the notebooks flip it so that
1 = malignant (the positive class).

## Files

* `comparison.ipynb`: audit, deployment and metric, fairness setup, tuning,
repeated-fold comparison, test evaluation, cost and calibration, nested CV
(R1, R2, R4, R5, R6, R7, R9). Writes `best\_params.json`.
* `scratch.ipynb`: NumPy KNN, Gaussian Naive Bayes and SVM decision function, checked
against scikit-learn on the whole test set (R3). Reads `best\_params.json`.
* `decision.md`: the decision and the limitations of the comparison (R8).
* `best\_params.json`: tuned hyperparameters handed from the first notebook to the second.

## Seeds

`SEED = 42`, used for: the train/test split (`train\_test\_split`), `CV`
(StratifiedKFold, the tuning splitter shared by every search), `RCV`
(RepeatedStratifiedKFold, 5 x 5), `OUTER` (StratifiedKFold, nested CV), the five
`RandomizedSearchCV` objects, and the `random\_state` of LogisticRegression,
RandomForestClassifier and SVC. No other random\_state is used.

## How to run

1. Python 3 with the packages in `requirements.txt` (scikit-learn >= 1.4 is needed for
`make\_scorer(response\_method=...)`).
2. Run `comparison.ipynb` top to bottom (a few minutes; nested CV is the slowest part).
3. Run `scratch.ipynb` top to bottom (it needs `best\_params.json` from step 2).
Verified here with numpy 2.4.4, pandas 3.0.2, scipy 1.17.1, scikit-learn 1.8.0.


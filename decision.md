**Deployment (R1).** Pathologists use the predictions as a second-read triage aid on
fine-needle aspirate samples. Predictions run per sample, a few dozen per day, batch
latency of seconds is fine. A pathologist reviews every flagged case, so each
prediction needs a per-case explanation. A missed malignancy is far costlier than an
extra review. Primary metric (set before any fit): recall on malignant at
precision >= 0.90 (the 0.90 floor is an assumption to confirm with the reviewers).

```text
WE CHOOSE      Logistic regression (standardised features, C = 0.0746 as tuned).

BECAUSE        Evidence (5x5 repeated folds on the training data):
               logreg 0.978 ± 0.020 vs SVM 0.987 ± 0.017 on recall@P>=0.90.
               Difference -0.009 (std 0.016); the SVM was ahead on 9 folds,
               logreg on 1, with 15 ties. Not shown to be different, though the
               SVM leans ahead. Logreg vs KNN: +0.008 (std 0.023), 5 vs 1 folds,
               19 ties: not shown to be different. No pair of models met the rule
               fixed in advance (|diff| > std and >= 20 of 25 folds).
               Nested CV (R9, 5 outer folds, training data only): logreg and the
               SVM scored identically on all 5 outer folds (nested mean 0.982
               each, difference 0.000), so the SVM's earlier lead (0.994 in
               tuning CV) was selection optimism, not a measured advantage.
               Logreg's estimate did not move (0.982 -> 0.982); the SVM's fell
               by 0.012. Both were ahead of or tied with RF, KNN and GNB on
               every outer fold. With 5 folds of about 34 malignant cases this
               is weak evidence, but it points the same way as the test set.
               Test set (42 malignant cases, scored once): logreg tied-best on
               recall@P>=0.90 (0.976), best ROC-AUC (0.998), and missed 2 of 42
               at the default threshold with 0 false alarms (SVM 4, KNN 6, RF 3,
               GNB 6). One case is 0.024 of recall, so the top models cannot be
               ranked reliably on 42 cases.
               Properties (R7): logreg stores 31 numbers and needs 30
               multiply-adds per prediction, and gives a per-feature breakdown
               (coefficient x scaled value). The tuned SVM is rbf (C = 110), so
               it can only point to support vectors; KNN stores 14,105 numbers
               and needs 455 distances; GNB's breakdown double-counts the
               correlated size features and it is the most overconfident model.
               Calibration on the test set: logreg log loss 0.102 (43 of 114
               predictions beyond 0.99/0.01, 0 wrong), vs KNN 0.123 (69, 0),
               SVM with Platt scaling 0.095 (73, 0), GNB 0.500 (107, 5 wrong).

WE GIVE UP     The SVM's slightly better calibrated probabilities (log loss
               0.095 vs 0.102, and the SVM was Platt-scaled on training data).
               KNN's explanation by actual similar cases, which a pathologist
               may find more convincing than coefficients. The SVM's apparent
               CV lead did not survive nested CV, so nothing measured is given
               up there. RF and GNB were not better on any comparison.

WE WOULD
RECONSIDER IF  1. A larger evaluation set (at least 150 malignant cases) shows the SVM
               ahead of logreg on recall@P>=0.90 by 0.03 or more over repeated
               folds at the same precision floor.
               2. Logreg's log loss on a new evaluation set exceeds the SVM's by
               more than 0.05, or logreg makes any confident error (>0.99 or
               <0.01). On the current test set the gap is 0.007 and there are
               no confident errors.
               3. Reviewers say they prefer case-based explanations (KNN) to
               per-feature ones.
               4. The precision floor changes from 0.90 (rerun the metric; the
               ranking could change).
```

## Limitations of the comparison (R5)

The repeated-fold scores in R5 reuse the tuning data. Each model's hyperparameters
were chosen on the training set with the shared cross-validation splitter, and the
same training rows were then used again to score those tuned models on repeated
folds. Every score is therefore optimistic, most of all for the models whose search
was most selective (the SVM had only 5 of 20 candidates within one standard
deviation of its best). The nested CV in R9 removes this reuse and shows the effect:
the SVM's estimate fell by 0.012 while logreg's did not move.

The repeated folds are also not independent. The 25 folds (5 x 5) are built from the
same 455 training rows, so their training sets overlap heavily and the fold-to-fold
differences are correlated. The standard deviation of a difference therefore
understates the true uncertainty, and "clearly different" is a rule of thumb that was
fixed before looking, not a significance test. The metric also moves in coarse steps
(about 34 malignant cases per validation fold, so one case is about 0.03), which
produces many ties, and GNB's large spread comes from folds where no threshold
reaches precision 0.90 and the score is 0.

## Other caveats

* **The headline metric is an upper bound.** recall@P>=0.90 picks its threshold using
the labels it scores. Before deployment, set the threshold on out-of-fold training
predictions. The confusion matrices in R6 use the default threshold.
* **Correlated features make coefficients unstable.** Radius, perimeter and area
correlate at 0.99+, so the breakdown can split one size signal across them
arbitrarily. Present it as "size-related features drove this", not three
independent facts.


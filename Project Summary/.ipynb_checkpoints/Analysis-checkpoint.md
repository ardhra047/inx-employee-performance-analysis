# Analysis: Choices and Reasoning

This document explains why each analytical and modelling decision was made. The
findings themselves are in `Summary.md`.

## 1. Problem Framing

`PerformanceRating` takes three observed values (2 Good, 3 Excellent, 4 Outstanding),
so the task was framed as **multi-class classification** rather than regression.
Classification metrics (per-class recall, confusion matrix) are easier for managers
to interpret than a continuous score, and the number of classes is small.

## 2. Data Processing

The raw dataset (1,200 employees, 28 columns) was unusually clean: **no missing
values, no duplicate rows, no duplicate employee IDs, and no inconsistent category
labels**. This meant imputation and deduplication code was written as a safety net
rather than an active fix — kept in the notebook in case a future data refresh
introduces issues, but not exercised on this dataset.

`Attrition` was excluded from all predictors. It is a *consequence* of an employee's
satisfaction and performance, not a cause of it — including it would blur, rather
than clarify, the true drivers.

## 3. Feature Engineering

The four raw tenure columns (`ExperienceYearsAtThisCompany`, `ExperienceYearsInCurrentRole`,
`YearsSinceLastPromotion`, `YearsWithCurrManager`) turned out to be strongly correlated
with each other (Spearman rho up to 0.93 between some pairs) — they mostly repeat the
same "length of service" signal. Five ratio features were engineered to capture the
*shape* of a career rather than its raw length:

| Feature | Formula | Captures |
|---|---|---|
| `PromotionGapRatio` | years since promotion / (years at company + 1) | Share of tenure spent without promotion |
| `RoleTenureRatio` | years in current role / (years at company + 1) | Stagnation in one role |
| `ManagerTenureRatio` | years with manager / (years at company + 1) | Management stability |
| `PriorExperienceYears` | total experience − years at company | Experience brought in from outside |
| `AverageTenurePerEmployer` | total experience / (companies worked + 1) | Job-hopping tendency |

## 4. Exploratory Analysis

- **Spearman correlation and Kruskal-Wallis** were used for numeric factors, since the
  target is ordinal, not continuous.
- **Chi-square test and Cramér's V** were used for categorical factors.
- With ~30 factors tested simultaneously, p-values were adjusted using the
  **Benjamini-Hochberg false discovery rate correction** to control for false positives
  from multiple testing. 16 of 30 factors remained significant after adjustment.
- **Effect size, not just p-value, ranked the factors** — with 1,200 employees, even
  trivial effects can appear statistically significant, so effect size was used to
  judge practical importance.
- A **Spearman correlation heatmap and pairwise correlation check** confirmed the
  tenure-related columns overlap heavily (several pairs above 0.6, one pair — promotion
  gap ratio and years since promotion — at 0.93), which shaped how the "top 3 factors"
  question was ultimately answered (see Section 6).

## 5. Modelling

### 5.1 Algorithms Compared

Eight approaches spanning the major algorithm families: a majority-class baseline,
logistic regression (linear), decision tree and random forest and gradient boosting
(tree-based), a support vector machine (kernel-based), k-nearest neighbours
(distance-based), and a neural network (multilayer perceptron). This breadth made the
final choice evidence-based rather than assumed.

### 5.2 Protocol

1. **80/20 stratified split**, test set untouched until final evaluation.
2. **5-fold stratified cross-validation** on the training portion only, for both model
   comparison and hyperparameter tuning.
3. **Preprocessing inside a pipeline** (standard scaling for numeric columns, one-hot
   encoding for categorical columns) so each cross-validation fold learns scaling
   parameters only from its own training data — preventing leakage.
4. **Macro F1 as the primary metric**, not accuracy, because the target is imbalanced
   (72.8% of employees are rated "Excellent"). A model that always predicts "Excellent"
   would already score 72.8% accuracy but a macro F1 of just 0.28 — accuracy alone
   would have been misleading.
5. **Class weighting** (`class_weight="balanced"`) was used wherever the algorithm
   supported it, to counter the imbalance directly.
6. A **95% Wilson confidence interval** was reported for test accuracy, since 240 test
   employees leaves some genuine sampling uncertainty.

### 5.3 Algorithm Selection

Gradient Boosting was the clear winner in cross-validation (0.888 macro F1, later
0.900 after tuning), well ahead of every other model, including the other two
tree-based methods. Its consistently strong margin, combined with a tight standard
deviation across folds (0.018), indicated the relationship between the input factors
and performance rating is **non-linear with threshold effects** — something tree-based
models capture naturally and linear or distance-based models do not.

### 5.4 Principal Component Analysis — Tested and Rejected

PCA was tested as a required check, compressing the 64 encoded columns down to 27
components (95% variance retained). **Result: cross-validated macro F1 collapsed from
0.900 to 0.540** — a large drop, not a minor trade-off. This happens because PCA
maximises variance explained, not predictive power for the target: the two strongest
predictors here are individual columns, not directions of broad variance, so
compression diluted their sharp signal by mixing them with many weaker, irrelevant
columns. **PCA was excluded from the final pipeline.**

### 5.5 Two Separate Models

- **Full model** (30 features, all employee attributes except identifiers and
  attrition): diagnoses why *current* employees perform as they do.
- **Hiring model** (10 pre-hire features only, explicitly excluding age, gender, and
  marital status): intended for screening candidates.

Age, gender, and marital status were excluded from the hiring model on fairness
grounds — even where a historical correlation might exist, using protected attributes
in a hiring decision is inappropriate and potentially unlawful.

The hiring model's test performance (31.3% accuracy, 0.281 macro F1 — statistically
no better than the majority-class baseline) is itself an important finding, not a
failure of process: it demonstrates that **pre-hire information does not meaningfully
predict performance at INX**. See `Summary.md` for the business implication.

## 6. Identifying the Top 3 Factors

Two independent methods were used and cross-checked:

1. **Statistical association** (Section 4 above): ranked factors by effect size.
2. **Permutation importance**: measured the drop in the trained model's macro F1 when
   each column was randomly shuffled on the *test set* — a model-based, unseen-data
   measure, immune to the assumption of linearity that correlation relies on.

Both methods agreed unambiguously on the top 2: `EmpEnvironmentSatisfaction` and
`EmpLastSalaryHikePercent`. The two methods initially appeared to disagree on the 3rd
position (`YearsSinceLastPromotion` by association vs. `EmpDepartment` by permutation
importance) — but this was resolved by recognising the promotion/tenure signal is
**split across seven correlated columns** rather than concentrated in one. Summed
together, this cluster's combined permutation importance (0.232) exceeds
`EmpDepartment`'s (0.080), confirming the statistical ranking was correct once the
redundancy is accounted for. See `Summary.md` for the final, consolidated top 3.

## 7. Identifying Employees Who May Need Support

- **Out-of-fold prediction** (`cross_val_predict`) was used so every employee is scored
  by a version of the model that never saw them during training — even though all
  1,200 employees were part of the original dataset.
- A **precision/recall trade-off table across thresholds** let the flagging cut-off be
  chosen deliberately rather than defaulted to 0.5 arbitrarily. A threshold of 0.5 was
  selected, balancing 87.2% precision and 88.1% recall — flagging 196 employees (16.3%
  of the workforce), concentrated in Sales and Finance.
- Flagged employees were **profiled on the top 3 factors** rather than left as an
  opaque score, giving explainable, actionable indicators (see `Summary.md`).

## 8. Limitations

1. **Association is not causation.** These factors show where to look, not proof that
   changing one will directly move performance.
2. **Snapshot data.** A single cross-section cannot establish *why* performance
   declined over time — only what differs between higher- and lower-rated employees
   today.
3. **Rating subjectivity.** `PerformanceRating` reflects a manager's judgement and may
   embed inconsistency or bias between departments and managers.
4. **Small sub-groups.** Data Science (20 employees) and some job roles have small
   headcounts; their averages carry more uncertainty than larger groups.
5. **Hiring model.** Performs at baseline level and should not be used as a stand-alone
   hiring filter — structured interviews and work samples remain the primary basis
   for hiring decisions.
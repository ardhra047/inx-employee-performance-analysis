# Project Summary: Employee Performance at INX Future Inc

## 1. Project Summary

**Study performed.** Employee records for **1,200 employees** across 28 original
attributes were cleaned, explored statistically, and used to train classification
models predicting `PerformanceRating`.

**Algorithm and training method.** Eight algorithms were compared using stratified
5-fold cross-validation: a majority-class baseline, logistic regression, decision
tree, random forest, gradient boosting, support vector machine, k-nearest neighbours,
and a neural network. **Gradient Boosting** was the clear winner and was tuned via
randomised search, then evaluated once on an untouched 20% test set.

**Headline model performance (test set, 240 employees):**
- Accuracy: **94.6%** (95% confidence interval: 91.0%–96.8%)
- Macro F1: **0.92**
- Majority-class baseline: 72.9% accuracy, 0.28 macro F1 — the model clearly beats
  naive guessing, not just on accuracy but on the harder, imbalance-robust metric.

**Features and dimensionality.** 30 predictors were used in the full model.
**Principal Component Analysis was tested and rejected** — it collapsed cross-validated
macro F1 from 0.900 to 0.540, because the signal is concentrated in a small number of
strong individual factors rather than spread across broad directions of variance that
PCA optimises for.

**Other techniques and tools.** Python (pandas, scikit-learn, scipy, statsmodels,
seaborn); Spearman correlation, Kruskal-Wallis and chi-square tests with
Benjamini-Hochberg multiple-testing correction; permutation importance; out-of-fold
probability scoring for support-priority flagging.

## 2. Feature Selection and Engineering

**2.1 Most important features and why.** Two independent methods — statistical
association (on the raw data) and permutation importance (measured on the trained
model's unseen test predictions) — were used and cross-checked. The consolidated
**top 3 factors** are:

| Rank | Factor | Statistical effect size | Model importance | Direction |
|---|---|---|---|---|
| 1 | **Employee Environment Satisfaction** | 0.397 | 0.308 | Higher → better rating |
| 2 | **Last Salary Hike Percentage** | 0.274 | 0.281 | Higher → better rating |
| 3 | **Career progression stagnation** (time since last promotion, tenure-in-role) | 0.270 (leading single column) | 0.232 (combined across correlated columns) | Longer without progression → worse rating |

Both methods agreed exactly on factors 1 and 2. Factor 3 required reconciling: the
"career progression" signal is spread across seven correlated tenure columns
(`YearsSinceLastPromotion`, `PromotionGapRatio`, `ExperienceYearsInCurrentRole`, and
others) rather than concentrated in one, which initially made department appear more
important in the model-only ranking. Summing the cluster's importance confirmed it
belongs in 3rd place — see `Analysis.md`, Section 6, for the full reasoning.

**2.2 Feature transformations.** Numeric predictors were standardised and text
predictors one-hot encoded inside the modelling pipeline (preventing data leakage).
Five ratio features were engineered from the raw tenure columns to separate career
*shape* from raw length of service. `Attrition` was excluded as a predictor (it is an
outcome of performance, not a cause); identifiers were excluded. The hiring model was
further restricted to pre-hire information only, with age, gender, and marital status
removed on fairness grounds.

**2.3 Correlations and interactions.** The four raw tenure columns are strongly
correlated with each other (Spearman rho up to 0.93 between `YearsSinceLastPromotion`
and `PromotionGapRatio`; 0.87 between `ExperienceYearsAtThisCompany` and
`ExperienceYearsInCurrentRole`). This was handled by (a) using a tree-based model that
tolerates correlated inputs natively, (b) engineering ratio features to add distinct
information, and (c) measuring importance by permutation on original columns rather
than relying on a single correlation figure. Environment satisfaction and salary hike
— the two leading factors — vary largely independently of this tenure cluster and of
each other.

## 3. Results, Analysis and Insights

**3.1 An interesting relationship that stands apart.** Department-wise performance
does **not** simply track the top 3 factors. Finance has the lowest average rating
(2.78) despite scoring *favourably* on all three leading factors (2nd-highest
satisfaction, highest salary hikes, shortest promotion gap of any department). This
points to a department-specific cause — likely workload, client-facing pressure, or
management style — not captured by the factors identified here, and worth investigating
directly with Finance leadership.

**3.2 Most important technique.** Permutation importance, cross-checked against
statistical association with multiple-testing correction. Using two independent
methods — one purely statistical, one model-based on unseen data — gave confidence
that the top factors are real signal, not an artifact of either method alone.

**3.3 Answers to the business questions**

| Question | Answer |
|---|---|
| **1. Department-wise performance** | Significant differences exist (χ² = 70.33, p < 0.0001). Ranking by average rating: **Development (3.09) > Data Science (3.05, n=20) > Human Resources (2.93) > Research & Development (2.92) > Sales (2.86) > Finance (2.78)**. The share of employees in the lowest rating band ranges from 3.6% (Development) to 30.6% (Finance) — nearly a 9x difference. |
| **2. Top 3 factors** | Employee environment satisfaction, salary hike percentage, and career-progression stagnation (see 2.1 table above). |
| **3. Trained model for hiring** | **Full model** (diagnosing current employees): 94.6% test accuracy, 0.92 macro F1 — strong and reliable. **Hiring model** (pre-hire information only): 31.3% test accuracy, 0.28 macro F1 — statistically no better than guessing. **Verdict: the hiring model should NOT be used as a stand-alone or primary hiring filter.** This is itself a key finding: performance at INX is driven by post-hire experience, not by anything visible on a resume. |
| **4. Recommendations** | See Section 5 below. |
| **Clear indicators of non-performing employees** | At a flagging threshold of 0.5, the model identifies **196 employees (16.3% of the workforce)** with 87.2% precision and 88.1% recall. Flagged employees are concentrated in **Sales (94) and Finance (15, 30.6% of that department)**, and are characterised by markedly **lower environment satisfaction (1.53 vs 2.95 average)** and **longer time since last promotion (3.77 vs 1.89 years average)** — notably, **not** by lower salary hikes, which were nearly identical between flagged and non-flagged groups. |

**3.4 Further business insights.**
- The gap between the full model (94.6% accuracy) and the hiring model (31.3%) is
  itself a finding: it shows performance at INX is determined much more by what
  happens **after** someone joins than by anything about their background.
- Age and gender showed **no statistically significant relationship** with performance
  rating — reassuring from a fairness perspective, and consistent with the decision to
  exclude them from the hiring model regardless.
- Overtime and attrition showed only weak, marginal relationships with rating and were
  correctly excluded from (or found unhelpful for) the top-factor analysis.

## 4. Protecting Morale While Acting on the Findings

1. Use the support-priority scores only to **prioritise supportive conversations** —
   every flagged employee should be reviewed by their manager before any action is
   taken.
2. The chosen threshold (0.5) deliberately keeps the flagged group to **16% of the
   workforce**, not a blanket sweep, directly addressing the CEO's morale concern.
3. Frame any resulting programme internally as **development and workload review**,
   not a search for underperformers.
4. Prioritise the **systemic** drivers (environment satisfaction, career progression)
   first, since they affect the wider workforce and reduce the number of individual
   cases that later need attention.
5. Re-run this analysis on a **regular cadence** (e.g. every 6 months) to track whether
   interventions are working — a single snapshot cannot show trends.

## 5. Recommendations

1. **Investigate and improve workplace environment satisfaction, especially in Sales
   and Finance.** This is the single strongest driver of performance found in the
   analysis (effect size 0.40, model importance 0.31). Recommended action: team-level
   pulse surveys and workload/management reviews in the two lowest-rated departments.
   *(Evidence: Figure 6, Figure 7, Section 3.1)*

2. **Review the salary hike / merit-increase process for consistency.** The second
   strongest driver (effect size 0.27, model importance 0.28) — but notably did *not*
   differ between flagged and non-flagged employees, suggesting pay alone won't fix
   the flagged group's issues; it matters more as a broader, company-wide lever.
   *(Evidence: Figure 7, Section 3, predict_model.ipynb Section 7)*

3. **Introduce structured career-path and promotion-readiness reviews at fixed
   intervals**, particularly for employees who have gone 3+ years without a promotion
   — this population is heavily overrepresented among flagged employees (average 3.77
   years vs 1.89 for others). *(Evidence: Figure 7, predict_model.ipynb Section 7)*

4. **Conduct a targeted, department-specific review of Finance.** Finance has the
   lowest average rating in the company despite performing well on all three top
   factors — the cause is not captured by this dataset and needs direct investigation
   (workload, client pressure, management practices). *(Evidence: Section 3.1)*

5. **Do not build or deploy a resume-based hiring predictor from this data.** The
   hiring model performed at baseline level; pre-hire attributes do not meaningfully
   predict performance at INX. Continue relying on structured interviews and work
   samples for hiring decisions. *(Evidence: Section 3.3, `Analysis.md` Section 5.5)*

6. **Use the support-priority scores as a coaching tool, not a penalty list.** At the
   chosen threshold, 196 employees (16.3% of the workforce) are flagged with 87%
   precision — small and targeted enough to avoid a company-wide morale impact, while
   still catching the large majority of genuinely low-rated employees.
   *(Evidence: Figure 9, Section 3.3)*

## 6. Compliance with Project Guidelines

| Guideline | Status |
|---|---|
| Scripts as Jupyter notebooks with markdown | 5 notebooks, each with markdown observations throughout |
| Directory structure as specified | Matches the required structure (see `README.md` if created, or the folder tree itself) |
| No acronyms/abbreviations in titles | Titles written in full |
| Summary describes study, data, and insights | This document |
| Methods describe preprocessing in detail | `Analysis.md`, Sections 2–5, and `data_processing.ipynb` |
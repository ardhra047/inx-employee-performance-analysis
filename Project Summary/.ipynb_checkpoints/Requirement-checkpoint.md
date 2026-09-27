# Requirements

## 1. Business Context

INX Future Inc is a data analytics and automation solutions provider with over 15
years of global presence, rated among the top 20 best employers for five consecutive
years. Recently, employee performance indicators have declined: service-delivery
escalations have increased, and client satisfaction has fallen by 8 percentage points.

The CEO, Mr. Brain, does not want to penalise low performers indiscriminately, as this
risks damaging morale across the company and its reputation as a top employer — both
central to attracting talent. This project uses data science to find the underlying,
evidence-based causes of the performance decline before any action is taken.

## 2. Required Deliverables

| # | Deliverable | Delivered in |
|---|---|---|
| 1 | Department-wise performance | `data_exploratory_analysis.ipynb`, Figure 2, `Summary.md` |
| 2 | Top 3 factors affecting performance | `train_model.ipynb` (permutation importance), cross-checked against `data_exploratory_analysis.ipynb` (statistical association), Figures 3, 6, 7 |
| 3 | Trained model predicting performance from input factors, for hiring | `trained_model_full.joblib`, `trained_model_hiring.joblib`, `predict_model.ipynb` |
| 4 | Recommendations to improve performance | `Summary.md` |
| 5 | Clear indicators of non-performing employees, limiting impact on morale | `predict_model.ipynb` — support-priority scores and factor profile of flagged employees |

## 3. Data Requirements

- **Source:** INX Future Inc employee performance dataset —
  `INX_Future_Inc_Employee_Performance_CDS_Project2_Data_V1.8.xls`
- **Size:** 1,200 employees, 28 original columns, target column `PerformanceRating`
- **Location:** original file stored unchanged in `data/raw/`
- **External data:** none used
- **Definitions:** see `references/data_dictionary.md`

## 4. Software Requirements

Python 3.10+ with: pandas, numpy, scikit-learn, scipy, statsmodels, matplotlib,
seaborn, joblib, jupyter (see `requirements.txt`).

## 5. Constraints and Guiding Principles

1. **No unfair or unlawful discrimination** — age, gender, and marital status are
   excluded from the hiring model.
2. **Support before sanction** — model outputs prioritise support and manager review;
   they are not automatic grounds for penalties.
3. **Reproducibility** — fixed random seed (42), preprocessing inside pipelines, raw
   data untouched, notebooks run in a fixed order.
4. **Honest evaluation** — models compared against a majority-class baseline; final
   numbers reported from a test set used exactly once.

## 6. Success Criteria

- Model macro F1 and per-class recall clearly exceed the majority-class baseline on the
  held-out test set.
- Top factors supported by two independent methods (statistical association and
  model-based permutation importance).
- Every recommendation traces back to a specific, named finding.
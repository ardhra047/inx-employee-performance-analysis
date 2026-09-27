# Data Dictionary

Reference for every column in the INX Future Inc employee performance dataset
(1,200 employees, 28 original columns + 6 engineered columns).

## Identifier and Target

| Column | Type | Meaning |
|---|---|---|
| `EmpNumber` | text | Unique employee identifier. Not used as a predictor. |
| `PerformanceRating` | integer | **Target.** 2 = Good, 3 = Excellent, 4 = Outstanding. No employees rated 1 (Low) are present in this dataset. |

## Personal and Background

| Column | Type | Meaning |
|---|---|---|
| `Age` | integer | Employee's age in years. Excluded from the hiring model (protected attribute). |
| `Gender` | text | Male / Female. Excluded from the hiring model (protected attribute). |
| `MaritalStatus` | text | Single / Married / Divorced. Excluded from the hiring model (protected attribute). |
| `EducationBackground` | text | Field of study (e.g. Life Sciences, Marketing, Technical Degree). |
| `EmpEducationLevel` | ordinal (1–5) | 1 = Below College, 2 = College, 3 = Bachelor, 4 = Master, 5 = Doctor. |
| `DistanceFromHome` | integer | Distance from home to workplace. |

## Job and Organisation

| Column | Type | Meaning |
|---|---|---|
| `EmpDepartment` | text | Sales, Human Resources, Development, Data Science, Research & Development, Finance. |
| `EmpJobRole` | text | Specific job title (19 distinct roles across departments). |
| `EmpJobLevel` | ordinal (1–5) | Seniority level within the organisation. |
| `BusinessTravelFrequency` | text | Non-Travel / Travel_Rarely / Travel_Frequently. |
| `OverTime` | text | Yes / No — whether the employee regularly works overtime. |
| `EmpHourlyRate` | integer | Hourly pay rate. |
| `EmpLastSalaryHikePercent` | integer | Percentage increase at the employee's most recent salary review. |
| `TrainingTimesLastYear` | integer | Number of training sessions attended in the last year. |

## Experience and Career Progression

| Column | Type | Meaning |
|---|---|---|
| `NumCompaniesWorked` | integer | Number of employers prior to INX. |
| `TotalWorkExperienceInYears` | integer | Total career length across all employers. |
| `ExperienceYearsAtThisCompany` | integer | Tenure at INX specifically. |
| `ExperienceYearsInCurrentRole` | integer | Years spent in the current job role. |
| `YearsSinceLastPromotion` | integer | Years elapsed since the employee's last promotion. |
| `YearsWithCurrManager` | integer | Years reporting to the current manager. |

## Survey Scores (ordinal 1–4; 1 = lowest, 4 = highest)

| Column | Meaning |
|---|---|
| `EmpEnvironmentSatisfaction` | Satisfaction with the physical/organisational working environment. |
| `EmpJobSatisfaction` | Satisfaction with the job itself. |
| `EmpRelationshipSatisfaction` | Satisfaction with workplace relationships. |
| `EmpJobInvolvement` | Degree of engagement/involvement in the job. |
| `EmpWorkLifeBalance` | 1 = Bad, 2 = Good, 3 = Better, 4 = Best. |

## Outcome (Not Used as a Predictor)

| Column | Meaning |
|---|---|
| `Attrition` | Yes / No — whether the employee has left the company. Treated as a *consequence* of performance and satisfaction, not a cause, so excluded from all model features. |

## Engineered Columns

Created in `data_processing.ipynb` to separate career *shape* from raw tenure length.

| Column | Formula | Meaning |
|---|---|---|
| `PromotionGapRatio` | YearsSinceLastPromotion / (ExperienceYearsAtThisCompany + 1) | Share of tenure spent without a promotion. |
| `RoleTenureRatio` | ExperienceYearsInCurrentRole / (ExperienceYearsAtThisCompany + 1) | Degree of stagnation in the current role. |
| `ManagerTenureRatio` | YearsWithCurrManager / (ExperienceYearsAtThisCompany + 1) | Management relationship stability. |
| `PriorExperienceYears` | TotalWorkExperienceInYears − ExperienceYearsAtThisCompany | Experience brought in from outside INX. |
| `AverageTenurePerEmployer` | TotalWorkExperienceInYears / (NumCompaniesWorked + 1) | Job-hopping tendency across a career. |
| `PerformanceLabel` | Maps `PerformanceRating` to text | 2 → "Good", 3 → "Excellent", 4 → "Outstanding". Used for readability in charts and tables. |
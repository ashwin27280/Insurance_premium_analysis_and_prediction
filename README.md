# Insurance Premium Prediction & Risk Analytics

Predicting annual health insurance premiums from demographic, financial, and health data, with a companion Power BI dashboard for underwriting/pricing stakeholders.

## Overview

This project takes a raw dataset of 50,000 policyholders and:
1. Cleans and engineers features from demographic, financial, and medical history data
2. Trains and compares three regression models to predict `annual_premium_amount`
3. Surfaces pricing/business insights (which risk factors drive premiums, and by how much)
4. Presents those insights in an interactive Power BI dashboard for non-technical stakeholders

## Dataset

`premiums.xlsx` — 50,000 rows × 13 columns:

| Column | Description |
|---|---|
| Age | Policyholder age |
| Gender | Male / Female |
| Region | Northeast / Northwest / Southeast / Southwest |
| Marital Status | Married / Unmarried |
| Number Of Dependants | Count of dependants |
| BMI Category | Underweight / Normal / Overweight / Obesity |
| Smoking Status | Non-smoker / Occasional / Regular |
| Employment Status | Salaried / Self-employed / Freelancer |
| Income Level | Bucketed income band |
| Income (Lakhs) | Numeric annual income |
| Medical History | Disease/condition combination, or "No Disease" |
| Insurance Plan | Bronze / Silver / Gold |
| **Annual Premium Amount** | **Target variable** |

## Pipeline

**1. Data cleaning**
- Dropped nulls and duplicate rows
- Corrected negative `Number Of Dependants` values (absolute value)
- Removed unrealistic ages (raw max was 356; capped at 100)
- Capped `Income (Lakhs)` at the 99.9th percentile (raw max was 930 lakhs)

**2. Exploratory data analysis**
- Univariate and bivariate distributions
- Correlation heatmaps
- Crosstabs (e.g. income level vs. plan tier)

**3. Feature engineering**
- `Medical History` mapped to a custom health risk score (0–14 scale), then min-max normalized
- `Insurance Plan` ordinally encoded (Bronze=1, Silver=2, Gold=3)
- Remaining nominal columns one-hot encoded

**4. Modeling** — 80/20 train/test split, `random_state=42`

| Model | R² | MAE | RMSE | MAPE |
|---|---|---|---|---|
| Linear Regression | 0.927 | ₹1,763.79 | ₹2,265.03 | 15.84% |
| Random Forest | 0.978 | ₹851.07 | ₹1,247.17 | 10.12% |
| **XGBoost (selected)** | **0.980** | **₹818.54** | **₹1,184.43** | **9.71%** |

**5. Feature importance (Random Forest)**

| Feature | Importance |
|---|---|
| Age | 59.0% |
| Insurance Plan | 33.8% |
| Health Risk Score | 2.3% |
| Smoking Status (Regular) | 1.5% |
| BMI Category (Obesity) | 1.3% |

## Key business insights

- **Age and plan tier dominate pricing** — together they explain ~93% of feature importance; most other factors (region, gender, employment) barely move the needle
- Heart disease, and heart disease combined with diabetes or high blood pressure, are the costliest medical histories (avg premium ₹20.7K–₹23.3K vs. ₹9.9K baseline for no disease)
- ~14.2% of policyholders are high-risk (2+ conditions or heart disease)
- Region has almost no effect on premium (₹15.68K–₹15.86K range across all four regions) — not a useful pricing lever
- Gold-plan holders pay roughly 3× Bronze-plan holders on average

## Power BI Dashboard

An interactive companion dashboard translates the model's findings for underwriting/pricing stakeholders. Planned pages:

1. **Overview** — policyholder count, avg/total premium, plan mix, age-vs-premium trend, top health conditions by cost — *built*
2. **Risk Drivers** — feature importance, high-risk segment size, regional comparison — *built*

Filters: Gender, Insurance Plan, Region, BMI Category.

### Known issues (tracked, not yet fixed)
- High-risk donut on the Risk Drivers page: the "standard" segment count is pulling an unfiltered total instead of `Total − High Risk`, causing the two slices to overstate the whole
- Gold-Plan Share KPI doesn't update correctly when the Insurance Plan slicer is applied (likely an `ALL`/`ALLEXCEPT` context issue in the measure)
- Age-band trend line reverses direction under some slicer combinations — needs axis sort verified against a numeric (not text) order
- Two KPI cards are missing unit formatting (`%`, `Cr`)

## Repo structure

```
├── Insurance_Premium_Model.ipynb   # cleaning, EDA, feature engineering, modeling
├── premiums.xlsx                   # raw dataset
├── dashboard/                      # Power BI file + theme + DAX reference
└── README.md
```

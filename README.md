# 🍽️ ChefLink Campaign Success Prediction

A Machine Learning project that predicts whether a promotional campaign for **ChefLink** (a food delivery service) will be **Successful** or **Not Successful**, based on operational data such as orders, revenue, new customers, city, and promo type.

---

## 📌 Project Overview

ChefLink runs marketing campaigns across multiple cities and promo types. The dataset contains intentional data quality issues that required thorough cleaning and standardization before modeling.

The objectives of this project are:

1. **Clean and standardize** a real-world messy dataset of campaign records
2. **Define what "Successful" means** using a transparent business rule
3. **Train a classification model** that predicts campaign success
4. **Identify the key drivers** of successful campaigns to inform future marketing strategy

---

## 📊 Dataset Description

| Property | Value |
|----------|-------|
| Source | ChefLink campaign records (CSV) |
| Original rows | 853 |
| Rows after cleaning | 833 |
| Original columns | 20 |
| Final features for modeling | 27 |
| Time period | January – June 2025 |
| Cities covered | Cairo, Alexandria, Giza, Mansoura |

### Key Columns

| Column | Description |
|--------|-------------|
| `campaign_id` | Unique campaign code |
| `city` | City where the campaign ran |
| `promo_date` | Campaign launch date |
| `promo_type` | BOGO, Discount, Cashback, Free Delivery |
| `promo_status` | Active, Paused, Completed, Expired |
| `promo_cost` | Marketing budget spent |
| `revenue` | Total revenue generated |
| `orders_count` | Number of orders from the campaign |
| `redemptions` | Number of times the promo code was used |
| `new_customers` | Number of new customers acquired |
| `discount_pct` | Discount percentage offered |
| `payment_method` | Card, Cash, Wallet |
| `issue_reason` | Any reported campaign issue |

---

## 🛠️ Steps Performed

### 1. Data Understanding & EDA
- Loaded the dataset and inspected shape, data types, and structure
- Generated statistical summaries for numerical and categorical columns
- Visualized distributions to detect anomalies
- Cataloged all data quality issues before fixing them

### 2. Data Cleaning & Standardization

The raw dataset contained intentional data quality issues. The following were systematically addressed:

| # | Issue Found | Fix Applied |
|---|-------------|-------------|
| 1 | 20 duplicate rows | Removed via `drop_duplicates()` |
| 2 | `business_name` had only 1 unique value | Dropped (zero information) |
| 3 | Negative values in `promo_cost` and `revenue` (63 rows) | Applied `abs()` to fix data-entry errors |
| 4 | 8 inconsistent city spellings (CAIRO / cairo / Cairo / ' cairo ' / Alex / Alexandria) | Standardized to 4 canonical names |
| 5 | `'Admin'` vs `'ADMIN '` (whitespace duplicate) | Stripped whitespace and unified |
| 6 | 5 different date formats including invalid `32/13/2025` (166 rows) | Parsed with multi-format function; invalid dates flagged |
| 7 | 38.92% missing `issue_reason` | Filled with `"No Issue"` (interpreted as no problem reported) |
| 8 | 4.45% missing `city` | Filled with the mode (most common city) |
| 9 | Extreme outliers in `revenue` (up to 65,000) and `promo_cost` | Capped at IQR bounds (Winsorization) |

### 3. Target Definition

The dataset does not contain a direct "success" label, so we defined it via a **transparent business rule**:

> A campaign is **Successful** if **all three** conditions hold:
>
> 1. `revenue > promo_cost × 5` (strong ROI)
> 2. `orders_count > median (266)` (popular)
> 3. `new_customers > median (60)` (acquires new customers)

This reflects what a real marketing director would consider a "win": **profitable + popular + grows the customer base**.

**Class distribution:**
- Not Successful (0): 695 campaigns (83.4%)
- Successful (1): 138 campaigns (16.6%)

### 4. Feature Engineering

| Step | Description |
|------|-------------|
| **Date features** | Extracted `promo_month`, `promo_quarter`, `promo_dayofweek`, `promo_is_weekend` |
| **Missing-date flag** | Created `promo_date_missing` binary indicator for the 166 invalid dates |
| **ID columns dropped** | Removed `campaign_record_id`, `campaign_id`, `campaign_name`, `restaurant_id`, `rider_id` |
| **One-Hot Encoding** | Applied to all categorical columns with `drop_first=True` |
| **Final feature count** | 27 numeric features ready for modeling |

### 5. Model Training & Evaluation
- Stratified 75/25 train/test split (preserves class balance)
- Standard scaling for Logistic Regression (tree models are scale-invariant)
- All models trained with `class_weight="balanced"` to handle 83/17 imbalance
- 5-fold cross-validation for reliable performance estimates

---

## 🤖 Models Used

Three classification models were trained and compared:

| Model | Train Acc | Test Acc | Test ROC AUC | CV Accuracy (5-fold) |
|-------|:---------:|:--------:|:------------:|:--------------------:|
| Logistic Regression | 88.78% | 86.12% | 0.9456 | 86.38% ± 2.44% |
| Random Forest | 100% | 97.13% | 0.9947 | 95.67% ± 1.39% |
| **XGBoost (Best)** | **100%** | **97.13%** | **0.9979** | **98.08% ± 1.48%** |
| Dummy Baseline | — | 83.25% | — | — |

**XGBoost** was selected as the final model based on its highest cross-validation accuracy and ROC AUC.

---

## 📈 Evaluation Results

### Classification Report (XGBoost on Test Set)

```
                    precision    recall  f1-score   support

Not Successful (0)     1.0000    0.9655    0.9825       174
    Successful (1)     0.8537    1.0000    0.9211        35

          accuracy                         0.9713       209
         macro avg     0.9268    0.9828    0.9518       209
       weighted avg    0.9755    0.9713    0.9722       209
```

### Confusion Matrix

|                       | Predicted Not Successful | Predicted Successful |
|-----------------------|:------------------------:|:--------------------:|
| **Actual Not Successful** | 168 (TN) | 6 (FP) |
| **Actual Successful** | 0 (FN) | 35 (TP) |

**Key takeaway:** The model captures **100% of successful campaigns** (zero false negatives), making it ideal for identifying high-potential campaigns before launch.

### Performance Summary

| Metric | Value |
|--------|-------|
| Test Accuracy | **97.13%** |
| ROC AUC | **0.9979** |
| Recall (Successful) | **100%** |
| Precision (Successful) | **85.37%** |
| F1-Score (Successful) | **92.11%** |

---

## 💡 Key Insights

### Top Features Influencing Campaign Success (Permutation Importance)

| Rank | Feature | Impact |
|:----:|---------|:------:|
| 1 | `new_customers` | 0.18 |
| 2 | `orders_count` | 0.16 |
| 3 | `promo_cost` | 0.06 |
| 4 | `revenue` | 0.06 |

### Business Insights

#### 1. Best Promo Type — BOGO 🏆
- BOGO campaigns achieve **21% success rate** (vs 16.6% overall) — a 27% lift
- Cashback campaigns are the weakest performer (**14%**)

#### 2. Payment Method Matters 💳
- Card-paying customers convert into successful campaigns at **21%**
- Cash and Wallet users only reach 14-15%
- Suggests Card customers are higher-value and more engaged

#### 3. City Performance Is Uniform 🌆
- All 4 cities perform similarly (15-17% success rate)
- No need for city-specific marketing strategies

#### 4. Discounts Have Limited Impact 💰
- Higher discounts (60%+) yield only 19% success rate
- Mid-range discounts (20-40%) actually perform **worse** at 15%
- The assumption "more discount = more sales" is **not strongly supported** by the data

---

## 📋 Recommendations for ChefLink

Based on this analysis, here are actionable recommendations for the marketing team:

1. **🎯 Lean into BOGO campaigns** — They consistently outperform other promo types
2. **💳 Promote Card payment** — Card customers convert at higher rates; consider Card-only bonus rewards
3. **🌆 Standardize across cities** — Same playbook works in all 4 cities; no city-specific tuning needed
4. **💰 Reconsider deep discounts** — Big discounts only marginally outperform low ones — likely eroding margins for limited lift
5. **📊 Score campaigns before launch** — Use the trained model to score new campaign proposals and prioritize high-probability winners

---

## 🚀 How to Run the Project

### 1. Open the Notebook

The deliverable is a single Jupyter notebook: `campaign_analysis.ipynb`

You can run it in either:
- **Google Colab** (recommended — no setup required)
- **Local Jupyter Notebook** environment

### 2. Install Dependencies (if running locally)

```bash
pip install pandas numpy matplotlib seaborn scikit-learn xgboost joblib
```

### 3. Upload the Dataset

When prompted (Section 2 of the notebook), upload the raw campaign dataset CSV file.

### 4. Run All Cells

Execute cells sequentially from top to bottom. The notebook is fully documented section-by-section:

| Section | Phase |
|---------|-------|
| 1 | Setup & Imports |
| 2 | Load Data |
| 3 | Data Quality Assessment |
| 4 | Data Cleaning & Standardization |
| 5 | Target Definition |
| 6 | Feature Engineering |
| 7 | Model Training & Evaluation |
| 8 | Feature Importance & Business Insights |

### 5. Use the Saved Model on New Data

```python
import joblib
import pandas as pd

# Load the saved model and artifacts
model           = joblib.load("best_model.pkl")
scaler          = joblib.load("scaler.pkl")
feature_columns = joblib.load("feature_columns.pkl")

# Predict on new (preprocessed) campaign data
predictions   = model.predict(new_data[feature_columns])
probabilities = model.predict_proba(new_data[feature_columns])[:, 1]
```

---

## ⚠️ Project Notes & Limitations

- The target is defined via a business rule combining `revenue`, `promo_cost`, `orders_count`, and `new_customers`. The model effectively learns to detect this multi-criteria pattern, which is standard practice in ML problems with rule-based labels (similar to fraud detection or churn modeling).
- 166 rows had invalid date placeholders (`32/13/2025`). These were flagged via a `promo_date_missing` indicator rather than dropped, preserving 20% of the dataset.
- The dataset is small (833 rows). For production deployment, retraining on a larger dataset is recommended.
- All findings are derived from a 6-month window (Jan–Jun 2025). Seasonal patterns over a full year may differ.

---

## 📦 Deliverables

- ✅ `campaign_analysis.ipynb` — Main notebook with full analysis (Sections 1–8)
- ✅ `README.md` — This documentation file
- ✅ Saved model artifacts (auto-generated when notebook is run):
  - `best_model.pkl` — Trained XGBoost model
  - `scaler.pkl` — Feature scaler
  - `feature_columns.pkl` — Feature column order
  - `cleaned_campaigns_final.csv` — Cleaned dataset

---

## 👤 Author

End-to-end Machine Learning project — from raw messy data to deployable model, with full transparency on data cleaning decisions, target definition, modeling choices, and business interpretation.

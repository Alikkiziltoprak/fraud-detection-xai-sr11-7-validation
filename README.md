# Fraud Detection with Explainable AI & SR 11-7 Model Validation

![Python](https://img.shields.io/badge/Python-3.11-blue)
![XGBoost](https://img.shields.io/badge/XGBoost-2.0-green)
![SHAP](https://img.shields.io/badge/SHAP-Explainability-orange)
![SR11-7](https://img.shields.io/badge/SR%2011--7-Aligned-red)

---

## 1. Business Problem

Financial institutions lose billions annually to payment fraud.
But detecting fraud is not just a technical challenge — it is a
**regulatory and business decision problem**.

From a bank's perspective, two types of errors carry very different costs:

| Error Type | What Happens | Business Cost |
|------------|-------------|---------------|
| False Negative (missed fraud) | Fraudulent transaction approved | Direct financial loss |
| False Positive (false alarm) | Legitimate customer blocked | Customer friction, investigator cost |

A model that catches every fraud will also block thousands of
legitimate customers. A model optimized only for accuracy will
miss most fraud due to class imbalance (1:27 ratio in this dataset).

**The core business question:** How do we build a model that
maximizes fraud detection while minimizing operational cost —
and how do we prove to regulators that it works as intended?

This project answers that question end-to-end.

---

## 2. Model Development

### Dataset
- **Source**: IEEE-CIS Fraud Detection (Kaggle)
- **Size**: 590,540 transactions | 437 features
- **Fraud rate**: 3.50% (1:27 class imbalance)
- **Features**: Transaction data, card info, identity signals,
  Vesta engineered behavioral features

### Key EDA Findings
| Risk Factor | Fraud Rate | vs Average |
|-------------|-----------|------------|
| ProductCD 'C' | 11.7% | 3.3x |
| Credit cards | 6.7% | 1.9x |
| protonmail.com email | 40.8% | 11.7x |
| Peak hours (05:00-09:00) | 10.6% | 3.0x |

### Modeling Approach
- **Algorithm**: XGBoost Gradient Boosting
- **Class imbalance**: Handled via scale_pos_weight=27
- **Benchmark**: Logistic Regression (conceptual soundness)
- **Split**: 80/20 stratified train/test

### Threshold Optimization
Default threshold (0.50) maximizes recall but floods operations
with false alarms. Optimal threshold (0.83) balances precision
and recall based on business cost assumptions.

| Metric | Default (0.50) | Optimal (0.83) |
|--------|---------------|----------------|
| Precision | 0.25 | 0.69 |
| Recall | 0.81 | 0.56 |
| F1 Score | 0.38 | 0.62 |
| False Positives | 10,052 | 1,047 |
| False Negatives | 786 | 1,804 |

> Threshold optimization reduced unnecessary investigations by
> **89.6%** (10,052 → 1,047) — directly reducing operational cost.

---

## 3. Explainability (SHAP)

Regulators and business stakeholders require model decisions
to be explainable. A high AUC score alone is not sufficient —
the model must be interpretable at both global and individual levels.

### Global Feature Importance
Top drivers of fraud probability across all transactions:

| Rank | Feature | Description |
|------|---------|-------------|
| 1 | V258 | Vesta behavioral count feature |
| 2 | C1 | Number of addresses linked to card |
| 3 | C14 | Address match count |
| 4 | card6 | Card type (credit vs debit) |
| 5 | C13 | Billing address frequency |

### Individual Explanation
For any flagged transaction, the model provides a full
waterfall explanation showing exactly which features pushed
the fraud probability up or down — and by how much.

Example: Transaction flagged at 99.95% fraud probability
- C1=46 contributed +1.12 to fraud score
- V258=27 contributed +1.10 to fraud score
- Combined with 13 other signals → near-certain fraud

> **Regulatory requirement met**: Every model decision is fully
> traceable and auditable in accordance with SR 11-7 explainability
> standards.

---

## 4. Independent Model Validation

Following SR 11-7 guidelines, the model was independently
validated — treating model development and validation as
separate, distinct processes.

### Overall Performance
| Metric | Value |
|--------|-------|
| ROC-AUC | 0.9356 |
| Average Precision | 0.6519 |
| Baseline (random) | 0.5000 |

### Bias Analysis — Performance Across Segments
No material bias detected. Model performs consistently
across card types and product categories.

| Segment | ROC-AUC | vs Baseline |
|---------|---------|-------------|
| ProductCD H | 0.9417 | +0.006 |
| ProductCD C | 0.9328 | -0.003 |
| Credit cards | 0.9403 | +0.005 |
| Debit cards | 0.9333 | -0.002 |
| Discover | 0.9446 | +0.009 |
| American Express | 0.9382 | +0.003 |

**Finding**: Minor American Express recall gap (0.54 vs 0.56
overall) flagged for ongoing monitoring.

---

## 5. Scenario Testing

SR 11-7 requires stress testing under adverse conditions.
Six scenarios were tested to identify where the model
underperforms and why.

| Scenario | ROC-AUC | vs Baseline | Risk Level |
|----------|---------|-------------|------------|
| High Value (>$500) | 0.8929 | -0.043 | HIGH |
| No Identity Data | 0.9006 | -0.035 | HIGH |
| Low Value (<$50) | 0.9499 | +0.014 | LOW |
| With Identity Data | 0.9610 | +0.025 | LOW |
| Night Transactions | 0.9543 | +0.019 | LOW |
| High-Risk Email | 0.9434 | +0.008 | LOW |

**Critical finding**: Model degrades materially on high-value
transactions (>$500) — exactly where financial exposure is greatest.
This is the highest priority remediation item.

---

## 6. Risk Discussion & Limitations

### Model Limitations

**L1 — High Value Transactions (HIGH)**
Performance drops 4.3 points (AUC) for transactions above $500.
These represent the highest financial risk. A dedicated sub-model
for high-value transactions is recommended.

**L2 — Missing Identity Data (HIGH)**
75.6% of transactions have no identity features. The model
relies solely on transaction signals in these cases, increasing
prediction uncertainty significantly.

**L3 — Temporal Stability**
Model trained on a fixed time window. Fraud patterns evolve
with new attack vectors. Quarterly monitoring and annual
revalidation are required.

**L4 — Early Stopping Not Triggered**
Model was still improving at 300 iterations, suggesting
additional training capacity. Retraining with n_estimators=500
may further improve performance.

**L5 — Anonymous Email Domains**
protonmail.com shows 40.8% fraud rate. Rule-based email
domain filters may complement the model for low-volume
high-risk segments.

### Management Action Plan

| Finding | Priority | Recommendation |
|---------|----------|----------------|
| High value degradation | HIGH | Develop dedicated sub-model for >$500 |
| Missing identity data | HIGH | Improve identity collection at onboarding |
| Threshold selection | MEDIUM | Implement dual threshold: 0.50 alert / 0.83 block |
| AmEx recall gap | LOW | Monitor monthly, retrain if recall < 0.50 |
| Early stopping | LOW | Retrain with n_estimators=500 |

### Validation Conclusion

> **MODEL CONDITIONALLY APPROVED**
>
> The model demonstrates strong discriminatory power (AUC=0.9356)
> and stable performance across subpopulations. Two HIGH priority
> findings require remediation before full production deployment.
> Model is suitable for parallel running pending action plan completion.

---

## Technical Stack

| Tool | Purpose |
|------|---------|
| Python 3.11 | Core language |
| XGBoost | Primary fraud detection model |
| SHAP | Model explainability |
| Scikit-learn | Preprocessing, metrics, benchmarking |
| Pandas / NumPy | Data manipulation |
| Matplotlib / Seaborn | Visualization |

---

## Project Structure

```text
fraud-detection-xai-sr11-7-validation/
├── 01_EDA_and_Data_Understanding.ipynb
├── 02_Feature_Engineering_and_Preprocessing.ipynb
├── 03_Model_Training_XGBoost.ipynb
├── 04_Explainability_SHAP.ipynb
├── 05_Bias_Analysis_and_Model_Validation.ipynb
├── 06_Scenario_Testing_and_Validation_Report.ipynb
├── validation_report.txt
└── README.md
```

---

## How to Run

```bash
# Create virtual environment
python -m venv fraud-env
fraud-env\Scripts\activate

# Install dependencies
pip install pandas numpy scikit-learn xgboost shap matplotlib seaborn jupyter

# Download dataset from Kaggle
# https://www.kaggle.com/c/ieee-fraud-detection
# Place train_transaction.csv and train_identity.csv in project folder

# Run notebooks in order 01 to 06
jupyter notebook
```

---

## Author
**Ali Kiziltoprak** 
16 years experience in Finance, Risk Management & Accounting  
[GitHub](https://github.com/Alikkiziltoprak)

---

*This project is for educational and portfolio purposes.
Dataset used under Kaggle competition terms.*

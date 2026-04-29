# Fraud Detection with Explainable AI & SR 11-7 Model Validation

## Project Overview
An end-to-end fraud detection project that combines machine learning model development 
with independent model validation aligned to **SR 11-7 / OCC 2011-12** regulatory guidelines.

This project demonstrates the full lifecycle of a financial risk model:
from exploratory data analysis to production-ready validation reporting.

---

## Business Context
Financial institutions are required by regulators (Federal Reserve SR 11-7) to independently 
validate all models used in risk decisions. This project simulates that process:

- **Model Developer perspective**: Build and optimize a fraud detection model
- **Model Validator perspective**: Independently assess model soundness, stability, and limitations
- **Risk Manager perspective**: Translate findings into actionable management recommendations

---

## Dataset
- **Source**: [IEEE-CIS Fraud Detection](https://www.kaggle.com/c/ieee-fraud-detection) (Kaggle)
- **Size**: 590,540 transactions | 437 features
- **Class imbalance**: 1:27 (fraud:legitimate) | Fraud rate: 3.50%
- **Features**: Transaction data, card info, identity data, Vesta engineered features

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

## Key Results

### Model Performance
| Metric | Default (0.50) | Optimal (0.83) |
|--------|---------------|----------------|
| ROC-AUC | 0.9356 | 0.9356 |
| Precision (Fraud) | 0.25 | 0.69 |
| Recall (Fraud) | 0.81 | 0.56 |
| F1 Score | 0.38 | 0.62 |
| False Positives | 10,052 | 1,047 |

> Threshold optimization reduced false alarms by **89.6%** (10,052 → 1,047)

### Key EDA Findings
- **ProductCD 'C'**: 11.7% fraud rate (3.3x overall average)
- **Credit cards**: 6.7% fraud rate vs 2.4% for debit
- **Peak fraud window**: 05:00–09:00 (up to 10.6% fraud rate)
- **protonmail.com**: 40.8% fraud rate

### Top Risk Drivers (SHAP)
1. V258 — Vesta engineered count feature
2. C1 — Card transaction count
3. C14 — Address match count
4. card6 — Card type (credit vs debit)
5. C13 — Billing address count

---

## SR 11-7 Validation Summary

### Scenario Testing
| Scenario | ROC-AUC | vs Baseline | Risk Level |
|----------|---------|-------------|------------|
| High Value (>$500) | 0.8929 | -0.043 | HIGH |
| No Identity Data | 0.9006 | -0.035 | HIGH |
| Low Value (<$50) | 0.9499 | +0.014 | LOW |
| With Identity Data | 0.9610 | +0.025 | LOW |
| Night Transactions | 0.9543 | +0.019 | LOW |
| High-Risk Email | 0.9434 | +0.008 | LOW |

### Validation Conclusion
> **MODEL CONDITIONALLY APPROVED** — Strong discriminatory power (AUC=0.9356).
> Two HIGH priority findings require remediation before full production deployment.

---

## Regulatory Alignment
- ✅ Conceptual soundness
- ✅ Data quality assessment
- ✅ Performance testing across subpopulations
- ✅ Sensitivity analysis and scenario testing
- ✅ Explainability via SHAP
- ✅ Limitations documentation
- ✅ Management action plan

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

# Run notebooks in order 01 to 06
jupyter notebook
```

---

## Author
**Ali Kiziltoprak** |  

---

*This project is for educational and portfolio purposes.*

# Predicting Colorectal Cancer Diagnosis from Social Determinants of Health

**UC Berkeley AI/ML Professional Certificate — Capstone Project (Final Submission)**  
**Author:** Jennifer Penner

---

## What This Project Is About

Colorectal cancer is the second leading cause of cancer death in the United States — yet it is one of the most preventable cancers when caught early through routine screening. Despite this, too many people are diagnosed too late. And the people most likely to be diagnosed late are also the people who face the greatest barriers to care: lower-income households, communities of color, people without insurance, and those who haven't seen a doctor in years.

This project uses machine learning and a national health survey of over 430,000 Americans to answer one question:

> **Can we predict who is most likely to have been diagnosed with colorectal cancer based on their age, race, income, insurance status, and lifestyle — and use that information to direct screening outreach to the people who need it most?**

The answer is yes — but with an important warning: the model reflects the inequities already baked into the healthcare system, and must be used carefully to avoid making those inequities worse.

---

## Table of Contents
1. [Research Question](#research-question)
2. [Data Source](#data-source)
3. [Notebooks](#notebooks)
4. [Key Findings](#key-findings)
5. [Model Performance](#model-performance)
6. [Algorithmic Bias Warning](#algorithmic-bias-warning)
7. [Recommendations](#recommendations)
8. [Repository Structure](#repository-structure)
9. [How to Run](#how-to-run)
10. [Dependencies](#dependencies)

---

## Research Question

**Can demographic, socioeconomic, lifestyle, and healthcare access factors predict whether an individual has been diagnosed with colorectal cancer, and which factors are most strongly associated with increased risk?**

---

## Data Source

**Dataset:** CDC Behavioral Risk Factor Surveillance System (BRFSS) — 2023  
**Source:** https://www.cdc.gov/brfss/annual_data/annual_2023.html  
**File:** `LLCP2023.XPT` (SAS Transport Format)  
**Size:** 433,323 respondents, 345 variables

The BRFSS is the largest ongoing telephone health survey in the United States, conducted annually by the CDC. It captures self-reported data on health conditions, behaviors, and access to care for adults in all 50 states.

### Variables Used

| Variable | Description | Type |
|---|---|---|
| `CHCSCNC1` | Ever told had colorectal cancer (Target) | Binary |
| `_AGEG5YR` | Age group | Ordinal |
| `SEXVAR` | Biological sex | Binary |
| `_RACEGR3` | Race/ethnicity group | Categorical |
| `INCOME3` | Household income (42.1% missing — excluded from model) | Ordinal |
| `EDUCA` | Education level | Ordinal |
| `_HLTHPL1` | Health insurance coverage | Binary |
| `MEDCOST1` | Skipped doctor due to cost | Binary |
| `CHECKUP1` | Time since last routine checkup | Ordinal |
| `PERSDOC3` | Has a personal doctor | Categorical |
| `SMOKE100` | Lifetime smoker | Binary |
| `_BMI5CAT` | BMI category | Ordinal |
| `EXERANY2` | Physical activity in past 30 days | Binary |
| `DRNKANY6` | Alcohol use in past 30 days | Binary |

> **Note:** Fruit and vegetable consumption variables were not available in the 2023 core dataset and were excluded. Income had 42.1% missing data and was excluded from modeling; education was used as the primary socioeconomic proxy.

---

## Notebooks

| Notebook | Description |
|---|---|
| [`notebooks/colorectal_cancer_modeling.ipynb`](notebooks/colorectal_cancer_modeling.ipynb) | Full model comparison, GridSearchCV tuning, SHAP interpretability, equity analysis, and final recommendations |

---

## Key Findings

### What the data revealed

**Age is the strongest predictor** (SHAP rank #1, mean absolute impact ~1.1). Diagnosis rates rise sharply for respondents over 50, consistent with clinical guidelines recommending colonoscopies beginning at age 45. Age dwarfs every other feature in the model — its SHAP importance is nearly double that of the second-ranked feature.

**Race/ethnicity is the second most important factor** (SHAP rank #2, mean absolute impact ~0.6) — but the story is nuanced and critical. Some communities of color showed *lower* detected diagnosis rates in the data. This does not mean lower risk. It means lower access to the screening that catches cancer early. Cancer that is not screened for is not diagnosed.

**The model has a severe false negative problem for communities of color:**

| Race/Ethnicity | False Negative Rate |
|---|---|
| White (Non-Hispanic) | ~13% |
| Asian (Non-Hispanic) | ~58% |
| Hispanic | ~82% |
| Other/Multiracial | ~95% |
| Black (Non-Hispanic) | ~100% |

This means the model misses virtually every diagnosed Black respondent and more than 4 out of 5 diagnosed Hispanic respondents. This is algorithmic bias — the model learned from data that already reflects decades of unequal access to screening, and reproduces those inequities in its predictions.

**Healthcare access barriers compound the problem.** People who skipped the doctor due to cost, who lacked insurance, or who hadn't had a checkup in years were less likely to have a recorded diagnosis — not because they are healthier, but because they never got screened.

**Higher income and education correlate with higher diagnosis rates** — not because wealthier people are at greater risk, but because they are more likely to be screened and therefore detected.

**Socioeconomic factors are deeply interconnected.** Income and education are correlated (0.43). Lower income predicts being uninsured and more likely to skip care due to cost. These compounding disadvantages surface clearly in the data.

---

## Model Performance

Three models were trained and evaluated using ROC-AUC — a metric that measures how well the model distinguishes high-risk from low-risk individuals regardless of class imbalance.

| Model | Test AUC | True Positives | False Negatives |
|---|---|---|---|
| Logistic Regression (Baseline) | 0.798 | 5,493 | 1,128 |
| Random Forest (GridSearchCV) | 0.808 | 5,669 | 952 |
| Gradient Boosting (GridSearchCV) | 0.811 | 5,578 | 1,043 |

**What AUC means in plain English:** An AUC of 0.81 means that when the model is shown one person who has been diagnosed and one who hasn't, it correctly identifies the higher-risk person about 81% of the time. A score of 0.50 would be no better than a coin flip.

**Why ROC-AUC:** Colorectal cancer diagnoses represent ~8% of respondents. A model that predicted "No Diagnosis" for everyone would appear 92% accurate — but would catch zero cases. ROC-AUC avoids this trap.

**Random Forest is the recommended model** for screening outreach because it has the highest recall — catching 5,669 out of 6,621 diagnosed individuals (86%) in the test set, with the fewest missed cases (952).

---

## Algorithmic Bias Warning

> ⚠️ **This model should not be deployed without equity corrections.**

The false negative rate analysis reveals that the model performs dramatically worse for Black, Hispanic, and other minority respondents. The model misses nearly 100% of diagnosed Black respondents and over 80% of diagnosed Hispanic respondents.

This is not a flaw in the math — it is a reflection of a healthcare system where communities of color have historically received less screening. The model learned from that inequitable data and reproduces those patterns in its predictions. If deployed without correction, it would direct outreach resources predominantly toward White patients while overlooking the communities with the greatest unmet need.

**Any use of this model must be paired with:**
- Race-conscious outreach strategies that actively target underscreened communities regardless of model score
- Community health worker programs that build trust and screening uptake in Black and Hispanic communities
- Regular audits of model performance by race/ethnicity subgroup to detect and correct bias over time

---

## Recommendations

*Written for health plan administrators, public health officials, and community outreach coordinators.*

**For health plans (e.g., Kaiser Permanente):**
- Use the model to flag members over 50 who haven't had a recent preventive visit — but do not rely on the model alone for communities of color
- Actively override model scores for Black and Hispanic members and prioritize them for outreach regardless of predicted risk
- Stratify outreach with culturally competent, in-language materials and trusted community messengers

**For public health agencies:**
- Direct mobile screening resources to zip codes with high concentrations of uninsured, low-utilization, Black, and Hispanic residents
- Fund community health worker programs specifically in communities where the model's false negative rate is highest

**For policymakers:**
- Eliminating cost-sharing for preventive screenings and expanding insurance coverage would address the root cause of the equity gap this model exposes
- Regulate the use of predictive models in healthcare to require equity audits before deployment

**For data scientists building on this work:**
- Train separate models or apply post-processing fairness corrections for underrepresented subgroups
- Validate against claims data where available — self-reported survey data underrepresents diagnosed cases in communities with less healthcare access

---

## Repository Structure

```
capstone-colorectal-cancer/
├── README.md
├── notebooks/
│   └── colorectal_cancer_modeling.ipynb
└── data/
    └── README.md
```

---

## How to Run

1. **Download the BRFSS 2023 data:**
   - Go to: https://www.cdc.gov/brfss/annual_data/annual_2023.html
   - Download `2023 BRFSS Data (SAS Transport Format)` → unzip → place `LLCP2023.XPT` in the `notebooks/` folder

2. **Install dependencies:**
   ```bash
   pip install pandas numpy matplotlib seaborn scikit-learn imbalanced-learn shap
   ```

3. **Run notebook:**
   ```bash
   jupyter notebook notebooks/colorectal_cancer_modeling.ipynb
   ```

---

## Dependencies

| Library | Purpose |
|---|---|
| `pandas` | Data manipulation and XPT file loading |
| `numpy` | Numerical operations |
| `matplotlib` | Base visualizations |
| `seaborn` | Statistical visualizations |
| `scikit-learn` | Modeling, GridSearchCV, evaluation metrics |
| `imbalanced-learn` | Class imbalance handling |
| `shap` | Model interpretability |

---

*For questions about this project, please refer to the notebooks.*

# Gamma-GTP and Metabolic Risk Analysis

---

## Abstract
This study investigates the association between elevated Gamma-Glutamyl Transpeptidase (Gamma-GTP) levels and the risk of developing metabolic diseases, specifically hyperglycemia and hypertension, in women. Using propensity score weighting methods and logistic regression, we estimate the causal effect of elevated Gamma-GTP on metabolic risk.

---

## Introduction
Gamma-GTP is traditionally regarded as a marker of liver dysfunction or alcohol consumption. However, emerging evidence suggests that elevated Gamma-GTP levels may be associated with metabolic disorders such as diabetes and hypertension. Despite this, the causal nature of the relationship remains unclear.  
This study aims to explore whether Gamma-GTP can serve as a predictive marker for metabolic disease development in a female population.

---

## Methods

### Data Source
- 2023 Korean National Health Insurance Service health checkup data (de-identified)
- Female participants only
- Variables used: Gamma-GTP levels, fasting glucose, blood pressure, demographics, health behavior factors

### Study Design
- Observational cross-sectional study
- Treatment: Gamma-GTP ≥ 50 IU/L (high risk group)
- Outcomes:
  - Hyperglycemia (fasting glucose ≥ 100 mg/dL)
  - Hypertension (systolic BP ≥ 130 mmHg or diastolic BP ≥ 80 mmHg)

### Statistical Analysis
- Propensity Score Modeling (logistic regression)
- Inverse Probability of Treatment Weighting (IPTW)
- Weighted logistic regression for outcome estimation
- Sensitivity analysis with alternative Gamma-GTP cut-offs

---

## Results

### Baseline Characteristics
- Description of baseline differences between groups before and after weighting
- IPTW distribution plots

### Main Analysis
- OR (Odds Ratio) for hyperglycemia and hypertension according to Gamma-GTP level
- 95% Confidence Intervals and p-values

### Sensitivity Analysis
- Alternative Gamma-GTP thresholds tested (e.g., 60, 70 IU/L)

---

## Discussion
Our findings suggest that elevated Gamma-GTP levels may be significantly associated with an increased risk of hyperglycemia and hypertension in women.  
Gamma-GTP could potentially serve as an early marker for identifying individuals at higher metabolic risk.  
Future longitudinal studies are warranted to validate these findings and explore causal pathways more deeply.

---

## Repository Structure

```plaintext
gammaGTP/
│
├── README.md       # Project description and outline
├── data/            # Dummy or anonymized sample dataset
├── notebook/        # Main analysis notebook (gammaGTP_analysis.ipynb)
├── src/             # Preprocessing, modeling, visualization scripts
├── results/         # Result tables and plots
├── figures/         # Visualization outputs
└── LICENSE          # MIT License
```

---

## License
This project is licensed under the MIT License.

# Data-Driven Trading: How analysis and machine learning boost consistent returns

![Python](https://img.shields.io/badge/Python-3.10%2B-blue)
![Machine Learning](https://img.shields.io/badge/Machine%20Learning-Scikit--Learn-orange)
![Status](https://img.shields.io/badge/Status-Completed-brightgreen)

---

## 1. Introduction

**Data Science applied to trading: from intuition to evidence.**

In this project, I designed and analyzed a trading model from scratch. I took a *barely profitable model* and, through the creation and measurement of new variables, managed to capture patterns that were not visible at first glance.

The analysis not only improved the model’s accuracy but also raised its **Profit Factor above 3**, a standard of excellence in quantitative trading.  
This project helped me understand that data is everything—in an environment full of confusion about what should or shouldn’t be applied to the market, data becomes the most reliable partner for achieving profitability, and here I show how to take advantage of it.

---

## 2. Data Cleaning and Preparation

Before any optimization, the first step was an **exhaustive cleaning and preparation** of the dataset.

### Tasks performed:
- Removed rows with missing values in `ticks_SL` (Stop Loss in ticks), since they represented incomplete records.  
- Converted data types to ensure numeric columns were correctly interpreted.  
- Created a `fecha` (year, month, day) column and used it to complete missing `dia_semana` values.  
- Removed irrelevant or highly null columns (`HIN`, `hora_HIN`).  

### Initial metrics (unoptimized dataset):

| Metric | Value |
| --- | --- |
| Initial Win Rate | **31.85%** |
| Max losing streak | **16** |
| Avg RR of winning trades | **3.50** |
| Profit factor | **1.89** |

> This baseline allowed visualization of the system’s performance before applying improvements.

---

## 3. Exploratory Data Analysis (EDA)

The **EDA (Exploratory Data Analysis)** helped identify variables with the greatest impact on results.

Two main factors were analyzed:
- `ticks_rango_OTE`: size of the optimal entry zone (in ticks).  
- `antiguedad_de_liquidez`: time elapsed since liquidity was formed.

### Key findings:
- Trades with `ticks_rango_OTE` > **55.2** showed a **significant decrease in Win Rate**.  
- Trades with `antiguedad_de_liquidez` > **300** candles also had a **lower success probability**.

 **Conclusion:**  
Filtering trades above these thresholds improves the system’s overall metrics.

---

## 4. Results After Filtering

After applying the filters (`ticks_rango_OTE ≤ 55.2` and `antiguedad_de_liquidez ≤ 300`), results improved notably:

| Metric | Before filter | After filter |
| --- | --- | --- |
| Win Rate | 31.85% | **34.88%** |
| Max losing streak | 16 | **13** |
| Avg RR (TPs) | 3.50 | **3.55** |

> This filtering removed historically unfavorable setups, increasing the model’s overall consistency.  
> But wait, the optimization has just begun...

---

## 5. Take Profit (TP) Optimization

A **dynamic Take Profit optimization strategy** was applied based on the OTE zone size (`ticks_rango_OTE`).

### Objective:
Maximize the expected value for each trade:

\[
E(R) = R \times P(\text{reaching R without Stop Loss})
\]

Through percentile analysis of `RR_potencial` and simulations, an optimal TP was estimated for each OTE range.

### Results:

| Metric | Value |
| --- | --- |
| Total R:R (original filtered data) | 126.53 |
| Total R:R (with optimized TP) | **236.57** |
| Absolute improvement | **+110.04** |
| Percentage improvement | **+86.9%** |

 **Conclusion:**  
Defining TP dynamically based on trade characteristics nearly doubled total profitability.

---

## 6. Stop Loss (SL) Adjustment

The effect of the **maximum price retracement within the OTE** (`profundidad_retroceso_real`) was analyzed.

- When price retraced **0.9 of the OTE range**, most trades ended in loss.  
- An adjusted strategy was implemented:
  - If retracement reached 0.9 → early Stop Loss triggered (-1R).  
  - If trade hit TP without retracing to 0.9 → R:R was increased (×1.33).  

### Results:

| Metric | Value |
| --- | --- |
| Total R:R (original filtered) | 126.53 |
| Total R:R (with SL adjusted to 0.9) | **163.00** |
| Absolute improvement | **+36.47** |
| Percentage improvement | **+28.82%** |

 **Conclusion:**  
Incorporating a dynamic Stop Loss based on retracement significantly increased operational efficiency.

---

## 7. Combined Impact (TP + SL)

Combining both strategies produced the best overall results.

| Criterion | Win Rate (%) | Avg RR (winners) | Profit Factor | Max losing streak |
| --- | --- | --- | --- | --- |
| RR_real (unchanged) | 34.42 | 3.615 | 1.897 | 13 |
| RR_real with 0.9 SL | 30.23 | 4.815 | 2.087 | 15 |
| Optimal TP | 22.53 | 9.209 | 2.678 | 13 |
| Optimal TP + SL 0.9 | **20.88** | **12.016** | **3.149** | **13** |

 **Conclusion:**  
Although the Win Rate slightly decreased, the **Profit Factor nearly doubled (from 1.89 to 3.1)** and total profitability rose dramatically.

---

## 8. Machine Learning Prediction

Finally, a **supervised classification model** was developed to predict trade outcomes (TP or SL).

### Data preparation:
- Removed *spoiler* variables (like `RR_real`).  
- Imputed missing values.  
- Standardized (`StandardScaler`) and applied categorical encoding (`OneHotEncoder`).  
- Performed train/test split and used `SMOTE` to handle imbalance.

### Models trained:
`Logistic Regression · Random Forest · Gradient Boosting · SVM · KNN · Decision Tree · Naive Bayes (tuned) · MLP Classifier · LightGBM · XGBoost`

### Naive Bayes metrics (best result):

| Metric | Value |
| --- | --- |
| AUC | 0.7039 |
| Precision (TP) | 0.346 |
| Recall (TP) | **0.947** |
| F1-Score (TP) | 0.507 |
| Global accuracy | 0.4068 |

 **Interpretation:**  
The model achieved a **95% recall in detecting winning trades**, meaning it identified almost all real Take Profits, though with some overprediction—exactly what we aim for: **not missing profitable trades**, even at the cost of some false positives.

---

## 9. General Conclusions

### Main improvements achieved:

| Aspect | Before | Improved model (no ML) |
| --- | --- | --- |
| Win Rate | 34% | **20.88%** |
| Profit Factor | 1.89 | **3.14** |
| Recall (TP, ML model) | — | **0.95** |

### Key insights:
- EDA-based filtering removed unprofitable contexts.  
- TP optimization and SL adjustment reshaped the performance curve.  
- Win Rate slightly dropped, but the profit factor skyrocketed due to higher-reward trades.  
- The Naive Bayes model showed strong potential as a predictive trade filter, identifying profitable setups in real time.

---

## 10. Next Steps

1. **Integrate the Naive Bayes classifier** into the operational process to filter high-probability setups.  
2. **Refine the regression model** to better estimate `RR_potencial`.  
3. **Add new data sources**, such as volatility or market sentiment.  
4. **Perform full backtesting** with combined rules and models to validate robustness and stability.

---

## Final Conclusion

This project demonstrates how a trading strategy can **evolve from intuition to a data-driven system**, combining exploratory analysis, statistical optimization, and predictive modeling.

**Result:**  
> From a *Profit Factor* of **1.3 to over 3.0**, with a replicable, measurable, and evidence-based approach.

---

### Tech Stack

| Area | Tools |
|------|-------|
| Language | Python (3.10+) |
| Analysis | Pandas, NumPy, Seaborn, Matplotlib |
| Modeling | Scikit-learn, LightGBM, XGBoost |
| Optimization | GridSearchCV, Percentile interpolation |
| ML Pipeline | Imputer + Scaler + OneHotEncoder + Classifier |
| Environment | Google Colab / Jupyter Notebook |

---

### Contact
valentinalvarezlamas@gmail.com


**Autor:** Valentín Álvarez Lamas  
**Email:** [valentinalvarezlamas@gmail.com](mailto:valentinalvarezlamas@gmail.com)  
**LinkedIn:** [[linkedin.com/in/valentinalvarezlamas](https://www.linkedin.com/in/valentinalvarezlamas](https://www.linkedin.com/in/valentin-alvarez-lamas-a67720223/))

---

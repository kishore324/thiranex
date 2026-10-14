# What Drives Diabetes Progression? An Exploratory Data Analysis

**Dataset:** Diabetes dataset (442 patients, 10 baseline measurements, plus a disease-progression score measured one year later). Bundled with scikit-learn; a copy is saved in `data/diabetes.csv`.
**Tools:** pandas, NumPy, SciPy, scikit-learn, Matplotlib
**Question:** Which baseline measurements are most strongly linked to how much a patient's disease progresses?

---

## 1. Executive summary

- **BMI and blood triglycerides are the two dominant factors.** They have the strongest individual correlations with progression (r = +0.59 and +0.57) and by far the largest permutation importance in a random forest (about 0.25 and 0.23 R² lost when shuffled; every other variable is 0.04 or less).
- **Blood pressure and glucose add smaller but real signal.** Blood pressure stays significant after controlling for everything else; glucose correlates with progression (r = +0.38) but adds little once BMI and triglycerides are known.
- **Sex and age are weak.** Raw progression does not differ by sex (p = 0.37). Age shows a modest upward trend across decades, but its effect disappears once other variables are controlled.
- **The cholesterol measures are heavily redundant.** Total cholesterol and LDL correlate at r = 0.90, and their variance inflation factors (59 and 39) mean their individual coefficients cannot be trusted. Judge them together, not separately.
- **Predictive power is moderate.** The measurements explain about half the variation in progression (linear model 5-fold CV R² = 0.48 ± 0.09), so a lot remains unexplained by these ten variables.

## 2. Data overview

| Check | Result |
|---|---|
| Rows × columns | 442 × 11 |
| Missing values | 0 |
| Duplicate rows | 0 |
| Target | `progression`: mean 152.1, median 140.5, std 77.1, range 25–346 |

Variables: `age`, `sex` (coded 1/2), `bmi`, `bp` (mean blood pressure), `total_chol`, `ldl`, `hdl`, `tc_hdl_ratio` (total cholesterol / HDL), `log_triglycerides`, `glucose`.

The data is clean, so no imputation or row removal was needed.

## 3. Statistical summaries

Full table: `outputs/02_summary_statistics.csv`.

- **Shapes are mostly mildly right-skewed.** Skewness runs from -0.23 (age) to +0.80 (HDL). Progression itself has skew +0.44, with more patients at low scores than high.
- **Outliers are few.** IQR rules flag at most 9 points in any variable (glucose), and none in progression, age, sex, or blood pressure. Nothing was removed.
- **No variable is strictly normal** (Shapiro-Wilk p < 0.05 throughout). Because of this, Spearman correlations and Kruskal-Wallis tests were run alongside the Pearson and ANOVA versions, and they agree.
- **`tc_hdl_ratio` is discrete.** Its histogram shows spikes at whole numbers, which suggests the source values were rounded. Treat its fine-grained variation with care.

![Distributions](figures/01_distributions.png)
![Target distribution](figures/02_target_distribution.png)

## 4. How progression differs across groups

| Group | n | Mean progression | Median |
|---|---|---|---|
| BMI < 25 | 188 | 109.3 | 95.0 |
| BMI 25–30 | 155 | 165.0 | 168.0 |
| BMI 30+ | 99 | 213.3 | 230.0 |
| Age < 30 | 44 | 127.4 | 108.5 |
| Age 30s | 73 | 138.4 | 128.0 |
| Age 40s | 97 | 139.6 | 121.0 |
| Age 50s | 125 | 163.5 | 151.0 |
| Age 60+ | 103 | 170.4 | 168.0 |
| Sex = 1 | 235 | 149.0 | 140.0 |
| Sex = 2 | 207 | 155.7 | 141.0 |

- **BMI shows a steep, graded effect.** Mean progression roughly doubles from the lowest to the highest BMI group (109 → 213). ANOVA p ≈ 2 × 10⁻³², confirmed by Kruskal-Wallis.
- **Age shows a modest rise.** Mean progression climbs from about 127 (under 30) to 170 (60+), significant at p = 0.001 (ANOVA).
- **Sex shows no meaningful difference** (Welch t-test p = 0.37).

![Progression by group](figures/03_target_by_group.png)

## 5. Correlation analysis

| Variable | Pearson r | Spearman ρ |
|---|---|---|
| bmi | +0.586 | +0.561 |
| log_triglycerides | +0.566 | +0.589 |
| bp | +0.441 | +0.416 |
| tc_hdl_ratio | +0.430 | +0.449 |
| hdl | -0.395 | -0.410 |
| glucose | +0.382 | +0.351 |
| total_chol | +0.212 | +0.232 |
| age | +0.188 | +0.198 |
| ldl | +0.174 | +0.196 |
| sex | +0.043 | +0.037 (not significant) |

- Every variable except sex is significantly correlated with progression (p < 0.001). HDL is the only negative one: higher "good" cholesterol goes with lower progression.
- **Pearson and Spearman agree closely**, so the relationships are not driven by a few extreme points.
- **Strong inter-feature correlations:** total_chol–ldl (0.90), hdl–tc_hdl_ratio (-0.74), ldl–tc_hdl_ratio (0.66), log_triglycerides–tc_hdl_ratio (0.62). Several predictors carry overlapping information.

![Correlation heatmap](figures/04_correlation_heatmap.png)
![Correlations with progression](figures/05_target_correlations.png)
![Strongest scatter plots](figures/06_top_scatter.png)

## 6. Key influencing factors

Three complementary views were used, because correlation alone cannot separate overlapping variables.

| Variable | \|Pearson r\| | Std. coefficient | OLS p | Permutation importance | VIF |
|---|---|---|---|---|---|
| bmi | 0.586 | +0.32 | < 0.001 | 0.248 | 1.5 |
| log_triglycerides | 0.566 | +0.46 | < 0.001 | 0.229 | 10.1 |
| bp | 0.442 | +0.20 | < 0.001 | 0.040 | 1.5 |
| glucose | 0.382 | +0.04 | 0.31 | 0.021 | 1.5 |
| sex | 0.043 | -0.15 | < 0.001 | 0.017 | 1.3 |
| hdl | 0.395 | +0.06 | 0.63 | 0.010 | 15.4 |
| tc_hdl_ratio | 0.430 | +0.11 | 0.27 | 0.005 | 8.9 |
| ldl | 0.174 | +0.29 | 0.16 | 0.002 | 39.2 |
| age | 0.188 | -0.01 | 0.87 | -0.002 | 1.2 |
| total_chol | 0.212 | -0.49 | 0.058 | -0.014 | 59.2 |

How to read this:

1. **BMI, triglycerides, and blood pressure hold up in every view.** They are significant after controlling for the other variables, and BMI and triglycerides also dominate the random forest.
2. **Sex is a good example of why correlation is not enough.** Its raw correlation is near zero (+0.04), yet it is significant in the regression (std. coefficient -0.15, p = 0.0001). Once BMI, lipids, and the rest are held constant, one sex group has lower progression than the other at the same measurements. The raw difference was hidden because sex is itself related to other variables, such as HDL (r = -0.38).
3. **The cholesterol variables cannot be separated.** `total_chol`, `ldl`, `hdl` and `tc_hdl_ratio` have VIFs from 9 to 59, and their coefficients flip sign and widen into very large confidence intervals (e.g. total_chol CI spans -1.00 to +0.02). Their apparent individual effects are unreliable. Their joint contribution is real (HDL alone correlates at -0.39), but this dataset cannot say which one matters.
4. **Glucose and age look important alone but add little jointly.** Their correlations (+0.38, +0.19) appear to be largely accounted for by their relationship with BMI, blood pressure and lipids (both have near-zero, non-significant coefficients in the joint model).

**Model fit.** A linear model explains 51.8% of variance in-sample and 47.8% ± 8.5% in 5-fold cross-validation. A random forest does no better (CV R² = 0.44 ± 0.09; held-out test R² = 0.49), which suggests the relationships are mostly linear and that the ceiling for these ten variables is roughly 0.5.

![Permutation importance](figures/07_feature_importance.png)
![Standardised coefficients](figures/08_standardised_coefficients.png)

## 7. Conclusions and recommendations

1. **Focus on BMI and triglycerides first.** Together they carry most of the predictive signal in this dataset.
2. **Do not read individual cholesterol effects from this data.** If a cholesterol feature is needed, use one of them (or the ratio), not all four.
3. **Expect a ceiling around R² ≈ 0.5** from these baseline variables. Better prediction would need new information (for example, treatment, diet, or genetics), not a more complex model.
4. **Treat sex coding with care.** The dataset does not document which code is which, so the sex result should be interpreted only as "the two groups differ after adjustment", not in a specific direction.

## 8. Limitations

- **Correlation is not causation.** This is observational data; nothing here shows that changing BMI or triglycerides would change progression.
- **Sample size is modest** (n = 442), which makes cross-validation estimates noisy (± 0.09) and group comparisons for small subgroups imprecise.
- **Standard-error and p-value calculations assume roughly linear, homoscedastic relationships.** Residual diagnostics were not part of this EDA.
- **Sex is coded 1/2 without a documented meaning** in the source data.
- **Many tests were run without multiple-comparison correction.** The strongest results (BMI, triglycerides, blood pressure) are far below any reasonable threshold, but borderline p-values (e.g. total_chol, p = 0.058) should not be over-interpreted.

## 9. Reproducing this analysis

```bash
pip install -r requirements.txt
python eda.py
```

All figures are regenerated into `figures/`, and all tables into `outputs/`.

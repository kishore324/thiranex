# Data Cleaning and Data Storytelling in Python

A hands-on project for the data-cleaning tutorial brief: take a messy orders file, clean it step by step with Pandas, prove it is clean, and turn it into charts and a short report with Matplotlib and Seaborn.

> **Note on the video:** the YouTube link could not be opened while building this project, so the project follows the topics listed in the brief (missing values, outliers, duplicates, Pandas / Matplotlib / Seaborn, visual reports) rather than the video's exact examples or dataset. If the video uses specific techniques or a specific dataset, swap them into the notebook; the structure stays the same.

## What is in the folder
| Path | What it is |
|---|---|
| `notebook/data_cleaning.ipynb` | The main deliverable. 12 steps, each explains the problem, shows evidence, fixes it. Outputs are already stored. |
| `data/raw_orders.csv` | Messy input: 2,435 rows, deliberately broken (see below) |
| `data/clean_orders.csv` | Output: 2,291 validated rows plus flag columns |
| `report/data_story_report.html` | Single-file visual report: KPIs, findings, charts, full cleaning log |
| `report/cleaning_log.csv` | Every fix with the number of rows or cells affected |
| `figures/` | The 9 charts as PNG |
| `scripts/make_messy_data.py` | Recreates the raw data (seeded, so it is reproducible) |
| `scripts/build_notebook.py` | Rebuilds the notebook from source |
| `scripts/run_notebook.py` | Runs the notebook headless and stores outputs (use if Jupyter is not installed) |

## The problems planted in the data
- Missing values spelled 7 ways (`""`, `N/A`, `na`, `NULL`, `-`, `?`, `None`)
- 135 duplicate rows (80 exact copies, 55 that differ only by case or spaces)
- Inconsistent text: 23 spellings of 6 categories, 29 of 8 cities, 18 of 5 payment methods
- Dates in four formats, plus unusable ones (`TBD`, `31/02/2025`)
- Prices stored as text (`₹1,299`, `Rs. 1,299`)
- Impossible values: age 150, rating 55, negative quantity, delivery in -3 days
- Statistical outliers: prices with two extra zeros, quantity 999, delivery 365 days
- Malformed emails

## Learning path
1. **Load as text** so nothing is silently converted
2. **Audit** before touching anything
3. **Real NaN** for every placeholder
4. **Standardise text** with an explicit mapping that fails loudly on anything new
5. **Fix types**: explicit date formats, strip currency symbols
6. **Duplicates** after text is standardised, with a conflict check
7. **Outliers**: IQR vs z-score vs domain rules, compared side by side; per-product price check
8. **Missing values**: a different strategy per column (recover, estimate, label, or leave)
9. **Validation**: 13 assertions that must pass
10. **Before / after** evidence
11. **Storytelling**: 5 charts whose titles state the finding, plus generated key findings
12. **Report**: HTML with embedded charts and the cleaning log

## Run it
```bash
pip install pandas numpy matplotlib seaborn jupyter
jupyter notebook notebook/data_cleaning.ipynb      # then Run All

# or without Jupyter
python scripts/build_notebook.py
python scripts/run_notebook.py
```
Tested with Python 3 and pandas 3.0; the code avoids features that need anything newer than pandas 2.

## Results on the sample data
- 2,435 raw rows became 2,291 clean rows (135 duplicates and 9 rows with no usable date removed)
- 13 of 13 validation checks pass
- Findings: October and November produce 23% of 2025 revenue; Electronics earns 51% of revenue while Fashion has the highest return rate (19%); each extra delivery day lowers average rating by about 0.21 points; Jaipur is the slowest city and has the lowest rating; the top 10% of customers bring 37% of revenue.

These numbers describe the synthetic dataset, not a real business.

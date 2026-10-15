# E-commerce Retail Analytics: Customer Segmentation and Repeat-Purchase Prediction

An end-to-end data science project in Python: raw transactions in, business recommendations out.
It follows the standard retail-analytics workflow taught in e-commerce Python courses
(load, clean, feature engineering, EDA, RFM segmentation, visualization, conclusions) and adds a
predictive model.

> **Data note:** the dataset is **synthetic**. It uses the schema of the public "Online Retail" dataset
> (InvoiceNo, StockCode, Description, Quantity, InvoiceDate, UnitPrice, CustomerID, Country) and contains
> deliberate data-quality problems (duplicates, cancellations, guest checkouts, bad prices, missing text).
> Findings describe this simulated business, not a real one. To use real data, replace
> `data/raw_transactions.csv` with your own file in the same schema and rerun `02_analysis.py`.

## Business question
1. Where does revenue come from (time, products, categories, countries)?
2. Which customers matter most, and who is drifting away?
3. Can we predict which customers will buy again in the next 90 days, so marketing can focus?

## Project structure
| File | Purpose |
|---|---|
| `01_generate_data.py` | Builds the messy raw dataset (50,660 rows, Oct 2025 - Sep 2026) |
| `02_analysis.py` | Cleaning, EDA, RFM, modelling, charts, `results.json` |
| `data/clean_transactions.csv` | Analysis-ready sales lines with revenue and date features |
| `data/rfm_segments.csv` | Recency, frequency, monetary scores and segment per customer |
| `data/customer_repurchase_scores.csv` | Model probability of buying again, per customer |
| `charts/` | 8 PNG visuals used below |

Run: `pip install pandas numpy matplotlib seaborn scikit-learn`, then `python 01_generate_data.py && python 02_analysis.py`.

## 1. Data cleaning
| Issue | Rows | Action |
|---|---|---|
| Exact duplicate lines | 900 | Dropped |
| Missing product descriptions | 1,500 | Recovered from the StockCode lookup |
| Cancelled invoices (`C...`, negative qty) | 1,315 | Excluded from sales |
| Zero / negative prices | 400 | Excluded |
| Guest checkouts (no CustomerID) | 9,919 | Kept for revenue totals, excluded from customer analysis |

50,660 raw rows became **48,057 clean sales lines**. Features added: `Revenue`, `Month`, `DayOfWeek`, `Hour`, `Category`.

## 2. Exploratory analysis
- **Revenue 3.86M** across 13,764 orders and 2,888 identified customers. Average order value is **280**.
- **Growth plus seasonality:** revenue rises through the year as the customer base grows; **December 2025 revenue was 2.3x January 2026**, a clear holiday spike (`charts/01_monthly_revenue.png`).
- **Toys & Games** is the largest category at 38% of revenue (`02_category_revenue.png`).
- **Concentration:** the top 20% of products produce 68% of revenue (`03_top_products.png`).
- **Geography:** the United Kingdom is 71% of revenue (`04_country_revenue.png`).
- **Timing:** Friday is the busiest order day (`05_order_heatmap.png`).
- **Retention:** only about 35-40% of a cohort is still buying in a typical later month (`06_cohort_retention.png`).

## 3. RFM customer segmentation
Each customer is scored 1-5 on Recency, Frequency and Monetary value, then grouped (`07_rfm_segments.png`):

| Segment | Customers | Share of revenue | Avg days since last order |
|---|---|---|---|
| Champions | 654 (23%) | **46%** | 14 |
| Loyal | 586 (20%) | 22% | 36 |
| At Risk (high value) | 277 (10%) | 14% | 122 |
| Needs Attention | 504 (17%) | 9% | 137 |
| Lost / Hibernating | 604 (21%) | 6% | 120 |
| New / Promising | 263 (9%) | 3% | 17 |

The top 20% of customers produce **61%** of revenue, and 78% of customers have ordered more than once.

## 4. Predictive model: will a customer buy again within 90 days?
- **Setup:** cutoff 2 Jul 2026. Features use only data before the cutoff (recency, frequency, spend, tenure, basket size, product variety, recent orders, country). The label is whether the customer purchased in the following 90 days. This avoids target leakage.
- **Population:** 2,281 customers; 62.3% bought again, so always predicting "yes" is already 62% accurate.
- **Results (25% hold-out):** logistic regression AUC **0.68**, gradient boosting AUC **0.67**.
- **Recency is the strongest driver**, followed by total spend, tenure and average order value (`08_model_performance.png`).

**Honest reading:** the signal is real but modest. The model ranks customers better than chance, but it is not accurate enough to automate decisions on its own. Use the scores to prioritize outreach, not as a precise forecast. Better features (browsing behaviour, discounts used, product returns) would likely help more than a fancier algorithm. Because the data is simulated, these metrics say little about performance on real data.

## 5. Recommendations
1. **Protect Champions (23% of customers, 46% of revenue):** early access, loyalty rewards, no heavy discounting.
2. **Win back "At Risk (high value)" first:** 277 customers worth 14% of revenue who have been inactive about 4 months. This is the best return on a win-back campaign.
3. **Convert New / Promising customers into a second order:** retention drops sharply after the first month in most cohorts, so a well-timed follow-up offer matters.
4. **Plan for December:** stock and staff for the holiday spike, which is about 2x a normal month.
5. **Reduce dependence on the UK (71%)** by testing targeted campaigns in the next-largest markets.
6. **Use the repurchase score** (`customer_repurchase_scores.csv`) to rank who gets outreach when budget is limited.

## Limitations and next steps
- Synthetic data: no real-world messiness such as promotions, stock-outs or marketing channels.
- Guest checkouts (20% of revenue) cannot be tied to customers; capturing email at checkout would fix this.
- Next steps: customer lifetime value model, market-basket analysis for cross-selling, a Streamlit or Power BI dashboard on top of the CSV outputs.



An end-to-end analysis of two years of real UK e-commerce transaction data — from cleaning and exploratory analysis to customer segmentation — uncovering that a small fraction of customers drive a disproportionate share of revenue, and identifying who they are.

## Dataset

[Online Retail II](https://archive.ics.uci.edu/dataset/502/online+retail+ii) (UCI Machine Learning Repository) — transaction-level data from a UK-based online gift retailer, December 2009 to December 2011. Combined from two sheets, ~1.07 million rows.

## Project Structure

1. **Data Cleaning**
2. **Exploratory Data Analysis (EDA)**
3. **RFM Feature Engineering**
4. **Customer Segmentation (K-Means Clustering)**

## 1. Data Cleaning

- **Missing values:** Dropped ~4,000 rows with missing `Description`. Rather than dropping the ~20-25% of rows with missing `Customer ID`, these were flagged with a `has_customer_id` column — preserving them for revenue/sales analysis while excluding them later from customer-level analysis.
- **Duplicate rows:** Investigated before removing — confirmed exact duplicates (identical Invoice, StockCode, Quantity, Price, and InvoiceDate) were accidental double entries, not genuine repeat purchases, and dropped them.
- **Invalid prices:** Removed rows with zero or negative prices (data errors/adjustments, not real sales).
- **Cancelled orders:** Flagged (not dropped) using the Invoice "C" prefix. Confirmed negative quantities were the exact same rows as cancelled invoices (19,103/19,103 match), simplifying cleanup to a single flag.

## 2. Exploratory Data Analysis

**Scope:** ~£19M total revenue, 5,939 unique customers, 4,932 unique products, 43 countries.

**Key findings:**
- **Seasonality:** Revenue rises sharply September–November each year, peaking in November (~£1.4M) — consistent with holiday shopping demand.
- **Data quality catch:** Initial "top products by revenue" included non-product entries (`POSTAGE`, `DOTCOM POSTAGE`) — filtered out to isolate genuine product performance.
- **Geographic concentration:** The UK accounts for ~85% of total revenue.
- **Customer concentration:** The top 20% of customers (1,187 people) generate 77.4% of total revenue — a near-textbook 80/20 pattern that motivates the segmentation work below.
- **Cancellations:** Overall cancellation rate is 17.14%. This varies notably by country — Japan (41.1%), Channel Islands (30.4%), and Italy (29.3%) cancel at well above the overall rate, suggesting potential friction in international fulfillment worth investigating further.

## 3. RFM Feature Engineering

Built a customer-level feature table using the industry-standard RFM framework:
- **Recency** — days since last purchase
- **Frequency** — number of unique orders
- **Monetary** — total revenue generated

Features were scaled using `StandardScaler` prior to clustering, since Recency, Frequency, and Monetary operate on very different numeric scales.

## 4. Customer Segmentation (K-Means, k=4)

The optimal number of clusters (k=4) was determined using the elbow method. Each resulting segment was interpreted using average Recency/Frequency/Monetary values:

| Cluster | Segment Name | Recency (avg) | Frequency (avg) | Monetary (avg) | Count | % of Revenue |
|---|---|---|---|---|---|---|
| 0 | At-Risk / Inactive | 466 days | 2.6 | £607 | 2,035 | 7.6% |
| 1 | Regular / Core Customers | 66 days | 8.7 | £2,751 | 3,858 | 65.1% |
| 2 | VIP / Whales | 3.5 days | 257 | £413,886 | 4 | 10.2% |
| 3 | High-Value Loyal | 20 days | 120.6 | £66,361 | 42 | 17.1% |

**Key insight:** revenue concentration isn't driven by one uniform "high-value" group. It splits into a large Regular/Core base (65.1% of revenue, the vast majority of customers) and a tiny, extreme VIP tier (Clusters 2+3 combined: 27.3% of revenue from just 46 customers, under 1% of the customer base). This suggests two distinct business strategies are needed: broad retention efforts for the core base, and targeted, high-touch account management for the VIP segment.

Clusters 2 and 3's extreme order frequency and spend suggest these may be wholesale/B2B accounts rather than typical retail shoppers — worth confirming with the business if this analysis were used operationally.

## Tech Stack

- Python, pandas, NumPy
- scikit-learn (StandardScaler, KMeans)
- matplotlib

## What I'd Improve Next

- Confirm whether Clusters 2/3 are wholesale accounts and consider analyzing them separately
- Investigate the higher cancellation rates in Japan, Channel Islands, and Italy
- Extend into churn prediction (classification) using RFM features to predict at-risk customers
- Explore Market Basket Analysis (Apriori) to identify frequently co-purchased products

---

*Personal portfolio project, built as part of ongoing Data Science studies.*

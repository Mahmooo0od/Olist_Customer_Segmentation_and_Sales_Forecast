# Olist Customer Segmentation and Sales Forecast

RFM-based customer segmentation (K-Means) and category-level sales forecasting (ARIMA) on the Olist Brazilian E-Commerce dataset.

## Overview

This project answers two business questions using the public Olist marketplace data:

1. **Who are our customers?** Customers are scored on Recency, Frequency and Monetary value (RFM) and grouped into segments with K-Means clustering.
2. **What will sales look like next?** Monthly revenue of the top product category is forecast 6 months ahead with an ARIMA model, validated on a held-out test period.

Everything lives in a single notebook: `Olist_Customer_Segmentation_and_Sales_Forecast.ipynb`.

## Dataset

[Brazilian E-Commerce Public Dataset by Olist](https://www.kaggle.com/datasets/olistbr/brazilian-ecommerce) (Kaggle). Files used:

| File | Used for |
|---|---|
| `olist_customers_dataset.csv` | Mapping orders to unique customers |
| `olist_orders_dataset.csv` | Order status and purchase dates |
| `olist_order_items_dataset.csv` | Item prices (forecast revenue) |
| `olist_order_payments_dataset.csv` | Order value (RFM monetary) |
| `olist_products_dataset.csv` | Product categories |
| `product_category_name_translation.csv` | Portuguese to English category names |

Download the CSVs and place them in the same folder as the notebook.

## Methodology

### 1. RFM analysis
- Keep only `delivered` orders (canceled, unavailable and in-progress orders are excluded).
- Aggregate payments per order **before** merging, so order values are not duplicated by multiple payment rows.
- Compute per `customer_unique_id`:
  - **Recency**: days since the last purchase (snapshot = last order date + 1 day)
  - **Frequency**: number of distinct orders
  - **Monetary**: total payment value

### 2. Clustering
- `log1p` transform to reduce the heavy skew in Frequency and Monetary, then `StandardScaler`.
- K selected using the **Elbow method** and **Silhouette score** (K=4 chosen for business interpretability; silhouette 0.40).
- Segment names are assigned automatically from each cluster's profile, not from the arbitrary cluster number.

### 3. Sales forecast
- Revenue is based on `price` from order items for delivered orders, grouped by English category name.
- The top category by revenue (`health_beauty`) is aggregated monthly.
- Incomplete early months (2016) are dropped; the series covers Jan 2017 to Aug 2018 (20 months).
- ARIMA order is chosen by AIC (with drift), evaluated on the last 3 months against a naive baseline, then refit on all data for a 6-month forecast with confidence intervals.

## Results

**Customer segments** (93,357 customers; 97% bought only once):

| Segment | Customers | Avg. Recency (days) | Avg. Frequency | Avg. Monetary |
|---|---:|---:|---:|---:|
| Repeat Buyers | 2,801 | 220 | 2.11 | 308.59 |
| Big Spenders (At Risk) | 32,111 | 272 | 1.00 | 296.50 |
| New Customers | 16,108 | 42 | 1.00 | 133.01 |
| Lapsed Low-Value | 42,337 | 288 | 1.00 | 68.37 |

**Forecast quality** (backtest on the last 3 months):

| Model | MAE | MAPE |
|---|---:|---:|
| ARIMA(2,1,2) with drift | 4,498 | 4.2% |
| Naive baseline | 15,353 | n/a |

## Key takeaways

- Retention is the main opportunity: almost all customers purchase once, and the largest group (Lapsed Low-Value) has not bought in about 9 months.
- **Big Spenders (At Risk)** are a high-value single-purchase group that has gone quiet, a good target for win-back campaigns.
- **Repeat Buyers** are few (about 3%) but the only group showing real loyalty.
- `health_beauty` revenue shows a strong upward trend through 2018.

## Limitations

- Because most customers have a Frequency of 1, segments are separated mainly by Recency and Monetary value.
- The forecast uses only 20 months of data with no seasonality component, so treat the 6-month projection as an approximate trend estimate rather than a precise prediction.

## Getting started

```bash
git clone https://github.com/<your-username>/Olist_Customer_Segmentation_and_Sales_Forecast.git
cd Olist_Customer_Segmentation_and_Sales_Forecast
pip install -r requirements.txt
jupyter notebook Olist_Customer_Segmentation_and_Sales_Forecast.ipynb
```

Place the Olist CSV files in the project folder before running the notebook.

## Tech stack

Python, pandas, NumPy, scikit-learn, statsmodels, matplotlib, seaborn

## Project structure

```
.
├── Olist_Customer_Segmentation_and_Sales_Forecast.ipynb
├── requirements.txt
└── README.md
```


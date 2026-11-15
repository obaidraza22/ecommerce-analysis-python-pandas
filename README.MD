# Olist Brazilian E-Commerce Analysis (Python & Pandas)

An end-to-end exploratory data analysis built using **Python and Pandas** on the Brazilian Olist marketplace dataset (2016–2018), covering **99,441 orders** and **112,650 items** across 9 relational CSV files.

The goal of this project was to use Pandas to clean, deduplicate, and join messy multi-table e-commerce data without inflating row counts, and answer three practical questions around delivery delays, category margins, and seasonal demand.

## Project Structure

```
ecommerce-business-analytics/
├── data/
│   ├── raw/                  # Original Kaggle CSVs (ignored in git)
│   └── processed/            # Cleaned master tables (order_masters, item_masters)
├── notebooks/
│   ├── 01_data_understanding.ipynb          # Schema checks, null audits, duplicate discovery
│   ├── 02_data_cleaning.ipynb               # Type conversions, translation fixes, deduplication & joins
│   └── 03_eda_and_business_analysis.ipynb   # Feature engineering, metrics & charts
├── reports/                  # Exported PNG charts
└── README.md
```

## Data Prep & Cleaning Notes

Before plotting anything, I spent most of Notebook 2 making sure the tables could be joined safely in Pandas:

* **Avoiding join duplication:** `order_items`, `order_payments`, and `order_reviews` all have multiple rows per `order_id` (split payments, multi-item carts, or duplicate reviews). I aggregated each of those tables down to a strict 1-row-per-`order_id` grain using `.groupby().agg()` before merging onto `orders`, keeping the final `order_masters` table at exactly **99,441 rows**.

* **Geolocation cleanup:** The raw geolocation file had over **1,000,163 rows** with **261,831 exact duplicates** and dozens of slightly different coordinates per zip code. I grouped by `geolocation_zip_code_prefix` and averaged the coordinates to shrink it to **19,015** unique zip codes.

* **Missing product categories & translations:** Filled `610` missing product categories with `"unknown"`, imputed 2 missing product dimension rows with column medians, and patched 3 missing Portuguese-to-English mappings (`pc_gamer`, `portateis_cozinha_e_preparadores_de_alimentos`, and `unknown`) via `pd.concat()` before merging.

* **Timestamps:** Parsed all date columns from strings to `datetime64` to calculate actual delivery times and delay deltas.

## Main Findings

### 1. Delivery delays have a sharp 3-day tolerance cliff

Out of **96,478 delivered orders**, **91.9%** arrived on time or early and averaged a **4.29** review score (median **5.0**). The **8.1%** that arrived late averaged **2.57** (median **2.0**).

Breaking delays down into buckets shows where customer patience runs out:

* **On time / early:** `4.29` avg score

* **1–3 days late:** `3.76` avg score

* **4–7 days late:** `2.32` avg score

* **7+ days late:** `1.73` avg score

**Takeaway:** Customers are relatively forgiving of a 1–3 day slip, but once an order crosses 3 days late, ratings fall off a cliff. Flagging orders at +2 days past their estimated delivery date would catch most of these before they turn into 1-star reviews.

### 2. High order volume doesn't always mean top revenue (or low shipping costs)

* **Health & Beauty** brought in the most revenue (**1.23M BRL** across 8,647 orders), even though **Bed, Bath & Table** had more orders (**9,272** orders, **1.02M BRL**).

* **Watches & Gifts** ranked #2 in revenue (**1.17M BRL**) with far fewer orders (5,495) thanks to a higher average item price (**199.04 BRL**) and the lowest shipping-to-total-spend ratio (**7.76%**).

* Bulky categories like **Furniture & Decor** (**19.13%**) and **Housewares** (**18.82%**) lose nearly a fifth of total customer spend to freight charges.

**Takeaway:** Light, high-ticket items like watches carry much healthier unit economics on Olist than heavy home goods, where high freight costs likely hurt conversion.

### 3. Black Friday 2017 was the major inflection point

After a quiet soft-launch period in late 2016 (`2016-09` to `2016-12`), monthly revenue climbed steadily throughout 2017 from \~**110K BRL** in January to \~**650K BRL** in October.

In **November 2017**, sales jumped nearly **50% month-over-month** to almost **1.0M BRL** (\~987K BRL) during Black Friday, and the platform held onto a higher baseline of **850K–980K BRL/month** through 2018.

## Tools Used

* Python 3, Pandas, NumPy

* Matplotlib, Seaborn

* JupyterLab

## How to Run

1. Download the [Brazilian E-Commerce Public Dataset by Olist](https://www.kaggle.com/datasets/olistbr/brazilian-ecommerce) from Kaggle and place the 9 CSV files inside `data/raw/`.

2. Run the notebooks in order (`01` $\rightarrow$ `02` $\rightarrow$ `03`). Notebook `02` will generate the processed CSVs in `data/processed/`, and Notebook `03` will reproduce the charts in `reports/`.
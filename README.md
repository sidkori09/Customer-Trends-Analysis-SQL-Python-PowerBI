# 🛍️ Customer Trends Analysis | SQL · Python · Power BI

**An end-to-end retail analytics case study — from raw data to an interactive Power BI dashboard.**

![Python](https://img.shields.io/badge/Python-Pandas-blue?logo=python&logoColor=white)
![SQL](https://img.shields.io/badge/SQL-PostgreSQL-336791?logo=postgresql&logoColor=white)
![Power BI](https://img.shields.io/badge/Power%20BI-Dashboard-F2C811?logo=powerbi&logoColor=black)
![License](https://img.shields.io/badge/License-MIT-green)

**Maintainer:** [Siddharth Kori](#-author)

---

## 📖 About This Repository

This repository documents my hands-on implementation of a retail customer-behavior analytics project, built as part of my learning journey toward a Data Analyst / Business Intelligence role.

I did not design the original business case or collect the dataset — both originate from a public learning project created by **Siddharth Kori** (see [Credits & Attribution](#-credits--attribution)). What you'll find here is **my own walkthrough**: I cleaned and transformed the data myself in Python, wrote and ran every SQL query independently to validate the results, built the Power BI dashboard from scratch, and recorded the insights and recommendations in my own words based on what the numbers actually showed. I'm sharing it as a transparent, reproducible record of that process rather than as original research.

---

## 🎯 Project Overview

The project simulates a real-world analytics engagement for a retail business, structured around the three tools most commonly expected of a Data Analyst:

| Stage | Tool | What I Did |
|---|---|---|
| **Data Preparation** | Python (Pandas) | Cleaned, validated, and engineered new features in a 3,900-row retail transactions dataset |
| **Business Analysis** | SQL (PostgreSQL) | Loaded the cleaned data into a relational database and wrote 10 queries to answer specific business questions |
| **Visualization** | Power BI | Built a single-page interactive dashboard with KPI cards, slicers, and category/age-group breakdowns |

The goal was to go through the full analyst workflow — not just analyze data, but also *interpret* it and turn it into something a business stakeholder could act on.

---

## 🧩 Business Problem

> *"How can the company leverage consumer shopping data to identify trends, improve customer engagement, and optimize marketing and product strategies?"*

A retail business wants to understand how its customers shop — across demographics, product categories, and purchase channels — in order to:

- Identify which factors (discounts, reviews, season, shipping preference, subscription status) actually influence spending and repeat purchases
- Segment customers meaningfully for targeted marketing
- Decide where discounting helps revenue versus where it simply erodes margin
- Surface which products and age groups drive the most revenue

The full brief is preserved in [`reports/business_problem_statement.pdf`](reports/business_problem_statement.pdf).

---

## 📊 Dataset Description

| Detail | Value |
|---|---|
| **Source** | [Customer Shopping Trends Dataset](https://www.kaggle.com/datasets/iamsouravbanerjee/customer-shopping-trends-dataset) (Kaggle, synthetic retail data) |
| **Rows** | 3,900 customer transactions |
| **Columns** | 18 |
| **Missing values** | 37 (in `Review Rating` only) |

**Key fields:**
- **Demographics:** Age, Gender, Location
- **Transaction details:** Item Purchased, Category, Purchase Amount (USD), Size, Color, Season
- **Behavioral signals:** Review Rating, Subscription Status, Discount Applied, Promo Code Used, Previous Purchases, Frequency of Purchases
- **Fulfillment:** Shipping Type, Payment Method

Raw data lives in [`data/customer_shopping_behavior.csv`](data/customer_shopping_behavior.csv).

---

## 🛠️ Tools Used

| Tool | Purpose |
|---|---|
| **Python** (Pandas, SQLAlchemy) | Data cleaning, feature engineering, and loading data into a SQL database |
| **SQL** (PostgreSQL) | Structured querying to answer 10 specific business questions |
| **Power BI** | Interactive dashboard for stakeholder-facing visualization |
| **Jupyter Notebook** | Documenting the cleaning and EDA process step by step |

---

## 🧹 Data Cleaning Process

All cleaning steps are documented in [`notebooks/customer_shopping_behavior_analysis.ipynb`](notebooks/customer_shopping_behavior_analysis.ipynb). Here's what I actually did and why:

1. **Initial inspection** — used `.info()` and `.describe(include='all')` to check data types, ranges, and overall structure before touching anything.
2. **Missing value handling** — found 37 nulls, all in `Review Rating`. Rather than dropping rows or filling with a single global average (which would mask category-level quality differences), I imputed each missing rating with the **median rating of that product's category**.
3. **Column standardization** — renamed all columns to `snake_case` (e.g. `Purchase Amount (USD)` → `purchase_amount`) so the schema would behave consistently once it moved into SQL and Power BI.
4. **Feature engineering — `age_group`** — bucketed `age` into four quartile-based segments (`Young Adult`, `Adult`, `Middle-aged`, `Senior`) using `pd.qcut`, so age-based revenue analysis didn't depend on arbitrary fixed cutoffs.
5. **Feature engineering — `purchase_frequency_days`** — converted the categorical `frequency_of_purchases` field (e.g. "Weekly", "Fortnightly", "Quarterly") into a numeric day-interval equivalent, making it usable in quantitative comparisons rather than just as a label.
6. **Redundancy check** — tested whether `discount_applied` and `promo_code_used` were duplicate signals. They matched on every single row, so I dropped `promo_code_used` to avoid double-counting the same effect in later analysis.
7. **Database load** — connected the cleaned DataFrame to a PostgreSQL database via SQLAlchemy so the SQL stage could run against a real relational table instead of a flat file.

---

## 🔍 Exploratory Data Analysis

Before jumping into business questions, I profiled the dataset to understand its shape and any biases I'd need to account for:

- **Gender split is uneven:** 2,652 male records vs. 1,248 female — important context, since raw revenue totals by gender are *not* directly comparable per customer.
- **Category mix:** Clothing (44.5%) and Accessories (31.8%) dominate transaction volume; Footwear (15.4%) and Outerwear (8.3%) make up the rest.
- **Season and payment method are both fairly evenly distributed** (no single season or payment type dominates), which told me seasonality and payment preference were unlikely to be major revenue drivers on their own.
- **Average review rating is nearly identical across genders** (3.74 vs. 3.75) — product satisfaction doesn't appear to differ by gender in this data.
- **Subscription is a minority behavior:** only ~27% of customers (1,053 of 3,900) are subscribed.

This profiling step shaped which SQL questions were worth digging into further.

---

## 💡 Key Insights

These are the findings I drew directly from running the SQL queries in [`sql/customer_behavior_queries.sql`](sql/customer_behavior_queries.sql) against the cleaned dataset:

1. **Higher male revenue is a volume effect, not a spending-power effect.** Male customers generated $157,890 vs. $75,191 from female customers — but once you normalize for the gender imbalance in the data, the *average spend per customer* is almost identical (~$59.5 male vs. ~$60.3 female). The takeaway isn't "men spend more," it's "there are more male customers in this dataset."
2. **Discount usage isn't a reliable signal of price sensitivity.** 839 customers used a discount *and* still spent above the average purchase amount ($59.76) — a meaningful chunk of "discount shoppers" are not actually budget-constrained.
3. **Subscribers don't outspend non-subscribers.** Average spend was virtually flat between subscribers ($59.49) and non-subscribers ($59.87). Subscription drives recurring engagement, not bigger basket sizes — which changes how a subscription program should be pitched internally.
4. **Customer segmentation is heavily skewed toward "Loyal."** Using the New (1 purchase) / Returning (2–10) / Loyal (10+) split, **79.9%** of customers already fall into "Loyal," while only **2.1%** are "New." That's a strong signal the acquisition funnel is thin relative to the retained base — worth flagging as a possible new-customer acquisition gap rather than a loyalty success story.
5. **Repeat purchase behavior and subscription aren't strongly linked.** Among customers with more than 5 previous purchases, only ~27.6% are subscribed — almost identical to the ~27% subscription rate across the *entire* customer base. Being a repeat buyer doesn't meaningfully predict subscription.
6. **Express shipping customers spend modestly more.** Average purchase amount is $60.48 for Express vs. $58.46 for Standard shipping — a small but consistent ~3.5% premium associated with faster delivery.
7. **The highest-rated products are also some of the most discount-dependent.** "Hat" tops the discount-rate list at 50% of its purchases involving a discount, yet still ranks in the top 5 by average rating (3.80). High satisfaction and heavy discounting aren't mutually exclusive — but it raises the question of whether the rating is partly a function of perceived deal value.
8. **Revenue is fairly evenly spread across age groups**, with Young Adults contributing the most ($62,143) and Seniors the least ($55,763) — a gap of roughly 10%, not a dramatic skew.

---

## 📈 Dashboard Features

The Power BI dashboard ([`dashboard/customer_behavior_dashboard.pbix`](dashboard/customer_behavior_dashboard.pbix)) is a single, densely-informative page built around quick stakeholder scanning:

- **KPI cards** — Number of Customers, Average Purchase Amount, and Average Review Rating at a glance
- **Subscription donut chart** — share of customers by subscription status
- **Revenue by Category** and **Sales (volume) by Category** — clustered column charts side by side, so revenue and order count can be compared directly
- **Revenue by Age Group** and **Sales by Age Group** — clustered bar charts using the engineered `age_group` field
- **Interactive slicers** for Subscription Status, Gender, Category, and Shipping Type — so any stakeholder can filter the entire page down to a specific customer slice without touching the underlying data

---

## ✅ Business Recommendations

Based on the insights above, here's what I'd recommend to the business:

- **Don't read raw gender revenue totals at face value.** Report per-customer averages alongside totals so leadership doesn't draw the wrong conclusion about gender-based spending power.
- **Re-evaluate the discount strategy.** Since a large share of discount users would likely have purchased anyway, consider tightening discount eligibility (e.g. first-time buyers, cart abandoners) rather than applying it broadly.
- **Reframe the subscription pitch around retention, not basket size.** Since subscribers don't spend more per order, marketing should emphasize convenience and exclusive access rather than implying bigger savings or higher-value purchases.
- **Invest more deliberately in top-of-funnel acquisition.** With "New" customers making up only ~2% of the base, the business may be retaining well but acquiring slowly — worth validating against actual marketing spend and channel data.
- **Promote Express shipping at checkout.** The modest spend premium associated with Express orders suggests upselling faster delivery could lift average order value with minimal friction.
- **Use top-rated, low-discount-dependency products in campaigns** (e.g. Sandals, Boots, Skirt) where customer satisfaction isn't being propped up by price cuts.

---

## 🚀 Future Enhancements

Things I'd like to explore if I extend this project further:

- Add **cohort or time-series analysis** — this dataset has no transaction date field, so all "frequency" analysis is categorical rather than truly time-based. A version with timestamps would allow real retention-curve analysis.
- Build a **customer lifetime value (CLV) estimate** combining purchase amount, frequency, and previous purchases into a single score.
- Add a **second Power BI page** dedicated to product-level performance (top products per category, discount-rate vs. rating scatter, etc.) instead of compressing everything into one page.
- Automate the **Python → SQL load** with a small script/Makefile instead of manual notebook cells, so the pipeline can be re-run end-to-end.
- Re-test the New/Returning/Loyal segmentation thresholds — the current cutoffs produce a heavily Loyal-skewed split, and revisiting them against actual business definitions of "loyalty" would make the segmentation more useful.

---

## 📁 Repository Structure

```
Customer-Trends-Analysis-SQL-Python-PowerBI/
├── README.md
├── LICENSE
├── data/
│   └── customer_shopping_behavior.csv
├── notebooks/
│   └── customer_shopping_behavior_analysis.ipynb     # Data cleaning, feature engineering, EDA, DB load
├── sql/
│   └── customer_behavior_queries.sql                 # 10 business-question queries
├── dashboard/
│   └── customer_behavior_dashboard.pbix               # Power BI dashboard
└── reports/
    ├── business_problem_statement.pdf
    ├── customer_shopping_behavior_analysis_report.pdf
    └── customer_shopping_behavior_analysis_presentation.pptx
```

---

## ▶️ How to Run This Project

1. **Clone the repository**
   ```bash
   git clone https://github.com/<your-username>/Customer-Trends-Analysis-SQL-Python-PowerBI.git
   cd Customer-Trends-Analysis-SQL-Python-PowerBI
   ```

2. **Run the notebook** — open `notebooks/customer_shopping_behavior_analysis.ipynb` to walk through data loading, cleaning, feature engineering, and the database load step. Update the database credentials in the connection cell to match your local PostgreSQL/MySQL/SQL Server setup.

3. **Run the SQL queries** — once the cleaned data is loaded into your database, open `sql/customer_behavior_queries.sql` and run each query against the `customer` table to reproduce the business-question answers above.

4. **Open the dashboard** — load `dashboard/customer_behavior_dashboard.pbix` in Power BI Desktop and point the data source to your database (or refresh from the CSV directly) to explore the visuals interactively.

---

## 🙏 Credits & Attribution

This project is based on a publicly shared learning project originally created by **[Amlan Mohanty](https://www.youtube.com/@amlanmohanty1)**, including the business problem framing, dataset selection, and overall project structure (see his [original YouTube walkthrough](https://www.youtube.com/@amlanmohanty1)). The underlying dataset is the [Customer Shopping Trends Dataset](https://www.kaggle.com/datasets/iamsouravbanerjee/customer-shopping-trends-dataset) on Kaggle.

I implemented, customized, and analyzed this version independently as a learning exercise — every line of cleaning code, SQL query, dashboard visual, and written insight in this repository reflects my own work on top of that original framing. I'm grateful to Amlan for putting together such a practical, well-scoped project for analysts to learn from.

---

## 📜 License

This project is licensed under the MIT License — see [`LICENSE`](LICENSE) for details. The original license and copyright notice from the source project have been retained as required by the MIT terms.

---

## 👤 Author

**Siddharth Kori**
Data Analyst in training — building a portfolio in SQL, Python, and Power BI.

If you're also learning data analytics, feel free to fork this, try it on the same dataset yourself, and compare notes — that's exactly how I used it.

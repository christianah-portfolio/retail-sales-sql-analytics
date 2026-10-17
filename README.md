# Retail Sales Analysis (SQL)

**A SQL Server project analysing retail sales data to identify sales trends, customer purchasing patterns, and differences in revenue across product categories. The project involved cleaning the dataset, validating records, and using T-SQL queries to answer business questions and extract actionable insights.**

---

## Project Overview

**Tools & Skills Used**
Microsoft SQL Server, T-SQL, Data Cleaning, Data Validation, Aggregations, Filtering, Grouping, Business Analysis.

**Project Goals**
To analyse retail transactions, understand customer purchasing patterns, compare sales performance across product categories, and identify trends that can support business decisions.

**Key Findings**
The original dataset contained 2,000 records, which was reduced to 1,997 after cleaning. The analysis examined revenue across product categories, customer purchasing behaviour, and shopping patterns at different times of the day. Evening transactions accounted for approximately 53% of orders.

---

## Business Questions

The analysis used SQL Server to answer the following ten business questions:

1. **Daily Sales**: What sales transactions were recorded on November 5, 2022?
2. **Clothing Sales**: Which clothing transactions in November 2022 involved purchases of four or more units?
3. **Category Performance**: How do total revenue and order counts compare across product categories?
4. **Customer Demographics**: What is the average age of customers purchasing Beauty products?
5. **High-Value Transactions**: Which individual transactions generated more than 1,000 in sales?
6. **Gender and Category Analysis**: How do transaction volumes differ by gender across product categories?
7. **Monthly Sales Performance**: Which month had the highest average transaction value in each year?
8. **Top Customers**: Who are the five customers with the highest total sales?
9. **Customer Reach**: How many unique customers purchased from each product category?
10. **Shopping Time Patterns**: How many orders were made in the morning, afternoon, and evening?

---

## Dataset

A retail sales table with **2,000 raw records** and 11 columns, covering **1 January 2022 to 31 December 2023**.

| Column | Meaning |
|---|---|
| `transactions_id` | Unique ID of each sale |
| `sale_date`, `sale_time` | When the sale happened |
| `customer_id` | Customer who made the purchase |
| `gender`, `age` | Customer details |
| `category` | Beauty, Clothing, or Electronics |
| `quantity` | Units bought |
| `price_per_unit` | Price of one unit |
| `cogs` | Cost of goods sold |
| `total_sale` | Total value of the sale |

---

## Data Cleaning

| Problem found | What I did | Result |
|---|---|---|
| The quantity column was misspelt (`quantiy`) | Renamed it with `sp_rename` | Column is now `quantity` |
| **10** records had a missing `age` | Replaced them with the average customer age (41) | No missing ages left |
| **3** records had missing quantity, price, COGS, and total sale | Deleted them, because the sale values could not be recovered | **1,997** clean records |

After cleaning, I checked the record count and confirmed that no missing ages remained.

---

## Key Findings

| Question | Result |
|---|---|
| Total clean transactions | **1,997** |
| Unique customers | **155** |
| Total revenue | **911,720** |
| Revenue by category | Electronics **313,810** (684 orders), Clothing **311,070** (701), Beauty **286,840** (612) |
| Category and gender | Beauty is the only category where more women than men bought (330 vs 282); Clothing and Electronics are almost equal |
| Highest average transaction value | **July 2022** (541.34) and **February 2023** (535.53) |
| Large transactions | **306** sales were worth more than 1,000 |
| Top customer | Customer **3**, with total sales of **38,440**, followed by customers 1, 5, 2, and 4 |
| Customers per category | Clothing 149, Electronics 144, Beauty 141 |
| Time of day | Evening **1,062** orders (about 53%), Morning **558**, Afternoon **377** |
| Beauty customers | Average age of about **40** |

**What this means:** the three categories are almost equally important, so no single category carries the business. Most orders come in the evening, so staffing and promotions could be timed for that period. A handful of customers account for a lot of the spending, which makes them worth keeping.

---

## Example Queries

**Revenue and orders by category**
```sql
SELECT
     category,
     SUM(total_sale) AS net_sales,
     COUNT(*) AS total_order
FROM Retail_Sales_Analysis
GROUP BY category
```

**Best month by average sale in each year** (window function)
```sql
SELECT Year, Month, Average_sale
FROM
(
SELECT YEAR(sale_date) AS Year,
       MONTH(sale_date) AS Month,
       AVG(total_sale) AS Average_sale,
       RANK() OVER(PARTITION BY YEAR(sale_date) ORDER BY AVG(total_sale) DESC) AS Rank
FROM Retail_Sales_Analysis
GROUP BY YEAR(sale_date), MONTH(sale_date)
) AS T1
WHERE Rank = 1
```

**Orders by time of day**
```sql
SELECT
     CASE
         WHEN DATEPART(HOUR, sale_time) < 12 THEN 'Morning'
         WHEN DATEPART(HOUR, sale_time) BETWEEN 12 AND 17 THEN 'Afternoon'
         ELSE 'Evening'
     END AS TimeOfDay,
     COUNT(*) AS No_Of_Orders
FROM Retail_Sales_Analysis
GROUP BY
     CASE
         WHEN DATEPART(HOUR, sale_time) < 12 THEN 'Morning'
         WHEN DATEPART(HOUR, sale_time) BETWEEN 12 AND 17 THEN 'Afternoon'
         ELSE 'Evening'
     END
```

The full script, with all ten business queries, is here: [sql/retail_sales_analysis.sql](sql/retail_sales_analysis.sql)

---

## Query Results

**Revenue and orders by category**
![Revenue by category](https://github.com/christianah-portfolio/retail-sales-sql-analytics/blob/main/results/01_revenue_by_category.png)
**Orders by time of day**
![Orders by time of day](results/02_orders_by_time_of_day.png)

**Month with the highest average sale in each year**
![Best month by average sale](results/03_best_month_by_average_sale.png)

---

## How to Run This Project

1. Open SQL Server Management Studio (SSMS) and create a database (the script does this in section 1).
2. Import `data/retail_sales_dataset.csv` into the database, and name the table `Retail_Sales_Analysis`. Keep the original column names, because the script renames the misspelt `quantiy` column itself.
3. Open `sql/retail_sales_analysis.sql` and run it section by section: setup, data check, cleaning, exploration, then the business analysis.
4. Run the cleaning section only once, because it renames a column and deletes records.

---

## Limitations

- Replacing 10 missing ages with the average age is a simple method. It slightly affects age-based results, but those records are a small share of the data.
- The dataset does not state the currency, so revenue is shown without a currency symbol.
- The data shows what happened, but not why.

---

## Project Files

```
retail-sales-sql-analytics/
├── README.md
├── sql/
│   └── retail_sales_analysis.sql
├── data/
│   └── retail_sales_dataset.csv
└── results/
    ├── 01_revenue_by_category.png
    ├── 02_orders_by_time_of_day.png
    └── 03_best_month_by_average_sale.png
```

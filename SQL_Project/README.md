# Brazilian Olist E-commerce Analysis (SQL and Python)

I loaded nine tables of Brazilian marketplace data into MySQL, checked and standardized them, built reporting views, and used SQL to answer ten business questions about sales, customers, delivery and sellers.

## Dataset

The Olist public dataset on Kaggle covers 99,441 orders placed between September 2016 and October 2018 by 96,096 unique customers, across 32,951 products. It comes as nine CSV files.

- Source: [Olist Brazilian E-commerce dataset on Kaggle](https://www.kaggle.com/datasets/olistbr/brazilian-ecommerce)
- Table definitions, keys and data types: [OLIST_DataDictionary.xlsx](OLIST_DataDictionary.xlsx)

## Tools

- **MySQL** for the database, views and analysis queries
- **Python** (pandas, SQLAlchemy, PyMySQL) to load the CSV files into MySQL

## What is in this folder

| File or folder | What it does |
| :--- | :--- |
| `Dataset/` | The nine source CSV files |
| `SQL_Import.ipynb` | Reads each CSV, checks the primary keys for duplicates, and loads the tables in foreign key order |
| `Environment_Setup.sql` | Creates the `olist` database and tables, fixes load issues, backs up the tables, explores and standardizes the data, and builds three views |
| `Business_Insights.sql` | The ten business questions and their queries |
| `SQL Results/` | Query output for questions 3 to 10 as CSV files. Results for questions 1 and 2 are in comments inside `Business_Insights.sql` |
| `OLIST_DataDictionary.xlsx` | Data dictionary for all nine tables |

## How to run it

1. Run the first steps of `Environment_Setup.sql` to create the database and empty tables.
2. Update the connection details in `SQL_Import.ipynb`, then run it to load the CSV files.
3. Run the rest of `Environment_Setup.sql` to check the load, standardize the data and create the views.
4. Run `Business_Insights.sql` to reproduce the results.

## Data checks and cleaning

- Row counts in MySQL matched the row counts in Python for every table.
- Loading failed on repeated `customer_unique_id` values, which is expected for repeat customers, so I dropped the unique constraint on that column.
- I backed up every table before changing anything.
- I trimmed text fields, confirmed state codes were consistent, and checked review scores stayed between 1 and 5.
- I added an English product category name, a delivery time in days, and a late delivery flag (delivered after the estimated date) to support the analysis.
- I grouped the `boleto` and `voucher` payment types into one updated payment type column.
- I created three views (`v_order_details`, `v_item_details`, `v_seller_performance`) so the analysis queries stay short.

## Business questions

1. Overall business snapshot
2. Overall performance
3. Product portfolio performance, revenue versus satisfaction
4. Monthly sales trend
5. Top spenders
6. Delivery timeliness
7. Delivery challenges by region
8. Seller performance
9. Month over month trends
10. Cumulative sales by product category

The queries use common table expressions, window functions (`LAG`, `PERCENT_RANK`, running totals), views and multi-table joins.

## Key findings

All amounts are in Brazilian reais.

- **Scale:** The average order was worth about R$161, and the average delivery took about 12.5 days, with a longest delivery of 210 days.
- **Revenue:** Health and beauty earned the most revenue (about R$1.44 million including freight), followed by watches and gifts and then bed, bath and table. Bed, bath and table sold the most items and orders.
- **Top spenders:** The top 10% of customers by lifetime spend (9,611 customers) averaged 1.1 orders each and waited about 14 days for delivery. Repeat buying is rare in this dataset.
- **Seasonality:** November 2017 was the peak month for both revenue and order count.
- **Delivery:** About 93% of orders arrived on or before the estimated date, and about 6.6% arrived late.
- **Regional gaps:** Late deliveries were most common in northeastern states, led by Alagoas (21.5% of orders), Maranhão (19.3%) and Sergipe (17.4%). Average freight in these states was between R$36 and R$38.

## Future work

- Visualize the findings in a Tableau or Power BI dashboard.
- Test whether freight cost explains the regional delivery gaps.
- Automate the load and refresh of the tables.

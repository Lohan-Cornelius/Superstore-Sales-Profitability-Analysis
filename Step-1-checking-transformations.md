## Check 1 - Row Counts
To confirm no rows were lost or duplicated during normalisation, I counted the rows in the raw superstoredata table and in fact_table. 
Each raw row is one order line, so the two counts should be identical. The raw table has 9994 rows and the fact table has 9994 rows, 
so the check passed.

```sql
SELECT 
	(SELECT COUNT(*) FROM superstoredata) AS count_source,
	COUNT(*) AS count_fact
FROM fact_table;
```
<img width="571" height="329" alt="image" src="https://github.com/user-attachments/assets/bfcd2ce4-8c2f-41fa-8556-8460c2c93846" />

---

## Check 2 - Totals

Next I compared total sales, total profit and total quantity between the raw table and the fact table, using the exact figures rather than rounded ones so that any precision issues would show up. Raw sales were [X] against [X] in the fact table, raw profit was [X] against [X], and raw quantity was [X] against [X]. [All three totals matched / The totals differed by X, which I traced to Y].

```sql
/*Aggregating Totals from both the Raw - & fact Table*/
WITH totals AS (
SELECT 
	ROUND(SUM(sales), 2)      AS total_sales_raw,
    ROUND(SUM(profit), 2)     AS total_profit_raw,
    SUM(quantity)             AS total_quantity_raw,
    (SELECT 
		ROUND(SUM(sales), 2) FROM fact_table)         AS total_sales_fact,
	(SELECT
		ROUND(SUM(profit), 2) FROM fact_table)        AS total_profit_fact,
	(SELECT 
		SUM(quantity) FROM fact_table)                AS total_quantity_fact
FROM superstoredata
)

/*Returning text whether the totals match across the tables*/
SELECT 
	total_sales_raw,
    total_sales_fact,
	CASE 
		WHEN total_sales_raw = total_sales_fact 
        THEN 'Match / Passed'
        ELSE 'Mismatch / Failed'
        END AS sales_match,
	total_profit_raw,
    total_profit_fact,
	CASE 
		WHEN total_profit_raw = total_profit_fact
        THEN 'Match / Passed'
        ELSE 'Mismatch Failed'
        END AS profit_match,
	total_quantity_raw,
    total_quantity_fact,
	CASE 
		WHEN total_quantity_raw = total_quantity_fact
        THEN 'Match / Passed'
        ELSE 'Mismatch / Failed'
        END AS quantity_match
FROM totals;
```
<img width="571" height="329" alt="image" src="https://github.com/user-attachments/assets/1fa16a06-8c56-49cf-abe7-cbe8e0551015" />

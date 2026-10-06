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

Next I compared total sales, total profit and total quantity between the raw table and the fact table, using the exact figures rather than rounded ones so that any precision issues would show up. Raw sales were 2,297,200.86 against 2,297,200.86 in the fact table, raw profit was 286,397.02 against 286,397.02, and raw quantity was 37,873 against 37,873. All three totals match and Passed the check.

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
        ELSE 'Mismatch / Failed'
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

---

## Check 3 - Orphaned Fact Rows
I checked that every fact row matches a record in each dimension by joining the fact table to all four dimensions and counting the matched keys. 
The fact table has 9,994 rows, and the matched counts for segment, location, product and shipping were (9,994), (9,994), (9,994) and (9,994). All four equalled the fact table's row count, so there were 0 orphaned rows.

```sql
/*This query checks that every fact row matches a record in each dimension table*/
WITH row_id_count AS (
SELECT 
	(SELECT COUNT(*) FROM fact_table) AS total_rows,
	COUNT(dm.segment_id) AS total_segment_id,
    COUNT(dl.location_id) AS total_location_id,
    COUNT(dp.product_id) AS total_product_id,
    COUNT(ds.shipping_id) AS total_shipping_id
FROM fact_table AS ft
    
LEFT JOIN dim_custsegment AS dm
	ON ft.segment_id = dm.segment_id
LEFT JOIN dim_location AS dl
	ON ft.location_id = dl.location_id
LEFT JOIN dim_products AS dp
	ON ft.product_id = dp.product_id
LEFT JOIN dim_shipping AS ds
	ON ft.shipping_id = ds.shipping_id
)

/*Returns text values to easily scan output and see whether the values match or not
  Each id count needs to match with the total_rows*/
SELECT 
	total_rows,
	total_segment_id,
    CASE 
		WHEN total_segment_id != total_rows
        THEN 'Mismatch / Failed'
        ELSE 'Match / Passed'
	END AS segment_match,
    total_location_id,
    CASE 
		WHEN total_location_id != total_rows
        THEN 'Mismatch / Failed'
        ELSE 'Match / Passed'
	END AS location_match,
    total_product_id,
    CASE 
		WHEN total_product_id != total_rows
        THEN 'Mismatch / Failed'
        ELSE 'Match / Passed'
	END AS product_match,
	total_shipping_id,
    CASE 
		WHEN total_shipping_id != total_rows
        THEN 'Mismatch / Failed'
        ELSE 'Match / Passed'
	END AS shipping_match
FROM row_id_count;
```
<img width="571" height="329" alt="image" src="https://github.com/user-attachments/assets/c317f540-4b07-42c1-a293-97899419d210" />

---

## Check 4 - Duplicate Dimension Keys
Duplicate keys in a dimension table would inflate sales and profit totals in any later join, so I checked that every key in each of the four dimension tables appears only once. For each dimension, I compared the total number of rows with the number of distinct keys. The segment dimension has 3 rows and 3 distinct keys, location has 632 and 632, product has 17 and 17, and shipping has 4 and 4. No duplicates were found across all dimensions.

```sql
/*Using UNION ALL to check if the total rows in the dimension tables matches the DISTINCT id per entry*/
SELECT 
	'dim_custsegment' AS table_name,
    COUNT(*) AS total_rows,
    COUNT(DISTINCT segment_id) AS distinct_keys,
	CASE 
		WHEN COUNT(*) = COUNT(DISTINCT segment_id)
        THEN 'Match / Passed'
        ELSE 'Mismatch / Failed'
	END AS test_result
FROM dim_custsegment

UNION ALL

SELECT 
	'dim_location',
    COUNT(*) AS total_rows,
    COUNT(DISTINCT location_id) AS distinct_keys,
	CASE 
		WHEN COUNT(*) = COUNT(DISTINCT location_id)
        THEN 'Match / Passed'
        ELSE 'Mismatch / Failed'
	END
FROM dim_location

UNION ALL

SELECT 
	'dim_products',
    COUNT(*) AS total_rows,
    COUNT(DISTINCT product_id) AS distinct_keys,
	CASE 
		WHEN COUNT(*) = COUNT(DISTINCT product_id)
        THEN 'Match / Passed'
        ELSE 'Mismatch / Failed'
	END
FROM dim_products

UNION ALL

SELECT 
	'dim_shipping',
    COUNT(*) AS total_rows,
    COUNT(DISTINCT shipping_id) AS distinct_keys,
	CASE 
		WHEN COUNT(*) = COUNT(DISTINCT shipping_id)
        THEN 'Match / Passed'
        ELSE 'Mismatch / Failed'
	END
FROM dim_shipping;
```
<img width="571" height="329" alt="image" src="https://github.com/user-attachments/assets/7a2dfe79-a417-4aeb-865c-b06baaaa145c" />

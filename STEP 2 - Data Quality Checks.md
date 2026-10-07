# STEP 2 - Data Quality Checks
Step 1 showed that the star schema matches the raw data. Step 2 is where I checked whether the raw superstoredata table is itself trustworthy enough to analyse. I ran five checks covering missing values, numeric ranges, category consistency, identical rows and margin sanity. The table is clean, and the only things worth flagging are a small set of identical rows and a large share of loss-making sales, which are findings rather than errors.

### What I found

* Check 6, NULLs and blanks: I tested all 13 columns for NULLs and the 8 text columns for blank values. Out of 9,994 rows, I found 0 NULLs and 0 blanks.
* Check 7, Numeric ranges: Sales ranged from 0.44 to 22,638.48, quantity from 1 to 14, and discount from 0 to 0.8. No row had zero or negative sales, a quantity     below 1, or a discount outside 0 to 1.
* Check 8, Category consistency: There are 3 segments, 4 ship modes, 4 regions, 3 categories, 17 sub-categories, 49 states and 1 country. Every sub-category sits    under one category and every state under one region, so group totals will not be split or inflated.
* Check 9, Identical rows: I found 17 groups of identical rows, covering 34 rows, so 17 rows are excess. With no order ID, I can't tell whether they are duplicate   entries or genuine repeat purchases, so I kept them. The excess makes up 0.17% of rows and 0.04% of sales, so keeping them does not change any conclusion.
* Check 10, Margin sanity: No row has profit above sales. 1,871 rows (18.72%) have negative profit, and 349 rows (3.49%) have a margin below -100%, with the worst   row-level margin at -275%. I kept these rows because heavy losses are part of the analysis, not a data error. The overall margin (total profit divided by total    sales) is 12.47%.

### Exclusions and decisions

I excluded no rows. All analysis in the following sections uses the full table of 9,994 rows. The identical rows were kept because there is no order ID to show they are duplicates, and the negative profit rows were kept because loss-making sales are what the analysis is about. The raw superstoredata table is unchanged, so the Step 1 totals still reconcile.

### Data limitations

The raw dataset has no dates, order IDs, customer IDs or product names. This analysis therefore cannot cover growth over time, seasonality, shipping speed, customer concentration or repeat purchasing, and product analysis is at category and sub-category level only.

---

## Check 6 - NULLs and blanks
I checked every column in the raw superstoredata table for missing values, because aggregate functions like SUM and AVG silently skip NULLs and would understate results. I ran two queries. The first tested the numeric columns (sales, quantity, discount and profit) and postal_code, which is stored as an integer, for NULLs only, since a number cannot be an empty string. The second tested the eight text columns (ship mode, segment, country, city, state, region, category and sub-category) for both NULLs and blank values, meaning empty strings or spaces-only entries. Out of 9994 rows, I found 0 NULLs across the numeric columns and postal_code, and 0 NULLs and 0 blanks across the text columns. No missing values were found.

#### Numeric NULLs
```sql
/*Checking Numeric Columns in Source for NULL Values*/
SELECT 
	COUNT(*) AS total_rows,
	(SELECT COUNT(*) FROM superstoredata WHERE sales IS NULL) AS sales_nulls,
    (SELECT COUNT(*) FROM superstoredata WHERE quantity IS NULL) AS quantity_nulls,
    (SELECT COUNT(*) FROM superstoredata WHERE postal_code IS NULL) AS postal_nulls,
    (SELECT COUNT(*) FROM superstoredata WHERE discount IS NULL) AS discount_nulls,
    (SELECT COUNT(*) FROM superstoredata WHERE profit IS NULL) AS profit_nulls
FROM superstoredata;
```
<img width="571" height="329" alt="image" src="https://github.com/user-attachments/assets/7be598a5-7bfe-44cd-8211-9c4f3c3fd564" />


#### Text NULLs & blanks
```sql
/*Checking Text values in Source for NULLs and Blanks*/
SELECT
	COUNT(*) AS total_rows,
	(SELECT COUNT(*) FROM superstoredata WHERE ship_mode IS NULL) AS ship_nulls,
    (SELECT COUNT(*) FROM superstoredata WHERE TRIM(ship_mode) = '') AS ship_blanks,
    (SELECT COUNT(*) FROM superstoredata WHERE segment IS NULL) AS segment_nulls,
    (SELECT COUNT(*) FROM superstoredata WHERE TRIM(segment) = '') AS segment_blanks,
    (SELECT COUNT(*) FROM superstoredata WHERE country IS NULL) AS country_nulls,
    (SELECT COUNT(*) FROM superstoredata WHERE TRIM(country) = '') AS country_blanks,
    (SELECT COUNT(*) FROM superstoredata WHERE city IS NULL) AS city_nulls,
    (SELECT COUNT(*) FROM superstoredata WHERE TRIM(city) = '') AS city_blanks,
    (SELECT COUNT(*) FROM superstoredata WHERE state IS NULL) AS state_nulls,
    (SELECT COUNT(*) FROM superstoredata WHERE TRIM(state) = '') AS state_blanks,
    (SELECT COUNT(*) FROM superstoredata WHERE region IS NULL) AS region_nulls,
    (SELECT COUNT(*) FROM superstoredata WHERE TRIM(region) = '') AS region_blanks,
    (SELECT COUNT(*) FROM superstoredata WHERE category IS NULL) AS category_nulls,
    (SELECT COUNT(*) FROM superstoredata WHERE TRIM(category) = '') AS category_blanks,
    (SELECT COUNT(*) FROM superstoredata WHERE sub_category IS NULL) AS scat_nulls,
    (SELECT COUNT(*) FROM superstoredata WHERE TRIM(sub_category) = '') AS scat_blanks
FROM superstoredata;
```
<img width="571" height="329" alt="image" src="https://github.com/user-attachments/assets/69bd71d4-6db3-4b2e-8e55-a06585596dbb" />

---

## Check 7 - Numeric ranges
I checked the minimum, maximum and average of sales, quantity, discount and profit to look for values that don't make sense, and then counted the rows that break each rule. Discount ranged from 0 to 0.8 (0% to 80%), quantity from 1 to 14, and sales from 0.44 to 22,638.48. Out of 9994 rows, there were 0 rows with zero or negative sales, 0 rows with a quantity below 1, and 0 rows with a discount outside the range 0 to 1. Profit ranged from -6599.98 to 8399.98, with an average of 28.66. 1871 rows (18.72%) had a negative profit and 65 rows had exactly zero profit. I kept the negative profit rows, because loss-making sales are part of the analysis and not a data error. No invalid values were found in sales, quantity or discount, so no rows were excluded.

#### Query 1
```sql
/*Calculating the range (MIN, MAX & AVG) of the numeric fields*/
SELECT
	COUNT(*) AS total_rows,
	MIN(sales) AS min_sales,
    MAX(sales) AS max_sales,
    ROUND(AVG(sales), 2) AS avg_sales,
    MIN(quantity) AS min_quantity,
    MAX(quantity) AS max_quantity,
    ROUND(AVG(quantity), 2) AS avg_quantity,
    MIN(discount) AS min_discount,
    MAX(discount) AS max_discount,
    ROUND(AVG(discount), 2) AS avg_discount,
    MIN(profit) AS min_profit,
    MAX(profit) AS max_profit,
    ROUND(AVG(profit), 2) AS avg_profit
FROM superstoredata;
```
<img width="571" height="329" alt="image" src="https://github.com/user-attachments/assets/0b3f7e1e-8670-4794-932f-577a49e6b4ca" />

#### Query 2
```sql
/*Identifying key aspects of the data where the data isn't necessarily wrong but it is breaking rules*/
SELECT
	COUNT(*) AS total_rows,
    (SELECT COUNT(*) FROM superstoredata WHERE sales <= 0) AS sales_zero_or_below,
    (SELECT COUNT(*) FROM superstoredata WHERE quantity < 1) AS quantity_below_1,
    (SELECT COUNT(*) FROM superstoredata WHERE discount > 1 OR discount < 0) AS discount_out_of_range,
    (SELECT COUNT(*) FROM superstoredata WHERE profit = 0) AS zero_profit_rows,
    (SELECT COUNT(*) FROM superstoredata WHERE profit < 0) AS neg_prof_rows,
    ROUND((SELECT COUNT(*) FROM superstoredata WHERE profit < 0) / (SELECT COUNT(*) FROM superstoredata) * 100, 2) AS neg_prof_pct
FROM superstoredata;
```
<img width="571" height="329" alt="image" src="https://github.com/user-attachments/assets/48e2fea1-116a-406a-b5f6-643394106398" />

---

## Check 8 - Category values and consistency

I counted the distinct values in each text column, both as stored and after trimming spaces. There are 3 segments, 4 ship modes, 4 regions, 3 categories, 17 sub-categories, 49 states and 1 country, and the trimmed and untrimmed counts matched in every column. I also checked the relationships between columns: each sub-category belongs to exactly one category, and each state belongs to exactly one region. I found no conflicts. Country contains only one value, so it adds nothing to the analysis.

#### Query 1
```sql
/*This query is to check unique text values*/
SELECT 
	'shipping' AS column_name,
	COUNT(DISTINCT ship_mode) AS text_distinct,
    COUNT(DISTINCT TRIM(ship_mode)) AS text_trimmed
FROM superstoredata
UNION ALL
SELECT 
	'segment',
	COUNT(DISTINCT segment),
    COUNT(DISTINCT TRIM(segment))
FROM superstoredata
UNION ALL
SELECT 
	'country',
	COUNT(DISTINCT country),
    COUNT(DISTINCT TRIM(country))
FROM superstoredata
UNION ALL
SELECT 
	'city',
	COUNT(DISTINCT city),
    COUNT(DISTINCT TRIM(city))
FROM superstoredata
UNION ALL
SELECT 
	'state',
	COUNT(DISTINCT state),
    COUNT(DISTINCT TRIM(state))
FROM superstoredata
UNION ALL
SELECT 
	'region',
	COUNT(DISTINCT region),
    COUNT(DISTINCT TRIM(region))
FROM superstoredata
UNION ALL
SELECT 
	'category',
	COUNT(DISTINCT category),
    COUNT(DISTINCT TRIM(category))
FROM superstoredata
UNION ALL
SELECT 
	'sub_category',
	COUNT(DISTINCT sub_category),
    COUNT(DISTINCT TRIM(sub_category))
FROM superstoredata;
```
<img width="571" height="329" alt="image" src="https://github.com/user-attachments/assets/8810b2bc-2309-4208-ba9e-57c9f81c3490" />

#### Query 2
```sql
/*Checking that a sub_category belongs to 1 category*/
SELECT 
	sub_category,
    COUNT(DISTINCT category) AS category_count
FROM superstoredata
	GROUP BY
		sub_category
	HAVING category_count > 1;
```
<img width="571" height="329" alt="image" src="https://github.com/user-attachments/assets/e2b071b2-e26f-407b-86b0-a88f6299369c" />

#### Query 3
```sql
/*Checking that a state belongs to 1 region*/
SELECT
	state,
    COUNT(DISTINCT region) AS region_count
FROM superstoredata
	GROUP BY
		state
	HAVING region_count > 1;
```
<img width="571" height="329" alt="image" src="https://github.com/user-attachments/assets/12a06f85-9a55-4440-98cd-094403dc6b59" />

---

## Check 9 - Fully identical rows
Because the table has no order ID, I checked for rows that are identical across every column. I found 17 groups of identical rows, covering 34 rows in total, which is 17 more rows than the number of groups. Without an order ID I can't tell whether these are duplicate entries or genuine repeat purchases of the same item, so I kept them.

```sql
/*Checking for duplicate rows, no unique identifier*/
SELECT 
	COUNT(*) AS row_counts, 
    ship_mode,
	segment,
	country,
	city,
	state,
	postal_code,
	region,
	category,
	sub_category,
	sales,
	quantity,
	discount,
	profit
FROM superstoredata 
	GROUP BY ship_mode, 
			segment,
            country,
            city,
            state,
            postal_code,
            region,
            category,
            sub_category,
            sales,
            quantity,
            discount,
            profit
		HAVING row_counts > 1;
```
<img width="571" height="329" alt="image" src="https://github.com/user-attachments/assets/2209582a-d4a0-4434-9118-d47e4d836623" />

I kept all of the identical rows. The table has no order ID, date or customer ID, so there is no way to tell whether two matching rows are the same sale entered twice or two separate purchases of the same item. A match across all 13 columns, including sales, quantity, discount and profit to four decimal places, is also what you would expect when different customers buy the same low-priced item, and the groups I looked at were mostly small Paper sales. Removing rows I cannot prove are errors would also mean my totals no longer reconcile with the source table. The excess rows make up 0.17% of the table and 0.04% of total sales, so keeping them does not change any conclusion. The raw superstoredata table is unchanged. If the original order data could confirm that these are duplicates, I would remove them and re-run the analysis.

```sql
/*Calculating the percentage of excess rows and sales from the duplicates using the previous query as a CTE*/
WITH duplicates AS (
SELECT 
	COUNT(*) AS row_counts, 
    ship_mode,
	segment,
	country,
	city,
	state,
	postal_code,
	region,
	category,
	sub_category,
	sales,
	quantity,
	discount,
	profit
FROM superstoredata 
	GROUP BY ship_mode, 
			segment,
            country,
            city,
            state,
            postal_code,
            region,
            category,
            sub_category,
            sales,
            quantity,
            discount,
            profit
		HAVING row_counts > 1
	)
    
    SELECT 
		ROUND((SUM((row_counts - 1) * sales) / (SELECT SUM(sales) FROM superstoredata)) * 100, 2) AS dup_pct_of_sales,
        ROUND(SUM(row_counts - 1) / (SELECT COUNT(*) FROM superstoredata) * 100 , 2) AS dup_pct_of_rows
	FROM duplicates;
```
<img width="571" height="329" alt="image" src="https://github.com/user-attachments/assets/57b3ff76-1e24-4e9d-a615-42a7dd0670ad" />

---

## Check 10 - Margin Sanity
I calculated profit margin (profit divided by sales) for every row to look for extreme values. Row-level margins ranged from -275% to 50%. There were 0 rows with a margin above 100%, which would mean profit exceeds sales, and 349 rows (3.49% of table) with a margin below -100%, meaning the sale lost more than it brought in. Sales were above zero in every row (Check 7), so no division by zero was possible. No impossible margins were found. Across the whole table, total profit divided by total sales is 12.47%.
```sql
/*Calculating MIN, MAX and AVG margin percentage*/
SELECT
	COUNT(*) AS total_rows,
	MIN((profit / sales) * 100) AS min_margin_pct,
    ROUND(AVG((profit / sales) * 100), 2) AS avg_margin_pct,
    MAX((profit / sales) * 100) AS max_margin_pct
FROM superstoredata;
```
<img width="571" height="329" alt="image" src="https://github.com/user-attachments/assets/7bf9b1ea-b170-4955-83fe-8680c54d744f" />

```sql
/*Checking for impossible values on the margin level*/
SELECT
	COUNT(*) AS total_rows,
    (SELECT COUNT(*) FROM superstoredata WHERE ((profit / sales) * 100) > 100) AS margin_above_100,
    (SELECT COUNT(*) FROM superstoredata WHERE ((profit / sales) * 100) < -100) AS margin_below_neg100
FROM superstoredata;
```
<img width="571" height="329" alt="image" src="https://github.com/user-attachments/assets/b9c8ae4b-cfe6-4ae5-ac67-017dff1baf81" />

```sql
/*Calculating total margin percentage*/
SELECT 
	ROUND(((SUM(profit) / SUM(sales)) * 100), 2) AS total_margin_pct
FROM superstoredata;
```
<img width="571" height="329" alt="image" src="https://github.com/user-attachments/assets/ee562c4b-7d90-4445-bd77-c40aeb54e7fc" />


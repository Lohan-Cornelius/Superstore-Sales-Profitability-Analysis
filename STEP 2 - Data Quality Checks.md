# STEP 2 - Data Quality Checks.md
Step 1 showed that the star schema matches the raw data. Step 2 asks whether the raw superstoredata table is itself trustworthy enough to analyse. 
I checked for missing values, impossible numbers, inconsistent category labels, identical rows and extreme margins, and recorded what I decided to exclude and why. 
Negative profit is not treated as an error, because loss-making sales are part of the analysis.

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

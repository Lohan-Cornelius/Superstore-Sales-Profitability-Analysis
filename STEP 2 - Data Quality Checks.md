# STEP 2 - Data Quality Checks.md
Step 1 showed that the star schema matches the raw data. Step 2 asks whether the raw superstoredata table is itself trustworthy enough to analyse. 
I checked for missing values, impossible numbers, inconsistent category labels, identical rows and extreme margins, and recorded what I decided to exclude and why. 
Negative profit is not treated as an error, because loss-making sales are part of the analysis.

## Check 6 - NULLs and blanks
I checked every column in the raw superstoredata table for missing values, because aggregate functions like SUM and AVG silently skip NULLs and would understate results. I ran two queries. The first tested the numeric columns (sales, quantity, discount and profit) and postal_code, which is stored as an integer, for NULLs only, since a number cannot be an empty string. The second tested the eight text columns (ship mode, segment, country, city, state, region, category and sub-category) for both NULLs and blank values, meaning empty strings or spaces-only entries. Out of 9994 rows, I found 0 NULLs across the numeric columns and postal_code, and 0 NULLs and 0 blanks across the text columns. No missing values were found.

Numeric NULLs
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

Text NULLs & blanks
```sql
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


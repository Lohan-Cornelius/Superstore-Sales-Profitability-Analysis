## Check 1 - Rown Counts
To confirm no rows were lost or duplicated during normalisation, I counted the rows in the raw superstoredata table and in fact_table. 
Each raw row is one order line, so the two counts should be identical. The raw table has 9994 rows and the fact table has 9994 rows, 
so the check passed.

```
SELECT 
	(SELECT COUNT(*) FROM superstoredata) AS count_source,
	COUNT(*) AS count_fact
FROM fact_table;

```
<img width="571" height="329" alt="image" src="https://github.com/user-attachments/assets/bfcd2ce4-8c2f-41fa-8556-8460c2c93846" />

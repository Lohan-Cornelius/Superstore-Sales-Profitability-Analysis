## Check 1 
* Counting the rows in the raw superstoredata table and the rows in fact_table. 
* They should be identical, because each row in the raw data is one order line. 
* If the numbers differ, note by how much, since that's the first clue to where rows were lost or duplicated.

```
SELECT 
	(SELECT COUNT(*) FROM superstoredata) AS count_source,
	COUNT(*) AS count_fact
FROM fact_table;

```
<img width="571" height="329" alt="image" src="https://github.com/user-attachments/assets/bfcd2ce4-8c2f-41fa-8556-8460c2c93846" />

# Step 3 - Definitions
These definitions apply to every number in the analysis that follows. All figures use the full table of 9,994 rows, and the raw superstoredata table is unchanged.

1. Sales: the sum of the sales column. The dataset has no data dictionary, so I treat it as the revenue recorded for each row, and the currency is not stated. I      use the term "sales" rather than "revenue" throughout.
2. Profit: the sum of the profit column. Negative values are losses.
3. Margin: total profit divided by total sales, expressed as a percentage. The overall margin is 12.47%. For any group, such as a sub-category or region, margin      is that group's total profit divided by that group's total sales. I do not average row-level margins, because that would weight a 0.44 sale the same as a          22,638.48 sale. Row-level margins are used only to find extreme values, as in Check 10.
4. Loss-making row: a row with profit below 0. There are 1,871 of them (18.72% of rows).
5. Loss-making sub-category (or region, segment, ship mode): a group whose total profit is below 0. A group can contain profitable rows and still be loss-making      overall, and the reverse is also true.
6. Discount: stored as a fraction from 0 to 0.8, so 0.2 means 20%. I group discounts into five bands: no discount, above 0% to below 25%, 25% to below 50%, 50% to    below 75%, and 75% and above. No discount is its own band, and every other band includes its lower limit and excludes its upper limit, so a discount of exactly    25% falls in the 25% to below 50% band. The bands hold 4,798, 3,803, 471, 622 and 300 rows, which total 9,994.
7. Share of total: a group's sales (or profit) divided by the table total, as a percentage.
8. Level of analysis: product analysis is at category and sub-category level, location analysis is at region, state and city level, and customer analysis is at       segment level.
9. Time periods: none are defined, because the dataset has no dates.
 
## 6 - Discount Bands
```sql
/*Grouping discounts into five bands: no discount, above 0 to below 25%, 25% to below 50%, 50% to below 75%, and 75% and above. Bands include their lower limit and exclude their upper limit, except no discount (exactly 0) and the top band (no upper limit).*/
SELECT 
	COUNT(*) AS total_rows,
    (SELECT COUNT(*) FROM superstoredata WHERE discount >= 0.75) AS disc_75_100,
    (SELECT COUNT(*) FROM superstoredata WHERE discount >= 0.5 AND discount < 0.75) AS disc_50_75,
    (SELECT COUNT(*) FROM superstoredata WHERE discount >= 0.25 AND discount < 0.5) AS disc_25_50,
	(SELECT COUNT(*) FROM superstoredata WHERE discount > 0 AND discount < 0.25) AS disc_1_25,
    (SELECT COUNT(*) FROM superstoredata WHERE discount = 0) AS no_disc
FROM superstoredata;
```
<img width="571" height="329" alt="image" src="https://github.com/user-attachments/assets/270e01df-1ef3-4acd-82d6-6a1bffda0d48" />

```sql
/*Grouping discounts into five bands: no discount, above 0 to below 25%, 25% to below 50%, 50% to below 75%, and 75% and above. Bands include their lower limit and exclude their upper limit, except no discount (exactly 0) and the top band (no upper limit).*/
WITH bands AS (
SELECT 
	COUNT(*) AS total_rows,
    (SELECT COUNT(*) FROM superstoredata WHERE discount >= 0.75) AS disc_75_100,
    (SELECT COUNT(*) FROM superstoredata WHERE discount >= 0.5 AND discount < 0.75) AS disc_50_75,
    (SELECT COUNT(*) FROM superstoredata WHERE discount >= 0.25 AND discount < 0.5) AS disc_25_50,
	(SELECT COUNT(*) FROM superstoredata WHERE discount > 0 AND discount < 0.25) AS disc_1_25,
    (SELECT COUNT(*) FROM superstoredata WHERE discount = 0) AS no_disc
FROM superstoredata
)

/*Using above CTE to calculate percentage of rows offered across all discounts*/
SELECT 
	ROUND((no_disc / total_rows) * 100, 2) AS pct_no_disc,
    ROUND((disc_1_25 / total_rows) * 100, 2) AS pct_1_25,
    ROUND((disc_25_50 / total_rows) * 100, 2) AS pct_25_50,
    ROUND((disc_50_75 / total_rows) * 100, 2) AS pct_50_75,
    ROUND((disc_75_100 / total_rows) * 100, 2) AS pct_75_100
FROM bands;
```
<img width="571" height="329" alt="image" src="https://github.com/user-attachments/assets/c55a402d-c263-40e0-b518-17efbad2a670" />

Nearly half of all rows (48.01%) have no discount, and a further 38.05% have a discount above 0% and below 25%. Only 13.94% of rows are discounted at 25% or more (4.71% in the 25% to below 50% band, 6.22% in the 50% to below 75% band and 3.00% at 75% and above), which is why I separated the heavier discount bands for the analysis.

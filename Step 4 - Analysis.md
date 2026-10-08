# Step 4 - Analysis

This section answers six business questions using the star schema, joining fact_table to its dimensions. Because Step 1 showed the dimensions have no duplicate keys and no orphaned rows, the joins cannot inflate totals. Every result is reconciled back to the source totals (sales of 2,297,200.86 and profit of 286,397.02), and every margin is total profit divided by total sales, as defined in Step 3. Each finding gives the number, what it means and a recommendation.

## Questions

1. Which categories and sub-categories make the most sales, and which make the most profit? Are they the same? Uses sales, profit, margin and share of total at       category and sub-category level.
2. Which sub-categories lose money, and how much do those losses cost? Uses total profit by sub-category, and compares the losses with the total profit of            286,397.02.
3. How much does discounting hurt profit, and at what discount level do sales turn unprofitable? Uses the five discount bands from Step 3, comparing sales, profit    and margin in each.
4. Which regions and states perform best and worst on sales and on margin? Uses the location dimension, with region first, then state.
5. Do customer segments differ in sales and profitability? Compares Consumer, Corporate and Home Office on sales, profit, margin and average discount. This is        segment level only, not individual customers.
6. Do ship modes differ in sales, profit and discount? Compares the four ship modes.

---

## Question 1: Which categories and sub-categories make the most sales, and which make the most profit? Are they the same?

### Category
I calculated total sales, total profit, margin, share of total sales and share of total profit for each category. By sales, Technology is the largest at 836,154.03 (36.40% of the total), followed by Furniture at 741,999.80 (32.30%) and Office Supplies at 719,047.03 (31.30%). By profit, Technology leads with 145,454.95 (50.79% of the total), followed by Office Supplies at 122,490.80 (42.77%), and Furniture is the weakest at 18,451.27 (6.44%). The rankings differ: Furniture ranks second by sales but last by profit, and its margin of 2.49% is far below the overall margin of 12.47%, while Technology (17.40%) and Office Supplies (17.04%) are both above it. The category totals reconcile to the source totals of 2,297,200.86 sales and 286,397.02 profit.

### Sub Category
At sub-category level, Phones makes the most sales at 330,007.05 (14.37% of the total), while Copiers makes the most profit at 55,617.82 (19.42% of the total) with a margin of 37.20%. Copiers ranks 8th by sales but 1st by profit, and Phones, the top seller, ranks 2nd by profit with a margin of 13.49%.

### What it means: 
The biggest sellers are not the biggest earners. Phones leads on sales but Copiers leads on profit, and Copiers is only 8th by sales, so profit depends more on margin than on sales volume. Furniture is the clearest case of selling a lot at a thin margin: it brings in 32.30% of sales but only 6.44% of profit, with a margin of 2.49% against 12.47% overall. Chairs show the same pattern within it, ranking 2nd by sales but 6th by profit with a margin of only 8.10%. Technology and Office Supplies both earn margins above 17%, so they carry the profit.

### Recommendation: 
Review pricing and discounting in Furniture first, starting with Chairs. Furniture brings in 32.30% of sales but earns a margin of only 2.49%, against 12.47% overall, and Chairs are the second-largest sub-category by sales at 328,449.10 with a margin of just 8.10%. The sub-categories with negative profit, which are also concentrated in Furniture, are examined in Question 2. For scale, if Furniture earned the overall margin, it would add roughly 74,000 in profit, about a quarter of the current total of 286,397.02. This is an illustration of the size of the gap, not a forecast. Protect and grow Technology and Office Supplies, which both earn margins above 17%, and look at whether high-margin, lower-selling sub-categories such as Copiers and Paper deserve more emphasis. I will test the discount link in Question 3 before concluding that discounting is the cause.

```sql
/*Calculating Category totals*/
SELECT 
    dp.category,
    ROUND(SUM(ft.sales), 2) cat_total_sales,
    ROUND(SUM(ft.sales) / (SELECT SUM(sales) FROM fact_table) * 100, 2) AS cat_sales_total_pct,
    ROUND(SUM(ft.profit), 2) AS cat_total_profit,
    ROUND(SUM(ft.profit) / (SELECT SUM(profit) FROM fact_table) * 100, 2) AS cat_profit_total_pct,
    ROUND((SUM(profit) / SUM(sales) * 100), 2) AS cat_margin
FROM
    fact_table AS ft
        LEFT JOIN
    dim_products AS dp ON ft.product_id = dp.product_id
GROUP BY dp.category;
```
<img width="571" height="329" alt="image" src="https://github.com/user-attachments/assets/535f4602-be69-45ba-b847-9192517aec4f" />

```sql
/*Sub Category Totals and ranking based on Sales and Profit*/
SELECT 
    dp.category,
    dp.sub_category,
    ROUND(SUM(ft.sales), 2) cat_total_sales,
    ROUND(SUM(ft.sales) / (SELECT SUM(sales) FROM fact_table) * 100, 2) AS subcat_sales_total_pct,
    ROUND(SUM(ft.profit), 2) AS subcat_total_profit,
    ROUND(SUM(ft.profit) / (SELECT SUM(profit) FROM fact_table) * 100, 2) AS subcat_profit_total_pct,
    ROUND((SUM(profit) / SUM(sales) * 100), 2) AS subcat_margin,
    RANK() OVER(ORDER BY SUM(ft.sales) DESC) AS sales_rank,
    RANK() OVER(ORDER BY SUM(ft.profit)DESC) AS profit_rank
FROM
    fact_table AS ft
		LEFT JOIN
			dim_products AS dp 
            ON ft.product_id = dp.product_id
GROUP BY dp.category, dp.sub_category;
```
<img width="650" height="400" alt="image" src="https://github.com/user-attachments/assets/95ce8d91-9f6b-4ef1-94a6-17b65c9e8a15" />

---

## Question 2: Which sub-categories lose money, and how much do those losses cost?

Using the sub-category results, I identified the sub-categories whose total profit is below 0, which is my definition of loss-making from Step 3. 3 of the 17 sub-categories are loss-making: Supplies, Bookcases and Tables. Together they lose 22,387.14 on sales of 368,519.07 (16.04% of total sales), a combined margin of -6.07%. The largest loss is Tables at 17,725.48 on sales of 206,965.53 (margin -8.56%), which is 79.18% of all the losses. Without these losses, total profit would be 308,784.16 instead of 286,397.02, which is 7.82% higher. Tables makes a loss despite ranking 4th by sales, so it is a popular but loss-making line. Two of the three loss-making sub-categories, Tables and Bookcases, sit in Furniture, and Supplies sits in Office Supplies.

### What it means: 
The losses are modest against total profit but concentrated. The three loss-making sub-categories lose 22,387.14 combined, which is 7.82% of total profit of 286,397.02, and Tables alone accounts for 79.18% of that. Two of the three (Tables and Bookcases) sit in Furniture, and they explain its thin margin. Furniture's profit of 18,451.27 is Chairs (26,590.17) and Furnishings (13,059.14) less the 21,198.04 lost on Tables and Bookcases. Without those two sub-categories, Furniture's margin would be 9.44% instead of 2.49%, so the category's problem is two sub-categories, not the whole category.

### Recommendation: 
Review Tables first. It ranks 4th of 17 sub-categories by sales, loses 8.56% of every sale, and its loss of 17,725.48 is almost as large as all of Furniture's profit (18,451.27).
```sql
/*Sub Category Totals based on Sales and Profit where the sub category profit is less than 0*/
SELECT 
    dp.category,
    dp.sub_category,
    ROUND(SUM(ft.sales), 2) cat_total_sales,
    ROUND(SUM(ft.sales) / (SELECT SUM(sales) FROM fact_table) * 100, 2) AS subcat_sales_total_pct,
    ROUND(SUM(ft.profit), 2) AS subcat_total_profit,
    ROUND(SUM(ft.profit) / (SELECT SUM(profit) FROM fact_table) * 100, 2) AS subcat_profit_total_pct,
    ROUND((SUM(profit) / SUM(sales) * 100), 2) AS subcat_margin
FROM fact_table AS ft
		LEFT JOIN
			dim_products AS dp 
            ON ft.product_id = dp.product_id
GROUP BY dp.category, dp.sub_category
		HAVING subcat_total_profit < 0;
```

<img width="571" height="400" alt="image" src="https://github.com/user-attachments/assets/d4765eeb-eea2-48a3-9635-c849f03efd58" />

---

## Question 3: How much does discounting hurt profit, and at what discount level do sales turn unprofitable?

I grouped every row into the five discount bands from Step 3 and calculated rows, sales, profit, margin, share of total sales, share of total profit and the percentage of loss-making rows (profit below 0) for each. Rows with no discount (4,798 rows) account for 47.36% of sales at a margin of 29.51%, and none of them lose money. Rows with a discount above 0% and below 25% (3,803 rows) have a margin of 11.91%, and 13.75% of them lose money. Margin turns negative in the 25% to below 50% band (471 rows) at -15.99%, with 90.45% of rows loss-making. It falls to -62.65% in the 50% to below 75% band (622 rows) and -180.03% in the 75% and above band (300 rows), where every row loses money. The bands reconcile to 9,994 rows, 2,297,200.86 in sales and 286,397.02 in profit.

Shares of total profit can exceed 100% for profitable groups and be negative for loss-making ones, because total profit is the net of both. The no-discount rows earn 112.08% of total profit because the three heaviest bands lose 13.38%, 23.23% and 10.66% between them.

### Exact discount levels

I then grouped by the exact discount value. Margin is positive at 10% (16.61%, 94 rows), 15% (5.15%, 52 rows) and 20% (11.82%, 3,657 rows). It is negative at 30% (-10.05%, 227 rows) and at every level above it: -16.50% at 32%, -19.81% at 40%, -45.45% at 45%, -34.80% at 50%, -89.46% at 60%, -98.66% at 70% and -180.03% at 80%. No rows sit between 20% and 30%, so the turning point lies somewhere in that gap and the data can't place it more precisely. Several levels hold few rows (11 at 45%, 27 at 32%, 52 at 15%), so their individual margins should not be over-read.

### Where the heavy discounts sit

The 1,393 rows with a discount of 30% or more are 13.94% of rows and 15.79% of sales, but they lose 135,376.06 combined, a margin of -37.32%. Of the 1,871 loss-making rows in the table, 1,348 (72.05%) carry a discount of 30% or more, and the other 523 carry a smaller discount. The losses are concentrated: Binders (38,510.50), Tables (30,698.22) and Machines (29,555.35) account for 72.96% of them. The four Furniture sub-categories (Furnishings, Chairs, Bookcases and Tables) hold 542 of the heavily discounted rows and 54,477.76 (40.24%) of the loss. Binders is profitable overall (30,221.76), so heavy discounting is cutting into an otherwise profitable line. Tables and Bookcases lose money overall in Question 2, but their heavily discounted rows lose more than their total losses (30,698.22 against 17,725.48 for Tables, and 11,097.76 against 3,472.56 for Bookcases), so their remaining rows are profitable. Supplies has no rows at 30% or above, so heavy discounting does not explain its loss. Not every heavily discounted row loses money: the 9 Copiers rows at 40% earn a margin of 12.90%.

### What it means: 
Discounting is where the profit goes. No row sold without a discount loses money, and from 30% upwards most rows do. The 13.94% of rows at 30% or above lose an amount equal to nearly half of the profit the whole business earns (135,376.06 against 286,397.02). This is about six times the 22,387.14 lost across the three loss-making sub-categories in Question 2, so discount level is a bigger source of loss than sub-category choice. It also explains most of Furniture's thin margin from Question 1. The data shows an association, not a cause: it cannot show what prices or volumes would have been without the discounts, and Step 3 notes that it is unclear whether sales is before or after discount.

### Recommendation: 
Put an approval step on any discount above 20%, the highest level in this data where margin is still positive, and review discounts of 30% or more first on Binders, Tables and Machines. Treat the 135,376.06 as the size of the problem, not as profit that can be recovered, because cutting discounts could also reduce volumes. Review Supplies separately, since discounting does not explain its loss.

```sql
/*Grouping rows into the five discount bands and calculating sales, profit, margin and loss-making rows per band*/
SELECT
	COUNT(*) AS total_rows,
    ROUND(SUM(ft.sales), 2) AS total_sales,
    ROUND(SUM(ft.sales) / (SELECT SUM(sales) FROM fact_table) * 100, 2) AS total_pct_of_sales,
    ROUND(SUM(ft.profit), 2) AS total_profit,
    ROUND(SUM(ft.profit) / (SELECT SUM(profit) FROM fact_table) * 100, 2) AS total_pct_of_profit,
    ROUND((SUM(CASE WHEN ft.profit <  0 THEN 1 ELSE 0 END) / COUNT(*) * 100), 2) AS pct_profit_below_0,
    ROUND((SUM(profit) / SUM(sales) * 100), 2) AS band_margin,
    CASE 
		WHEN discount >= 0.75 THEN '5. Upper Band (75% and above)'
        WHEN discount >= 0.5 AND discount < 0.75 THEN '4. Upper Middle Band (50 - 75%)'
        WHEN discount >= 0.25 AND discount < 0.5 THEN '3. Lower Middle Band (25 - 50%)'
        WHEN discount > 0 AND discount < 0.25 THEN '2. Lower Band(1 - 25%)'
        ELSE '1. No Discount (0%)'
        END AS disc_band
FROM fact_table AS ft
	GROUP BY disc_band
		ORDER BY disc_band;
```

<img width="750" height="400" alt="image" src="https://github.com/user-attachments/assets/f1201304-05bc-4afe-981a-ae8c479fddcb" />

```sql
/*Grouping rows by the discount amount and calculating sales, profit, margin and loss-making rows per band */
SELECT
	COUNT(*) AS total_rows,
    ROUND(SUM(ft.sales), 2) AS total_sales,
    ROUND(SUM(ft.sales) / (SELECT SUM(sales) FROM fact_table) * 100, 2) AS total_pct_of_sales,
    ROUND(SUM(ft.profit), 2) AS total_profit,
    ROUND(SUM(ft.profit) / (SELECT SUM(profit) FROM fact_table) * 100, 2) AS total_pct_of_profit,
    ROUND((SUM(CASE WHEN ft.profit <  0 THEN 1 ELSE 0 END) / COUNT(*) * 100), 2) AS pct_profit_below_0,
    ROUND((SUM(profit) / SUM(sales) * 100), 2) AS band_margin,
    (discount * 100)AS discount_pct,
    CASE 
		WHEN discount >= 0.75 THEN '5. Upper Band (75% and above)'
        WHEN discount >= 0.5 AND discount < 0.75 THEN '4. Upper Middle Band (50 - 75%)'
        WHEN discount >= 0.25 AND discount < 0.5 THEN '3. Lower Middle Band (25 - 50%)'
        WHEN discount > 0 AND discount < 0.25 THEN '2. Lower Band(1 - 25%)'
        ELSE '1. No Discount (0%)'
        END AS disc_band
FROM fact_table AS ft
	GROUP BY discount
		ORDER BY discount
```

<img width="750" height="400" alt="image" src="https://github.com/user-attachments/assets/ae5ca435-191b-46b7-9554-48be274cb453" />

```sql
/*Heavily discounted rows (30% and above) by sub-category: rows, sales, profit, margin and share of rows that lose money*/
SELECT
	COUNT(*) AS total_rows,
    dp.category,
    dp.sub_category,
    ROUND(SUM(ft.sales), 2) AS total_sales,
    ROUND(SUM(ft.sales) / (SELECT SUM(sales) FROM fact_table) * 100, 2) AS total_pct_of_sales,
    ROUND(SUM(ft.profit), 2) AS total_profit,
    ROUND(SUM(ft.profit) / (SELECT SUM(profit) FROM fact_table) * 100, 2) AS total_pct_of_profit,
    ROUND((SUM(CASE WHEN ft.profit <  0 THEN 1 ELSE 0 END) / COUNT(*) * 100), 2) AS pct_profit_below_0,
    ROUND((SUM(profit) / SUM(sales) * 100), 2) AS margin
FROM fact_table AS ft
	JOIN dim_products AS dp
		ON ft.product_id = dp.product_id
	WHERE ft.discount >= 0.30
		GROUP BY category, sub_category
			ORDER BY total_profit DESC;
```

<img width="750" height="400" alt="image" src="https://github.com/user-attachments/assets/8be4f429-da4f-4458-af6b-cc1e33ebca1d" />

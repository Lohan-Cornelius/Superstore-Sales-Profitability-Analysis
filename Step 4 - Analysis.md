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
Review Tables first. It is among the top sellers by sales, it ranks 4th of 17 sub-categories by sales loses 8.56% of every sale, and its loss of 17,725.48 is almost as large as all of Furniture's profit. Then review Bookcases and Supplies, which lose smaller amounts. I will test in Question 3 whether heavy discounting is behind these losses before recommending a pricing change.

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

I grouped every row into the five discount bands defined in Step 3 and calculated rows, total sales, total profit, margin, share of total sales, share of total profit and the percentage of loss-making rows for each band. Rows with no discount made [X] of sales at a margin of [X]%, and rows with a discount above 0% and below 25% made [X] at [X]%. Margin turns negative in the [band] band, where [X] rows made [X] in sales and lost [X], a margin of [X]%. The three bands at 25% and above hold [X]% of rows but account for [X]% of sales, and together lose [X]. [X]% of rows in the 25% to below 50% band are loss-making, against [X]% of rows with no discount. The five bands hold 4,798, 3,803, 471, 622 and 300 rows, which total 9,994, and their sales and profit reconcile to 2,297,200.86 and 286,397.02. [Check the exact discount values: the finer breakdown shows the turning point at [X]%.]

### What it means: 
[one or two sentences: where profit falls away, whether the heavy bands lose money outright, and how much profit the heavy bands cost against the no-discount rows].

### Recommendation: 
[one action tied to the numbers, such as a discount cap or approval threshold at the level where margin turns negative].

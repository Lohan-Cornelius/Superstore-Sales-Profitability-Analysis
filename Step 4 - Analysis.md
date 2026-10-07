# Step 4 - Analysis

This section answers six business questions using the star schema, joining fact_table to its dimensions. Because Step 1 showed the dimensions have no duplicate keys and no orphaned rows, the joins cannot inflate totals. Every result is reconciled back to the source totals (sales of 2,297,200.86 and profit of 286,397.02), and every margin is total profit divided by total sales, as defined in Step 3. Each finding gives the number, what it means and a recommendation.

## Questions

1. Which categories and sub-categories make the most sales, and which make the most profit? Are they the same? Uses sales, profit, margin and share of total at          category and sub-category level.
2. Which sub-categories lose money, and how much do those losses cost? Uses total profit by sub-category, and compares the losses with the total profit of 286,397.02.
3. How much does discounting hurt profit, and at what discount level do sales turn unprofitable? Uses the five discount bands from Step 3, comparing sales, profit       and margin in each. This is likely your headline finding.
4. Which regions and states perform best and worst on sales and on margin? Uses the location dimension, with region first, then state. You can go down to city if        something stands out.
5. Do customer segments differ in sales and profitability? Compares Consumer, Corporate and Home Office on sales, profit, margin and average discount. This is           segment level only, not individual customers.
6. Do ship modes differ in sales, profit and discount? Compares the four ship modes. You can't compare speed or delays, because there are no dates.

---

## Question 1: Which categories and sub-categories make the most sales, and which make the most profit? Are they the same?

### Category
I calculated total sales, total profit, margin, share of total sales and share of total profit for each category. By sales, Technology is the largest at 836,154.03 (36.40% of the total), followed by Furniture at 741,999.80 (32.30%) and Office Supplies at 719,047.03 (31.30%). By profit, Technology leads with 145,454.95 (50.79% of the total), followed by Office Supplies at 122,490.80 (42.77%), and Furniture is the weakest at 18,451.27 (6.44%). The rankings differ: Furniture ranks second by sales but last by profit, and its margin of 2.49% is far below the overall margin of 12.47%, while Technology (17.40%) and Office Supplies (17.04%) are both above it. The category totals reconcile to the source totals of 2,297,200.86 sales and 286,397.02 profit.

### Sub Category
At sub-category level, Phones makes the most sales at 330,007.05 (14.37% of the total), while Copiers makes the most profit at 55,617.82 (19.42% of the total) with a margin of 37.20%. Copiers ranks 8th by sales but 1st by profit, and Phones, the top seller, ranks 2nd by profit with a margin of 13.49%.

What it means: [one or two sentences, e.g. whether the biggest sellers are also the most profitable, and whether any group sells a lot at a thin or negative margin].

Recommendation: [one action tied to the numbers, e.g. where to push, and where to review pricing or discounts].

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
GROUP BY dp.category
```
<img width="571" height="329" alt="image" src="https://github.com/user-attachments/assets/535f4602-be69-45ba-b847-9192517aec4f" />

```sql
/*Sub Category Totals and ranking based on Sales and Profit*/
SELECT 
    dp.category,
    dp.sub_category,
    ROUND(SUM(ft.sales), 2) cat_total_sales,
    ROUND(SUM(ft.sales) / (SELECT SUM(sales) FROM fact_table) * 100, 2) AS cat_sales_total_pct,
    ROUND(SUM(ft.profit), 2) AS cat_total_profit,
    ROUND(SUM(ft.profit) / (SELECT SUM(profit) FROM fact_table) * 100, 2) AS cat_profit_total_pct,
    ROUND((SUM(profit) / SUM(sales) * 100), 2) AS cat_margin,
    RANK() OVER(ORDER BY SUM(ft.sales) DESC) AS sales_rank,
    RANK() OVER(ORDER BY SUM(ft.profit)DESC) AS profit_rank
FROM
    fact_table AS ft
		LEFT JOIN
			dim_products AS dp 
            ON ft.product_id = dp.product_id
GROUP BY dp.category, dp.sub_category
```
<img width="650" height="400" alt="image" src="https://github.com/user-attachments/assets/05d068b2-72b5-49c0-abbb-b6d8ef39544c" />


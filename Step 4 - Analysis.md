# Step 4 - Analysis

This section answers six business questions using the star schema, joining fact_table to its dimensions. Because Step 1 showed the dimensions have no duplicate keys and no orphaned rows, the joins cannot inflate totals. Every result is reconciled back to the source totals (sales of 2,297,200.86 and profit of 286,397.02), and every margin is total profit divided by total sales, as defined in Step 3. Each finding gives the number, what it means and a recommendation.

## Questions

1. Which categories and sub-categories make the most sales, and which make the most profit? Are they the same? Uses sales, profit, margin and share of total at          category and sub-category level.
2. Which sub-categories lose money, and how much do those losses cost? Uses total profit by sub-category, and compares the losses with the total profit of 286,397.02.
3. How much does discounting hurt profit, and at what discount level do sales turn unprofitable? Uses the five discount bands from Step 3, comparing sales, profit       and margin in each. This is likely your headline finding.
4. Which regions and states perform best and worst on sales and on margin? Uses the location dimension, with region first, then state. You can go down to city if        something stands out.
5. Do customer segments differ in sales and profitability? Compares Consumer, Corporate and Home Office on sales, profit, margin and average discount. This is           segment level only, not individual customers.
6. Do ship modes differ in sales, profit and discount? Compares the four ship modes. You can't compare speed or delays, because there are no dates.

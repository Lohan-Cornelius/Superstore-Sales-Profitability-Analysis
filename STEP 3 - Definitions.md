# Step 3: Definitions
These definitions apply to every number in the analysis that follows. All figures use the full table of 9,994 rows, and the raw superstoredata table is unchanged.

1. Sales: the sum of the sales column. The dataset has no data dictionary, so I treat it as the revenue recorded for each row, and the currency is not stated. I use the term "sales" rather than "revenue" throughout.
2. Profit: the sum of the profit column. Negative values are losses.
3. Margin: total profit divided by total sales, expressed as a percentage. The overall margin is 12.47%. For any group, such as a sub-category or region, margin is that group's total profit divided by that group's total sales. I do not average row-level margins, because that would weight a 0.44 sale the same as a 22,638.48 sale. Row-level margins are used only to find extreme values, as in Check 10.
4. Loss-making row: a row with profit below 0. There are 1,871 of them (18.72% of rows).
5. Loss-making sub-category (or region, segment, ship mode): a group whose total profit is below 0. A group can contain profitable rows and still be loss-making overall, and the reverse is also true.
6. Discount: stored as a fraction from 0 to 0.8, so 0.2 means 20%. I group discounts into bands: [list your bands, e.g. no discount, up to 20%, 21% to 40%, above 40%].
7. Share of total: a group's sales (or profit) divided by the table total, as a percentage.
8. Level of analysis: product analysis is at category and sub-category level, location analysis is at region, state and city level, and customer analysis is at segment level.
9. Time periods: none are defined, because the dataset has no dates.

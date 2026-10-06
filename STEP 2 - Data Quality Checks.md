# STEP 2 - Data Quality Checks.md
Step 1 showed that the star schema matches the raw data. Step 2 asks whether the raw superstoredata table is itself trustworthy enough to analyse. 
I checked for missing values, impossible numbers, inconsistent category labels, identical rows and extreme margins, and recorded what I decided to exclude and why. 
Negative profit is not treated as an error, because loss-making sales are part of the analysis.

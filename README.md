# Projects 
my sql project based on the 8 week sql challenge online.

Question and Solution

-Data Exploration
1. What day of the week is used for each week_date value?

SELECT DISTINCT(TO_CHAR(week_date, 'day')) AS week_day 
FROM clean_weekly_sales;
Answer:

week_day
monday
Monday is used for the week_date value.
2. What range of week numbers are missing from the dataset?

First, generate a range of week numbers for the entire year from 1st week to the 52nd week using the GENERATE_SERIES() function.
Then, perform a LEFT JOIN with the clean_weekly_sales. Ensure that the join sequence is the CTE followed by the clean_weekly_sales as reversing the sequence would result in null results (unless you opt for a RIGHT JOIN instead!).
WITH week_number_cte AS (
  SELECT GENERATE_SERIES(1,52) AS week_number
)
  
SELECT DISTINCT week_no.week_number
FROM week_number_cte AS week_no
LEFT JOIN clean_weekly_sales AS sales
  ON week_no.week_number = sales.week_number
WHERE sales.week_number IS NULL; -- Filter to identify the missing week numbers where the values are `NULL`.
Answer:

I'm posting only the results of 5 rows here. Ensure that you have retrieved 28 rows!

week_number
1
2
3
37
41
The dataset is missing a total of 28 week_number records.
3. How many total transactions were there for each year in the dataset?

SELECT 
  calendar_year, 
  SUM(transactions) AS total_transactions
FROM clean_weekly_sales
GROUP BY calendar_year
ORDER BY calendar_year;
Answer:

calendar_year	total_transactions
2018	346406460
2019	365639285
2020	375813651
4. What is the total sales for each region for each month?

SELECT 
  month_number, 
  region, 
  SUM(sales) AS total_sales
FROM clean_weekly_sales
GROUP BY month_number, region
ORDER BY month_number, region;
Answer:

I'm only showing the results for the month of March.

month_number	region	total_sales
3	AFRICA	567767480
3	ASIA	529770793
3	CANADA	144634329
3	EUROPE	35337093
3	OCEANIA	783282888
3	SOUTH AMERICA	71023109
3	USA	225353043
5. What is the total count of transactions for each platform?

SELECT 
  platform, 
  SUM(transactions) AS total_transactions
FROM clean_weekly_sales
GROUP BY platform;
Answer:

platform	total_transactions
Retail	1081934227
Shopify	5925169
6. What is the percentage of sales for Retail vs Shopify for each month?

WITH monthly_transactions AS (
  SELECT 
    calendar_year, 
    month_number, 
    platform, 
    SUM(sales) AS monthly_sales
  FROM clean_weekly_sales
  GROUP BY calendar_year, month_number, platform
)

SELECT 
  calendar_year, 
  month_number, 
  ROUND(100 * MAX 
    (CASE 
      WHEN platform = 'Retail' THEN monthly_sales ELSE NULL END) 
    / SUM(monthly_sales),2) AS retail_percentage,
  ROUND(100 * MAX 
    (CASE 
      WHEN platform = 'Shopify' THEN monthly_sales ELSE NULL END)
    / SUM(monthly_sales),2) AS shopify_percentage
FROM monthly_transactions
GROUP BY calendar_year, month_number
ORDER BY calendar_year, month_number;
Answer:

Although I am only displaying the rows for the year 2018, please note that the overall results consist of 20 rows.

calendar_year	month_number	retail_percentage	shopify_percentage
2018	3	97.92	2.08
2018	4	97.93	2.07
2018	5	97.73	2.27
2018	6	97.76	2.24
2018	7	97.75	2.25
2018	8	97.71	2.29
2018	9	97.68	2.32
7. What is the percentage of sales by demographic for each year in the dataset?

WITH demographic_sales AS (
  SELECT 
    calendar_year, 
    demographic, 
    SUM(sales) AS yearly_sales
  FROM clean_weekly_sales
  GROUP BY calendar_year, demographic
)

SELECT 
  calendar_year, 
  ROUND(100 * MAX 
    (CASE 
      WHEN demographic = 'Couples' THEN yearly_sales ELSE NULL END)
    / SUM(yearly_sales),2) AS couples_percentage,
  ROUND(100 * MAX 
    (CASE 
      WHEN demographic = 'Families' THEN yearly_sales ELSE NULL END)
    / SUM(yearly_sales),2) AS families_percentage,
  ROUND(100 * MAX 
    (CASE 
      WHEN demographic = 'unknown' THEN yearly_sales ELSE NULL END)
    / SUM(yearly_sales),2) AS unknown_percentage
FROM demographic_sales
GROUP BY calendar_year;
Answer:

calendar_year	couples_percentage	families_percentage	unknown_percentage
2019	27.28	32.47	40.25
2018	26.38	31.99	41.63
2020	28.72	32.73	38.55
8. Which age_band and demographic values contribute the most to Retail sales?

SELECT 
  age_band, 
  demographic, 
  SUM(sales) AS retail_sales,
  ROUND(100 * 
    SUM(sales)::NUMERIC 
    / SUM(SUM(sales)) OVER (),
  1) AS contribution_percentage
FROM clean_weekly_sales
WHERE platform = 'Retail'
GROUP BY age_band, demographic
ORDER BY retail_sales DESC;
Answer:

age_band	demographic	retail_sales	contribution_percentage
unknown	unknown	16067285533	40.5
Retirees	Families	6634686916	16.7
Retirees	Couples	6370580014	16.1
Middle Aged	Families	4354091554	11.0
Young Adults	Couples	2602922797	6.6
Middle Aged	Couples	1854160330	4.7
Young Adults	Families	1770889293	4.5
The majority of the highest retail sales accounting for 42% are contributed by unknown age_band and demographic. This is followed by retired families at 16.73% and retired couples at 16.07%.

9. Can we use the avg_transaction column to find the average transaction size for each year for Retail vs Shopify? If not - how would you calculate it instead?

SELECT 
  calendar_year, 
  platform, 
  ROUND(AVG(avg_transaction),0) AS avg_transaction_row, 
  SUM(sales) / sum(transactions) AS avg_transaction_group
FROM clean_weekly_sales
GROUP BY calendar_year, platform
ORDER BY calendar_year, platform;
Answer:

calendar_year	platform	avg_transaction_row	avg_transaction_group
2018	Retail	43	36
2018	Shopify	188	192
2019	Retail	42	36
2019	Shopify	178	183
2020	Retail	41	36
2020	Shopify	175	179
The difference between avg_transaction_row and avg_transaction_group is as follows:

avg_transaction_row calculates the average transaction size by dividing the sales of each row by the number of transactions in that row.
On the other hand, avg_transaction_group calculates the average transaction size by dividing the total sales for the entire dataset by the total number of transactions.
For finding the average transaction size for each year by platform accurately, it is recommended to use avg_transaction_group.

🧼 C. Before & After Analysis
This technique is usually used when we inspect an important event and want to inspect the impact before and after a certain point in time.

Taking the week_date value of 2020-06-15 as the baseline week where the Data Mart sustainable packaging changes came into effect. We would include all week_date values for 2020-06-15 as the start of the period after the change and the previous week_date values would be before.

Using this analysis approach - answer the following questions:

1. What is the total sales for the 4 weeks before and after 2020-06-15? What is the growth or reduction rate in actual values and percentage of sales?

Before we proceed, we determine the the week_number corresponding to '2020-06-15' to use it as a filter in our analysis.

SELECT DISTINCT week_number
FROM clean_weekly_sales
WHERE week_date = '2020-06-15' 
  AND calendar_year = '2020';
week_number
25
The week_number is 25. I created 2 CTEs:

packaging_sales CTE: Filter the dataset for 4 weeks before and after 2020-06-15 and calculate the sum of sales within the period.
before_after_changes CTE: Utilize a CASE statement to capture the sales for 4 weeks before and after 2020-06-15 and then calculate the total sales for the specified period.
WITH packaging_sales AS (
  SELECT 
    week_date, 
    week_number, 
    SUM(sales) AS total_sales
  FROM clean_weekly_sales
  WHERE (week_number BETWEEN 21 AND 28) 
    AND (calendar_year = 2020)
  GROUP BY week_date, week_number
)
, before_after_changes AS (
  SELECT 
    SUM(CASE 
      WHEN week_number BETWEEN 21 AND 24 THEN total_sales END) AS before_packaging_sales,
    SUM(CASE 
      WHEN week_number BETWEEN 25 AND 28 THEN total_sales END) AS after_packaging_sales
  FROM packaging_sales
)

SELECT 
  after_packaging_sales - before_packaging_sales AS sales_variance, 
  ROUND(100 * 
    (after_packaging_sales - before_packaging_sales) 
    / before_packaging_sales,2) AS variance_percentage
FROM before_after_changes;
Answer:

sales_variance	variance_percentage
-26884188	-1.15
Since the implementation of the new sustainable packaging, there has been a decrease in sales amounting by $26,884,188 reflecting a negative change at 1.15%. Introducing a new packaging does not always guarantee positive results as customers may not readily recognise your product on the shelves due to the change in packaging.

2. What about the entire 12 weeks before and after?

We can apply a similar approach and solution to address this question.

WITH packaging_sales AS (
  SELECT 
    week_date, 
    week_number, 
    SUM(sales) AS total_sales
  FROM clean_weekly_sales
  WHERE (week_number BETWEEN 13 AND 37) 
    AND (calendar_year = 2020)
  GROUP BY week_date, week_number
)
, before_after_changes AS (
  SELECT 
    SUM(CASE 
      WHEN week_number BETWEEN 13 AND 24 THEN total_sales END) AS before_packaging_sales,
    SUM(CASE 
      WHEN week_number BETWEEN 25 AND 37 THEN total_sales END) AS after_packaging_sales
  FROM packaging_sales
)

SELECT 
  after_packaging_sales - before_packaging_sales AS sales_variance, 
  ROUND(100 * 
    (after_packaging_sales - before_packaging_sales) / before_packaging_sales,2) AS variance_percentage
FROM before_after_changes;
Answer:

sales_variance	variance_percentage
-152325394	-2.14
Looks like the sales have experienced a further decline, now at a negative 2.14%! If I'm Danny's boss, I wouldn't be too happy with the results.

3. How do the sale metrics for these 2 periods before and after compare with the previous years in 2018 and 2019?

I'm breaking down this question to 2 parts.

Part 1: How do the sale metrics for 4 weeks before and after compare with the previous years in 2018 and 2019?

Basically, the question is asking us to find the sales variance between 4 weeks before and after '2020-06-15' for years 2018, 2019 and 2020. Perhaps we can find a pattern here.
We can apply the same solution as above and add calendar_year into the syntax.
WITH changes AS (
  SELECT 
    calendar_year,
    week_number, 
    SUM(sales) AS total_sales
  FROM clean_weekly_sales
  WHERE week_number BETWEEN 21 AND 28
  GROUP BY calendar_year, week_number
)
, before_after_changes AS (
  SELECT 
    calendar_year,
    SUM(CASE 
      WHEN week_number BETWEEN 13 AND 24 THEN total_sales END) AS before_packaging_sales,
    SUM(CASE 
      WHEN week_number BETWEEN 25 AND 28 THEN total_sales END) AS after_packaging_sales
  FROM changes
  GROUP BY calendar_year
)

SELECT 
  calendar_year, 
  after_packaging_sales - before_packaging_sales AS sales_variance, 
  ROUND(100 * 
    (after_packaging_sales - before_packaging_sales) 
    / before_packaging_sales,2) AS variance_percentage
FROM before_after_changes;
Answer:

calendar_year	sales_variance	variance_percentage
2018	4102105	0.19
2019	2336594	0.10
2020	-26884188	-1.15
In 2018, there was a sales variance of $4,102,105, indicating a positive change of 0.19% compared to the period before the packaging change.

Similarly, in 2019, there was a sales variance of $2,336,594, corresponding to a positive change of 0.10% when comparing the period before and after the packaging change.

However, in 2020, there was a substantial decrease in sales following the packaging change. The sales variance amounted to $26,884,188, indicating a significant negative change of -1.15%. This reduction represents a considerable drop compared to the previous years.

Part 2: How do the sale metrics for 12 weeks before and after compare with the previous years in 2018 and 2019?

Use the same solution above and change to week 13 to 24 for before and week 25 to 37 for after.
WITH changes AS (
  SELECT 
    calendar_year, 
    week_number, 
    SUM(sales) AS total_sales
  FROM clean_weekly_sales
  WHERE week_number BETWEEN 13 AND 37
  GROUP BY calendar_year, week_number
)
, before_after_changes AS (
  SELECT 
    calendar_year,
    SUM(CASE 
      WHEN week_number BETWEEN 13 AND 24 THEN total_sales END) AS before_packaging_sales,
    SUM(CASE 
      WHEN week_number BETWEEN 25 AND 37 THEN total_sales END) AS after_packaging_sales
  FROM changes
  GROUP BY calendar_year
)

SELECT 
  calendar_year, 
  after_packaging_sales - before_packaging_sales AS sales_variance, 
  ROUND(100 * 
    (after_packaging_sales - before_packaging_sales) 
    / before_packaging_sales,2) AS variance_percentage
FROM before_after_changes;
Answer:

calendar_year	sales_variance	variance_percentage
2018	104256193	1.63
2019	-20740294	-0.30
2020	-152325394	-2.14
There was a fair bit of percentage differences in all 3 years. However, now when you compare the worst year to their best year in 2018, the sales percentage difference is even more stark at a difference of 3.77% (1.63% + 2.14%).

When comparing the sales performance across all three years, there were noticeable variations in the percentage differences. However, the most significant contrast emerges when comparing the worst-performing year in 2020 to the best-performing year in 2018. In this comparison, the sales percentage difference becomes even more apparent with a significant gap of 3.77% (1.63% + 2.14%).

# SQL-Exploratory-Data-Analysis

- This portfolio contains Exploratory Data Analysis project for SQL. We are going to analyse the layoffs data set that has already been cleaned in the previous project. 
- We need to answer some few questions such as which year had most layoffs, which company, industry or country was affect the most and why so.
- We will rank both companies and industries to see the company or the industry that had the highest number of layoffs in the year.
- Finally, we will investigate what could have caused these layoffs and how we can reduce the number of layoffs.  

## Data Sources

layoffs: The primary dataset used for analyzing recent mass layoffs and discover useful insights and patterns. You can find this data at [kaggle](https://www.kaggle.com/).

## Tools

SQL Server

## Date Range of the Data Set

1.  What is the date range of our data set? We need to check from what date and until what date this data set is for. We can see that based on the results, this analysis will be based on the information that is recorded from March 11, 2020 until March 6, 2023, about a period of 3 years.

```sql   
SELECT MIN(date), MAX(date)
FROM layoffs_staging2;
```

2.  Let us check the maximum total laid off and maximum percentage of the layoffs that ever took place in one time.  We found that 12000 employees lost their jobs in one day. We also get 1 for the maximum percentage of the layoffs, and this represents 100%, meaning that the company laid off all its employees.
   
```sql
 SELECT MAX(total_laid_off), MAX(percentage_laid_off)
FROM layoffs_staging2;
```

3.  I would like to know the company that laid off 12000 employees at once, and the results shows that it is Google. 

```sql
SELECT *
FROM layoffs_staging2
WHERE total_laid_off = 12000
```

4.  Let us now check the companies that laid off everyone. 

```sql
SELECT *
FROM layoffs_staging2
WHERE percentage_laid_off = 1
ORDER BY total_laid_off DESC;
```

5.  Checking the funds raised by these companies that laid off their employees and the amount is in millions and the currency is USD.  

```sql
SELECT *
FROM layoffs_staging2
WHERE percentage_laid_off = 1
ORDER BY funds_raised_millions DESC;
```

6. Let us check the total number of employees each company laid off from March 2020 until March 2023.

```sql
SELECT company, SUM(total_laid_off)
FROM layoffs_staging2
GROUP BY company;
```

7.  We can now check the companies which had the highest number of lay offs as sort our finding by the total_laid_off. We can see that Amazon is leading, followed by Google. 

```sql
SELECT company, SUM(total_laid_off)
FROM layoffs_staging2
GROUP BY company
ORDER BY SUM(total_laid_off) DESC;
```

8.  Which industries are the most affected? Consumer with the total number of 45182 laid off employees is leading, followed by Retail industry which has total number of 43613 laid off employees. 

```sql
SELECT industry, SUM(total_laid_off)
FROM layoffs_staging2
GROUP BY industry
ORDER BY 2 DESC;
```

9.  Which countries are the most affected? United States is leading with a total number of 256420 and the following country is India with 35793 laid off employees. United States laid off more employees compared to all countries put together and the reason for this could be the following: 
- Some countries didn’t correctly record the total number of laid off employees. This could happen if the company does not disclose the total number of employees, it has laid off. 
- United States in a leading country when it comes to economy, so it has more companies and employees based in the United States, hence the laid off number of employees is so high. 

```sql
SELECT country, SUM(total_laid_off)
FROM layoffs_staging2
GROUP BY country
ORDER BY 2 DESC;
```

10. Checking which year had the most layoffs. The year 2022 had the most laid off with the total number of 160322. However, it should be noted that in the first 3 months of the year 2023, the total number of 125677 employees has been laid off, and this number is expected to increase. There is a high possibility that 2023 will have the most laid off employees. Our next project will focus on this because I am already curious about the outcome of the remaining months of 2023. 

```sql
SELECT YEAR(date), SUM(total_laid_off)
FROM layoffs_staging2
GROUP BY YEAR(date)
ORDER BY 1 DESC;
```

11. Checking the stage the companies were during the layoff. Stage Post-IPO shows that 204073 employees were laid off. The companies in the stage Post-IPO are developed and big companies such as Amazon, Google and etc. This shows that big companies laid off more employees that other companies in the lower stages. 

```sql
SELECT stage, SUM(total_laid_off)
FROM layoffs_staging2
GROUP BY stage
ORDER BY 2 DESC;
```

12.  We need to roll this data per month, and we start by grouping by month and get the totals of laid off each month. After managing to get each month and its total, then we will apply rolling to the data. We will use CTE for Rolling.

```sql
SELECT SUBSTRING(date, 1, 7) AS 'month', SUM(total_laid_off)
FROM layoffs_staging2
WHERE SUBSTRING(date, 1, 7) IS NOT NULL
GROUP BY month
ORDER BY 1;

WITH Rolling_Total AS
(
SELECT SUBSTRING(date, 1, 7) AS Month,  SUM(total_laid_off) AS total_off_job
FROM layoffs_staging2
WHERE SUBSTRING(date, 1, 7) IS NOT NULL
GROUP BY month
ORDER BY 1 ASC
)
SELECT Month, total_off_job, SUM(total_off_job) OVER(ORDER BY Month) AS rolling_total
FROM Rolling_Total;
```

13. Checking which company has the highest laid off within each year. We can see that Google is leading so far in 2023 with the current total number of 12000. Meta had the highest laid off in 2022 of total number of 11000. Uber led all companies in 2020 with the total number of 7525, and I suspect Covid-19 to be the cause of this since it caused the close down in many countries. 

```sql
SELECT company, YEAR(date), SUM(total_laid_off)
FROM layoffs_staging2
WHERE total_laid_off IS NOT NULL
GROUP BY company, YEAR(date)
ORDER BY 3 DESC;

WITH Company_Year (company, year, total_laid_off) AS
(
SELECT company, YEAR(date), SUM(total_laid_off)
FROM layoffs_staging2
WHERE total_laid_off IS NOT NULL
GROUP BY company, YEAR(date)
)
SELECT *
FROM Company_Year
```

14.  Let us check which company laid off more people in the year. We need to partition based on the year and then rank it based on how many employees the company laid off in that year. This will help us to see the company that laid off more people in that particular year. 

```sql
WITH Company_Year (company, years, total_laid_off) AS
(
SELECT company, YEAR(date), SUM(total_laid_off)
FROM layoffs_staging2
WHERE total_laid_off IS NOT NULL
GROUP BY company, YEAR(date)
)
SELECT *, DENSE_RANK() OVER (PARTITION BY years ORDER BY total_laid_off DESC) AS ranking
FROM Company_Year
WHERE years IS NOT NULL
ORDER BY ranking ASC;
```

15. Next step, I need to filter by Ranking to select the top 10 companies per year with the highest laid offs

```sql
WITH Company_Year (company, years, total_laid_off) AS
(
SELECT company, YEAR(date), SUM(total_laid_off)
FROM layoffs_staging2
WHERE total_laid_off IS NOT NULL
GROUP BY company, YEAR(date)
), Company_Year_Rank AS
(
SELECT *, DENSE_RANK() OVER (PARTITION BY years ORDER BY total_laid_off DESC) AS ranking
FROM Company_Year
WHERE years IS NOT NULL
)
SELECT *
FROM Company_Year_Rank
WHERE ranking <= 10
```

16.  Let us also check the Top 5 Industries with the highest laid offs. Transportation and Travel industries were mostly affected in 2020 and this could have been caused by Covid-19. 

```sql
WITH Industry_Year (industry, years, total_laid_off) AS
(
SELECT industry, YEAR(date), SUM(total_laid_off)
FROM layoffs_staging2
WHERE total_laid_off IS NOT NULL
GROUP BY industry, YEAR(date)
), Industry_Year_Rank AS
(
SELECT *, DENSE_RANK() OVER (PARTITION BY years ORDER BY total_laid_off DESC) AS ranking
FROM Industry_Year
WHERE years IS NOT NULL
)
SELECT *
FROM Industry_Year_Rank
WHERE ranking <= 5
```

17.   Analysis

The transportation and travel industries experienced significant layoffs in 2020 primarily due to the COVID-19 pandemic. Here are the key reasons: Travel Restrictions and Lockdowns, Decrease in Demand, Economic Uncertainty, Operational Challenges, Reduced Capacity, Event Cancellations, and Border Closures. These factors combined to create a severe downturn in the transportation and travel industries, forcing many companies to lay off employees in an effort to cut costs and survive the economic impact of the pandemic.

In the first three months of 2023, both Google and Microsoft, along with other major tech companies, experienced significant layoffs. The key reasons for these layoffs include: Post-Pandemic Adjustments, Economic Uncertainty, Overexpansion, Cost-Cutting Measures, Shifts in Business Focus, and Stock Market Pressures. These factors combined to create a situation where even well-established and highly profitable tech companies like Google and Microsoft found it necessary to conduct significant layoffs.


18.   Suggestions
Avoiding layoffs, especially in challenging economic times, requires strategic planning and proactive management. Here are some suggestions for companies to  consider: 

Financial Planning and Management:

-	Build Reserves: Maintain a healthy cash reserve to manage through downturns without needing to lay off employees.
-	Cost Management: Regularly review and manage costs to ensure the company is operating efficiently.

Flexible Work Arrangements:

-	Part-Time Work: Offer part-time work or reduced hours during downturns instead of full layoffs.
-	Remote Work: Utilize remote work to reduce overhead costs associated with physical office spaces.

Reskilling and Upskilling Employees:

-	Training Programs: Invest in continuous learning and development programs to help employees adapt to new roles within the company.
-	Career Development: Create clear career pathways that allow employees to transition to areas with higher demand.

Employee Engagement and Retention:

-	Transparent Communication: Maintain open and honest communication with employees about the company's performance and challenges.
-	Involvement in Decision-Making: Involve employees in decision-making processes, particularly those that affect their jobs.

19. Conclusion: There are many other questions one can answer using this data. We just answered these few questions but they are very important questions to be addressed. 






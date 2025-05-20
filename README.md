# SQL project to explore and analyze layoffs data

This project demonstrates an exploratory data analysis (EDA) workflow using SQL on a global layoffs dataset. The aim was to uncover patterns, trends, and insights such as which companies had the largest layoffs, which industries were most affected, and how layoffs evolved over time.

---

## Dataset Overview  
The dataset includes records of layoffs across various companies, with fields such as:

- company  
- location  
- industry  
- total_laid_off  
- percentage_laid_off  
- date  
- stage  
- country  
- funds_raised_millions  

---

## Tools Used  
- MySQL  
- SQL (Aggregations, Window Functions, CTEs, Date Functions)  
- GitHub  

---

## Data Exploration Steps  

```sql

-- Preview the dataset

SELECT * FROM world_layoffs.layoffs_staging;

-- We are going to explore the data and find trends or patterns or anything interesting like outliers

-- The Largest Single Layoff

SELECT MAX(total_laid_off)
FROM world_layoffs.layoffs_staging;

-- Extreme percentage layoffs (complete shutdowns)

SELECT MAX(percentage_laid_off),  MIN(percentage_laid_off)
FROM world_layoffs.layoffs_staging
WHERE  percentage_laid_off IS NOT NULL;

-- 1 is basically 100 percent of the company laid off

SELECT *
FROM world_layoffs.layoffs_staging
WHERE  percentage_laid_off = 1;

-- these are mostly startups it looks like who all went out of business during this time

-- if we order by funds_raised_millions we can see how big some of these companies were

SELECT *
FROM world_layoffs.layoffs_staging
WHERE  percentage_laid_off = 1
ORDER BY funds_raised_millions DESC;

-- BritishVolt looks like an EV company, Quibi raised like 2 billion dollars and went under

-- Companies with the biggest single Layoff

SELECT company, total_laid_off
FROM world_layoffs.layoffs_staging
ORDER BY 2 DESC
LIMIT 5;

-- that's just on a single day

-- Companies with the most Total Layoffs

SELECT company, SUM(total_laid_off)
FROM world_layoffs.layoffs_staging
GROUP BY company
ORDER BY 2 DESC
LIMIT 10;

-- by location

SELECT location, SUM(total_laid_off)
FROM world_layoffs.layoffs_staging
GROUP BY location
ORDER BY 2 DESC
LIMIT 10;

-- this it total in the past 3 years or in the dataset

-- by country

SELECT country, SUM(total_laid_off)
FROM world_layoffs.layoffs_staging
GROUP BY country
ORDER BY 2 DESC;

-- by year

SELECT YEAR(date), SUM(total_laid_off)
FROM world_layoffs.layoffs_staging
WHERE YEAR(date) IS NOT NULL
GROUP BY YEAR(date)
ORDER BY 1 ASC;

-- by industry

SELECT industry, SUM(total_laid_off)
FROM world_layoffs.layoffs_staging
GROUP BY industry
ORDER BY 2 DESC;

-- by stage

SELECT stage, SUM(total_laid_off)
FROM world_layoffs.layoffs_staging
GROUP BY stage
ORDER BY 2 DESC;

-- Earlier we looked at Companies with the most Layoffs. Now let's look at that per year.

WITH Company_Year AS 
(
  SELECT company, YEAR(date) AS years, SUM(total_laid_off) AS total_laid_off
  FROM layoffs_staging
  GROUP BY company, YEAR(date)
)
, Company_Year_Rank AS (
  SELECT company, years, total_laid_off, DENSE_RANK() OVER (PARTITION BY years ORDER BY total_laid_off DESC) AS ranking
  FROM Company_Year
)
SELECT company, years, total_laid_off, ranking
FROM Company_Year_Rank
WHERE ranking <= 3
AND years IS NOT NULL
ORDER BY years ASC, total_laid_off DESC;

-- Rolling Total of Layoffs Per Month

SELECT SUBSTRING(date,1,7) as dates, SUM(total_laid_off) AS total_laid_off
FROM layoffs_staging
GROUP BY dates
ORDER BY dates ASC;

-- now use it in a CTE so we can query off of it

WITH DATE_CTE AS 
(
SELECT SUBSTRING(date,1,7) as dates, SUM(total_laid_off) AS total_laid_off
FROM layoffs_staging
GROUP BY dates
ORDER BY dates ASC

)
SELECT dates, SUM(total_laid_off) OVER (ORDER BY dates ASC) as rolling_total_layoffs
FROM DATE_CTE
ORDER BY dates ASC;

-- Now that we have done a basic Exploratory Data Analysis on this dataset, we have a better understanding of the Layoffs Data

```

# Summary

By performing this SQL-based exploration, we identified key companies, industries, and countries most affected by layoffs. We also tracked layoff trends across months and years, and highlighted extreme cases such as companies with 100% workforce reduction.

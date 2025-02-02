# Layoffs Data Cleaning & EDA

This repository contains a comprehensive SQL project that demonstrates an end-to-end process for cleaning, transforming, and analyzing a layoffs dataset. The project includes duplicate removal, data standardization, handling of missing values, and various exploratory analyses to extract insights about industry trends, company performance, geographical distribution, and funding impacts.

## Table of Contents

- [Overview](#overview)
- [Project Structure](#project-structure)
- [Data Cleaning Process](#data-cleaning-process)
- [Exploratory Data Analysis (EDA)](#exploratory-data-analysis-eda)


## Overview


Layoffs can have significant economic and social impacts. This project is designed to:
- Clean and standardize a raw layoffs dataset.
- Remove duplicates and handle missing or inconsistent data.
- Perform exploratory analysis to uncover trends and patterns over time, across industries, companies, and countries.
- Analyze the relationship between funds raised and layoffs.

The project is implemented entirely in SQL and is ideal for learning how to prepare real-world data for further analysis.

## Project Structure

- **README.md**: This file.
- **layoffs_analysis.sql**: Contains all the SQL scripts for the data cleaning and exploratory analysis.
- **layoffs.csv**: The primary dataset uced for this analysis.

## Data Cleaning Process

The SQL scripts in this project follow these main steps:

1. **Creating a Staging Table**:  
   - A copy of the original `layoffs` table is created to work on a staging environment.

2. **Removing Duplicates**:  
   - A unique column is added using a window function.
   - Duplicates are removed based on all columns to ensure each record is unique.

3. **Standardizing Data**:  
   - Trimming whitespace from key columns.
   - Correcting inconsistencies (e.g., standardizing industry names).
   - Converting date strings to a proper `DATE` format.
   - Cleaning up country names by removing unwanted characters.

4. **Handling Missing Data**:  
   - Identifying and updating nulls or blanks.
   - Filling in missing industry values based on related records (matching by company, location, and country).
   - Removing unnecessary rows that lack critical information.
  
```sql
SELECT *
FROM layoffs;

-- 1. DATA CLEANING
		-- 1. Removing Duplicates
		-- 2. Standardizing the Data
		-- 3. Checking for Nulls and Blanks
		-- 4. Removing Unnecessary Rows and Columns

-- CREATING A COPY OF THE DATASET
 
CREATE TABLE layoffs_staging
LIKE layoffs;

INSERT INTO layoffs_staging
SELECT *
FROM layoffs;

SELECT *
FROM layoffs_staging;


-- IDENTIFYING AND REMOVING DUPLICATES

-- adding a unique column
CREATE TABLE `layoffs_staging2` (
  `company` text,
  `location` text,
  `industry` text,
  `total_laid_off` int DEFAULT NULL,
  `percentage_laid_off` text,
  `date` text,
  `stage` text,
  `country` text,
  `funds_raised_millions` int DEFAULT NULL,
  `row-num` int
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

ALTER TABLE layoffs_staging2
RENAME COLUMN `row-num` TO `row_num`;

INSERT INTO layoffs_staging2
SELECT *, row_number() OVER(
PARTITION BY company, location, industry, total_laid_off, percentage_laid_off, `date`, stage, country, funds_raised_millions) as row_num
FROM layoffs;

-- removing duplicates
DELETE
FROM layoffs_staging2
WHERE row_num > 1;

SELECT *
FROM layoffs_staging2;


-- STANDARDIZING THE DATA

-- standardizing the individual columns
SELECT *
FROM layoffs_staging2;

UPDATE layoffs_staging2
SET company = TRIM(company), location = TRIM(location), industry = TRIM(industry), stage = TRIM(stage), country = TRIM(country);

-- checking each column to identify standardization issues

SELECT DISTINCT location
FROM  layoffs_staging2;
-- no issues identified

SELECT DISTINCT company
FROM  layoffs_staging2;
-- no issues identified

SELECT DISTINCT industry
FROM  layoffs_staging2
ORDER BY 1;
-- one issue identified
-- inconsistencies with the Crypto Currency industry

UPDATE layoffs_staging2
SET industry = 'Crypto Currency'
WHERE industry LIKE 'Crypto%';


SELECT DISTINCT stage
FROM  layoffs_staging2
ORDER BY 1;
-- no issues identified


SELECT DISTINCT country
FROM  layoffs_staging2
ORDER BY 1;

-- removing '.' from the ends of the country names
UPDATE layoffs_staging2
SET country = TRIM(TRAILING '.' FROM country);

-- standardizing the date column
SELECT `date`
FROM layoffs_staging2;

-- converting the date column into a date format
UPDATE layoffs_staging2
SET `date` = STR_TO_DATE(`date`, '%m/%d/%Y');

ALTER TABLE layoffs_staging2
MODIFY COLUMN `date` DATE;


-- CHECKING FOR NULLS AND BLANKS

SELECT *
FROM layoffs_staging2
WHERE company IS NULL OR location IS NULL OR industry IS NULL OR country IS NULL
OR company ='' OR location ='' OR industry ='' OR country ='';
-- blanks and nulls identified in the industry column


-- filling missing industry values where possible
UPDATE layoffs_staging2
SET industry = NULL
WHERE industry = '';

-- checking companies in the same location and country to fill in missing industries
-- logic: a company in the same country and location is likely to be in the same industry 
SELECT T1.industry, T2.industry
FROM layoffs_staging2 T1
JOIN layoffs_staging2 T2
	ON T1.company = T2.company
    AND T1.location = T2.location
    AND T1.country = T2.country
WHERE T1.industry IS NULL
	AND T2.industry IS NOT NULL;

-- updating the table to fill in missing industries
UPDATE layoffs_staging2 T1
JOIN layoffs_staging2 T2
	ON T1.company = T2.company
SET T1.industry = t2.industry
WHERE T1.industry IS NULL
	AND T2.industry IS NOT NULL;


-- REMOVING UNNECESSARY ROWS

SELECT *
FROM layoffs_staging2
WHERE total_laid_off IS NULL
	AND percentage_laid_off IS NULL;
-- these rows are unnecessary because these two columns are needed for analysis and cannot be both empty

DELETE
FROM layoffs_staging2
WHERE total_laid_off IS NULL
	AND percentage_laid_off IS NULL;

-- dropping the unique column
ALTER TABLE layoffs_staging2
DROP COLUMN row_num;

SELECT *
FROM layoffs_staging2;
-- everything looks good
```

## Exploratory Data Analysis (EDA)

After cleaning, the project performs several analyses, including:

- **Overall Layoffs Analysis**:  
  - Calculating total layoffs and summarizing trends over time (yearly and monthly breakdowns).

- **Industry Insights**:  
  - Analyzing layoffs by industry and exploring trends across different time periods.

- **Company Insights**:  
  - Identifying the top companies with the highest number of layoffs.
  - Analyzing layoffs over time for specific companies (e.g., Amazon).

- **Geography Insights**:  
  - Summarizing layoffs by country and by country & industry.
  - Observing geographical trends over time.

- **Funding Insights**:  
  - Grouping companies by funds raised ranges and analyzing how this correlates with the number of layoffs.
  - Generating funding range labels based on the sum of funds raised per company.

- **Outlier Detection**:  
  - Identifying companies with unusually high layoffs based on statistical outlier detection.

```sql
-- 2. EDA

-- OVERVIEW

-- total layoffs
SELECT SUM(total_laid_off) AS total_layoffs
FROM layoffs_staging2;

-- yearly trends
SELECT YEAR(`date`) AS `Year`, SUM(total_laid_off) AS layoffs_per_year 
FROM layoffs_staging2
GROUP BY YEAR(`date`)
HAVING `Year` IS NOT NULL
ORDER BY 1;

-- monthly breakdown
SELECT DATE_FORMAT(`date`, '%Y-%m') AS `year_month`, SUM(total_laid_off) AS layoffs_per_month
FROM layoffs_staging2
GROUP BY `year_month`
HAVING `year_month` IS NOT NULL
ORDER BY 1;

-- INDUSTRY INSIGHTS

-- layoffs by industry
SELECT industry, SUM(total_laid_off) AS layoffs_per_industry
FROM layoffs_staging2
WHERE industry IS NOT NULL
GROUP BY 1
ORDER BY 2 DESC;

-- yearly layoffs by industry
SELECT industry, YEAR(`date`) AS `year`, SUM(total_laid_off) AS layoffs_per_industry
FROM layoffs_staging2
WHERE industry IS NOT NULL 
	AND YEAR(`date`) IS NOT NULL
GROUP BY 1, 2
ORDER BY 2 DESC;


-- COMPANY INSIGHTS

-- top 10 companies with most latoffs
SELECT company, SUM(total_laid_off) AS layoffs
FROM layoffs_staging2
WHERE company IS NOT NULL
GROUP BY 1
ORDER BY 2 DESC
LIMIT 10;

-- layoffs over time for a specific company (Amazon in this case)
SELECT DATE_FORMAT(`date`, '%Y-%m') AS `year_month`, SUM(total_laid_off) AS layoffs
FROM layoffs_staging2
WHERE company = 'Amazon'
GROUP BY 1
HAVING `year_month` IS NOT NULL
ORDER BY 1;

-- GEOGRAPHY INSIGHTS

-- layoffs by country
SELECT country, SUM(total_laid_off) AS layoffs
FROM layoffs_staging2
GROUP BY 1
ORDER BY 2 DESC;

-- layoffs by country and industry
SELECT country, industry, SUM(total_laid_off) AS layoffs
FROM layoffs_staging2
GROUP BY 1, 2
ORDER BY 1,3 DESC;

-- layoffs by country over time
SELECT country, DATE_FORMAT(`date`, '%Y-%m') AS `year_month`, SUM(total_laid_off) AS layoffs
FROM layoffs_staging2
GROUP BY country, `year_month`
HAVING `year_month` IS NOT NULL
ORDER BY 1, 2;

-- FUND-RAISED INSIGHTS

-- total layoffs by funds raised range
SELECT CASE
			WHEN funds_raised_millions = 0 THEN '0M'
            WHEN funds_raised_millions < 10 THEN '<10M'
            WHEN funds_raised_millions BETWEEN 10 AND 100 THEN '10M-100M'
            WHEN funds_raised_millions BETWEEN 100 AND 1000 THEN '100M-1B'
            WHEN funds_raised_millions BETWEEN 1000 AND 10000 THEN '1B-10B'
            WHEN funds_raised_millions BETWEEN 10000 AND 100000 THEN '10B-100B'
            ELSE '>100B'
		END AS funding_range,
        SUM(total_laid_off) AS total_layoffs
FROM layoffs_staging2
WHERE funds_raised_millions IS NOT NULL
	AND funds_raised_millions != ''
GROUP BY funding_range
ORDER BY 1 DESC;


-- total layoffs by company showing their total funds raised range 
SELECT DISTINCT company, SUM(funds_raised_millions) OVER (PARTITION BY company)AS total_funds_raised_millions,
		CASE
			WHEN SUM(funds_raised_millions) OVER (PARTITION BY company) = 0 THEN '0M'
            WHEN SUM(funds_raised_millions) OVER (PARTITION BY company) < 10 THEN '<10M'
            WHEN SUM(funds_raised_millions) OVER (PARTITION BY company) BETWEEN 10 AND 100 THEN '10M-100M'
            WHEN SUM(funds_raised_millions) OVER (PARTITION BY company) BETWEEN 100 AND 1000 THEN '100M-1B'
            WHEN SUM(funds_raised_millions) OVER (PARTITION BY company) BETWEEN 1000 AND 10000 THEN '1B-10B'
            WHEN SUM(funds_raised_millions) OVER (PARTITION BY company) BETWEEN 10000 AND 100000 THEN '10B-100B'
            ELSE '>100B'
		END AS funding_range, SUM(total_laid_off) OVER (PARTITION BY company) AS total_layoffs
FROM layoffs_staging2
WHERE industry IS NOT NULL
ORDER BY 2 DESC;


-- IDENTIFYING OUTLIERS

-- companies with unusually high layoffs
SELECT company, total_laid_off
FROM layoffs_staging2
WHERE total_laid_off > (
		SELECT AVG(total_laid_off) + 2 * STDDEV(total_laid_off) 
        FROM layoffs_staging2
        )
ORDER BY total_laid_off DESC;
```

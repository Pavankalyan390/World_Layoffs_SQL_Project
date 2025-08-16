# World_Layoffs_SQL_Project

# Overview
- This project focuses on cleaning and preparing raw data related to global layoffs from various companies. Key steps include setting up the database, importing the dataset, handling NULL values, removing duplicate records, and dropping unnecessary columns to ensure a clean and analysis-ready dataset.

## Things to clean in data: 
- Removed duplicates
- Standardized the data
- Removed Null and Blank Values
- Eliminated unwanted columns and rows


## Project Structure

- Data Import: Inserting sample data into the tables.
- Data Cleaning: Handling null values and ensuring data integrity.
- Duplicate Table: For best practice Created duplicate data like layoff_staging to keep original raw data safe.

## Creating Table
```sql
create table layoffs(
Company varchar(50),
Location varchar(30),
Industry text,
Total_laid_off int,
Percentage_laid_off int,
Date date,
Stage text,
Country varchar(30),
Funds_raised_Millions FLOAT
);

```

## Data Import
### 1.Data Cleaning and removing duplicates rows
``` sql
-- For best practice Created duplicate data like layoff_staging to keep original raw data safe
CREATE Table layoffs_staging
LIKE layoffs;

INSERT INTO layoffs_staging
SELECT *
FROM layoffs;
```

``` sql
select * from layoffs_staging;

--used as row_num to identify unique rows from the table where ther's > 2 it's duplicates
SELECT *,
ROW_NUMBER() OVER(
PARTITION BY company, industry, total_laid_off, percentage_laid_off, `date`) AS row_num
FROM layoffs_staging;

WITH duplicates_CTE As(
SELECT *,
ROW_NUMBER() OVER(
PARTITION BY company, location, industry, total_laid_off, percentage_laid_off, `date`, 
stage, country, funds_raised_millions) AS row_num
FROM layoffs_staging
)
SELECT *
FROM duplicates_CTE
WHERE row_num > 1;

--Created another Table like layoffs_staging2 because we can delete and filter from those row_num which > 1
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
  `row_num` INT
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

INSERT INTO layoffs_staging2
SELECT *,
ROW_NUMBER() OVER(
PARTITION BY company, location, industry, total_laid_off, percentage_laid_off, `date`, 
stage, country, funds_raised_millions) AS row_num
FROM layoffs_staging;

SELECT *
FROM layoffs_staging2
WHERE row_num > 1;

DELETE
FROM layoffs_staging2
WHERE row_num > 1;
```

### 2. Standardizing data
```sql
SELECT company, Trim(company)
FROM layoffs_staging2;

UPDATE layoffs_staging2
SET company = Trim(company);

-- Corrected the two same Industry crypto and crypto currency to one name like "crypto"
SELECT *
FROM layoffs_staging2
WHERE industry LIKE 'Crypto%';

UPDATE layoffs_staging2
SET industry = 'Crypto'
WHERE industry LIKE 'Crypto%';

-- Removing Period at the end for unique country name.
SELECT DISTINCT country, TRIM(TRAILING '.' FROM country)
FROM layoffs_staging2
ORDER BY 1;

UPDATE layoffs_staging2
SET country  = TRIM(TRAILING '.' FROM country)
WHERE country LIKE 'United States%';

SELECT `date`,
STR_TO_DATE(`date`, '%m/%d/%Y')
FROM layoffs_staging2;

UPDATE layoffs_staging2
SET `date` = STR_TO_DATE(`date`, '%m/%d/%Y');

ALTER Table layoffs_staging2
MODIFY COLUMN `date` DATE;
```

### 3.Removing Null or Blank Values
```sql
SELECT *
FROM layoffs_staging2
WHERE total_laid_off IS NULL
AND percentage_laid_off IS NULL;

DELETE 
FROM layoffs_staging2
WHERE total_laid_off IS NULL
AND percentage_laid_off IS NULL;

SELECT *
FROM layoffs_staging
WHERE Industry IS NULL
OR Industry = '';

UPDATE layoffs_staging2
SET industry = NULL
WHERE industry = '';

--The same company's industry name is blank in one record and not blank in another. It should be the same name, so I used a JOIN.
SELECT T1.industry, T2.industry
FROM layoffs_staging2 T1
JOIN layoffs_staging2 T2 
    ON T1.company = T2.company
WHERE (T1.industry IS NULL OR T1.industry = '')
AND T2.industry IS NOT NULL;

UPDATE layoffs_staging2 T1
JOIN layoffs_staging2 T2 
    ON T1.company = T2.company
SET T1.industry = T2.industry
WHERE T1.industry IS NULL
AND T2.industry IS NOT NULL;
```

### 4. Eliminating unwanted columns and rows:
```sql
-- At the very end, we need to eliminate the row_num column, as we don't need it anymore.

ALTER Table layoffs_staging2
DROP COLUMN row_num;
```
## Conclusion
This project highlights my ability to handle complex SQL queries and provides solutions to real-world problems in the context of a SQL data cleaning. The approach taken here demonstrates a structured problem-solving methodology, data manipulation skills, and the ability to derive actionable insights from data.

## Notice 
All customer names and data used in this project are computer-generated using AI and random functions. They do not represent real data associated with any other entity. This project is solely for learning and educational purposes, and any resemblance to actual company, businesses, or events is purely coincidental.

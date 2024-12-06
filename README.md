SQL Project: Data Cleaning Documentation
Overview
This documentation details the process of cleaning and preparing the layoffs dataset from Kaggle, specifically using SQL to identify and resolve issues such as duplicates, missing or inconsistent values, and incorrect data formatting. The cleaned data is saved in a staging table (layoffs_staging2) for further analysis.

Steps
1. Initial Setup
* The raw data is loaded into the table world_layoffs.layoffs.
* A staging table, layoffs_staging, is created to ensure the raw data remains intact: 
CREATE TABLE world_layoffs.layoffs_staging 
LIKE world_layoffs.layoffs;
INSERT INTO world_layoffs.layoffs_staging 
SELECT * FROM world_layoffs.layoffs; 

2. Data Cleaning Process
The cleaning process involves several key steps:
2.1 Remove Duplicates
1. Identify Duplicates:
Use the ROW_NUMBER() function to identify duplicate rows based on specific columns.
Rows with ROW_NUMBER() > 1 are considered duplicates. SELECT company, location, industry, total_laid_off, percentage_laid_off, `date`, stage, country, funds_raised_millions,
ROW_NUMBER() OVER (PARTITION BY company, location, industry, total_laid_off, percentage_laid_off, `date`, stage, country, funds_raised_millions) AS row_num
FROM world_layoffs.layoffs_staging;
 Remove Duplicates:
Create a temporary column row_num and delete rows with row_num > 1.
 DELETE FROM world_layoffs.layoffs_staging2
WHERE row_num >= 2; 

2.2 Standardize and Fix Data
1. Null and Blank Values:
Replace blank values with NULL for easier handling: UPDATE world_layoffs.layoffs_staging2
SET industry = NULL
WHERE industry = '';
 Populate NULL values where possible by referencing other rows in the dataset:  UPDATE layoffs_staging2 t1
JOIN layoffs_staging2 t2
ON t1.company = t2.company
SET t1.industry = t2.industry
WHERE t1.industry IS NUL
AND t2.industry IS NOT NULL;
  
1. Normalize Data:
Fix inconsistencies in text fields like industry and country:  UPDATE layoffs_staging2
SET industry = 'Crypto'
WHERE industry IN ('Crypto Currency', 'CryptoCurrency');
UPDATE layoffs_staging2
SET country = TRIM(TRAILING '.' FROM country);

1. Standardize Date Formats:
Convert date from string to proper DATE format: UPDATE layoffs_staging2
SET `date` = STR_TO_DATE(`date`, '%m/%d/%Y');
ALTER TABLE layoffs_staging2
MODIFY COLUMN `date` DATE; 
2.3 Handle Missing or Invalid Data
1. Delete Unusable Rows:
Rows where both total_laid_off and percentage_laid_off are NULL are removed: DELETE FROM world_layoffs.layoffs_staging2
WHERE total_laid_off IS NULL
AND percentage_laid_off IS NULL;
  
1. Remove Helper Columns:
Drop columns (e.g., row_num) used temporarily for processing: ALTER TABLE layoffs_staging2
DROP COLUMN row_num; 

3. Post-Cleaning Verification
Validate the cleaned data:  SELECT * FROM world_layoffs.layoffs_staging2;
Check for duplicates, NULL values, and formatting issues to confirm the dataset is clean:  SELECT DISTINCT industry FROM world_layoffs.layoffs_staging2 ORDER BY industry;
SELECT DISTINCT country FROM world_layoffs.layoffs_staging2 ORDER BY country; 

Key Outcomes
* Duplicates Removed: Rows with identical information have been cleaned.
* Standardized Data: Fields such as industry, country, and date have been normalized for consistency.
* Clean and Usable Dataset: Rows with incomplete data were removed to ensure analysis-ready data.

Next Steps
* Perform exploratory data analysis (EDA) or additional transformations as needed.
* Export the cleaned dataset from world_layoffs.layoffs_staging2 for use in other tools or processes.

This structured approach ensures data integrity, consistency, and usability for further analytics or reporting.

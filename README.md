# Layoffs

## 📚 Table of Contents
- [Business Task](#business-task)
- [Data Cleaning and Transformation](#-data-cleaning--transformation)
- [A. Pizza Metrics](#a-pizza-metrics)
- [B. Runner and Customer Experience](#b-runner-and-customer-experience)
- [C. Ingredient Optimisation](#c-ingredient-optimisation)
- [D. Pricing and Ratings](#d-pricing-and-ratings)

Please note that all the information regarding the case study has been sourced from the following link: [here](https://8weeksqlchallenge.com/case-study-2/).

***

## Business Task

## 🧼 Data Cleaning & Transformation

### Step 1: Create a Staging Table
````sql
CREATE TABLE layoff_staging
LIKE layoffs
;
`````
THEN
````sql
INSERT layoff_staging
SELECT *
FROM layoffs
;
`````

### Step 2: Remove Duplicates

- First we need to identify possivle duplicates within the dataset. This can be done by assiging each entry a `row_number`.
````sql
WITH dublicate_cte AS
(
SELECT *, 
row_number() OVER (PARTITION BY company, location, industry, total_laid_off, percentage_laid_off, `date`, stage, country, funds_raised) AS row_num
FROM layoff_staging
)
SELECT *
FROM dublicate_cte
WHERE row_num > 1
;
`````
- We have identified two potential companies Beyond Meat and Cazoo
PICTURE

- Next, we can furhter this by looking into the entries into this dataset for each of the companies to determine their status as a dublicate.
  - Cazoo
    ````sql
    SELECT *
    FROM layoff_staging
    WHERE company = 'Cazoo'
    ;
    `````
      - A dublicate is confirmed here found in the last two rows.
      - PICTURE\

  - Beyond Meat
   ````sql
    SELECT *
    FROM layoff_staging
    WHERE company = 'Cazoo'
    ;
    `````
     - A dublicate is confirmed here found in the sencond row.
      - PICTURE\

- Further, we'll create another Staging table so we can filer on the row_num
````sql
    CREATE TABLE `layoff_staging` (
  `company` text,
  `location` text,
  `industry` text,
  `total_laid_off` text,
  `percentage_laid_off` text,
  `date` text,
  `stage` text,
  `country` text,
  `funds_raised` text,
  `row_num` INT -- Add row_num
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;
`````

````sql
  INSERT INTO layoff_staging2
  SELECT *, 
  row_number() OVER (PARTITION BY company, location, industry, total_laid_off, percentage_laid_off, `date`, stage, country, funds_raised) AS row_num
  FROM layoff_staging
  ;
`````

- Finally, we can delete the dublicate rows
````sql
DELETE
FROM layoff_staging2
WHERE row_num > 1
;
`````

### Step 3: Standardize the data

- At first glance I noticed that some of the spacing is off in the company names so I applied a `trim` to take off the whitespace off.
````sql
SELECT company, TRIM(company)
FROM layoff_staging2
;
`````
- PICTURE

- Apply trim
````sql
UPDATE layoff_staging2
SET company = TRIM(company)
;
`````
-Next, I reviewed the company’s location data and identified errors related to diacritics—special accent marks used in many languages (like é, ñ, ü) that change how letters are pronounced or understood. For example, in the dataset, the city of "Düsseldorf" appeared incorrectly as "DÃ¼sseldorf." Below are the errors I identified and corrected along with the respective SQL queries.

  - DÃ¼sseldorf -> Düsseldorf
  ````sql
  UPDATE layoff_staging2
  SET location = 'Düsseldorf'
  WHERE location = 'DÃ¼sseldorf'
  ;
  `````
 - FÃ¸rde -> FØRDE
  ````sql
  UPDATE layoff_staging2
  SET location = 'FØRDE'
  WHERE location = 'FÃ¸rde'
  ;
  `````
 - FlorianÃ³polis -> Florianópolis
  ````sql
  UPDATE layoff_staging2
  SET location = 'FØRDE'
  WHERE location = 'FÃ¸rde'
  ;
  `````
- MalmÃ¶ -> Malmö 
  ````sql
  UPDATE layoff_staging2
  SET location = 'Malmö'
  WHERE location = 'MalmÃ¶'
  ;
  `````
- WrocÅ‚aw -> Wrocław 
  ````sql
  UPDATE layoff_staging2
  SET location = 'Malmö'
  WHERE location = 'MalmÃ¶'
  ;
  `````
- Another similar issue was found in the country column, where some rows used "UAE" while others used "United Arab Emirates." I will standardize the data to use the full spelling instead of the abbreviation for consistency.
  ````sql
  UPDATE layoff_staging2
  SET country = 'United Arab Emirates'
  WHERE country = 'UAE'
  ;
  `````
  
- I will change the data column from text to a date format to enable time series data analysis.
  ````sql
  ALTER TABLE layoff_staging2
  MODIFY COLUMN `date` DATE
  ;
  `````


### Step 4: Null Values or blank values
### Step 5: Remove Any Columns

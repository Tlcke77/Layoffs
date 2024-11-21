# Layoffs

## 📚 Table of Contents
- [About the Dataset](#about-the-dataset)
- [Data Cleaning & Transformation](#-data-cleaning--transformation)
  - [Step 1: Create a Staging Table](#step-1-create-a-staging-table)
  - [Step 2: Remove Duplicates](#step-2-remove-duplicates)
  - [Step 3: Standardize the data](#step-3-standardize-the-data)
  - [Step 4: Null Values or Blank Values](#step-4-null-values-or-blank-values)
  - [Step 5: Remove Any Columns](#step-5-remove-any-columns)

Please note that the data has been sourced from the following link: [here](https://www.kaggle.com/datasets/swaptr/layoffs-2022).
:grey_exclamation: This dataset is constantly being updated, so results may vary.  :grey_exclamation:

***

## About the Dataset

This project uses a dataset that tracks recent tech layoffs, sourced from platforms like Bloomberg, San Francisco Business Times, TechCrunch, and The New York Times. 

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

- First we need to identify possible duplicates within the dataset. This can be done by assigning each entry a `row_number`.
````sql
WITH duplicate_cte AS
(
SELECT *, 
row_number() OVER (PARTITION BY company, location, industry, total_laid_off, percentage_laid_off, `date`, stage, country, funds_raised) AS row_num
FROM layoff_staging
)
SELECT *
FROM duplicate_cte
WHERE row_num > 1
;
`````
- We have identified two potential companies Beyond Meat and Cazoo
<img src="https://github.com/Tlcke77/pics/blob/main/Capture.PNG" alt="Image" width="800" height="55">

- Next, we can further this by looking into the entries into this dataset for each of the companies to determine their status as a duplicate.
  - Cazoo
    ````sql
    SELECT *
    FROM layoff_staging
    WHERE company = 'Cazoo'
    ;
    `````
      - A duplicate is confirmed here found in the last two rows.
<img src="https://github.com/Tlcke77/pics/blob/main/Capture%202.PNG" alt="Image" width="704" height="69">

  - Beyond Meat
````sql
SELECT *
FROM layoff_staging
WHERE company = 'Cazoo'
;
````
- A duplicate is confirmed here found in the sencond row.
<img src="https://github.com/Tlcke77/pics/raw/main/Capture%203.PNG" alt="Image" width="704" height="69">

- Further, we'll create another staging table so we can filter on the row_num
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
````

````sql
  INSERT INTO layoff_staging2
  SELECT *, 
  row_number() OVER (PARTITION BY company, location, industry, total_laid_off, percentage_laid_off, `date`, stage, country, funds_raised) AS row_num
  FROM layoff_staging
  ;
````

- Finally, we can delete the duplicate rows
````sql
DELETE
FROM layoff_staging2
WHERE row_num > 1
;
````

### Step 3: Standardize the data

- At first glance I noticed that some of the spacing is off in the company names so I applied a `trim` to take off the whitespace off.
````sql
SELECT company, TRIM(company)
FROM layoff_staging2
;
`````
<img src="https://github.com/Tlcke77/pics/blob/main/Capture%204.PNG" alt="Image" width="188" height="136">

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

### Step 4: Null Values or Blank Values

- The blank space I looked at where in the Industry column and here I indentified one blank value. It was with the company called "Appsmith". Out of the available industry options used in the dataset I felt that it fell best under the 'Other' category.
  ````sql
  SELECT *
  FROM layoff_staging2
  WHERE industry IS NULL 
  OR industry = ''
  ;
  `````

  ````sql
  UPDATE layoff_staging2
  SET industry = 'Other'
  WHERE industry = ''
  ;
  ````` 
- Then I looked at the location column and found a blank column for the company "Product Hunt".
  ````sql
  SELECT *
  FROM layoff_staging2
  WHERE location IS NULL 
  OR location = ''
  ;
  `````
  
  ````sql
  UPDATE layoff_staging2
  SET location = 'SF Bay Area'
  WHERE location = ''
  ;
  `````
:grey_exclamation: Discalmier there is still some blank values in the total_laid_off, percentage_laid_off, stage, and funds_raised because I fell like its not possible to populate the values with data due to the data availbe with the dataset. :grey_exclamation:

### Step 5: Remove Any Columns

:grey_exclamation: This section is completely option and is based on your goal for the dataset. :grey_exclamation:

- I'm cleaning this dateset with the goal of performing some exploaratory data analaysis. So with that said I will be removing columns where total_laid_off and percentage_laid_off are blank becuase they service no additonal value to me in my future process with my future data analysis.
  
 ````sql
  DELETE
  FROM layoff_staging2
  WHERE percentage_laid_off = ''
  AND total_laid_off = ''
  ;
  `````

- Then I no longer need the row_num column as well.
 ````sql
  ALTER TABLE layoff_staging2
  DROP COLUMN row_num
  ;
  ````` 

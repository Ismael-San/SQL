# BACI : International Trade Database Analysis

### Table of Contents

### Introduction

BACI (Base pour l'Analyse du Commerce International) is a bilateral trade flow dataset published by Guillaume Gaulier and Soledad from the CEPII, based on data collected from the United Nations COMTRADE. Several reasons explain the computation of BACI by Gaulier and Soledad:

1. Incomplete Data: COMTRADE presents some missing values/flows in the country reports sent to the United Nations, leading to incomplete data.
2. Classification Issues: Some countries use the SITC classification, an older system instead of the current Harmonized System, can lead to lack of readibility.
3. Data Discrepancies: There are differences in the data provided by importers and exporters for the same trade flow. One solution to this issue was to remove the trade cost value (CIF) from import values.

### Project Overview

The main objective of this project was to explore the most principals queries use in SQL that I've learned and, at the end, extract insights from international trade products between countries using BACI. 

### Data Sources

BACI_HS17_Y2021_V202301 : csv file used contained detailed informations about bilateral trade exchange by providing :

- t: Year
- k : Product category (HS 6-digit code)
- i : Exporter (ISO 3-digit country code)
- j : Importer (ISO 3-digit country code)
- v : Value of trade flow (in thousands of USD)
- q : Quantity (in metric tons)

Country codes : 
- Associates the ISO 3-digit country codes to country names

Product codes :
- Associates the HS 6-digit product codes to product names : give more detailed information about products

### Tools
- MySQLWorkbench : Data Analysis (add directlink toward sql file)

- Microsoft Excel : Dashboard Creation (add directlink toward xlsx file - add pdf to present dashboard)

### Entity Relationship Diagram


### Data Cleaning/Preparation
In the initial data preparation phase, we performed the following tasks:
- Loading the csv dataset
- Handling missing values/duplicates
- Data formatting :
  - Handling format issues with country name and special characters from Microsoft Excel
  - Data formatting in header to increase readibility, here is the queries use from MySQL Workbench :

```sql
SELECT *
FROM BACI_db.country_codes_modif;

/*
RENAME TABLE NAME 
FROM baci_hs17_y2021_v202301 TO baci_hs17_23
FROM country_codes_modif TO country_codes
FROM product_codes_hs17_v202301 TO product_codes
*/

ALTER TABLE baci_hs17_y2021_v202301 RENAME baci_hs17_23;
ALTER TABLE country_codes_modif RENAME country_codes;
ALTER TABLE product_codes_hs17_v202301 RENAME product_codes;


/* RENAME COLUMNS NAME FROM EACH TABLE
TABLE baci_hs17_23 : 
- FROM t TO year
- FROM i TO exporter
- FROM j TO importer
- FROM k TO product_category
- FROM v TO value
- FROM q TO quantity
*/

ALTER TABLE baci_hs17_23 RENAME COLUMN t TO year_;
ALTER TABLE baci_hs17_23 RENAME COLUMN i TO exporter;
ALTER TABLE baci_hs17_23 RENAME COLUMN j TO importer;
ALTER TABLE baci_hs17_23 RENAME COLUMN k TO product_category;
ALTER TABLE baci_hs17_23 RENAME COLUMN v TO value_;
ALTER TABLE baci_hs17_23 RENAME COLUMN q TO quantity;

/*
TABLE country_codes:
- FROM country code TO country_code
- FROM country name abbreviation TO country_name_abbreviation
- FROM country name full TO country_name_full
- FROM iso two digit alpha TO iso_2_digit_alpha 
- FROM iso three digit alpha full TO iso_3_digit_alpha
*/

ALTER TABLE country_codes RENAME COLUMN `country code` TO country_code;
ALTER TABLE country_codes RENAME COLUMN `country name abbreviation` TO country_name_abbreviation;
ALTER TABLE country_codes RENAME COLUMN `country name full` TO country_name_full;
ALTER TABLE country_codes RENAME COLUMN `iso two digit alpha` TO iso_2_digit_alpha;
ALTER TABLE country_codes RENAME COLUMN `iso three digit alpha` TO iso_3_digit_alpha;

/*
TABLE product_codes:
No modification needed
*/


```
### Exploratory Data Analysis

### Data Analysis
```sql
-- BACI CEPII SCRIPT PROJECT --

-- TO RETRIEVE BILATERAL EXCHANGE DETAILS --
SELECT
	*
FROM baci_hs17_23
;

-- TO RETRIEVE PRODUCT DETAILS --
SELECT
	*
FROM product_codes
;

-- TO RETRIEVE CONTRY DETAILS
SELECT
	*
FROM country_codes
;

-- #1 TOTAL TRADE VOLUME -- IT WOULD HAVE BEEN GOOD TO HAVE THE AMOUNT FROM PREVIOUS YEARS - TIME SERIES ANALYSIS
SELECT
	year_,COUNT(quantity)
FROM baci_hs17_23
GROUP BY year_
;

-- #2 TOTAL TRADE VALUE -- IT WOULD HAVE BEEN GOOD TO HAVE THE AMOUNT FROM PREVIOUS YEARS - TIME SERIES ANALYSIS
SELECT
	year_,CONCAT(ROUND(SUM(value_),2), ' ', '$') AS total_trade_value
FROM baci_hs17_23
GROUP BY year_  
;

-- #3 AVERAGE TRANSACTION VALUE -- 
SELECT
	year_,CONCAT(ROUND(AVG(value_),2), ' ', '$') AS average_transaction_value
FROM baci_hs17_23
GROUP BY year_
;
/*
-- # TOTAL AMOUNT PRODUCT IMPORTED BY COUNTRY -- #6
SELECT 
	importer,
    country_name_full,
    SUM(value_)
FROM baci_hs17_23
JOIN country_codes
	ON baci_hs17_23.importer = country_codes.country_code
GROUP BY importer, country_name_full
ORDER BY SUM(value_) DESC
;

-- # TOTAL AMOUNT PRODUCT EXPORTED BY COUNTRY -- #5
SELECT 
	exporter,
    country_name_full,
    SUM(value_)
FROM baci_hs17_23
JOIN country_codes
	ON baci_hs17_23.exporter = country_codes.country_code
GROUP BY exporter, country_name_full
ORDER BY SUM(value_) DESC
;
*/

-- #3 EXPORT CONCENTRATION (WHEN COUNTRY_CODE IN EXPORT COLUMN) -- ADD ANOTHER COLUMN : RATIO % EXP CONCENTRATION
SELECT
	country_name_full,
    CEIL(SUM(value_)) AS total_amount,
    SUM(CEIL(SUM(value_))) OVER () AS grand_total,
    ROUND(((CEIL(SUM(value_))/SUM(CEIL(SUM(value_))) OVER ()*100)),2) AS ratio
FROM baci_hs17_23
JOIN country_codes
	ON baci_hs17_23.exporter = country_codes.country_code
GROUP BY country_name_full
ORDER BY total_amount DESC
;

-- #4 IMPORT CONCENTRATION -- ADD ANOTHER COLUMN : RATIO % IMP CONCENTRATION
SELECT
	country_name_full,
    CEIL(SUM(value_)) AS total_amount,
    SUM(CEIL(SUM(value_))) OVER () AS grand_total,
    ROUND(((CEIL(SUM(value_))/SUM(CEIL(SUM(value_))) OVER ()*100)),2) AS ratio
FROM baci_hs17_23
JOIN country_codes
	ON baci_hs17_23.importer = country_codes.country_code
GROUP BY country_name_full
ORDER BY total_amount DESC
;

-- #5 TOP EXPORTERS --
SELECT
	year_,
    exporter,
    country_name_full,
    SUM(value_) AS total_export_value
FROM baci_hs17_23
JOIN country_codes
	ON baci_hs17_23.exporter = country_codes.country_code
GROUP BY year_, exporter, country_name_full
ORDER BY total_export_value DESC
;

-- #6 TOP IMPORTERS -- SUBQUERY
SELECT
	year_,
    country_name_full as country,
    total_import_value,
    rank_
FROM (
SELECT
	year_,
    country_name_full,
    SUM(value_) AS total_import_value,
    DENSE_RANK() OVER(ORDER BY SUM(value_) DESC) as rank_
FROM baci_hs17_23
JOIN country_codes
	ON baci_hs17_23.importer = country_codes.country_code
GROUP BY year_, country_name_full
ORDER BY total_import_value DESC
) as subquery_test
WHERE rank_ <= 10
-- LIMIT 10 -- Top 10 Rank --
;

-- #7 MOST FREQUENT TRADE PRODUCT --
SELECT
	year_,
    product_category,
    description,
    COUNT(*) AS frequency
FROM baci_hs17_23
JOIN product_codes
	ON baci_hs17_23.product_category = product_codes.code
GROUP BY year_, product_category, description
ORDER BY frequency DESC
;

-- (# TOP 3) MOST VALUABLE BILATERAL EXCHANGE BY (COUNTRY) AS EXPORTER -- TOP 3 + VALUE - MAYBE TRY TO COUNT AMOUNT OF TIME COUNTRY IS 1ST IMPORTER
WITH test AS (SELECT
	country_codes.country_name_full as exporter,
    ROW_NUMBER() OVER(PARTITION BY country_codes.country_name_full ORDER BY SUM(value_)DESC) as rank_nb,
    imported_country.country_name_full as importer,
    SUM(value_) as value_
FROM baci_hs17_23
JOIN country_codes
	ON baci_hs17_23.exporter = country_codes.country_code
JOIN country_codes as imported_country
	ON baci_hs17_23.importer = imported_country.country_code
GROUP BY country_codes.country_name_full, imported_country.country_name_full
ORDER BY exporter
)
SELECT
	exporter,
    importer,
    -- rank_nb,
    value_
FROM test
WHERE rank_nb <= 1
GROUP BY exporter, importer
ORDER BY exporter, value_ DESC;


-- #8 MOST FREQUENT BILATERAL EXCHANGE -- TRY TO DISPLAY TOP 10 MOST BILATERAL TRADE FOR EACH COUNTRY
-- INSIGHTS : DISTANCE - HISTORICAL PAST - LANGUAGE ARE KEY VARIABLE (EXPORTER - FREQUENCY - ROW_NUM - IMPORTER)
WITH test AS (SELECT
	country_codes.country_name_full as exporter,
    COUNT(*) as frequency,
    imported_country.country_name_full as importer,
    ROW_NUMBER() OVER (PARTITION BY country_codes.country_name_full ORDER BY COUNT(*) DESC) AS row_num
FROM baci_hs17_23
JOIN country_codes
	ON baci_hs17_23.exporter = country_codes.country_code
JOIN country_codes as imported_country
	ON baci_hs17_23.importer = imported_country.country_code
GROUP BY country_codes.country_name_full, imported_country.country_name_full
ORDER BY exporter, frequency DESC
)
SELECT
	exporter,
    frequency,
	row_num,
    importer  
FROM test
WHERE row_num <= 10
ORDER BY exporter, frequency DESC;
	
SELECT
	*
FROM baci_hs17_23;

-- #9 MOST FREQUENT BILATERAL EXCHANGE TOP 3 BY COUNTRY + VALUE -- QUERY ABOVE DISPLAY TOTAL BILATERAL VALUE BETWEEN COUNTRY (EXPORTER - IMPORTER - VALUE)
WITH test AS (SELECT
	country_codes.country_name_full as exporter,
    COUNT(*) as frequency,
    imported_country.country_name_full as importer,
    ROW_NUMBER() OVER (PARTITION BY country_codes.country_name_full ORDER BY COUNT(*) DESC) AS row_num,
    SUM(value_) as value_
FROM baci_hs17_23
JOIN country_codes
	ON baci_hs17_23.exporter = country_codes.country_code
JOIN country_codes as imported_country
	ON baci_hs17_23.importer = imported_country.country_code
GROUP BY country_codes.country_name_full, imported_country.country_name_full
ORDER BY exporter, frequency DESC
)
SELECT
	exporter,
--    frequency,
    importer,
--    row_num,
    value_
FROM test
-- WHERE row_num <= 3
GROUP BY exporter, frequency, importer, row_num
ORDER BY exporter, value_ DESC;

-- #10 DISPLAY MOST FREQUENT PRODUCT TRADE FOR TOP 3/5 FREQUENT BILATERAL TRADE PARTNER + VALUE
-- NEED TO FIND THE RIGHT QUERY --
SELECT
	country_codes.country_name_full as exporter,
    imported_country.country_name_full as importer,
    product_codes.description as product,
    value_
FROM baci_hs17_23
JOIN country_codes
	ON baci_hs17_23.exporter = country_codes.country_code
JOIN country_codes as imported_country
	ON baci_hs17_23.importer = imported_country.country_code
GROUP BY country_codes.country_name_full, imported_country.country_name_full
ORDER BY exporter, frequency DESC
LIMIT 5
;


-- # TOP 5 MOST FREQUENT PRODUCT EXPORT BY COUNTRY
WITH test_1 AS (
SELECT
	country_codes.country_name_full as exporter,
	baci_hs17_23.product_category,
    product_codes.description,
    baci_hs17_23.value_,
    imported_country.country_name_full as importer
FROM baci_hs17_23
JOIN product_codes
 	ON baci_hs17_23.product_category = product_codes.code
JOIN country_codes
	ON baci_hs17_23.exporter = country_codes.country_code
JOIN country_codes as imported_country
	ON baci_hs17_23.importer = imported_country.country_code
),
test_2 AS (
SELECT
	exporter,
    product_category,
    SUM(value_) as sum_final,
    COUNT(*) as frequency,
    ROW_NUMBER() OVER(PARTITION BY exporter ORDER BY COUNT(*) DESC) as frq_rank
FROM test_1
GROUP BY exporter, product_category
ORDER BY exporter, frequency DESC
)
SELECT
	exporter,
    product_category,
    frequency,
    frq_rank,
	-- value_rank,
    product_codes.description
FROM test_2
JOIN product_codes
	ON test_2.product_category = product_codes.code
WHERE frq_rank <= 5 AND exporter LIKE "United Arab%" -- value_rank
ORDER BY exporter, frequency DESC 
;


-- # TOP 5 MOST FREQUENT PRODUCT IMPORT BY COUNTRY
WITH test_1 AS (
SELECT
	country_codes.country_name_full as importer,
	baci_hs17_23.product_category,
    product_codes.description,
    baci_hs17_23.value_,
    exporter_country.country_name_full as exporter
FROM baci_hs17_23
JOIN product_codes
 	ON baci_hs17_23.product_category = product_codes.code
JOIN country_codes
	ON baci_hs17_23.importer = country_codes.country_code
JOIN country_codes as exporter_country
	ON baci_hs17_23.exporter = exporter_country.country_code
),
test_2 AS (
SELECT
	importer,
    product_category,
    -- SUM(value_),
    COUNT(*) as frequency,
    ROW_NUMBER() OVER(PARTITION BY importer ORDER BY COUNT(*) DESC) as frq_rank
FROM test_1
GROUP BY importer, product_category
ORDER BY importer, frequency DESC
)
SELECT
	importer,
    product_category,
    frequency,
    frq_rank,
    product_codes.description
FROM test_2
JOIN product_codes
	ON test_2.product_category = product_codes.code
WHERE frq_rank <= 5 AND importer LIKE "United Arab%"
ORDER BY importer, frequency DESC 
;

-- # TOP 5 MOST IMPORTANT PRODUCT EXPORTED BY COUNTRY (IN VALUE)
WITH test_1 AS (
SELECT
	country_codes.country_name_full as exporter,
	baci_hs17_23.product_category,
    product_codes.description,
    baci_hs17_23.value_,
    imported_country.country_name_full as importer
FROM baci_hs17_23
JOIN product_codes
 	ON baci_hs17_23.product_category = product_codes.code
JOIN country_codes
	ON baci_hs17_23.exporter = country_codes.country_code
JOIN country_codes as imported_country
	ON baci_hs17_23.importer = imported_country.country_code
),
test_2 AS (
SELECT
	exporter,
    product_category,
    SUM(value_) as sum_final,
    COUNT(*) as frequency,
    ROW_NUMBER() OVER(PARTITION BY exporter ORDER BY  SUM(value_) DESC) as value_rank
FROM test_1
GROUP BY exporter, product_category
ORDER BY exporter, frequency DESC
)
SELECT
	exporter,
    product_category,
    sum_final,
	value_rank,
    product_codes.description
FROM test_2
JOIN product_codes
	ON test_2.product_category = product_codes.code
WHERE value_rank <= 5 AND exporter LIKE "United Arab%"
ORDER BY exporter, sum_final DESC 
;

-- # TOP 5 MOST IMPORTANT PRODUCT IMPORTED BY COUNTRY (in value) - maybe adding number of country that export the product the importer to highlight dependancies/concentration
WITH test_1 AS (
SELECT
	country_codes.country_name_full as importer,
	baci_hs17_23.product_category,
    product_codes.description,
    baci_hs17_23.value_,
    exported_country.country_name_full as exporter
FROM baci_hs17_23
JOIN product_codes
 	ON baci_hs17_23.product_category = product_codes.code
JOIN country_codes
	ON baci_hs17_23.importer = country_codes.country_code
JOIN country_codes as exported_country
	ON baci_hs17_23.exporter = exported_country.country_code
),
test_2 AS (
SELECT
	importer,
    product_category,
    SUM(value_) as sum_final,
    COUNT(*) as frequency,
    ROW_NUMBER() OVER(PARTITION BY importer ORDER BY  SUM(value_) DESC) as value_rank
FROM test_1
GROUP BY importer, product_category
ORDER BY importer, frequency DESC
)
SELECT
	importer,
    product_category,
    sum_final,
	value_rank,
    product_codes.description
FROM test_2
JOIN product_codes
	ON test_2.product_category = product_codes.code
WHERE value_rank <= 5 AND importer LIKE "United Arab%"
ORDER BY importer, sum_final DESC 

;
-- # TRADE BALANCE (SUM EXPORTS AND IMPORTS) BY COUNTRY (USING CASE STATEMENT)
WITH export as (
SELECT
	country_codes.country_name_full as country,
    SUM(baci_hs17_23.value_) as total_export_value
FROM baci_hs17_23
	JOIN country_codes
		ON baci_hs17_23.exporter = country_codes.country_code
GROUP BY country
ORDER BY country
),
import as (
SELECT
	import_codes.country_name_full as country_2,
    SUM(baci_hs17_23.value_) as total_import_value
FROM baci_hs17_23
	JOIN country_codes as import_codes
	ON baci_hs17_23.importer = import_codes.country_code
GROUP BY country_2
ORDER BY country_2
)
SELECT
	export.country,
    ROUND(export.total_export_value,3),
	ROUND(total_import_value,3),
	ROUND(export.total_export_value - total_import_value,3) as trade_balance,
    CASE
		WHEN export.total_export_value - total_import_value > 0 THEN "Surplus"
        WHEN export.total_export_value - total_import_value < 0 THEN "Deficit"
        ELSE "0"
			END AS "Status"
FROM import
JOIN export
	ON import.country_2 = export.country
LIMIT 5
;
-- ADD A SUBQUERY TO CREATE A RATIO FOR AMOUNT COUNTRY IN SURPLUS AND IN DEFICIT

    
-- # NUMBER OF TRADE PARTNER BY COUNTRY (COUNTRY AS EXPORTER) -- HAVE MORE INFORMATION (CONTINENT - TRADE PARTNERSHIP)
SELECT
	country_codes.country_name_full as exporter,
-- baci_hs17_23.product_category,
 --   product_codes.description,
 --   baci_hs17_23.value_,
    COUNT(DISTINCT imported_country.country_name_full) as nb_importer_partners
FROM baci_hs17_23
-- JOIN product_codes
-- 	ON baci_hs17_23.product_category = product_codes.code
JOIN country_codes
	ON baci_hs17_23.exporter = country_codes.country_code
JOIN country_codes as imported_country
	ON baci_hs17_23.importer = imported_country.country_code
GROUP BY country_codes.country_name_full
ORDER BY nb_importer_partners DESC
;

-- # NUMBER OF TRADE PARTNER BY COUNTRY (COUNTRY AS IMPORTER) -- HAVE MORE INFORMATION (CONTINENT - TRADE PARTNERSHIP)
SELECT
	country_codes.country_name_full as importer,
-- baci_hs17_23.product_category,
 --   product_codes.description,
 --   baci_hs17_23.value_,
    COUNT(DISTINCT exported_country.country_name_full) as nb_exporter_partners
FROM baci_hs17_23
-- JOIN product_codes
-- 	ON baci_hs17_23.product_category = product_codes.code
JOIN country_codes
	ON baci_hs17_23.importer = country_codes.country_code
JOIN country_codes as exported_country
	ON baci_hs17_23.exporter = exported_country.country_code
GROUP BY (country_codes.country_name_full)
ORDER BY nb_exporter_partners DESC
;

-- # HIGHLIGHT PRODUCT BY CATEGORY (USING CASE STATEMENT) - SUM ALL AND FIND RATIO PRODUCT CONCENTRATION (PRODUCT BY CATEGORY)
SELECT
	*
FROM product_codes
WHERE code > 880260
;
    
	
-- MOST EXCHANGE PRODUCT (IN VALUE) --
SELECT
	product_category,
    SUM(value_) AS total_product_sum_value,
    product_codes.description
FROM baci_hs17_23
JOIN product_codes
	ON baci_hs17_23.product_category = product_codes.code
GROUP BY product_category, product_codes.description
ORDER BY SUM(value_) DESC
LIMIT 5
;

-- PRODUCT TRADE BY COUNTRY --
SELECT
	exporter,
    country_codes.country_name_full as country,
    product_category,
    product_codes.description as description
FROM baci_hs17_23
JOIN country_codes
	ON baci_hs17_23.exporter = country_codes.country_code
JOIN product_codes
	ON baci_hs17_23.product_category = product_codes.code
WHERE country_codes.country_name_full = "Mali"
ORDER BY product_category
; 


```
### Results/Findings

### Recommendations

### Limitations

### References

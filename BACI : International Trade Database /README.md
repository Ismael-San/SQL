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

### Results/Findings

### Recommendations

### Limitations

### References

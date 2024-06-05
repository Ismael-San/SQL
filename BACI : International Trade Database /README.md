# BACI : International Trade Database Analysis

### Table of Contents

### Introduction

### Project Overview
The main objective of this project was to extract insights from international trade products between countries using BACI (Base pour l'Analyse du Commerce International), a bilateral trade flow dataset published by Guillaume Gaulier and Soledad from the CEPII, based on data collected from the United Nations COMTRADE. Several reasons explain the computation of BACI by Gaulier and Soledad:

1. Incomplete Data: COMTRADE has missing values/flows in the country reports sent to the United Nations,       leading to incomplete data.
2. Classification Issues: Some countries use the older SITC classification instead of the current Harmonized   System.
3. Data Discrepancies: There are differences in the data provided by importers and exporters for the same trade flow. One solution to this issue was removing the trade cost value (CIF) from import values.


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
``` sql

```

### Exploratory Data Analysis

### Data Analysis

### Results/Findings

### Recommendations

### Limitations

### References

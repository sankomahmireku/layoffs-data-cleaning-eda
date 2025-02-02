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


# AWS Academy Data Engineering Capstone Project

## Overview and Objectives

This capstone project challenges building a solution using multiple AWS services without step-by-step guidance. It involves launching and configuring an AWS Cloud9 IDE, transforming CSV data to Parquet format and uploading to S3, creating Glue crawlers, querying with Athena, creating views, and analyzing data.

The project uses the Sea Around Us dataset, which provides historical fisheries information globally from 1950 to 2018, including yearly catches, countries, areas, tonnes, and values in 2010 US dollars.


### 1. Configuring the Development Environment
- Launched and configured an AWS Cloud9 integrated development environment (IDE) instance named CapstoneIDE, using a t2.micro EC2 instance in the Capstone VPC.
- Created two S3 buckets in us-east-1: `data-source-lmasb0` for data files and `query-results-lmasb0` for Athena query outputs.
- Downloaded three CSV source data files from Sea Around Us: SAU-GLOBAL-1-v48-0.csv (global high seas), SAU-HighSeas-71-v48-0.csv (Pacific Western Central), SAU-EEZ-242-v48-0.csv (Fiji EEZ).
- Observed the data structure using `head` command, noting columns like year, fishing_entity, tonnes, landed_value.
- Installed pandas, pyarrow, fastparquet on Cloud9.
- Converted SAU-GLOBAL-1-v48-0.csv to Parquet format using Python and pandas.
- Uploaded the Parquet file to the data-source S3 bucket using AWS CLI.

![Screenshot of AWS console access](<Screenshot 2025-12-15 at 11.38.58.png>)

### 2. Using AWS Glue Crawler and Querying Multiple Files with Athena
- Connected to the data source and prepared for ingestion into the data lake. 


![Screenshot of data lake connection](<Screenshot 2025-12-15 at 11.40.18.png>)

### 3. Transforming a New File and Adding to the Dataset
- Created an AWS Glue database named `fishdb`.
- Created an AWS Glue crawler named `fishcrawler` using CapstoneGlueRole IAM role to crawl the data-source S3 bucket and output to fishdb.
- Ran the crawler to infer schema and create table metadata (e.g., `data_source_lmasb0`).
- Verified the table structure and ran initial queries in Athena to confirm data categorization, such as `SELECT DISTINCT area_name FROM fishdb.data_source_lmasb0;`.

![Screenshot of table schema exploration](<Screenshot 2025-12-15 at 11.41.20.png>)
![Screenshot of data preview](<Screenshot 2025-12-15 at 11.50.58.png>)
![Screenshot of column details](<Screenshot 2025-12-15 at 12.01.29.png>)
![Screenshot of initial query development](<Screenshot 2025-12-15 at 22.44.57.png>)


### 5. Data Transformation
- Analyzed the SAU-EEZ-242-v48-0.csv file, noting column name mismatches (fish_name vs common_name, country vs fishing_entity).
- Used pandas to rename columns and convert to Parquet format.
- Uploaded the transformed EEZ file to the data-source bucket.
- Re-ran the Glue crawler to update table metadata with new columns.

![Screenshot of transformation application](<Screenshot 2025-12-15 at 22.54.21-1.png>)
![Screenshot of casting operations](<Screenshot 2025-12-15 at 23.07.37.png>)

### 4. Analysis and Documentation
- Executed various queries in Athena, including those for open seas and EEZ catches by Fiji.
- Created Athena views, such as the challenge view for Fiji's all high seas catch, and MackerelsCatch view for mackerel data analysis.
- Ran queries on views to analyze trends, like top mackerel catch by country per year.

![Screenshot of query execution](<Screenshot 2025-12-15 at 23.07.37-1.png>)
![Screenshot of initial results](<Screenshot 2025-12-15 at 23.10.12.png>)
![Screenshot of error resolution](<Screenshot 2025-12-15 at 23.12.12.png>)
![Screenshot of successful query run](<Screenshot 2025-12-15 at 23.15.28.png>)
![Screenshot of optimized execution](<Screenshot 2025-12-15 at 23.16.13.png>)
![Screenshot of SQL query writing](<Screenshot 2025-12-15 at 22.50.05.png>)
![Screenshot of filtering logic](<Screenshot 2025-12-15 at 22.54.21.png>)
![Screenshot of aggregation query](<Screenshot 2025-12-15 at 23.17.00.png>)


- Aggregated the results by `year` and `entity` to identify trends.



![Screenshot of trend identification](<Screenshot 2025-12-15 at 23.18.44.png>)
![Screenshot of analysis results](<Screenshot 2025-12-15 at 23.19.10.png>)


In addition to queries specified by the project description i have created the following queries
#### Custom Query 1: Top 5 Fishing Entities by Total Historical Value
**Goal:** Identify which countries/entities have generated the highest total revenue in the dataset's history. This helps establish the major market players.
![Screenshot of custom query 2 execution](<Screenshot 2025-12-15 at 23.44.38.png>)


```sql
SELECT 
    fishing_entity AS Country, 
    CAST(SUM(landed_value) AS DECIMAL(38,2)) AS Total_Landed_Value
FROM fishdb.data_source_lmasb0
GROUP BY fishing_entity
ORDER BY Total_Landed_Value DESC
LIMIT 5;
```

#### Custom Query 2: Most Valuable Fishing Areas (Pacific vs. Atlantic vs. Indian)
**Goal:** To determine which ocean region yields the highest economic return, filtering specifically for the major oceans.



```sql
SELECT 
    area_name, 
    CAST(AVG(landed_value) AS DECIMAL(38,2)) AS Average_Catch_Value
FROM fishdb.data_source_lmasb0
WHERE area_name LIKE '%Pacific%' 
   OR area_name LIKE '%Atlantic%' 
   OR area_name LIKE '%Indian%'
GROUP BY area_name
ORDER BY Average_Catch_Value DESC;
```

#### Custom Query 3: Global Economic Trend (Year-over-Year)
**Goal:** To analyze the global health of the fishing economy by tracking the total landed value across all entities for every year available.


![Screenshot of trend analysis results](<Screenshot 2025-12-15 at 23.45.41.png>)

```sql
SELECT 
    year, 
    CAST(SUM(landed_value) AS DECIMAL(38,2)) AS Global_Annual_Value
FROM fishdb.data_source_lmasb0
GROUP BY year
ORDER BY year DESC;
```

## Architecture
- **Initial Setup:** Cloud9 IDE, S3 buckets.
- **Final Architecture:** S3 for data storage, Glue for cataloging, Athena for querying, with  a view for analysis.

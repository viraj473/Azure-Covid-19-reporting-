# Azure Data Factory COVID-19 Reporting

### Concept of the Project 💡
- This project involves the acquisition of several Covid-19 Datasets from the ECDC website. These datasets are subsequently processed through diverse ADF components to effect transformations. These transformations are executed using ADF, HDInsight, and Databricks. The resultant data is then loaded into SQL Datawarehouse with the intention of enabling the Analytics team to draw meaningful and practical insights from these datasets. The primary objective is to comprehensively understand the influence of Covid on the entirety of the European Region throughout the course of the year 2020.

### Task 🎯
- This project's mission is to ingest data from multiple data sources, clean it up, and alter it so that it is more robust and suitable for the goal. Once the data has been cleaned, it should be imported into a central repository, such as a data warehouse or datalake, so that the analytics team can utilize Power BI and other BI tools to access it. The data warehouse will contain information on confirmed cases, regrettable death rates, hospitalization and ICU cases from our weekly data lake counts, as well as testing statistics. Additionally, we can utilize these data to construct ML Models that forecast the spread of COVID in the European region.

### Source Data: 📤
- ECDC (https://www.ecdc.europa.eu/en/covid-19)
- Population Data From Azure Blob Storage (eurostat_data)

### Destination: 📥📍
- Azure Data Lake Gen2 Storage.

## Tools ⚙

### Data Integration/Ingestion

- ADF Data Flows within the Data Factory

### Transformation

- Data Flows within the Data Factory
- DataBricks

### Data Warehouse Solution

- Azure SQL Database

### Visualization

- Power BI Desktop
- Power BI Service

### All the steps performed in this project are available as images in the ...... folder in this repository.

### Approach

### Environment Setup
- Azure Subscription
- Data Factory
- Azure Blob Storage Account
- Data Lake Storage Gen2
- Azure SQL Database
- Azure Databricks Cluster
- HD Insight Cluster

# Solution Architecture Overview
![Solution_architecture](https://github.com/user-attachments/assets/600a3a49-b863-4168-8a5b-baa2979897d1)

### DATA EXTRACTION/ INGESTION
Four different datasets were ingested from both the ECDC website and azure blob storage into Datalake Gen2. They are - 

- Cases and Deaths Data
- Hospital Admissions Data
- Population Data
- Test Conducted Data

We used various components of ADF Pipeline activities to ingest the data from both HTTP Data Source and Azure Storage Account to Azure DataLake. Some of those activities are:

- Validation Activity
- Get Metadata Activity
- Copy Activity

### Population Data : Load into Storage Account and move it to Destination Data Lake
Ingest "population by age" data for all EU Countries into the Data Lake to support the machine learning models with the data to predict an increase in Covid-19 mortality rate.

### Solution Flow

![Module_Solution](https://github.com/user-attachments/assets/45fd8ce3-50e4-4051-84c3-8f3b247b5bda)

### Steps:
1. Create a Linked Service To Azure Blob Storage
2. Create a Source Data Set
3. Create a Linked Service To Azure Data Lake storage (GEN2)
4. Create a Sink Data set
5. Create a Pipeline:
- Execute Copy activity when the file becomes available
- Check metadata counts before loading the data using the IF Condition
- Finally Load Data into our destination
6. ScheduleTrigger

### ECDC Data from Web to Destination Data Lake

### ECDC Data Content - Four files of CSV :
1. Case & Deaths Data.csv
2. Hospital Admission Data.csv
3. testing.csv
4. country_response.csv


### Solution Flow
![image](https://github.com/user-attachments/assets/d268350a-abe0-4a30-9a39-bea9a5e90f1b)


Steps:
1. Create a Linked Service using an HTTP connector
2. Create a Source Data Set
3. Create a Linked Service To Azure Data Lake storage (GEN2)
4. Create a Sink Data set
5. Create a Pipeline With Parameters & Variables
6. Lookup to get all the parameters from json file, then pass it to ForEach ECDC DATA as shown below
7. Schedule Trigger

![image](https://github.com/user-attachments/assets/14fbe052-fca7-447b-a209-8e436914bc80)

# 2. DATA TRANSFORMATION

The Cases and Deaths data together with the Hospital admissions data was transformed using ADF Data flows. The Data Flows transformation used on both dataset include

- Select transformation
- Lookup transformation
- Filter transformation
- Join transformation
- Sort transformation
- Conditional split transformation
- Derived columns transformation
- Sink transformation

# Data Flows (1) Cases & Deaths Data:

### Solution Flow
![image](https://github.com/user-attachments/assets/d4404640-2841-4794-991b-56e5e22925dc)


### Steps:
1. Cases And Deaths Source (Azure Data Lake Storage Gen2 )
2. Filter Europe-Only Data
3. Select only the required columns
4. PivotCounts using indicator Columns(confirmed cases, deaths) and get the sum of daily cases count
5. Lookup Country to get country_code_2_digit,country_code_3_digit columns
6. Select Only the required columns for the Sink
7. Create a Sink dataset (Azure Data Lake Storage Gen2)
8. Used Schedule Trigger

# Data Flows (2) Hospital Admissions Data:

### Solution Flow
![image](https://github.com/user-attachments/assets/214ccb49-9447-4488-91c1-7afbe9034a4e)


### Steps:
1. Hospital Admissions Source (Azure Data Lake Storage Gen2 )
2. Select only the required columns
3. Lookup Country to get country_code_2_digit,country_code_3_digit columns
4. Select only the required columns
5. Condition Split Weekly, Daily Split condition
  - indicator=='Weekly new hospital admissions per 100k' || indicator=='Weekly new ICU admissions per 100k'
  - indicator== "Daily hospital occupancy" || indicator=="Daily ICU occupancy"
6. For Weekly Path
- Join with Date to get ecdc_Year_week, week_start_date, week_End_date
- Piovt Counts using indicator Columns(confirmed cases, deaths) and get the sum of daily cases count
- Sort data using reported_year_week ASC and Country DESC
- Select only required columns for sink
- Create a sink dataset (Azure Data Lake Storage Gen2)
- Schedule Trigger
7. For Daily Path
- Pivot Counts using indicator Columns(confirmed cases, deaths) and get the sum of daily cases count
- Sort Data using reported_year_week ASC and Country DESC
- Select only required columns for sink
- Create a sink dataset (Azure Data Lake Storage Gen2)
- Used Schedule Trigger

![image](https://github.com/user-attachments/assets/6c1da2fd-32fc-4791-a00a-f755b5d5c304)


# Databricks Activity (3) -- Population File:

![image](https://github.com/user-attachments/assets/91cbd9ec-193f-4086-bba6-bc0da8423f30)


# 3. Copy Data to Azure SQL
1- Copy Cases and Deaths
2- Copy hospital admissions data
3- Copy testing data

![image](https://github.com/user-attachments/assets/ecbb2436-5e46-4bc9-9da1-a268f370a32a)

![image](https://github.com/user-attachments/assets/cce37126-8c84-4df8-b90a-48800103c818)




# 4. Reporting via Power BI

1. Create a Connection from Azure SQL to Power Bi and load the data
2. Analyze the data to get the total confirmed cases and deaths count
3. Identify the trends in data based on reporting date
4. Publish the report to Power BI Server
5. Publish to web

# Covid-19 Trend in the EU/EEA & UK 2020 by Cases, Deaths, Hospital Occupancy, and ICU Occupancy

![image](https://github.com/user-attachments/assets/74260b22-0d92-44f2-a654-08ce411efaa9)


# Covid-19 Cases and Death breakdown by population in the UK, France, and Germany

![image](https://github.com/user-attachments/assets/13787cbf-c0c4-4fe6-a37a-51571313c136)



# Total Number of covid tests carried out vs Confirmed Cases

![image](https://github.com/user-attachments/assets/2f31643b-123e-4414-9383-74e5fbf89e4b)





# Used Technologies
- Azure DataFactory
- Azure HDinsight (Hive)
- Azure Databricks (Pyspark, SparkSql)
- Azure Storage Account
- Azure Data Lake Gen2
- Azure SQL Database

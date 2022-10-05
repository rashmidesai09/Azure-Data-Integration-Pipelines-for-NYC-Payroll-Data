# Azure-Data-Integration-Pipelines-for-NYC-Payroll-Data

## Overview 
This projects creates data pipelines for a  Data Analytics platform that are dynamic, can be automated, and monitored for efficient operation. The source data resides in Azure Data Lake Gen2 and is processed in a NYC data warehouse in Azure Synapse Analytics. The source datasets consist of CSV files with Employee master data and monthly payroll data entered by various City agencies.

Two primary objectives:

 1. Analyze how the City's financial resources are allocated and how much of the City's budget is being devoted to overtime.
 2. Make the data available to the interested public to show how the City’s budget is being spent on salary and overtime pay for all municipal employees.
 

## Schema 
<img width="597" alt="image" src="https://user-images.githubusercontent.com/97893144/194106049-43ba9943-b74a-48aa-b5d1-ac44d6b2f7d5.png">

## Project steps
### Step 1: Data Infrastructure Preparation
1. Azure data lake Gen2 to be created and data to be uploaded

   Azure Data Lake Storage Gen2 (storage account) and associated storage container resource named adlsnycpayroll-yourfirstname-lastintial along with three directories    in this storage container named dirpayrollfiles , dirhistoryfiles , dirstaging created.
   Files named EmpMaster.csv, AgencyMaster.csv , TitleMaster.csv, nycpayroll_2021.csv uploaded to the dirpayrollfiles folder
   File named nycpayroll_2020.csv (historical data) uploaded to dirhistoryfiles folder

2. Azure Data Factory Resource is created

3. SQL Database to store the current year of the payroll data is created
   SQL Database resource named db_nycpayroll is created
   Table called NYC_Payroll_Data created in db_nycpayroll in the Azure Query Editor with this SQL script
   <img width="358" alt="image" src="https://user-images.githubusercontent.com/97893144/194109808-277dac7f-4996-46be-84f8-3738f21923cf.png">

4. Synapse Analytics workspace is created
   A new Azure Data Lake Gen2 is created in the Synapse Analytics workspace along with SQL dedicated pool (DW100c as performance level).
   Below tables are created- Emplyee Master Data, Job Title , Agency Master, Payroll transaction data
   
   <img width="274" alt="image" src="https://user-images.githubusercontent.com/97893144/194113605-67fad80d-0784-4e73-a02e-cd3acc0b95bf.png">  
   <img width="274" alt="image" src="https://user-images.githubusercontent.com/97893144/194113682-0dbcab0d-9eb5-4859-8ea0-4f6bc25ae5c4.png">
   <img width="317" alt="image" src="https://user-images.githubusercontent.com/97893144/194114021-bcdfa029-4919-42e9-91ab-243b505eb3f9.png">
   <img width="329" alt="image" src="https://user-images.githubusercontent.com/97893144/194114079-54a69d88-0436-4b30-96dc-16389233e6d1.png">
   
### Step 2: Linked Services creation

1. Linked Service for Azure Data Lake, SQL Database that has the current (2021) data and Synapse Analytics is created.

### Step 3: Datasets creation in Azure Data Factory 

1. Datasets for the 2021 Payroll file on Azure Data Lake Gen2cis created by selecting DelimitedText and the path is set to the nycpayroll_2021.csv in the Data Lake
   Data is previewed to ensure it is correctly parsed
2. Same process repated to create datasets for the rest of the data files in the Data Lake - EmpMaster.csv , TitleMaster.csv, AgencyMaster.csv
3. Dataset for transaction data table that should contain current (2021) data in SQL DB is created
4. Finally, the datasets for destination (target) tables in Synapse Analytics is created i,e dataset for NYC_Payroll_EMP_MD, NYC_Payroll_TITLE_MD,
   NYC_Payroll_AGENCY_MD, NYC_Payroll_Data

### Step 4 : Data Flows creation

1. In Azure Data Factory, the data flow created to load 2021 Payroll Data to SQL DB transaction table (in the future NYC will load all the transaction data into this table).
2. Pipeline created to load 2021 Payroll data into transaction table in the SQL DB
3. Data flows created to load the data from the data lake files into the Synapse Analytics data tables
   i.e the data flows for loading Employee, Title, and Agency files into corresponding SQL pool tables on Synapse Analytics for each Employee, Title, and Agency file          sink the data into each target Synaspe table
4. Data flow created to load 2021 data from SQL DB to Synapse Analytics
5. Pipelines created for Employee, Title, Agency, and year 2021 Payroll transaction data to Synapse Analytics containing the data flows.
6. Trigger and monitor the Pipelines

### Step 5: Data Aggregation and Parameterization

1. A Summary table in Synapse is created with the following SQL script along with a dataset named table_synapse_nycpayroll_summary
   <img width="291" alt="image" src="https://user-images.githubusercontent.com/97893144/194117189-9ffe6291-e916-42d3-af12-d47a9354920e.png">

2. A new dataset created for the Azure Data Lake Gen2 folder that contains the historical files (dirhistoryfiles selected in the data lake as the source)

3. New data flow created and called as Dataflow Aggregate Data
    A data flow level parameter for Fiscal Year created, first Source added for table_sqldb_nyc_payroll_data table , second Source added for the Azure Data Lake           history folder
  
4. A new Union activity in the data flow created along with the Union with history files

5. Filter activity after Union is added (In Expression Builder, enter toInteger(FiscalYear) >= $dataflow_param_fiscalyear )

6. New TotalPaid column derived (In Expression Builder, enter RegularGrossPaid + TotalOTPaid+TotalOtherPay)

7. An Aggregate activity added to the data flow next to the TotalPaid activity (Under Group By, AgencyName and Fiscal Year selected)

8. Sink activity added to the Data Flow
  (Selected the dataset to target (sink) the data into the Synapse Analytics Payroll Summary table, selected Truncate Table in settings)
  
9. A new Pipeline created and added the Aggregate data flow
   (Created a new Global Parameter (This will be the Parameter at the global pipeline level that will be passed on to the data flow
   In Parameters, selected Pipeline Expression by Choosing the parameter created at the Pipeline level)
   
10. Final step is to Validate, Publish and Trigger the pipeline. Enter the desired value for the parameter.

11. Monitor the Pipeline run 
![image](https://user-images.githubusercontent.com/97893144/194118858-00c13912-96cb-4f69-8884-0ba506c8f5fa.png)

### Step 6 : Connected the Azure Data Factory to Github and published all the objects created




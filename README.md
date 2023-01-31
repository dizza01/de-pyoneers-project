## Project Summary

ETL pipline that migrates data from OLTP Database to OLAP Datawarehouse



## The Minimum Viable Product (MVP)
- Two S3 buckets (one for ingested data and one for processed data). Both buckets should be structured and well-organised so that data is easy to find.
- A Python application that ingests all tables from the source database (details below). The data should be saved in files in the "ingestion" S3 bucket in a suitable format. The application must:
  - operate automatically on a schedule
  - log progress to Cloudwatch
  - follow good security practices (for example, preventing SQL injection and maintaining password security)
- A Python application that remodels data into a predefined schema suitable for a data warehouse and stores the data in `parquet` format in the "processed" S3 bucket. The application must:
  - trigger automatically when it detects the completion of an ingested data job
  - be adequately logged and monitored
- A Python application that loads the data into a prepared data warehouse at defined intervals. Again the application should be adequately logged and monitored.




## Technical Details
The primary data source for the project is a moderately complex (but not very large) database called `totesys` which is meant to simulate the back end data of a commercial application. Data is inserted and updated into this database several times a day.


The solution is hosted with a special AWS sandbox.



The overall structure of the resulting data warehouse is shown [here](https://dbdiagram.io/d/637b4c6dc9abfc6111741e65).

**Components**
1. A job scheduler to run the ingestion job. AWS Eventbridge is used for this. 
2. An S3 bucket which acts as a "landing zone" for ingested data.
3. A Lambda Function that checks for the changes to the database tables and ingest any new or updated data. AWS Lambda is the computing solution for this. The data should be saved in the "ingestion" S3 bucket in a suitable format. Status and error messages should be logged to Cloudwatch.
4. A second S3 bucket for "processed" data.
5. A Lambda Function to transform data landing in the "ingestion" S3 bucket and place the results in the "processed" S3 bucket. The data is transformed to conform to the warehouse schema (see below). The job triggered on a schedule. 
6. A Lambda Function that periodically schedules an update of the data warehouse from the data in S3.

The tables that are ingested from the source database are:
|tablename|
|----------|
|counterparty|
|currency|
|department|
|design|
|staff|
|sales_order|
|address|
|payment|
|purchase_order|
|payment_type|
|transaction|

The list of tables in the complete warehouse is:
|tablename|
|---------|
|fact_sales_order|
|fact_purchase_orders|
|fact_payment|
|dim_transaction|
|dim_staff|
|dim_payment_type|
|dim_location|
|dim_design|
|dim_date|
|dim_currency|
|dim_counterparty|



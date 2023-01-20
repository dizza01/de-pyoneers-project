Introduction

The directory overall, is built to deploy three specific lambda functions onto aws using the deployment script, the functionality of the three lambda fucntion are to extract, load, and transform respectively.

To use this script, you will need to have AWS credentials set up and added to GitHub secrets.

First, create an instance of the Lambda_script class, passing in the necessary arguments. Then, call the deploy method on the instance to deploy the Lambda function.

deployer = Lambda_script(lambda_function_path='path/to/lambda_function.py', 
                         zip_file_name='lambda_function.zip', 
                         function_name_template='my_lambda_function', 
                         schedule='0 0 * * *')
deployer.deploy()




_________________________________________________________________________________


## The Minimum Viable Product (MVP)
- Two S3 buckets (one for ingested data and one for processed data). Both buckets should be structured and well-organised so that data is easy to find.
- A Python application that ingests all tables from the `totesys` database (details below). The data should be saved in files in the "ingestion" S3 bucket in a suitable format. The application must:
  - operate automatically on a schedule
  - log progress to Cloudwatch
  - trigger email alerts in the event of failures
  - follow good security practices (for example, preventing SQL injection and maintaining password security)
- A Python application that remodels data into a predefined schema suitable for a data warehouse and stores the data in `parquet` format in the "processed" S3 bucket. The application must:
  - trigger automatically when it detects the completion of an ingested data job
  - be adequately logged and monitored
- A Python application that loads the data into a prepared data warehouse at defined intervals. Again the application should be adequately logged and monitored.




## Technical Details
The primary data source for the project is a moderately complex (but not very large) database called `totesys` which is meant to simulate the back end data of a commercial application. Data is inserted and updated into this database several times a day. (The data itself is entirely fake and meaningless, as a brief inspection will confirm.)


The solution is hosted with a special AWS sandbox.



The overall structure of the resulting data warehouse is shown [here](https://dbdiagram.io/d/637b4c6dc9abfc6111741e65).

### Required Components
1. A job scheduler to run the ingestion job. AWS Eventbridge is the recommended way to do this. Since data has to be visible in the data warehouse within 30 minutes from being written to the database, you need to schedule a your job to check for changes much more frequently.
1. An S3 bucket which will act as a "landing zone" for ingested data.
1. A Python application to check for the changes to the database tables and ingest any new or updated data. It is strongly recommended that you use AWS Lambda as your computing solution. It is possible to use EC2, but it will be much harder to create event-driven jobs, and harder to log events in Cloudwatch. The data should be saved in the "ingestion" S3 bucket in a suitable format. Status and error messages should be logged to Cloudwatch.
1. A Cloudwatch alert should be generated in the event of a major error - this should be sent to email or Slack.
1. A second S3 bucket for "processed" data.
1. A Python application to transform data landing in the "ingestion" S3 bucket and place the results in the "processed" S3 bucket. The data should be transformed to conform to the warehouse schema (see below). The job should be triggered by either an S3 event triggered when data lands in the ingestion bucket, or on a schedule. Again, status and errors should be logged to Cloudwatch, and an alert triggered if a serious error occurs.
1. A Python application that will periodically schedule an update of the data warehouse from the data in S3. As before

The tables that are ingested from the `totesys` source database are:
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



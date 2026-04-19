# Serverless-CSV-Data-Pipeline-on-AWS
This repository contains the architecture and implementation details for an automated ETL (Extract, Transform, Load) pipeline. The system processes raw CSV files uploaded to Amazon S3 and prepares them for visualization in Amazon QuickSight using AWS Lambda, Glue, and Athena.

🏗️ Architecture Overview
The pipeline follows a structured flow from raw data ingestion to final visualization:
1. **Ingestion:** Raw CSV files are uploaded to the csv-raw-data S3 bucket.

2. **Preprocessing:** An S3 event triggers an AWS Lambda function that cleans, filters, or re-formats the raw data.

3. **Intermediate Storage:** Preprocessed data is stored in the csv-processed-data S3 bucket.

4. **Cataloging:** An AWS Glue Crawler scans the processed data to infer the schema and update the AWS Glue Data Catalog.

5. **ETL Transformation:** An AWS Glue Job performs complex transformations (e.g., changing schemas or joining datasets) and writes the
final output to the csv-final-data bucket.
 
6. **Visualization:** Amazon QuickSight connects to the final dataset to create interactive dashboards and business reports.


<img width="1412" height="574" alt="image" src="https://github.com/user-attachments/assets/420e5fa9-7cf9-47cb-907e-d7f3c1e4307c" />


    
🚀 # **Key Components**

**Amazon S3:** Scalable object storage acting as the data lake's landing, processing, and gold zones.

**AWS Lambda:** Event-driven compute used for lightweight initial data validation and cleaning.

**AWS Glue:** A fully managed ETL service that automates data discovery and transformation.

**Amazon QuickSight:** A cloud-scale business intelligence (BI) service for data visualization.

**IAM Role and Policies:** Ensure secure access to S3, Lamnda, Glue and Quicksight... (Permmission)

## Step 1: Create 3 S3 Buckets

1. Go to S3 Console > Create bucket.
2. Create: csv-raw-data, csv-processed-data, and csv-final-data

<img width="1346" height="353" alt="image" src="https://github.com/user-attachments/assets/62393e69-266c-43cd-a161-64680b29c83c" />


## Step 2:

Create an IAM Role for Lambda:

Go to the IAM Console → Roles → Create role.
Select AWS Service as the trusted entity and choose Lambda.

<img width="1344" height="520" alt="Screenshot 2026-04-18 155638" src="https://github.com/user-attachments/assets/f26c70ff-576b-404f-b398-a9537d640557" />

. AmazonS3FullAccess (to read/write S3 buckets).
. AWSGlueServiceRole (for Glue operations).

<img width="1345" height="437" alt="image" src="https://github.com/user-attachments/assets/a239ab07-f3fb-4b1f-8f9d-22f44562a6b1" />


🔒 #@ Security Note: Attach only the permissions necessary for your pipeline to reduce security risks.


## Step 3: Setup QuickSight Visualization 

Go to QuickSight > New Dataset.
Choose S3 or Athena (Athena is easier if you use the Glue Catalog)

<img width="1359" height="640" alt="Screenshot 2026-04-18 172511" src="https://github.com/user-attachments/assets/cfdcb695-068d-41e0-b58f-
 99e3014215dc" />



## Step 2: Configure the Lambda Function 

Create Function: Choose "Author from scratch" and select Python 3.

<img width="1103" height="621" alt="image" src="https://github.com/user-attachments/assets/2b9f8024-6a33-4e59-8f32-e0daa31fceee" />

⚙️ Environment Variables (Lambda) 
Set these in the Configuration > Environment variables tab of your Lambda function. 

<img width="1071" height="476" alt="image" src="https://github.com/user-attachments/assets/0ab53dc8-9e5f-48dc-b5b3-4667832ff1af" />










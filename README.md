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


## This AWS Lambda function is designed to automatically process CSV files uploaded to an S3 bucket. Here’s how it works in simple terms:

1. Triggered by S3 Upload — Whenever a file is uploaded to a specific S3 bucket (csv-raw-data), this function runs automatically.

2. Reads the File — It fetches the CSV file from S3 and reads its content.

3. Cleans the Data — The function removes rows that contain missing values, keeping only complete rows.
   
4. Creates a New CSV — It writes the cleaned data into a new CSV file in memory.

5. Uploads the Processed File — Finally, the function saves the cleaned file into a different S3 bucket (csv-processed-data).


   <img width="1066" height="546" alt="image" src="https://github.com/user-attachments/assets/98aae8bf-2470-413c-bf9e-9824fc4b2cbd" />



```bash
import json
import boto3
import pandas as pd
import os
import io
import urllib.parse

# Initialize S3 client using the REGION_NAME from your environment variables
s3_client = boto3.client('s3', region_name=os.environ.get('REGION_NAME', 'us-east-1'))

def lambda_handler(event, context):
    try:
        # 1. Get the bucket and file key from the S3 trigger event
        source_bucket = event['Records']['s3']['bucket']['name']
        file_key = urllib.parse.unquote_plus(event['Records']['s3']['object']['key'], encoding='utf-8')
        
        # 2. Pull the target bucket name directly from your Environment Variables
        target_bucket = os.environ['PROCESSED_BUCKET']
        
        print(f"Processing: {file_key} from {source_bucket}")

        # 3. Read the CSV file into a DataFrame
        response = s3_client.get_object(Bucket=source_bucket, Key=file_key)
        df = pd.read_csv(io.BytesIO(response['Body'].read()))
        
        # 4. DATA CLEANING: Example steps
        # - Remove rows that are entirely empty
        # - Fill any remaining empty cells with 'N/A'
        df_cleaned = df.dropna(how='all').fillna('N/A')
        
        # 5. Convert cleaned data back to CSV string
        csv_buffer = io.StringIO()
        df_cleaned.to_csv(csv_buffer, index=False)
        
        # 6. Upload to csv-processed-data-ayo
        target_key = f"processed_{file_key}"
        s3_client.put_object(
            Bucket=target_bucket, 
            Key=target_key, 
            Body=csv_buffer.getvalue()
        )
        
        print(f"SUCCESS: {target_key} uploaded to {target_bucket}")
        
        return {
            'statusCode': 200,
            'body': json.dumps(f"Cleaned file {target_key} is ready!")
        }

    except Exception as e:
        print(f"ERROR: {str(e)}")
        return {
            'statusCode': 500,
            'body': json.dumps("Error during CSV processing.")
        }

```

## Setup S3 Event Trigger for Lambda

Now that our Lambda function is set up, we need to configure the raw data S3 bucket to automatically trigger the Lambda function whenever a new file is uploaded in the bucket.

. Go to the S3 Console and select your csv-raw-data-ayo bucket.
. Navigate to the Properties tab and scroll down to Event notifications.

<img width="1347" height="338" alt="Screenshot 2026-04-18 162604" src="https://github.com/user-attachments/assets/99746883-7645-4b5a-8017-89a33536ccb7" />

<img width="1365" height="545" alt="Screenshot 2026-04-18 162945" src="https://github.com/user-attachments/assets/b98488c5-102e-4345-8641-07d4e124cc78" />

<img width="1346" height="550" alt="Screenshot 2026-04-18 163020" src="https://github.com/user-attachments/assets/9f80d98a-74f8-4100-ad0f-016db7413979" />

<img width="1347" height="530" alt="Screenshot 2026-04-18 163057" src="https://github.com/user-attachments/assets/7edc98ea-c7d1-4c0f-86ff-e517fa173b48" />


## Step 4:

Upload a Sample CSV File:

Go to the S3 Console and navigate to your csv-raw-data-ayo bucket.

<img width="1354" height="438" alt="image" src="https://github.com/user-attachments/assets/837958a7-2caa-4807-a2b9-cf5b6aedaf78" />


 ## Step 5:
 
 Setup an AWS Glue Data Catalog

 . Navigate to the AWS Glue Console.
 . Click Data Catalogs → Databases → Add Database.
 . Provide the following details:
 . Database Name: csv_data_pipeline_catalog
 . Click Create database.

 
<img width="1353" height="439" alt="image" src="https://github.com/user-attachments/assets/1eb6a70c-401f-4451-ba24-54583621bf47" />


## Step 6:

 Create a Crawler to Discover Data Schema

 . Go to Crawlers and click Create Crawler.

 <img width="1333" height="627" alt="Screenshot 2026-04-18 173846" src="https://github.com/user-attachments/assets/36d23512-d761-44f6-a614-82c47e44121b" />


<img width="1350" height="550" alt="image" src="https://github.com/user-attachments/assets/2b50aaf8-3127-4211-be97-e54a1f6f0e34" />


<img width="1356" height="605" alt="image" src="https://github.com/user-attachments/assets/5cafdbfb-bf38-4107-82a4-ee08c98afa46" />


<img width="1354" height="550" alt="image" src="https://github.com/user-attachments/assets/3749ae41-4326-4b38-a754-1f76af91655b" />

<img width="1365" height="380" alt="image" src="https://github.com/user-attachments/assets/c67bf7b6-1668-49bc-9882-02014c897e8a" />


**Select the created Crawler and click on Run.**

<img width="1369" height="547" alt="image" src="https://github.com/user-attachments/assets/fdb46f5e-855c-4533-9e57-949493cb243d" />


<img width="1344" height="548" alt="image" src="https://github.com/user-attachments/assets/c3223db1-6ca6-457c-b1cf-1db342cea959" />


## Step 7:

Create and Configure an AWS Glue Job Using Visual ETL

Click ETLJob.

Select Visual ETL.

<img width="1338" height="553" alt="image" src="https://github.com/user-attachments/assets/f9f6c1f0-27fd-4d8c-8db1-922d76d68347" />

## Define the Source:

In the visual canvas, click on the add button and go to Data Source.

. Choose the AWS Glue Data Catalog. Under the database, choose the created csv_data_pipeline_catalog database.

. Under the table, choose the csv_processed_data table created by the Crawler job.



<img width="1362" height="544" alt="image" src="https://github.com/user-attachments/assets/90846cef-c821-44b8-bd11-1bd5cb4b0e8a" />


Add Transformations:

Click the + button after the source block and choose Change Schema for basic transformations.


<img width="1357" height="537" alt="image" src="https://github.com/user-attachments/assets/72009a9a-cab2-47ee-a039-481c6a5e12e4" />


<img width="643" height="499" alt="image" src="https://github.com/user-attachments/assets/75c9579c-a67e-4eef-a92d-604503640a9c" />


## Define the Target:

Click the + button after the transformation and select Data Target.

. Choose S3 as the target.

. Enter the S3 path where the transformed CSV file should be stored (e.g., s3://csv-final-data/).

. Format: Select CSV as the output format.

. Compression: Choose GZIP as the compression type

<img width="1355" height="546" alt="image" src="https://github.com/user-attachments/assets/95e20bde-62e4-45d6-9954-30932ea649fb" />



## Configure Job Properties:

Click the Job Details tab on the right panel and provide the following details:

Name: CSVDataTransformation

IAM Role: Select an existing Glue role with access to S3 or create a new one.

Leave other advanced settings as default.


<img width="1247" height="493" alt="image" src="https://github.com/user-attachments/assets/2d899174-6f80-499b-84d8-bfd001e1ac49" />


Save and Run the Job:

Click Save and then Run.

Monitor the job status in the Runs tab. It may take a few minutes to complete.


<img width="1337" height="226" alt="image" src="https://github.com/user-attachments/assets/a8ee6900-f426-424e-a4da-e719aea45341" />


## Verify and Prepare Transformed Data for Visualization

Navigate to the S3 Bucket:

. Open the AWS S3 Console.

. Locate and select the bucket where the transformed data is stored (e.g., csv-final-data-Ayo).


<img width="1328" height="384" alt="image" src="https://github.com/user-attachments/assets/40380be1-07ca-4144-aa38-b9a00321c64a" />




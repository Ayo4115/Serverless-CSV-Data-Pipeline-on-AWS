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


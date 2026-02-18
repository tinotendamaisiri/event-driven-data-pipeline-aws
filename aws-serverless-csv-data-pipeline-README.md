# AWS Serverless CSV Data Pipeline

Automated ingestion → preprocessing → transformation → visualization
using Amazon S3, AWS Lambda, AWS Glue, and Amazon QuickSight.

------------------------------------------------------------------------

## 🏗 Architecture


<img width="1523" height="620" alt="Ux4GVVgIR2CGgxirKCSV_Data-pipeline-intermediate-projs drawio" src="https://github.com/user-attachments/assets/7db5fa9f-c4e4-47f5-a12e-dac706e1a5c2" />

------------------------------------------------------------------------

## 📌 Overview

This project implements a fully serverless, event-driven data pipeline
on AWS that transforms raw CSV uploads into analytics-ready datasets
automatically.

Instead of manually cleaning and preparing CSV files for analysis, the
system:

1.  Ingests raw CSV files via Amazon S3\
2.  Triggers AWS Lambda for preprocessing\
3.  Uses AWS Glue for structured ETL transformations\
4.  Stores final outputs in S3\
5.  Visualizes insights using Amazon QuickSight

This project demonstrates modern cloud-native data engineering patterns
using managed AWS services.

------------------------------------------------------------------------

## 🚀 Architecture Flow

1.  CSV file uploaded to **Raw S3 Bucket**
2.  S3 event triggers **AWS Lambda**
3.  Lambda preprocesses and cleans CSV data
4.  Cleaned file stored in **Processed S3 Bucket**
5.  AWS Glue Crawler discovers schema
6.  AWS Glue ETL Job transforms data
7.  Transformed output stored in **Final S3 Bucket**
8.  Amazon QuickSight connects via S3 manifest for visualization

------------------------------------------------------------------------

## 🛠 AWS Services Used

-   **Amazon S3** --- Raw, processed, and final data storage\
-   **AWS Lambda (Python)** --- Event-driven preprocessing\
-   **AWS Glue Data Catalog** --- Metadata repository\
-   **AWS Glue Crawler** --- Schema discovery\
-   **AWS Glue Studio (Visual ETL)** --- Data transformation\
-   **Amazon QuickSight** --- Interactive dashboards\
-   **IAM Roles & Policies** --- Secure service access

------------------------------------------------------------------------

## 📂 S3 Bucket Structure

    csv-raw-data/
        └── raw/

    csv-processed-data/
        └── processed/

    csv-final-data/

------------------------------------------------------------------------

## ⚙️ Setup & Configuration

### 1️⃣ Create S3 Buckets

Create three S3 buckets in the same region:

-   `csv-raw-data`
-   `csv-processed-data`
-   `csv-final-data`

> Bucket names must be globally unique.

------------------------------------------------------------------------

### 2️⃣ Configure IAM Roles

#### Lambda Role

Role Name: `Lambda-S3-Glue-Role`

Permissions: - AmazonS3 access - AWSGlueServiceRole access

#### Glue Role

Role Name: `Glue-Service-Role`

Permissions: - AWSGlueServiceRole - AmazonS3 access

> In production, restrict policies to least privilege.

------------------------------------------------------------------------

## 🔄 Data Ingestion & Preprocessing

### Lambda Function

Triggered when a file is uploaded to:

    csv-raw-data/raw/

### Responsibilities

-   Reads uploaded CSV file
-   Removes rows with missing values
-   Writes cleaned data to processed bucket

### Lambda Code

``` python
import boto3
import csv
import io

s3 = boto3.client('s3')
PROCESSED_BUCKET = "csv-processed-data"

def lambda_handler(event, context):
    bucket_name = event['Records'][0]['s3']['bucket']['name']
    file_key = event['Records'][0]['s3']['object']['key']

    response = s3.get_object(Bucket=bucket_name, Key=file_key)
    csv_content = response['Body'].read().decode('utf-8')

    processed_rows = []
    reader = csv.reader(io.StringIO(csv_content))
    header = next(reader)

    for row in reader:
        if all(row):
            processed_rows.append(row)

    output_csv = io.StringIO()
    writer = csv.writer(output_csv)
    writer.writerow(header)
    writer.writerows(processed_rows)

    processed_file_key = file_key.replace('raw/', 'processed/')

    s3.put_object(
        Bucket=PROCESSED_BUCKET,
        Key=processed_file_key,
        Body=output_csv.getvalue()
    )
```

------------------------------------------------------------------------

## 🧠 Data Transformation with AWS Glue

### Glue Data Catalog

Database Name:

    csv_data_pipeline_catalog

### Crawler

-   Source: Processed S3 bucket
-   Output: Table in Glue Data Catalog
-   Schedule: Run on demand

### Glue ETL Job (Visual ETL)

-   Source: Glue Data Catalog table
-   Transformation: Modify schema (example: drop unused columns)
-   Target: Final S3 bucket
-   Output Format: CSV (GZIP compressed)

------------------------------------------------------------------------

## 📊 Data Visualization with QuickSight

### Steps

1.  Create S3 manifest file
2.  Connect dataset in QuickSight
3.  Build analysis:
    -   Bar charts
    -   Line charts
4.  Publish dashboard

------------------------------------------------------------------------

## 🔍 Key Engineering Concepts Demonstrated

-   Event-driven architecture
-   Serverless compute patterns
-   Managed ETL workflows
-   Metadata cataloging
-   Secure IAM configuration
-   Cloud-native analytics delivery

------------------------------------------------------------------------

## 🔮 Future Improvements

-   Convert output format to Parquet for performance
-   Add automated data validation checks
-   Orchestrate pipeline with AWS Step Functions
-   Add CloudWatch monitoring and alerting
-   Implement fine-grained IAM policies

------------------------------------------------------------------------

## 👤 Author

Tinotenda Maisiri\
Cloud & Data Systems Engineer

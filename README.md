### Step-by-Step Guide
This project involves building a serverless data pipeline on AWS for processing CSV files. The pipeline automates the ingestion, transformation, and visualization of data. CSV files are uploaded to a raw data S3 bucket (csv-raw-data), which triggers an AWS Lambda function to preprocess the data and store it in the processed data bucket (csv-processed-data).

AWS Glue is then used for additional ETL (Extract, Transform, Load) operations, and the final data is stored in the final data bucket (csv-final-data). Finally, Amazon QuickSight is used to create interactive dashboards and reports for visualizing the processed data.

## Architectural Diagram
<img width="695" height="319" alt="Screenshot 2025-10-22 152123" src="https://github.com/user-attachments/assets/436afff4-9906-46f8-b2ca-3c1eb961ff70" />
## 1.Create Three S3 Buckets
- csv-raw-data for raw data
- csv-processed-data for processed data
- csv-final-data for final data
- Region: Choose us-east-1
## 2.Configure IAM Roles and Policies
-Create IAM roles to grant permissions to Lambda and AWS Glue.
# Create an IAM Role for Lambda
-Go to the IAM Console and click Create role.
-Select AWS Service as the trusted entity.
-Attach the following policies:
AmazonS3FullAccess
AWSGlueServiceRole
# Create an IAM Role for Glue
-Go to the IAM Console and click Create role.
-Select AWS Service as the trusted entity.
-Attach the following policies:
-AmazonS3FullAccess
-AWSGlueServiceRole
## 3.Set Up Amazon QuickSight
-In the AWS Console, search for QuickSight and open the service.
-If you don’t have an account, click Sign up for QuickSight.
-Add your email address.
-Check Use IAM federated identities & QuickSight-managed users.
-Choose the US-East region (same region as your buckets).
-Add a QuickSight Account Name.
-For the IAM role, check Use QuickSight-managed role (default).
-Configure QuickSight Settings:
-Under Allow Access, click Select S3 Buckets.
-Select the three S3 buckets you created earlier and click Finish.
-Uncheck Add Pixel Perfect Reports and click Finish.
QuickSight is now ready to access and visualize data from S3!
## 4. Create a Lambda Function
Create a Lambda function to automate the ingestion and preprocessing of CSV files.
This function will trigger automatically when a file is uploaded to the raw data S3 bucket.
-Choose Author from scratch.
-Function Name: CSVPreprocessorFunction
-Runtime: Python 3.13
-Role: Use an existing role and select Lambda-S3-Glue-Role created earlier.
## 5. Write and Deploy the Lambda Function Code
Write a basic preprocessing Lambda function to clean and filter CSV files.
The code can be found in your GitHub repository folder.
Replace the default code with the provided one.
- Update the bucket name placeholder <YOUR_PROCESSED_BUCKET_NAME> with your actual processed data bucket name.
- Click Deploy to save the code.
## 6. Set Up S3 Event Trigger for Lambda
- Configure the raw data S3 bucket to automatically trigger the Lambda function whenever a new file is uploaded.
- Go to the S3 Console.
- Select the csv-raw-data bucket.
- Navigate to the Properties tab and scroll to Event notifications.
- Click Create event notification and configure the following:
- Name: CSVUploadTrigger
- Event types: Select PUT (for new file uploads).
- Prefix: raw/ (All raw files are uploaded to this folder).
- Destination: Choose Lambda Function and select CSVPreprocessorFunction.
- Click Save changes.
## 7. Test Data Ingestion and Preprocessing
Upload a sample CSV file to the csv-raw-data bucket.
If configured correctly, the Lambda function will trigger and store the processed CSV in the processed data bucket.
- Navigate to your csv-processed-data bucket.
- Verify that the processed CSV file is present in the correct folder.
## 8. Set Up an AWS Glue Data Catalog
-Navigate to the AWS Glue Console.
-Click Data Catalogs → Add Database.
-Database Name: csv_data_pipeline_catalog
- click Create.
## 9. Create a Crawler to Discover the Data Schema
- A crawler scans the data and creates metadata tables automatically.
- Go to Crawlers and click Add Crawler.
- Name: ProcessedCSVDataCrawler
- Data Source: Choose S3 and provide the location of the csv-processed-data bucket.
- IAM Role: Select Glue-Service-Role.
- Database: Choose csv_data_pipeline_catalog.
- Schedule: Select Run on demand.
- Click Next → Create Crawler.
- Select the created crawler and click Run.
## 10. Create and Configure an AWS Glue Job Using Visual ETL
- In the AWS Glue Console, go to AWS Glue Studio.
- Click Create job → Select Visual ETL.
# Define the Source:
- Click Add node → Data source.
- Choose AWS Glue Data Catalog.
- Select the database csv_data_pipeline_catalog.
- Choose the table created by the crawler.
  # Add Transformations:
- Click the + button after the source block → Select Change Schema for basic transformations.
- Check the column(s) to modify (e.g., icon column).
  # Define the Target:
- Click the + button after the transformation → Select Data target.
- Choose S3 as the target.
- Enter the S3 path where the transformed file should be stored (e.g., s3://csv-final-data/).
- Format: CSV
- Compression: GZIP
# Configure Job Properties:
- Job Name: CSVDataTransformation
- IAM Role: Select an existing Glue role with S3 access.
- Leave other settings as default.
- Click Save and then Run.
- Monitor the job status in the Runs tab. It may take a few minutes to complete.
## 11. Verify and Prepare Transformed Data for Visualization
After the Glue job execution, the transformed data will be stored as a compressed file in the target S3 bucket.
If the file lacks a .csv extension:
- Download and extract the file.
- Rename it with a .csv extension.
- Upload it back to the csv-final-data bucket.
Once confirmed, the file is ready for visualization.
Click on the CSV file and copy its Object URL for the next steps.
## 12. Connect to the Data Source in QuickSight
-In the QuickSight Console, go to Datasets → Click New Dataset.
-Select S3 as the data source.
-Data Source Name: ProcessedCSV
-Manifest File: Create a JSON manifest file locally with the following content:

{
  "fileLocations": [
    {
      "URIs": [
        "ENTER YOUR OBJECT URL HERE"
      ]
    }
  ],
  "globalUploadSettings": {
    "textqualifier": "\""
  }
}
- Upload the manifest file and click Connect.
- Click Visualize to load the data into QuickSight.
## 13. Build a QuickSight Dashboard
- Create a dashboard to visualize insights from the transformed CSV data.
- Click Add to create a new analysis.
- Select the ProcessedCSVData dataset.
# Create visualizations such as:
-Bar Chart: Compare values across columns.
-Line Chart: Analyze trends over time.
### QuickSight Dashboard (Visualization Output)
Below is the sample output showing the final dashboard running on Amazon QuickSight, displaying processed CSV data with interactive charts and insights.

<img width="1498" height="907" alt="Screenshot 2025-10-21 235541" src="https://github.com/user-attachments/assets/b97dbb1c-5f2d-4836-974a-d65ee4404fbd" />


<img width="1281" height="910" alt="Screenshot 2025-10-21 235952" src="https://github.com/user-attachments/assets/9d9b495d-74e9-4f46-bc6c-83d537e00049" />

## 14. Share and Publish the Dashboard

<img width="624" height="695" alt="Screenshot 2025-10-22 000430" src="https://github.com/user-attachments/assets/a9bb1d39-b6f1-4985-a665-135fb17a354c" />

Once the analysis is complete, click Publish Dashboard to make it accessible to other users.

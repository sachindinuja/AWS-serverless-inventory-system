# Event Driven Serverless Inventory Tracking System on AWS

A fully serverless, event driven inventory ingestion and real-time stock alerting pipeline built using **Amazon S3, AWS Lambda (Python/Boto3), Amazon DynamoDB (with DynamoDB Streams), Amazon SNS, and AWS IAM**.

Automatically processes store inventory uploads via S3, syncs with DynamoDB, updates a live Dashboard, and triggers real-time SNS notifications for out-of-stock items.

## Architecture Diagram

![Architecture Diagram](docs/architecture-diagram.png)

### Workflow
1. **Data Upload:** Global retail stores upload CSV inventory files to an **Amazon S3** bucket.
2. **Automated Ingestion:** S3 Object Creation events trigger the `Load-Inventory` **AWS Lambda** function, which parses the CSV file and writes records into an **Amazon DynamoDB** table.
3. **Database Write:** Lambda parses CSV rows and updates the **Amazon DynamoDB** `Inventory` table.
4. **Stream Processing:** Item-level updates in the DynamoDB table are captured in real time via **DynamoDB Streams**, which automatically trigger the decoupled `Check-Stock` **AWS Lambda** function.
5. **Real-time Alerting:** If `Count == 0`, `Check-Stock` Lambda publishes a notification to **Amazon SNS**, triggering instant Email/SMS alerts.

---

## Step-by-Step Implementation Guide

### Step 1: Configuring IAM Roles & Policies
Before provisioning compute resources, create the execution roles required for both Lambda functions.

#### A. Create `Lambda-Load-Inventory-Role`
1. Open the **IAM Console** -> **Roles** -> **Create role**.
2. Select **AWS service** and choose **Lambda** as the use case.
3. Attach the following **AWS Managed Policies**:
   - `AmazonDynamoDBFullAccess`
   - `AmazonS3ReadOnlyAccess`
4. Name the role `Lambda-Load-Inventory-Role` and create it.
5. Open the created role -> **Add permissions** -> **Create inline policy** -> Switch to the **JSON** tab and paste the following CloudWatch Logs policy (`CWLogsPolicy`):

```json
{
	"Version": "2012-10-17",
	"Statement": [
		{
			"Action": [
				"logs:CreateLogGroup",
				"logs:CreateLogStream",
				"logs:PutLogEvents"
			],
			"Resource": "arn:aws:logs:*:*:*",
			"Effect": "Allow"
		}
	]
}

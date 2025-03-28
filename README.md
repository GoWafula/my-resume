# AWS Cloud Resume Challenge

This repository contains the code and configuration for my AWS Cloud Resume Challenge.

## Steps to Complete the Challenge

### 1. Set Up AWS Services

#### Step 1.1: Configure an Amazon S3 Bucket for Static Website Hosting
1. Log in to the AWS Management Console.
2. Navigate to the S3 service.
3. Click "Create bucket".
4. Provide a unique bucket name (e.g., `my-resume-bucket`).
5. Choose the AWS Region closest to you.
6. Under "Object Ownership", select "ACLs enabled".
7. Leave "Block all public access" checked for now (we'll change it later).
8. Uncheck "Bucket Versioning" (for simplicity).
9. Leave the "Default encryption" section set to "None".
10. Click "Create bucket".

#### Step 1.2: Deploy the Static Resume Files to the S3 Bucket
1. Prepare your resume files (HTML, CSS, etc.).
2. In the S3 console, select your bucket.
3. Click "Upload" and "Add files" to upload your resume files.
4. After uploading, select the files, click "Actions", then "Make public".
5. Confirm the action.

#### Step 1.3: Set Up CloudFront with SSL for the S3 Bucket
1. Navigate to the CloudFront service in the AWS Management Console.
2. Click "Create Distribution" and select "Web" as the delivery method.
3. Under "Origin Settings":
   - Origin Domain Name: Select your S3 bucket from the dropdown.
   - Origin Path: Leave empty.
   - Origin ID: Use the default value.
   - Restrict Bucket Access: Yes.
   - Origin Access Identity: Create a new identity.
   - Grant Read Permissions on Bucket: Yes, update bucket policy.
4. Under "Default Cache Behavior Settings":
   - Viewer Protocol Policy: Redirect HTTP to HTTPS.
   - Allowed HTTP Methods: GET, HEAD.
   - Cache Based on Selected Request Headers: None.
   - Object Caching: Use Origin Cache Headers.
5. Under "Distribution Settings":
   - Price Class: Use Only US, Canada, and Europe (or choose based on your target audience).
   - Alternate Domain Names (CNAMEs): Add your custom domain if you have one.
   - SSL Certificate: Choose "Custom SSL Certificate" and select your certificate from ACM.
   - Default Root Object: `index.html`.
6. Click "Create Distribution" and wait for it to deploy.

### 2. Set Up Back-End and Certificate

#### Step 2.1: Create a DynamoDB Table to Store Visitor Count
1. Navigate to the DynamoDB service.
2. Click "Create table".
3. Table name: `VisitorCount`.
4. Primary key: `id` (String).
5. Leave the "Default settings" checked.
6. Click "Create".

#### Step 2.2: Develop a Lambda Function to Update the Visitor Count in DynamoDB
1. Navigate to the Lambda service.
2. Click "Create function".
3. Choose "Author from scratch":
   - Function name: `UpdateVisitorCount`.
   - Runtime: Python 3.9.
4. Click "Create function".
5. In the function code editor, replace the default code with the following:

```python
import json
import boto3
from boto3.dynamodb.conditions import Key

dynamodb = boto3.resource('dynamodb')
table = dynamodb.Table('VisitorCount')

def lambda_handler(event, context):
    visitor_id = 'visitor_count'

    response = table.get_item(
        Key={'id': visitor_id}
    )

    if 'Item' in response:
        visitor_count = response['Item']['count']
    else:
        visitor_count = 0

    visitor_count += 1

    table.put_item(
        Item={
            'id': visitor_id,
            'count': visitor_count
        }
    )

    return {
        'statusCode': 200,
        'body': json.dumps({'visitor_count': visitor_count}),
        'headers': {
            'Content-Type': 'application/json',
            'Access-Control-Allow-Origin': '*'
        }
    }
```

6. Under "Configuration" > "Permissions", click on the role name under "Execution role".
7. Click "Add inline policy".
   - Service: DynamoDB.
   - Actions: `GetItem`, `PutItem`.
   - Resources: Specify the ARN of your DynamoDB table.
   - Click "Review policy", name it (e.g., `DynamoDBAccessPolicy`), and create the policy.

#### Step 2.3: Implement API Gateway to Trigger the Lambda Function
1. Navigate to the API Gateway service.
2. Click "Create API" and choose "REST API".
3. Create a new resource:
   - Resource name: `visitor-count`.
   - Resource path: `/visitor-count`.
4. Create a method (e.g., `GET`):
   - Integration type: Lambda Function.
   - Lambda region: Your Lambda function's region.
   - Lambda function: `UpdateVisitorCount`.
5. Deploy the API and note the endpoint URL.


#### Step 2.4: Configure GitHub Actions for CI/CD to Automate Deployments
1. Create a `.github/workflows` directory in your GitHub repository.
2. Add a workflow YAML file (e.g., `deploy.yml`) to automate the deployment process.
3. Configure the workflow to deploy changes to your AWS services.

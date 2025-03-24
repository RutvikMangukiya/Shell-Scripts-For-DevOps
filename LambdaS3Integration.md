# AWS Lambda S3 Integration Tutorial

This repository contains an implementation of an AWS Lambda function that is triggered by an S3 event when a new object is uploaded to a specified bucket. The function processes the file and uploads the modified version to another S3 bucket.

## Prerequisites
- AWS Account
- AWS CLI installed and configured
- Python 3.x installed
- IAM role with necessary permissions
- An S3 bucket to trigger the Lambda function

## Steps to Implement

### 1. Create an S3 Bucket
```sh
aws s3 mb s3://your-source-bucket-name
```

### 2. Create an IAM Role for Lambda
- Go to **IAM** in AWS Console.
- Create a new role with the following permissions:
  - `AmazonS3FullAccess`
  - `AWSLambdaBasicExecutionRole`

### 3. Create a Lambda Function
- Open AWS Lambda and create a new function.
- Use the **Python 3.x** runtime.
- Assign the IAM role created in step 2.

### 4. Write the Lambda Function Code
```python
import boto3
import os
import uuid
from urllib.parse import unquote_plus
from PIL import Image

s3_client = boto3.client('s3')

def resize_image(image_path, resized_path):
    with Image.open(image_path) as image:
        image.thumbnail(tuple(x / 2 for x in image.size))
        image.save(resized_path)

def lambda_handler(event, context):
    for record in event['Records']:
        bucket = record['s3']['bucket']['name']
        key = unquote_plus(record['s3']['object']['key'])
        tmpkey = key.replace('/', '')
        download_path = f'/tmp/{uuid.uuid4()}{tmpkey}'
        upload_path = f'/tmp/resized-{tmpkey}'

        s3_client.download_file(bucket, key, download_path)
        resize_image(download_path, upload_path)
        s3_client.upload_file(upload_path, f'{bucket}-resized', f'resized-{key}')
```

### 5. Package and Deploy the Lambda Function
```sh
mkdir package
pip install --target=package pillow boto3
cd package
zip -r ../lambda_function.zip .
cd ..
zip lambda_function.zip lambda_function.py
aws lambda update-function-code --function-name YourLambdaFunctionName --zip-file fileb://lambda_function.zip
```

### 6. Configure S3 Event Trigger
- Go to **S3** in AWS Console.
- Open your bucket and go to **Properties** > **Event Notifications**.
- Create a new event notification for `s3:ObjectCreated:Put` and link it to your Lambda function.

### 7. Test the Setup
Upload a file to your S3 bucket:
```sh
aws s3 cp test-image.jpg s3://your-source-bucket-name/
```
Check CloudWatch logs for function execution:
```sh
aws logs tail /aws/lambda/YourLambdaFunctionName --follow
```

## Cleanup
To remove all resources created:
```sh
aws s3 rb s3://your-source-bucket-name --force
aws s3 rb s3://your-source-bucket-name-resized --force
aws lambda delete-function --function-name YourLambdaFunctionName
aws iam delete-role --role-name YourLambdaRoleName
```

## Conclusion
This tutorial demonstrates how to integrate AWS Lambda with S3 to automatically process images upon upload. Modify the Lambda function as needed for additional use cases.

---

### 📌 Author
GitHub: [YourGitHubUsername](https://github.com/YourGitHubUsername)


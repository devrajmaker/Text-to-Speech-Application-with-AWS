# 🔊 Text-to-Speech Application with AWS

A serverless text-to-speech application that converts text input into audio files using AWS Polly, stores metadata in DynamoDB, and serves content through a web interface hosted on S3.

## 📋 Table of Contents
- [Overview](#overview)
- [Architecture](#architecture)
- [Features](#features)
- [Technologies](#technologies)
- [Setup Instructions](#setup-instructions)
- [API Documentation](#api-documentation)
- [Deployment](#deployment)
- [Troubleshooting](#troubleshooting)


## 📖 Overview

This project demonstrates a fully serverless text-to-speech application built on AWS. Users can input text through a web interface, select a voice, and have it converted to speech using Amazon Polly. The application automatically processes requests, stores metadata in DynamoDB, and hosts audio files in S3.

## 🏗️ Architecture
User Interface → API Gateway → PostReader_NewPost Lambda → DynamoDB
↓
SNS Topic
↓
ConvertToAudio Lambda → Amazon Polly → S3 Bucket
↓
PostReader_GetPost Lambda ← DynamoDB

## ✨ Features

- ✅ **Multiple Voice Options** - Choose from various Amazon Polly voices
- ✅ **Real-time Processing** - Track status from PROCESSING to UPDATED
- ✅ **Scalable Architecture** - Fully serverless, auto-scaling
- ✅ **RESTful API** - Easy integration with other applications
- ✅ **Web Interface** - User-friendly HTML/JS frontend
- ✅ **Secure** - IAM roles with least privilege principle
- ✅ **Cost-effective** - Pay-per-use pricing model

## 🛠️ Technologies

| Service | Purpose | Version |
|---------|---------|---------|
| **AWS Lambda** | Serverless computing | Python 3.13 |
| **Amazon DynamoDB** | NoSQL database | - |
| **Amazon S3** | Object storage & web hosting | - |
| **Amazon Polly** | Text-to-speech conversion | - |
| **Amazon SNS** | Notification service | - |
| **API Gateway** | REST API endpoints | - |
| **IAM** | Access management | - |

## 📁 Project Structure
text-to-speech-aws/
├── lambda-functions/
│ ├── PostReader_NewPost.py # Creates new posts
│ ├── ConvertToAudio.py # Converts text to speech
│ └── PostReader_GetPost.py # Retrieves posts
├── web-interface/
│ ├── index.html # Frontend interface
│ └── CloudAge-Logo.png # Application logo
├── infrastructure/
│ └── iam-role-policy.json # IAM role permissions
├── screenshots/
│ ├── architecture.png # System architecture
│ └── demo.gif # Application demo
├── README.md # This file
├── LICENSE # License file
└── .gitignore # Git ignore file

## 🚀 Setup Instructions

### Prerequisites
- **AWS Account** with appropriate permissions
- **AWS CLI** configured with credentials
- **Basic understanding** of AWS services
- **Python 3.13** knowledge

### Step-by-Step Deployment

#### 1. Create DynamoDB Table
```bash
aws dynamodb create-table \
    --table-name AudioPost \
    --attribute-definitions AttributeName=id,AttributeType=S \
    --key-schema AttributeName=id,KeyType=HASH \
    --billing-mode PAY_PER_REQUEST
```

#### 2. Create S3 Buckets
```bash
# Audio storage bucket
aws s3api create-bucket \
    --bucket audioposts-YOUR-NAME \
    --region us-east-1

# Web hosting bucket
aws s3api create-bucket \
    --bucket text-to-speech-web-YOUR-NAME \
    --region us-east-1
```

#### 3. Create SNS Topic
```bash
aws sns create-topic \
    --name audiopost \
    --attributes DisplayName="New posts"
```
#### 4. Create IAM Role
Create a role with the following trust policy:
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Principal": {
                "Service": "lambda.amazonaws.com"
            },
            "Action": "sts:AssumeRole"
        }
    ]
}
```

#### 5. Deploy Lambda Functions
Upload each Lambda function through AWS Console or using:
```bash
aws lambda create-function \
    --function-name PostReader_NewPost \
    --runtime python3.13 \
    --role arn:aws:iam::ACCOUNT:role/CloudAge-Lambda-Role \
    --handler lambda_function.lambda_handler \
    --zip-file fileb://PostReader_NewPost.zip
```

#### 6. Configure API Gateway
```bash
aws apigateway create-rest-api \
    --name PostReaderAPI \
    --region us-east-1
```

## 📚 API Documentation
#### Base URL
```text
https://{api-id}.execute-api.{region}.amazonaws.com/{stage}/
```

#### Endpoints
#### POST / - Create New Audio Post
Request:
```json
{
    "voice": "Joanna",
    "text": "Hello, this is a test message."
}
```
Response:
```json
{
    "postId": "123e4567-e89b-12d3-a456-426614174000"
}
```

#### GET /?postId={id} - Retrieve Post Information
Parameters:

postId (string): Post identifier, use * for all posts

Response:
```json
[
    {
        "id": "123e4567-e89b-12d3-a456-426614174000",
        "text": "Hello, this is a test message.",
        "voice": "Joanna",
        "status": "UPDATED",
        "url": "https://s3.amazonaws.com/audioposts/123e4567.mp3"
    }
]
```
## 🔧 Configuration
Environment Variables
Lambda Function	Variable	Value
PostReader_NewPost	SNS_TOPIC	SNS Topic ARN
PostReader_NewPost	DB_TABLE_NAME	posts
ConvertToAudio	DB_TABLE_NAME	DynamoDB Table ARN
ConvertToAudio	BUCKET_NAME	S3 Bucket Name
PostReader_GetPost	DB_TABLE_NAME	posts

### IAM Role Permissions
The CloudAge-Lambda-Role requires permissions for:
Polly: synthesizeSpeech
DynamoDB: Query, Scan, PutItem, UpdateItem
S3: PutObject, PutObjectAcl, GetBucketLocation
SNS: Publish
CloudWatch Logs: CreateLogGroup, CreateLogStream, PutLogEvents

## 🎮 Usage Example
Access the Web Interface
Navigate to your S3 static website URL.

Create Audio

Enter text in the text area

Select a voice (e.g., Joanna, Matthew)

Click "Say it!"

View Results

Audio file is generated and stored

URL appears in the results section

Click "Play" to listen

listen

## 🚨 Troubleshooting
### Common Issues
Issue	Solution
Lambda Timeout	Increase timeout to 5 minutes for ConvertToAudio
S3 Permission Denied	Check bucket policy and ACL settings
Polly Character Limit	Split text into chunks < 3000 characters
CORS Errors	Verify API Gateway CORS configuration
Missing Environment Variables	Check Lambda function configuration

### Debugging Steps
1. Check CloudWatch Logs
``` bash
aws logs filter-log-events \
    --log-group-name /aws/lambda/PostReader_NewPost \
    --start-time $(date -d '1 hour ago' +%s000)
```

2. Test Lambda Functions
``` bash
aws lambda invoke \
    --function-name PostReader_NewPost \
    --payload '{"voice":"Joanna","text":"test"}' \
    output.json
```

3. Verify S3 Objects
``` bash
aws s3 ls s3://audioposts-YOUR-NAME/
```

## 📊 Monitoring
CloudWatch Metrics: Lambda invocations, errors, duration
S3 Access Logs: File downloads and access patterns
API Gateway Metrics: Request count, latency, 4XX/5XX errors
Cost Monitoring: AWS Cost Explorer for budget tracking

## 🔮 Future Enhancements
Add user authentication
Implement text preprocessing
Add support for multiple languages
Implement audio concatenation for long texts
Add WebSocket for real-time status updates
Implement caching layer


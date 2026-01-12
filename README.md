Project Overview
A serverless text-to-speech application that converts text input into audio files using AWS Polly, stores metadata in DynamoDB, and serves content through a web interface hosted on S3.

Architecture
text
User Input → API Gateway → Lambda (PostReader_NewPost) → DynamoDB → SNS → 
Lambda (ConvertToAudio) → Polly → S3 → Web Interface
Technologies Used
AWS Lambda (Python 3.13) - Serverless computing

Amazon DynamoDB - NoSQL database for storing post metadata

Amazon S3 - Object storage for audio files and web hosting

Amazon Polly - Text-to-speech service

Amazon SNS - Notification service

API Gateway - REST API endpoints

IAM - Identity and access management

Project Structure
text
├── lambda-functions/
│   ├── PostReader_NewPost.py    # Creates new posts
│   ├── ConvertToAudio.py        # Converts text to speech
│   └── PostReader_GetPost.py    # Retrieves posts
├── web-interface/
│   ├── index.html              # Frontend interface
│   └── CloudAge-Logo.png       # Application logo
└── infrastructure/
    └── iam-role-policy.json    # IAM role permissions
Setup Instructions
Prerequisites
AWS Account with appropriate permissions

Basic understanding of AWS services

Python 3.13 knowledge

Deployment Steps
Create DynamoDB Table

Table name: AudioPost

Partition key: id (String)

Additional attributes: status, text, voice, url

Create S3 Buckets

Bucket 1: audioposts (for audio files)

Bucket 2: (for web hosting with public access)

Create SNS Topic

Name: audiopost

Display name: New posts

Create IAM Role

Name: CloudAge-Lambda-Role

Attach the provided policy with permissions for:

Polly (text-to-speech)

DynamoDB (database operations)

S3 (file storage)

SNS (notifications)

CloudWatch (logging)

Deploy Lambda Functions

PostReader_NewPost: Creates new posts

ConvertToAudio: Processes text to audio

PostReader_GetPost: Retrieves posts

Configure API Gateway

Create REST API: PostReaderAPI

Configure POST and GET methods

Enable CORS

Set up query parameters and mapping templates

Deploy Web Interface

Upload index.html and logo to S3

Enable static website hosting

Update API Gateway endpoint in HTML

API Endpoints
POST /
Purpose: Create new text-to-speech request

Request Body:

json
{
  "voice": "Joanna",
  "text": "Your text here"
}
Response: Post ID

GET /?postId={id}
Purpose: Retrieve post information

Parameters: postId (use * for all posts)

Response: Post metadata

Security Considerations
S3 buckets have appropriate ACLs and bucket policies

IAM role follows principle of least privilege

Lambda functions have appropriate timeouts

API Gateway configured with CORS

Monitoring & Logging
Lambda logs in CloudWatch

S3 access logs (if configured)

API Gateway execution logs

Workflow
User submits text via web interface

API Gateway triggers PostReader_NewPost

Lambda stores data in DynamoDB with "PROCESSING" status

SNS notification triggers ConvertToAudio

Polly converts text to audio

Audio file stored in S3

DynamoDB updated with "UPDATED" status and S3 URL

User can retrieve and play audio

Features
Multiple voice options via Amazon Polly

Scalable serverless architecture

Real-time processing status

Publicly accessible audio files

Responsive web interface

RESTful API

Troubleshooting
Check CloudWatch logs for Lambda errors

Verify IAM role permissions

Confirm S3 bucket policies allow public read

Ensure API Gateway CORS settings are correct

Validate environment variables in Lambda functions

Notes
Maximum text length per Polly call: ~3000 characters

Supported Polly voices: Joanna, Matthew, etc.

Audio format: MP3

Region-specific S3 URLs

Note: Ensure all AWS resources are properly configured in the same region for optimal performance and to avoid cross-region data transfer costs.

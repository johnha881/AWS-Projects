# Serverless Application with API Gateway, Lambda, and DynamoDB

## Table of Contents
- [Introduction](#introduction)
- [Setup](#setup)
  - [Create IAM Roles](#create-iam-roles)
  - [Create Lambda Function](#create-lambda-function)
  - [Create DynamoDB Table](#create-dynamodb-table)
  - [Create API Gateway](#create-api-gateway)
- [Usage](#usage)
- [Scaling](#scaling)
- [Availability](#availability)
- [Cost Optimization](#cost-optimization)
- [Security](#security)
- [Issues I ran into](#issues)

## Introduction
This project demonstrates how to build a serverless application using AWS API Gateway, AWS Lambda, and Amazon DynamoDB. It covers the steps to set up IAM roles, Lambda functions, DynamoDB tables, and API Gateway, along with considerations for scaling, availability, cost optimization, and security.

## Setup

### Create IAM Roles
Create roles for Lambda that provide access to DynamoDB with the proper permissions. 
   ```json
    {
    "Version": "2012-10-17",
    "Statement": [
    {
      "Sid": "statementidlambdatodynamodb",
      "Action": [
        "dynamodb:DeleteItem",
        "dynamodb:GetItem",
        "dynamodb:PutItem",
        "dynamodb:Query",
        "dynamodb:Scan",
        "dynamodb:UpdateItem"
      ],
      "Effect": "Allow",
      "Resource": "*"
    }
   ```

- **Statement ID**: An optional identifier for individual policies within a role, useful for distinguishing policies.
- **Action**: A list of actions permitted on the DynamoDB table.
- **Effect**: Use "Allow" to permit actions (note: "Deny" takes precedence over "Allow").
- **Resource**: Currently set to `*`, but for better security, specify the actual ARN.

### Create Lambda Function
Create a Lambda function to manage and execute the logic required for interacting with the DynamoDB table.
- **Import Boto3**: This allows Python code to interact with AWS services.
- **Import JSON**: Necessary for handling JSON data.
- **Logic**: Check if the `tableName` is provided in the event and perform the corresponding actions on the DynamoDB table based on the operations specified in the event.

### Create DynamoDB Table
- **Primary Key**: Define a primary key as a unique identifier to distinguish rows from each other.

### Create API Gateway
- **Use REST API**: Configure a POST method for multiple actions.
- **Create Resource and Method**: Create a resource and configure a POST method to interact with the Lambda function created earlier.
- **Deploy API**: Deploy the API, specifying the stage (DEV, Test, Prod), and use the URL endpoint for testing with tools like curl or Postman.

## Usage
Check if the API injection works by using POST commands to interact with the DynamoDB table and retrieve items.

## Scaling

### DynamoDB
- **Enable Auto Scaling**: Adjust read/write capacity automatically.
- **Partition Keys**: Use effectively to avoid hot partitions.
- **On-Demand Mode**: Suitable for unpredictable workloads.

### API Gateway
- **Throttle Settings**: Limit the number of requests per second.
- **Enable Caching**: Store responses to reduce repetitive requests.
- **API Keys and Usage Plans**: Manage quotas and limits for API usage.
- **Regional Endpoints**: Provide faster access.

### Lambda
- **Increase Reserved Concurrency**: Handle simultaneous invocations.
- **Provisioned Concurrency**: Ensure critical functions have enough invocations.
- **Minimize Cold Starts**: Use minimal dependencies and initialize variables outside the handler function.
- **Lambda Layers**: Manage common dependencies and reduce deployment package size.

## Availability

### DynamoDB
- **Global Tables**: Cross-region redundancy and disaster recovery.
- **Backups**: Configure backups and point-in-time recovery.

### API Gateway
- **Edge-Optimized Endpoints**: Use CloudFront CDN for global distribution.
- **Multi-Region Deployment**: Distribute traffic based on request location using DNS.
- **Global Accelerator**: Redirect traffic to healthy regions.

### Lambda
- **Multi-Region Deployment**: Use health checks and latency-based routing.
- **Automate Deployment**: Ensure consistency across distributed locations.
- **VPC Configuration**: Ensure multiple subnets in different Availability Zones (AZs) for redundancy.
- **Multiple NAT Gateways**: For outbound internet access across different AZs.

## Cost Optimization

### DynamoDB
- **Provision Capacity**: For predictable workloads, it's more cost-effective than on-demand.
- **Auto-Scaling**: Avoid over-provisioning of capacity.
- **Efficient Queries**: Prefer queries over scans and use secondary indexes.
- **DAX**: Use for caching data to lower retrieval time.
- **Monitoring**: Use CloudWatch for monitoring metrics.

### API Gateway
- **Efficient API Requests**: Reduce the number of API requests per second with throttling and quotas.
- **Field Filtering**: Retrieve only necessary data to reduce data transfer.
- **Optimal Endpoints**: Ensure endpoints are in the proper region.

### Lambda
- **Optimal Memory**: Adjust memory settings to balance cost and performance.
- **Execution Time Limit**: Optimize function code to reduce execution time.
- **Reserved Concurrency**: Manage unexpected usage spikes.

## Security

### DynamoDB
- **IAM Access**: Restrict access to authorized users/groups/roles.
- **Encryption**: Use AWS KMS for encryption at rest.
- **Monitoring**: Use CloudTrail and CloudWatch for auditing.
- **VPC Endpoints**: Securely access DynamoDB within a VPC.
- **Continuous Backups**: Protect against accidental deletions or corruption.

### API Gateway
- **IAM Access**: Secure access to authorized users/groups/roles.
- **API Logs**: Use CloudTrail for monitoring API calls.
- **Usage Plans**: Manage API keys with quotas and limits.
- **Rotate API Keys**: Regularly update API keys.
- **WAF and CloudFront**: Protect against web attacks.

### Lambda
- **IAM Role**: Secure access for Lambda to interact with DynamoDB.
- **Environment Variables**: Avoid hardcoding sensitive information.
- **VPC Security**: Use inbound and outbound rules for Lambda within a VPC.
- **Auditing**: Use CloudWatch for monitoring and logging.
- **AWS X-Ray**: Trace and analyze application performance.

### Issues
- **AWSLambda_FullAccess**: Used initially but needed roles for more secure access.
- **x-api-key**: Took some time to find this value. This is the default AWS key name to access API Gateway.
- **Eventual Consistency**: The first GetItem call returned null, which was unexpected. However, the second call, made after waiting a few     seconds, returned the expected result.

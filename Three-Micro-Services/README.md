serverless with API gateway, lambda and dynamodb

1. create roles for lambda that provides access to dynamodb with the proper roles to interact with dynamoDB
   a. permissions are set up as follows:

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
   -statement ID provides an OPTIONAL identifier. Good for providing specific way to find out what an individial policy does inside a role.
   -Action gives an array of things you can do. Here are all things you can do to the dynamoDB table.
   -effect - Deny has precedence over allow, good way to stop deletes
   - resource is currently use * - good way to resstrict access is to have the actual ARN in here

2. Create a lambda function that will handle all the logic into performation actions into dynamoDb table
   a. Import Boto3- this lets your python3 code interact with AWS services.
   b. Import Json -
   -if 'tableName' in event:
        dynamo = boto3.resource('dynamodb').Table(event['tableName'])
         if this table name matches what is provided by APIgateway then actions provided by the api are triggered in the lambda function and pushed to dynamoDB
      -testing the lambda function -
                  if operation in operations:
                       return operations[operation](event.get('payload'))
         This return statement will output whatever data we put in while not being linked to a dynamoDB table


3. Create dynamoDB table
   a. primary Key is very important as it is a unique identifier that distinguishes rows from each other

4. Create APIGateway
   a. use REST - POST for multiple actions
      - create resource and then create a method that it will use to do some action
      - POST method will ask for what you want to interact with. In this case lambda and we can select the the lambda function we already created
   b. Deploy API
      - will need to indicate which STAGE
      - DEV,Test,Prod
      - select the URL endpoint
   c. Use curl or postman to inject actions into API Gateway

5. Check if our inject works and use post commands to retreieve list/items inside the table


SCALING 
1. DynamoDB
   a. enable auto scaling - adjust reading/write capcity
   b. Good use of partition keys. else they become hot(used by too mamy items)
   c. On-demand mode to avoid unpredictable workloads.
2. API Gateway
   a. Configure throttle settings to limit number of request per second
   b. enable caching to store responses so lambda calls and dynamodb request aren't constantly being used for repeating requests.
   c. API keys and usage plans to manange - put quotas and limits for api
   d. Use regional endpoints for faster access
3. Lambda
   a. Increease Reserved concurrency limit to handle simultaneous invocations
   b. Provisioned concurrency - makes sure critical functions have enough invokcations
   c. cold starts - Use minimal dependencies and initialize variables outside the handler function.
   d. Lambda Layers to manage common dependencies and reduce deployment package size.

AVAILIBILITY
1. DynamoDB
   a. Global Tables - cross region redundency and disasert recovery.
   b. Back-ups - Configure DynamoDB backups and point-in-time recovery to protect your data from accidental deletions or corruption.
2. API Gateway
   a. Edge optimzed endpoints - CND(cloudFront)
   b. Multi-region deployment - via DNS traffic distribuation depending on requestion location
   c. Global Accelerator -redirect traffic to healthy regions
3. Lambda
   a. Multi-region - use health checks and latecy to route correctly
   b. automate depolyment to make all distributated locations consistent
   c.VPC Configuration- If your Lambda functions need to access resources within a VPC, ensure you have multiple subnets in different Availability Zones (AZs) to provide redundancy.
   d.Use multiple NAT gateways for outbound internet access, distributed across different AZs.

COST OPTIMIZATION!!!!!!!!

Cost optimization for DynamoDB, API Gateway, and Lambda involves adopting strategies to efficiently use resources while maintaining performance and availability. Here are several approaches to optimize costs for each service:

1. DynamoDB
Provisioned Capacity vs. On-Demand Mode:

Evaluate your workload patterns. For predictable workloads, use provisioned capacity to save costs compared to on-demand pricing.
Use Auto Scaling to automatically adjust provisioned capacity based on demand, avoiding over-provisioning.
Optimize Data Models:

Design efficient data models to minimize read and write operations.
Avoid using scans where possible; prefer queries or using secondary indexes.
DynamoDB Accelerator (DAX):

Use DAX for caching frequently accessed data to reduce read costs and improve response times.
Monitor and Review:

Monitor CloudWatch metrics to track usage and adjust provisioned capacity based on actual demand.
Review and optimize your table designs and indexes regularly.
2. API Gateway
Optimize API Design:

Design efficient APIs to minimize the number of API requests and reduce data transfer.
Use response caching to reduce the number of calls to backend services.
Usage Plans and Quotas:

Implement usage plans with throttling and quotas to control API usage and prevent abuse.
Use API keys to monitor and control access to your APIs.
Regional Endpoints:

Use regional endpoints instead of edge-optimized endpoints if your API primarily serves users within the same AWS region.
3. Lambda
Right-Sizing Memory and Execution Time:

Adjust Lambda function memory allocation to match the workload. Higher memory settings increase CPU and network resources.
Optimize function execution time to minimize billing duration.
Concurrency and Provisioned Concurrency:

Use reserved concurrency to limit the maximum number of concurrent executions and avoid unexpected spikes in costs.
Consider using provisioned concurrency for critical functions to ensure consistent performance.
Optimize Deployment Packages:

Minimize the size of deployment packages to reduce cold start times and associated costs.
Use Lambda Layers for shared code or libraries to reduce function package size.

SECURING THESE SOLUATIONS
1. DynamoDB
Authentication and Access Control:

Use AWS Identity and Access Management (IAM) to control access to DynamoDB resources.
Assign fine-grained permissions using IAM policies to restrict access based on roles and permissions.
Consider using AWS Identity Federation to integrate with existing identity systems (e.g., Active Directory).
Encryption:

Enable DynamoDB encryption at rest to encrypt your data stored in DynamoDB tables using AWS Key Management Service (KMS) keys.
Use client-side encryption to encrypt sensitive data before sending it to DynamoDB.
Monitoring and Auditing:

Enable CloudTrail to log all API calls made to DynamoDB, providing visibility into who accessed the data and when.
Use CloudWatch Logs to monitor and analyze DynamoDB activity for suspicious behavior.
Backup and Restore:

Implement point-in-time recovery (PITR) to protect against accidental data loss by enabling continuous backups of your DynamoDB tables.
2. API Gateway
Authentication and Authorization:

Use AWS IAM roles and policies to control access to your APIs and resources integrated with API Gateway.
Implement OAuth 2.0 authorization for APIs requiring user authentication.
Traffic Management:

Use API keys and usage plans to throttle and control access to your APIs. Rotate API keys regularly.
Consider AWS WAF (Web Application Firewall) to protect APIs from common web exploits and attacks.
Encryption:

Enable TLS (Transport Layer Security) for API Gateway to encrypt data in transit between clients and your API endpoints.
Monitoring and Logging:

Enable CloudWatch Logs for API Gateway to capture detailed logs of API requests and responses for auditing and debugging purposes.
Use Amazon CloudFront integration for caching and protection against DDoS attacks.
3. Lambda
Execution Permissions:

Use AWS IAM roles to grant Lambda functions access permissions to specific AWS services like DynamoDB.
Avoid overly permissive roles and follow the principle of least privilege.
Environment Variables and Secrets:

Store sensitive configuration information using AWS Secrets Manager or AWS Systems Manager Parameter Store.
Use environment variables to securely pass sensitive information to your Lambda functions.
Network Isolation:

Deploy Lambda functions inside a VPC (Virtual Private Cloud) for enhanced network security, using private subnets and security groups to control inbound and outbound traffic.
Logging and Monitoring:

Enable CloudWatch Logs to monitor Lambda function execution and capture logs for troubleshooting and auditing purposes.
Use AWS X-Ray for tracing and debugging distributed Lambda applications.

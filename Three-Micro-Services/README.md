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
   - Statement ID provides an OPTIONAL identifier. Good for providing specific way to find out what an individial policy does inside a role.
   - Action gives an array of things you can do. Here are all things you can do to the dynamoDB table.
   - Effect - Deny has precedence over allow, good way to stop deletes
   - Resource is currently use * - good way to resstrict access is to have the actual ARN in here

2. Create a lambda function that will handle all the logic into performation actions into dynamoDb table
   A. Import Boto3- this lets your python3 code interact with AWS services.
   B. Import Json -
   - if 'tableName' in event:
        dynamo = boto3.resource('dynamodb').Table(event['tableName'])
         if this table name matches what is provided by APIgateway then actions provided by the api are triggered in the lambda function and pushed to dynamoDB
      -testing the lambda function -
                  if operation in operations:
                       return operations `[operation](event.get('payload'))`
         This return statement will output whatever data we put in while not being linked to a dynamoDB table


3. Create dynamoDB table
   a. Primary Key is very important as it is a unique identifier that distinguishes rows from each other

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

COST OPTIMIZATION
1. DynamoDB
   a.Provision capacity for predicable workoads. More cost effecive than On-demand.
   b.Auto-Scaling to avoid over Provision of capacity.
   c.Avoid using scans where possible; prefer queries or using secondary indexes.
   d.Use DAX for caching data to lower retrieval time.
   e.Use CloudWatch to monitor metrics.
2. API Gateway
   a. API request efficency by reducing number of API request per second with throttling and quotas
   b. Field filtering to lower amount being retrieved 
   c. Endpoints should be in proper region
3. Lambda
   a. Optimal mememory(which also adjust cpu/IOPS)
   b. Execution time limit 
   c. Reserved concurrency incase of unexepected usage

SECURITY 
1. DynamoDB
   a. IAM access for authorized users/groups/roles
   b. Encryption at rest with KMS
   c. Cloudtrail and CloudWatch for monitoring
   d. VPC endpoint access
   e. Continuous backup incase of accidental deletion
2. API Gateway
   a. IAM access for authorized users/groups/roles
   b. Cloudtrail API logs
   c. Usage plan for API keys
   d. Rotate API keys
   e. WAF and CloudFront to prevent against attacks
3. Lambda
   a. IAM role for lambda to access DynamoDB
   b. Environmental variables instead of hardcoding into code
   c. VPC with inbound and outbound rules
   d. Cloudwatch for auditing
   e. X-ray
   
   
   



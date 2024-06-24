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
   
         

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


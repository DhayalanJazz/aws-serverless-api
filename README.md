# aws-serverless-api  
This Project consists of a microservice architecture leveraging AWS Services such as API Gateway to create an API backed by a lambda function, dynamo DB as a database and CloudWatch for Real-time logging for monitoring and optimization.

![AWS Serverless Architecture](https://github.com/DhayalanJazz/aws-serverless-api/blob/images/AWS%20Serverless%20Architecture.png)
A resource is created and a method(POST) is defined and a method is backed by Lambda function and when the API is called through an HTTPS endpoint, Amazon API Gateway invokes the Lambda function.

The POST method on the DynamoDBManager resource supports the following DynamoDB operations:

* Create, update, and delete an item.
* Read an item.
* Scan an item.

The request payload you send in the POST request identifies the DynamoDB operation and provides necessary data. For example:

The following is a sample request payload for a DynamoDB create item operation:

```
{
    "operation": "create",
    "tableName": "lambda-apigateway",
    "payload": {
        "Item": {
            "id": "1",
            "name": "Max"
        }
    }
}
```

The following is a sample request payload for a DynamoDB read item operation:
```
{
    "operation": "read",
    "tableName": "lambda-apigateway",
    "payload": {
        "Key": {
            "id": "1"
        }
    }
}
```

## Setup

### Create Lambda IAM Role
Create the execution role that gives your function permission to access AWS resources.

To create an execution role

Step 1:Open the roles page in the IAM console.
Step 2:Choose Create role.
Step 3:Create a role with the following properties.
  * Trusted entity – Lambda.
  * Role name – lambda-apigateway-role.
  * Permissions – Custom policy with permission to DynamoDB and CloudWatch Logs. This custom policy has the permissions that the function needs to write data to DynamoDB and upload logs.
```
{
"Version": "2012-10-17",
"Statement": [
{
  "Sid": "Stmt1428341300017",
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
},
{
  "Sid": "",
  "Resource": "*",
  "Action": [
    "logs:CreateLogGroup",
    "logs:CreateLogStream",
    "logs:PutLogEvents"
  ],
  "Effect": "Allow"
}
]
}
```
### Create Lambda Function

**To create the Function**
 1. Click "Create function" in AWS Lambda Console
    

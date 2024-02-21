# aws-serverless-api  
This Project consists of a microservice architecture leveraging AWS Services such as API Gateway to create an API backed by a lambda function, dynamo DB as a database and CloudWatch for Real-time logging for monitoring and optimization.

![AWS Serverless Architecture](https://github.com/DhayalanJazz/aws-serverless-api/blob/main/images/AWS%20Serverless%20Architecture.png)
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
    
    ![Create function](https://github.com/DhayalanJazz/aws-serverless-api/blob/main/images/Create%20a%20function.png)
 3. Select "Author from scratch". Use name LambdaFunctionOverHttps , select Python 3.7 as Runtime. Under Permissions, select "Use an existing role", and select lambda-apigateway-role that we created, from the drop down
 4. Click "Create function"
    
    ![Lambda function setup](https://github.com/DhayalanJazz/aws-serverless-api/blob/main/images/Lambda%20Function%20Setup.png)
 5. Replace the boilerplate coding with the following code snippet and click "Save"
 ```
from __future__ import print_function

import boto3
import json

print('Loading function')


def lambda_handler(event, context):
    '''Provide an event that contains the following keys:

      - operation: one of the operations in the operations dict below
      - tableName: required for operations that interact with DynamoDB
      - payload: a parameter to pass to the operation being performed
    '''
    #print("Received event: " + json.dumps(event, indent=2))

    operation = event['operation']

    if 'tableName' in event:
        dynamo = boto3.resource('dynamodb').Table(event['tableName'])

    operations = {
        'create': lambda x: dynamo.put_item(**x),
        'read': lambda x: dynamo.get_item(**x),
        'update': lambda x: dynamo.update_item(**x),
        'delete': lambda x: dynamo.delete_item(**x),
        'list': lambda x: dynamo.scan(**x),
        'echo': lambda x: x,
        'ping': lambda x: 'pong'
    }

    if operation in operations:
        return operations[operation](event.get('payload'))
    else:
        raise ValueError('Unrecognized operation "{}"'.format(operation))
 ```
    

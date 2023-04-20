# Serverless-CRUD
Serverless CRUD

I have used 3 AWS services for this project, API Gateway, Lambda and DynamoDB. When you call the API through an HTTPS endpoint, Amazon API Gateway invokes the Lambda function. Lambda contains the logic for CRUD operations on DynamoDB table.


Setup:

Create Lambda IAM role : Create a lambda role with following details :
  Trusted entity – Lambda.
  Role name – lambda-apigateway-role.
  Permissions – Custom policy with permission to DynamoDB and CloudWatch Logs. This custom policy has the permissions that the function needs to write data to DynamoDB     and upload logs.
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
          
          
          
          Create Lambda Function: 
          
          Create a lambda function with name LambdaFunctionOverHttps , select Python 3.7 as Runtime. Under Permissions, select "Use an existing role", and select                 .lambda-apigateway-role . Below is the code :
          
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
    
    
    
    Create DynamoDB Table :
     Create a table with the following settings.
      Table name – lambda-apigateway
      Primary key – id (string)
      
    Create API :
     Go to API Gateway console. Click Create API and select "Build" for REST API, name it as "DynamoDBOperations"
     Create Resource : Input "DynamoDBManager" in the Resource Name, Resource Path will get populated. Click "Create Resource"
     Create a POST method : With the "/dynamodbmanager" resource selected, Click "Actions" again and click "Create Method".
     The integration will come up automatically with "Lambda Function" option selected. Select "LambdaFunctionOverHttps" function.
     
    Deploy the API:
     1. Click "Actions", select "Deploy API" and give the stage name as prod.
     2. To invoke our API endpoint, we need the endpoint url. In the "Stages" screen, expand the stage "Prod", select "POST" method, and copy the "Invoke URL" from             screen.
     
     Test run :
      To execute the API, I used Postman :
       Select "POST" , paste the API invoke url. Then under "Body" select "raw" and paste the below JSON. Click "Send". API should execute and return "HTTPStatusCode"        200. 
             {
          "operation": "create",
          "tableName": "lambda-apigateway",
          "payload": {
              "Item": {
                  "id": "1234ABCD",
                  "number": 5
              }
          }
      }


      Verify the table is created :
      
       Select "lambda-apigateway" table in DynamoDB from the console, select "Items" tab, and the newly inserted item should be displayed.
       


   

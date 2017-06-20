# s3-lambda-dynamodb

```
lambda
========

Create an s3 bucket "awstrainingtestbucket"
Create dynamodb table "table" and item "key"
Create IAM role    "lambda_dynamodb_role" --> trust --> lambda
Attach "AWSLambdaRole" managed policies
Attach inline policy to access dynamodb
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "Stmt1497430311000",
            "Effect": "Allow",
            "Action": [
                "dynamodb:*"
            ],
            "Resource": [
                "arn:aws:dynamodb:us-west-2:398818754185:table/table"
            ]
        }
    ]
}
```

Attach inline policy to access S3 bucket

```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "Stmt1497430489000",
            "Effect": "Allow",
            "Action": [
                "s3:*"
            ],
            "Resource": [
                "arn:aws:s3:::awstrainingtestbucket",
                "arn:aws:s3:::awstrainingtestbucket/*"
            ]
        }
    ]
}
```
lambda function code

````
import re
import json
import traceback
import boto3

s3_resource = boto3.resource('s3')
s3_client = boto3.client('s3')
dynamodb_client = boto3.client('dynamodb')

table_name = 'table'

def lambda_handler(event, context):
    bucket_name = event['Records'][0]['s3']['bucket']['name']
    key = event['Records'][0]['s3']['object']['key']
    if not key.endswith('/'):
        try:
            split_key = key.split('/')
            file_name = split_key[-1]
            s3_client.download_file(bucket_name, key, '/tmp/'+file_name)
            item = {'key': {'S': file_name}}
            dynamodb_client.put_item(TableName=table_name, Item=item)
        except Exception as e:
            print(traceback.format_exc())

    return (bucket_name, key)

```
Lambda input file

```
{
    "Records": [
        {"s3": 
            {"bucket": 
                {"name": "awstrainingtestbucket"},
            "object":
                {"key": "test/Untitled.rtf"}
            }
        }
    ]
}
```

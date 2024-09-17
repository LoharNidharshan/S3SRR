# S3 Replication Setup with AWS Lambda
## Overview
This project sets up S3 replication between two buckets (source and destination) using AWS SAM and includes a simple AWS Lambda function that returns a "hello world" response. The setup is defined in the CloudFormation template, and the replication allows copying objects between buckets automatically.

## Features
### S3 Replication: 
Configures replication between a source S3 bucket and a destination S3 bucket.
### Lambda Function: 
A simple AWS Lambda function that returns a 'hello world' response when invoked.
### IAM Role: 
An IAM role is created for the replication with the necessary policies to allow S3 replication actions.
## Architecture
### S3 Buckets: 
One source bucket and one destination bucket are created, both with versioning enabled.
### IAM Role & Policy: 
A role with specific permissions is created to manage S3 replication.
### Lambda Function: 
A basic function that responds with 'hello world' for API Gateway requests.
### SAM Template
The SAM template defines the resources needed for S3 replication and the Lambda function.

## SAM Template
```yaml
AWSTemplateFormatVersion: '2010-09-09'
Transform: AWS::Serverless-2016-10-31
Description: S3 Replication Setup

Resources:
  SourceBucket:
    Type: AWS::S3::Bucket
    Properties:
      VersioningConfiguration:
        Status: Enabled
      ReplicationConfiguration:
        Role: !GetAtt 'SourceBucketReplicationRole.Arn'
        Rules:
          - Destination:
              Bucket: !GetAtt 'DestinationBucket.Arn'
            Status: Enabled

  DestinationBucket:
    Type: AWS::S3::Bucket
    Properties:
      VersioningConfiguration:
        Status: Enabled

  SourceBucketReplicationRole:
    Type: 'AWS::IAM::Role'
    Properties:
      AssumeRolePolicyDocument:
        Statement:
          - Action:
              - 'sts:AssumeRole'
            Effect: Allow
            Principal:
              Service:
                - s3.amazonaws.com

  SourceBucketReplicationPolicy:
    Type: 'AWS::IAM::Policy'
    Properties:
      PolicyName: SourceBucketReplicationPolicy
      Roles:
        - !Ref SourceBucketReplicationRole
      PolicyDocument:
        Statement:
          - Action:
              - 's3:GetReplicationConfiguration'
              - 's3:ListBucket'
            Effect: Allow
            Resource: !GetAtt 'SourceBucket.Arn'
          - Action:
              - 's3:GetObjectVersion'
              - 's3:GetObjectVersionAcl'
            Effect: Allow
            Resource: !Sub '${SourceBucket.Arn}/*'
          - Action:
              - 's3:ReplicateObject'
              - 's3:ReplicateDelete'
            Effect: Allow
            Resource: !Sub '${DestinationBucket.Arn}/*'
```
## Lambda Function Code
The project also includes a simple AWS Lambda function that can be triggered via API Gateway. This function responds with a "hello world" message.

## Lambda Function (lambdaHandler)
```javascript
/**
 *
 * Event doc: https://docs.aws.amazon.com/apigateway/latest/developerguide/set-up-lambda-proxy-integrations.html#api-gateway-simple-proxy-for-lambda-input-format
 * @param {Object} event - API Gateway Lambda Proxy Input Format
 *
 * Context doc: https://docs.aws.amazon.com/lambda/latest/dg/nodejs-prog-model-context.html 
 * @param {Object} context
 *
 * Return doc: https://docs.aws.amazon.com/apigateway/latest/developerguide/set-up-lambda-proxy-integrations.html
 * @returns {Object} object - API Gateway Lambda Proxy Output Format
 */

export const lambdaHandler = async (event, context) => {
    const response = {
      statusCode: 200,
      body: JSON.stringify({
        message: 'hello world',
      }),
    };

    return response;
};
```
## API Endpoints
### Lambda Function: 
The Lambda function can be invoked via an API Gateway endpoint and will return a simple 'hello world' response.
## Example Response:
```json
{
  "message": "hello world"
}
```
## Setup & Deployment
### Prerequisites
### AWS Account: 
You need an AWS account with permissions to create S3 buckets, Lambda functions, and IAM roles.
### Node.js: 
Ensure Node.js is installed for local development.
### AWS SAM CLI: 
Install the AWS SAM CLI to build and deploy the application.
## Steps to Deploy
### Build the application:
```bash
sam build
```
## Deploy the application:
```bash
sam deploy --guided
```
Follow the prompts to specify parameters like stack name, region, etc.

Invoke the Lambda function: You can invoke the Lambda function via the API Gateway endpoint after deployment to see the 'hello world' response.

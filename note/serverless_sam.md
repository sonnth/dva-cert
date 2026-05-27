# AWS Serverless: SAM (Serverless Application Model)

## Overview

- Framework for developing and deploying serverless applications
- All the configuration is YAML code
- Generate complex CloudFormation for a simple SAM YAML file
- Supports anything from CloudFormation: Outputs, Mappings, Parameters, Resources,
- SAM can use CodeDeploy to deploy Lambda functions
- SAM can help run Lambda, API Gateway, DynamoDB locally

### SAM - Recipe
- Transform Header indicates its SAM template
  - Transform: AWS::Serverless-2016-10-31
- Write Code
  - AWS::Serverless::Function
  - AWS::Serverless::Api
  - AWS::Serverless::SimpleTable
- Package & Deploy: sam deploy (optionally preceded by "sam package")
- Quickly sync local changes to AWS Lambda (SAM Accelerate): sam sync --watch

### SAM Accelerate (sam sync)
- SAM Accelerate is a set of features to reduce latency while deploying resources to AWS
- sam sync
  - Synchronizes a project declared in SAM template to AWS
  - Synchronizes code changes to AWS without updating infrastructure (uses service APIs & bypass CloudFormation)
- sam sync (no options)
  - Synchronizes both infrastructure and code changes to AWS
- sam sync --code
  - synchronizes only code changes to AWS (bypass CloudFormation, update in seconds)
- sam sync --code --resource <resource_name>(AWS::Serverless::Function)
  - synchronizes only all Lambda functions and their dependencies
- sam sync --resource-id HelloWorldLambdaFunction
  - synchronizes only the specified resource by its ID
- sam sync --watch
  - Monitor for file changes and automatically synchronize them to AWS
  - If changes include configuration, it uses sam sync
  - If changes are code only, it uses sam sync --code

## SAM Policy Templates

- List of tempaltes to apply permissions to Lambda Functions
- Important examples:
  - S3ReadPolicy
  - SQSPollerPolicy
  - DynamoDBCrudPolicy

## SAM with CodeDeploy

- SAM framework natively uses CodeDeploy to update Lambda functions
- Traffic Shifting feature
- Pre- and Post-traffic hooks features to validate deployment (before traffic starts and after it ends)
- Easy & automated rollback using CloudWatch Alarms

- AutoPublishAlias:
  - detects when new code is being deployed
  - creates and publishes an updated version of function with latest code
  - points alias to the updated version
- DeploymentPreference
  - Canary, Linear, AllAtOnce
- Alarms
  - Alarms that can trigger a rollback
- Hooks
  - Pre- and post-traffic shifting Lambda functions to test your deployment
## SAM – Local Capabilities

- Locally start AWS Lambda
  - `sam local start-lambda`
  - starts a local endpoint that emulates AWS Lambda
  - can run automated tests against this local endpoint
- Locally Invoke Lambda Functions
  - `sam local invoke`
  - invokes a Lambda function locally
  - Invoke Lambda function with payload once and quit after invocation completes
  - helpful for generating test cases
  - if a function makes API calls to AWS, using a correct --profile option
- Locally start an API Gateway Endpoint
  - `sam local start-api`
  - starts a local HTTP server that hosts all functions
  - changes to functions are auto-reloaded
- Generate AWS Events for Lambda Functions
  - `sam local generate-event`
  - generates sample payloads for event sources
  - S3, API Gateway, SNS, Kinesis, DynamoDB...

## SAM – multiple environments
- samconfig.toml file to manage multiple environments
- sam deploy --config-env <env_name>
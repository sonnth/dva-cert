# Cloud Development Kit (CDK)

## Overview
- define cloud infrastructure using familiar languages
  - JavaScript, TypeScript, Python, Java, .NET
- contains high-level components called constructs
- the code compiled into CloudFormation template (JSON/YAML)
- therefore, deploy infrastructure and application runtime code together
  - great for Lambda functions
  - great for Docker containers in ECS/EKS
### CDK vs SAM
- SAM
  - severless focused
  - write template declaratively in YAML or JSON
  - great for quickly getting started with Lambda
  - Leverages CloudFormation
- CDK
  - All AWS services
  - write infra in a programming language
  - Leverages CloudFormation
- Can use SAM CLI to locally test CDK apps
## Constructs

- CDK Construct is a component that encapsulates everything CDK needs to create final CloudFormation stack
- Can represent a single or multiple AWS resource
- AWS Construct Library
  - a collection of Constructs included in AWS CDK
  - Contains three different levels of Constructs available (L1, L2, L3)
- Construct Hub: contains additional Constructs from AWS, third parties, and open source CDK community

### Layer 1 Constructs (L1)
- called CFN Resources, represents all resources directly available in CloudFormation
- Constructs are periodically generated from CloudFormation Resource Specification
- Construct names start with `Cfn` ( e.g. `CfnBucket` for S3 bucket)
- Must explicitly configure all resource properties

### Layer 2 Constructs (L2)
- represents AWS resources but with a higher level (intent-based API)
- Similar functinality as L1 but with more convenient defaults and boilerplate
  - Don't need to know all details about resource properties
- Provide methods that make it simpler (e.g., `bucket.addLifecycleRule()`)

### Layer 3 Constructs (L3)
- called Patterns, which represents multiple related resources
- helps you complete common tasks in AWS
- examples
  - `aws-apigateway.LambdaRestApi` represents an API Gateway backed by a Lambda function
  - `aws-ecs-patterns.ApplicationLoadBalancedFargateService` represents a Fargate service with an Application Load Balancer

## Commands & Bootstrapping

### Bootstrapping
- the process of provisioning resources for CDK before deploying CDK apps into AWS environment
- AWS environment = account & region
- CloudFormation Stack called CDKToolkit is created and contains
  - S3 bucket to store files
  - IAM Roles to grant permission to perform deployments
- Must run command `cdk bootstrap aws://ACCOUNT_ID/REGION` for each new environment
- Otherwise, error message `Policy contains a statement with one or more invalid principal` will be displayed

## Unit Testing
- to test CDK apps, use CDK Assertions Module combined with popular test frameworks such as Jest (JavaScript), Pytest(Python), or JUnit (Java)
- Verify we have specific resources, rules, conditions, parameters...
- Two types of tests:
  - Fine-grained Assertions (common): test specific aspects of the CloudFormation template (e.g.,check if a resource has this property with this value)
  - Snapshot Tests: test the synthesized CloudFormation template against a previously stored baseline template
- To import a template
  - Template.fromStack(MyStack): stack built in CDK

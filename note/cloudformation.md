# CloudFormation

## Overview

- Declarative way of outlining AWS Infrastructure, for any resources  
- CloudFormation creates in the right order, with the exact configuration   
- Visualizing using Infrastructure Composer

### Benefit

- Infrastructure as code  
  - No resources manually created, easy to control  
  - Code can be version controlled  
  - Changes to Infrastructure reviewed through code  
- Cost  
  - Each resources within the stack is tagged with an identifier  
  - Can estimate the costs of resources  
  - Savings strategy: could automation deletion of templates at 5PM and recreated at 8AM  
- Productivity  
  - Destroy and re-create infrastructure on the cloud on the fly  
  - Auto generation of Diagram for templates  
  - Declarative programming(no need to figure out ordering and orchestration)  
- Separation of concern: create many stacks for many apps, and many layers  
  - VPC stacks  
  - Network stacks  
  - App stacks  
- Don’t re-invent the wheel  
  - Leverage existing templates on the web  
  - Leverage documentation

### How CloudFormation Works

- Templates must be uploaded in S3 and then referenced in CloudFormation  
- To update template \-\> re-upload a new version of template to AWS  
- Stacks are identified by a name  
- Deleting a stack deletes every single artifact that was created by CloudFormation

### Deploying CloudFormation Templates

- Manual way:  
  - Editing templates in Infrastructure Composer or code editor  
  - Using console to input parameters  
- Automated way  
  - Editing templates in YAML file  
  - Using AWS CLI to deploy templates, using CD tool

### Building Blocks

- Template’s components  
  - AWSTemplateFormationVersion  
  - Description  
  - Resources(MANDATORY)  
  - Parameters  
  - Mappings  
  - Outputs  
  - Conditionals  
- Template’s Helpers  
  - References  
  - Functions

## YAML

- YAML and JSON can use for CloudFormation, but YAML is great so many ways  
- Key value pairs, nested objects, support arrays, multi line strings, can include comments

## Resources

- Resources are the core of CloudFormation template (MANDATORY)  
- Represent different types of AWS Components created and configured  
- Resources are declared and can reference each other  
- AWS figures out creation, updates and deletes of resources for us  
- 700 types of resources  
- Form: **service-provider::service-name::data-type-name**  
- All resources: [https://docs.aws.amazon.com/AWSCloudFormation/latest/TemplateReference/aws-template-resource-type-ref.html](https://docs.aws.amazon.com/AWSCloudFormation/latest/TemplateReference/aws-template-resource-type-ref.html)

## Parameters

- Parameters are a way to provide inputs to AWS CloudFormation template  
- Useful  
  - Want to reuse templates across company  
  - Some inputs can not be determined ahead of time  
- Parameters are extremely powerful, controlled, and can prevent errors from happening in templates

#### When should use a Parameter

- Is this CloudFormation resource configuration change in future. If so, make it a parameter  
- Won’t have to re-upload a template to change its content

#### Parameters Settings

- Parameters can be controlled by all these settings:  
  - Type  
  - Description  
  - ConstraintDescription  
  - Min/MaxLength  
  - Min/MaxValue  
  - Default  
  - AllowedValues  
  - AllowedPattern  
  - NoEcho

#### Reference a Parameter

- Use **Fn:Ref**   
- Parameters can be used anywhere in template  
- The shorthand for this YAML is **\!Ref**  
- The function can also reference other elements within template

#### Pseudo Parameters

- AWS offers us Pseudo Parameters in any CloudFormation template  
- Can be used any time and enabled by default  
- Important pseudo parameters  
  - AWS::AccountId  
  - AWS::Region  
  - AWS::StackId  
  - AWS::StackName  
  - AWS::NotificationARNs  
  - AWS::NoValue

## Mappings

- Mappings are fixed variables within CloudFormation  
- Handy to differentiate between different environments(dev vs prod), regions, AMI types  
- All values are hardcoded within template

#### Access Mapping Values

- Use **Fn::FindInMap** to return a named value from specific key  
- \!FindInMap\[MapName, TopLevelKey, SecondLevelKey\]

#### When Mappings, when Parameters

- Mappings when known in advance all values that are taken and can be deduced from variables (regions, AZ, AWS Account, Environment,..)   
- Parameters when values are really user specific 

## Output & Exports

- Outputs section declares optional outputs values that can import to other stacks(must export first)  
- View outputs in AWS Console or using AWS CLI  
- Useful define network CloudFormation, and output the variables such as VPC ID and Subnet IDs  
- Collaboration cross stack  
- Use **Fn::ImportValue** function  
- Can not delete the underlying stack until all the references are deleted

## Conditions

- Used to control the creation of resources or outputs based on a condition  
- Each condition can reference other condition, param value or mapping  
- Conditions can be applied to resources/ouputs/etc/…

```
// define
Conditions:
  CreateProdResources: !Equals [ !Ref EnvType, prod ] 
// apply
ResourcesL
  MountPoint:
    Type: AWS::EC2::VolumeAttachment
    Condition: CreateProdResources
```

## Intrinsic Functions

- Ref  
- Fn::GetAtt  
- Fn::FindInMap  
- Fn::ImportValue  
- Condition Functions (Fn::If, Fn::Not, Fn::Equals, etc,..)  
- Fn::Base64

### Fn::Ref

To reference

- Parameters \- returns value of the parameter  
- Resources \- returns physical ID of the underlying resource (EC2 ID,..)

The shorthand for this in YAML is \!Ref

### Fn::GetAtt

Attributes are attached to any resources created

### Fn::FindInMap

Return a named value from a specific key  
\!FindInMap\[MapName, TopLevelKey, SecondLevelKey\]

### Fn::ImportValue

Import values that are exported in other stacks

### Fn::Base64

Convert String to its Base64 representation  
Sample:

```
Resources:
 WebServer:
  Type: AWS::EC2::Instance
  Properties:
  ...
   UserData:
    Fn::Base64: |
	#!/bin/bash
	dnf update -y
	dnf install -y httpd
```

## Rollbacks

Stack creation fails:

- Default everything rolls back(gets deleted)  
- Option to disable rollback and troubleshoot what happen

Stack update fails:

- Auto rolls back to the previous known working state

## Service Role

- IAM role that allows CloudFormation to create/update/delete stack resources on your behalf  
- Give ability to users to create/update/delete the stack resources even if they don’t have permissions to work with the resources in the stack  
- Use cases:  
  - Achieve the least privilege principle  
  - Don’t want to give user all required permissions to create stack resources  
- User must have iam:PassRole permissions

## Capabilities

- CAPABILITY\_NAMED\_IAM and CAPABILITY\_IAM  
  - Use when CloudFormation template is creating or updating IAM resources (IAM User, Role, Group, Policy, Access Keys, Instance Profile,...)  
  - Specify CAPABILITY\_NAMED\_IAM if the resources are named  
- CAPABILITY\_AUTO\_EXPAND  
  - Use when CloudFormation template includes Macros or Nested Stacks  
  - Template change before deploying  
- InsufficientCapabilitiesException  
  - CloudFormation exception if capabilities haven’t been acknowledged when deploying a template 

## Deletion Policy

### DeletePolicy Delete

- Control when CloudFormation template is deleted or resource is removed  
- Extra safety measure to preserve and backup resources

Default DeletePolicy=Delete

- Delete won’t work on an S3 bucket if it is not empty

### DeletePolicy Retain

DeletePolicy=Retain

- Specify on resources to preserve in case of CloudFormation deletes  
- Work with any resources

### DeletePolicy Snapshot

DeletePolicy=Snapshot

- Create one final snapshot before deleting the resource  
- Supported resources: EBS Volume, ElastiCache, RDS,... 

## Stack Policy

- During a CloudFormation Stack update, all update actions are allowed on all resources (default)  
- A Stack Policy is a JSON defines the update actions that are allowed on specific resources during Stack updates  
- Protect from unintentional updates  
- When set Stack Policy, all resources in the Stack are protected by default  
- Specify an explicit ALLOW for the resources to be updated 

Sample: Allow updates on all resources except the ProductionDatabase

## Termination Protection

To prevent accidental deletes of CloudFormation Stacks, use TerminationProtection

## Custom Resources

Used to

- Define resources not supported by CloudFormation  
- Define custom provisioning logic for resources outside of CloudFormation (on-premises, 3rd party,..)  
- Have custom scripts run during process through Lambda functions

Defined in the template using AWS::CloudFormation::CustomResource or Custom::MyCustomResourceTypeName (recommend)

Backed by a Lambda function(most common) or an SNS topic

### Define a Custom Resource

- ServiceToken specifies where CloudFormation sends request to, such as Lambda ARN or SNS ARN(required and must be in the same region)  
- Input data parameters(optional)

Use case \- Delete content from an S3 bucket (use lambda to empty bucket and then delete)

## StackSets

- Create, update or delete stacks across multiple accounts and regions with a single operation/template  
- Target accounts to create, update,  delete stack instances from StackSets  
- When update a stack set, all associated stack instances are updated throughout all accounts and regions  
- Can be applied into all accounts of an AWS Organization
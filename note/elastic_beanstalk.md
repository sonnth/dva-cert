# AWS Elastic Beanstalk

## Overview

### Developer problem

- managing infra  
- deploying code  
- configuring db, load balancer,..  
- scaling concerns  
- Most web apps have the same architecture (ALB \+ ASG)

### Elastic Beanstalk

- Elastic Beanstalk is a developer centric view of deploying an app on AWS  
- uses all the components: EC2, ASG, ELB, RDS,...  
- managed service  
  - auto handles capacity provisioning, load balancing, scaling, app health monitoring, instance configuration,..  
- still have full control over the configuration  
- Beanstalk is free, pay for underlying instances

### Elastic Beanstalk Component

- Application (env, versions, configurations,..)  
- Application Version  
- Environment  
  - Collection of AWS resources running app version(1 app version at a time)  
  - Tiers: Web Server Environment Tier & Worker Environment Tier  
  - Create multiple env (dev, test, prod,..)

### Supported Platforms

- Go, Java SE, Java with Tomcat, .NET Core on Linux, .NET on Windows Server, Node.js, PHP, Python, Ruby, Packer Builder, Single Container Docker, Multi-container DOcker, Preconfigured Docker

### Deployment Modes

- Single instance (for dev)  
- High Availability with Load Balancer (for prod)

## First Environment

- Beanstalk \> Events to see events happening  
- CloudFormation \> Events all events within CloudFormation template  
- CloudFormation \> Application Composer visualize service created by CloudFormation

## SecondEnvironment

- High availability option \-\> multi AZ  
- Using load balancer \-\> no need to assign public IP  
- Enable Database in Beanstalk then it is linked to the lifecycle of environment. Delete environment \> DB go away

## Deployment Modes

### Deployment Options for Updates

- **All at once** (deploy all in one go) \- fastest, but it has some downtime  
- **Rolling** update few instances at a time(bucket), then move onto next bucket once the first is healthy  
- **Rolling with additional batches** like rolling, but spins up new instances to move the batch (old app still available)  
- **Immutable** spins up new instances in a new ASG, deploys version to these instances, and then swaps all instances when healthy  
- **Blue green**  create new environment and switch over when ready  
- **Traffic splitting**  canary testing \- send a small % of traffic to new deployment

#### All at once

- Fastest  
- Downtime  
- For quick iterations in develop environment  
- No additional cost

#### Rolling

- App running below capacity  
- Can set bucket size  
- App is running both versions simultaneously  
- No additional cost  
- Long deployment

#### Rolling with additional batches

- Running at capacity  
- Can set bucket size  
- App is running both versions simultaneously  
- Small additional cost  
- Additional batch is removed at the end of the deployment  
- Longer deployment  
- Good for prod 

#### Immutable

- #### Zero downtime

- New code is deployed to new instances on temp ASG  
- High cost, double capacity  
- Longest deployment  
- Quick rollback in case of failures (terminate new ASG)  
- Great for prod

#### Blue / Green

- Not a “direct feature” of Beanstalk  
- Zero downtime and release facility  
- Create new “stage” environment and deploy v2 there  
- The new environment (green) can be validated independently and rollback if issues  
- Route 53 setup using weighted policies to redirect a little bit traffic to the stage environment  
- Using Beanstalk, “swap URLs” when done with environment test

#### Traffic Splitting

- Canary Testing  
- New app version is deployed to a temp ASG with the same capacity  
- A small % of traffic is sent to temp ASG for configurable amount of time  
- Deployment health is monitored  
- If there’s a deployment failure, this trigger an automated rollback(very quick)  
- No downtime  
- New instances are migrated from the temp to original ASG  
- Old app version is terminated

## 

## CLI and Deployment Process

Speed up efficiency when using the CLI

- Describe dependencies ( requirements.txt for Python, package.json for [Node.js](http://Node.js))  
- Package code as zip, and describe dependencies  
- Console: upload zip file(creates new app version), and then deploy   
- CLI: create new app version using CLI  
- Elastic Beanstalk will deploy the zip on each EC2 instance, resolve dependencies and start the app

## Extensions

- All parameters set in the UI can be configured with code using files  
- Requirements:  
  - In the .ebextensions/   
  - YAML/JSON format  
  - .config extensions (example: logging.config)  
  - Able to modify some default setting using: option\_settings  
  - Ability to add resources such as RDS, ElastiCache, DynamoDB, …  
- Resources managed by .ebextensions get deleted if the environment goes away 

## CloudFormation

- Elastic Beanstalk relies on CloudFormation   
- CloudFormation is used to provision other AWS services  
- Use case: define CloudFormation resources in .ebextensions to provision ElastiCache, S3 bucket,.. 

## Cloning and Migrations

- Clone an environment with exact same configuration  
- Useful for deploying “test” version  
- All resources and configuration are preserved  
  - Load Balancer type and configuration  
  - RDS database type (data not preserved)  
  - Environment variables

## Migration

### Load Balancer

- After creating Elastic Beanstalk, can not change the Elastic Load Balancer type (only the configuration)  
- To migrate  
1. Create new environment with the same configuration except LB(can’t clone)  
2. Deploy app onto the new environment  
3. Perform CNAME swap or Route 53 update

### RDS 

- Not great for prod as db lifecycle is tied to Beanstalk environment  
- Best practice is separately create an RDS database and provide our EB app with the connection string

#### Depcouple RDS

1. Create snapshot of RDS DB  
2. Protect RDS from deletion in RDS console  
3. Create new Elastic Beanstalk environment, without RDS, point app to existing RDS  
4. Perform CNAME swap(blue/green) or Route 53 update, confirm working  
5. Terminate old environment  
6. Delete CloudFormation stack
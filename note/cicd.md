# AWS CICD, CodeCommit, CodePipeline, CodeBuild, CodeDeploy

## Introduction

- all about automating the deployment process
- CodeCommit: stores source code
- CodePipeline: automating the pipeline from code to Elastic Beanstalk
- CodeBuild: build and test the application
- CodeDeploy: deploying code to EC2 instances (not Elastic Beanstalk)
- CodeStar: manage software development activities in one place
- CodeArtifact: store, publish, and share software packages
- CodeGuru: automated code review using machine learning

### Continuous Integration (CI)
- developers push the code to a repository (GitHub, CodeCommit, Bitbucket, etc.)
- a testing / build server checks the code as soon as it's pushed (CodeBuild, Jenkins CI, etc.)
- developer gets feedback about the tests and checks that have passed / failed
- find bugs early, then fix them
- deliver faster as the code is tested
- deploy often, unblocked developer work
### Continuous Delivery (CD)
- ensures that the software can be released reliably whenever needed
- it ensures deployments often happen and are quick
- shift away from "one release every 3 months" to "5 releases a day"
- it usually means automated deployment (CodeDeploy, Jenkins CD, Spinnaker, etc.)

## CodeCommit overview

- version control system
- benefits: collaborate, backed-up, viewable, and auditable

### Why AWS CodeCommit
- Private Git repositories
- No size limit on repositories (scale seamlessly)
- Fully managed, highly available
- Code only in an AWS Cloud account (security, compliance)
- Integrates with AWS services
### Security
- Authentication
  - SSH Keys
  - HTTPS
- AUthorization
  - IAM policies
- Encryption
  - repositories are auto-encrypted at rest using KMS
  - encrypted in transit
- cross-account access
  - not share ssh keys or aws credentials
  - use IAM roles and use AWS STS (AssumeRole API)

## CodePipeline overview

- Visual Workflow to orchestrate CICD
- Source: CodeCommit, ECR, GitHub, S3, etc.
- Build: CodeBuild, Jenkins, CloudBees, TeamCity
- Test: CodeBuild, AWS Device Farm, third-party
- Deploy: CodeDeploy, Elastic Beanstalk, CloudFormation, ECS, S3, etc.
- Invoke: AWS Lambda, Step Functions.
- Consists of stages:
  - Each stage can have sequential actions and / or parallel actions
  - Example: Build -> Test -> Deploy -> Load Testing -> ...
  - Manual approval can be defined at any stage

### Artifacts
- each pipeline stage can create artifacts
- artifacts stored in an S3 bucket and passed on to the next stage

### Troubleshooting
- For CodePipeline Pipeline/Action/Stage Execution State Changes
- Use CloudWatch Events (Amazon EventBridge)
  - create events for failed pipelines
  - create events for cancelled stages
- If CodePipeline fails a stage, pipeline stops
- If the pipeline can't perform an action, make sure "IAM Service Role" attached does have enough IAM permissions (IAM Policy)
- AWS CLoudTrail can be used to audit AWS API calls

## CodeBuild overview

- Source: CodeCommit, GitHub, Bitbucket, S3
- Build instructions: Code file buildspec.yml or insert manually in Console
- Output logs can be stored in S3 & CloudWatch Logs
- Use CloudWatch Metrics to monitor build statistics
- Use EventBridge to detect failed builds and trigger notifications
- Use CloudWatch Alarms to notify if need "thresholds" for failures
- Build Projects can be defined within CodePipeline or CodeBuild
- Supported environments: Java, Ruby, Python, Go, Node.js, .NET Core, Android, PHP, Docker, etc.

### buildspec.yml
- must be at the root of code
- env: defined environment variables
  - variables: plaintext variables
  - parameter-store: variables stored in SSM Parameter Store
  - secrets-manager: variables stored in AWS Secrets Manager
- phases: specify commands to run
  - install: install dependencies
  - pre_build: commands to run before build
  - build: commands to run during build
  - post_build: finishing touches (e.g., zip output)
- artifacts: what to upload to S3 (encrypted with KMS)
- cache: file to cache (usually dependencies) to S3 for future build speedup

## CodeDeploy overview

- deployment service that automates application deployments
- deploy new app versions to EC2, On-premises, Lambda, and ECS
- Automated Rollback capability in case of failed deployments or trigger CloudWatch Alarm
- Gradual deployment control
- appspec.yml defines how the deployment happens

### EC2/On-premises Platform

- perform in-place deployments or blue/green deployments
- must run the CodeDeploy agent on the target instances
- Define deployment speed
  - AllAtOnce: most downtime
  - HalfAtATime: reduced capacity by 50%
  - OneAtATime: slowest, lowest availability impact
  - Custom: define %

### CodeDeploy agent
- The CodeDeploy agent must be running on EC2 instances as a pre-requisite
- It can be installed and updated automatically if using Systems Manager
- EC2 instances must have enough permissions to access S3 to get deployment bundles

### Lambda Platform
- CodeDeploy automates traffic shift for Lambda alias
- Feature is integrated within the SAM framework
- Linear: grow traffic every N minutes until 100%
  - LambdaLinear10PercentEvery1Minute
  - LambdaLinear10PercentEvery10Minute
- Canary: try X percent then 100%
  - LambdaCanary10Percent5Minutes
  - LambdaCanary10Percent10Minutes
- AllAtOnce: immediate

### ESC Platform
- CodeDeploy automates the deployment of a new ESC Task Definition
- Only Blue/Green Deployments
- Linear: grow traffic every N minutes until 100%
  - ECSLinear10PercentEvery1Minute
  - ECSLinear10PercentEvery10Minute
- Canary: try X percent then 100%
  - ECSCanary10Percent5Minutes
  - ECSCanary10Percent10Minutes
- AllAtOnce: immediate

## CodeDeploy for EC2 and ASG

### EC2
- define how to deploy the app using appspec.yml + Deployment Strategy
- will do in-place update to EC2 instances
- use hooks to verify the deployment after each deployment phase

### ASG
- in-place deployment
  - updates existing ec2 instances
  - newly created ec2 instances by an ASG will also get automated deployments
- blue/green deployment
  - a new Auto-Scaling Group is created (settings are copied)
  - choose how long to keep old ec2 instances (olg ASG)
  - must be using an ELB

### Redeploy and Rollback
- Rollback = redeploy the previous version
- Deployments can be rolled back:
  - Automatically: when a new deployment fails or rollback when a CloudWatch Alarm threshold is met
  - Manually
- Disable rollback: do not perform rollbacks for this deployment
- If a rollback happens, CodeDeploy redeploys the last known good revision as a new deployment (not a restored version)

## CodeArtifact overview

- Software packages depend on each other to be built (also called code dependencies), and new ones are created
- Storing and retrieving these dependencies is called artifact management
- Traditionally, need to set up artifact management manually
- CodeArtifact is a secure, scalable, and cost-effective artifact
- Works with common dependency management tools such as Maven, Gradle, npm, yarn, twine, pip, and NuGet
- Developers and CodeBuild can then retrieve dependencies straight from CodeArtifact

### EventBridge Integration
- Event is created when the Package version is created, modified, or deleted
- EventBridge can be used to trigger some services to Rebuild & Redeploy an App with the latest security fixes

### Resource Policy
- it can be used to authorize another account to access CodeArtifact
- A given principal can either read all the packages in a repository or none of them

## CodeGuru overview

- An ML-powered service for automated code reviews and application performance recommendations
- Provides 2 functionalities:
  - CodeGuru Reviewer: automated code reviews for static code analysis (development)
  - CodeGuru Profiler: visibility/recommendations for application performance (production)

### CodeGuru Reviewer
- Identify critical issues, security vulnerabilities, and hard-to-find bugs
- Example: common coding best practices, resource leaks, security detections, input validation
- Uses Machine Learning and automated reasoning
- Hard-learned lessons across millions of code reviews on 1000s of open-source and Amazon repositories
- Supports Java and Python
- Integrates with GitHub, Bitbucket, and AWS CodeCommit

### CodeGuru Profiler
- Helps understand the runtime behavior of app
- Example: identify app is consuming excessive CPU capacity on a logging routine
- Features:
  - Identify and remove code inefficiencies
  - Improve app performance (e.g., reduce CPI utilization)
  - Decrease compute costs
  - Provides heap summary (identify which objects are using up memory)
  - Anomaly Detection
- Support app running on AWS or on-premise
- Minimal overhead on application

## CodeGuru – Agent configuration
- MaxStackDepth: maximum stack depth in the code that represents in the profile
  - example: if CodeGuru Profiler finds a methodA, which calls methodB, which calls methodC, which calls methodD, then the depth is 4
  - if MaxStackDepth is set to 2, then the profiler evaluates A and B
- MemoryUsageLimitPercent: memory percentage used by the profiler
- MinimumTimeForReportingInMilliseconds: minimum time between sending reports
- ReportingIntervalInMilliseconds: the reporting interval used to report profiles (milliseconds)
- SamplingIntervalInMilliseconds: the sampling interval that is used to profile samples 
  - reduce to have a higher sampling rate
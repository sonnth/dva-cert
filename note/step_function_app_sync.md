# Other Serverless: Step Functions & AppSync

## Overview
### AWS Step Functions
- Model workflows as state machines (one per workflow)
  - Order fulfillment, Data processing
  - Web app, any workflow
  - Written in JSON
  - Visualization workflow and the execution, as well as history
  - Start the workflow with SDK call, API Gateway, Event Bridge (CloudWatch Events)
### Step Function – Task States
- do some work in a state machine
- invoke one AWS service
  - Lambda
  - Run AWS Batch job
  - Run an ECS task and wait for completion
  - Insert item from DynamoDB
  - Publish a message to SNS, SQS
  - Launch another Step Function workflow
- Run a one Activity
  - EC2, ECS, on-premises
  - Activities poll the Step functions for work
  - Activities send results back to Step Functions

### Step Function – States
- Choice State: Test for a condition to send to a branch (or default branch)
- Fail or Succeed State: Stop execution with failure or success
- Pass State: Pass its input to its output without performing work. Can be used to transform the data or add new data
- Wait State: Provide a delay for a certain amount of time or until a specified time / date
- Map State: Dynamically iterate steps
- Parallel State: begin parallel branches of execution

## Error Handling
- any state can encounter runtime errors for various reasons
  - state machine definition issues (e.g., no matching rule in a Choice state)
  - task failures (e.g., an exception in a Lambda function)
  - transient issues (e.g., network partition events)
- Use Retry (to retry a failed state) and Catch (transient to a failure path) in the State Machine to handle errors instead of inside Application Code
- predefined error codes:
  - States.ALL: matches any error name
  - States.TIMEOUT: Task ran longer than TimeoutSeconds or no heartbeat receieved
  - States.TaskFailed: execution failure
  - States.Permissions: not enough privileges to execute code
- The state may report its own errors

### Retry (Task or Parallel State)
- Evaluate from top to bottom
- ErrorEquals: match a specific kind of error
- IntervalSeconds: initial delay before retrying
- BackoffRate: multiplier for delay between retries
- MaxAttempts: maximum number of retry attempts (default: 3, set to 0 for never retried)
- When max attempts are reached, the Catch kicks in

### Catch (Task or Parallel State)
- Evaluate from bottom to top
- ErrorEquals: match a specific kind of error
- Next: state to send to
- ResultPath: a path that determines what input is sent to the state specified by Next

### ResultPath
- include the error in the input

## Wait for Task Token
- allows pausing Step Functions during a Task until a Task Token is returned
- Task might wait for other AWS services, human approval, or third party integration, call a legacy system...
- Append .waitForTaskToken to the Resource field to tell Step Functions to wait for a Task Token to be returned
- Task will pause until it receives a Task Token back with a SendTaskSuccess or SendTaskFailure API call

## Activity Tasks
- Enables to have Task work performed by an Activity Worker
- Activity Worker apps can be running on an EC2, Lambda, mobile device...
- Activity Worker poll for a Task using GetActivityTask API call
- After Activity Worker completes, it sends a response of its success/failure using SendTaskSuccess or SendTaskFailure API call
- To keep the Task active:
  - Configure how long the Task can wait by setting TimeoutSeconds
  - Periodically send a heartbeat from Activity Worker using SendTaskHeartbeat within the time you set in HeartBeatSeconds
- By configuring a long TimeoutSeconds and actively sending a heartbeat, Activity Task can wait up to 1 year

## Standard vs. Express
|                   | Standard                                        | Express                                                 |
|:------------------|:------------------------------------------------|:--------------------------------------------------------|
| Max. Duration     | up to 1 year                                    | up to 5m                                                |
| Execution Model   | Exactly one Execution                           | async at least one, sync at most one                    |
| Execution Rate    | over 2k/second                                  | over 100k/second                                        |
| Execution History | up to 90d or using CloudWatch                   | CloudWatch Logs                                         |
| Pricing           | number of State Transitions                     | number of executions, duration, mem consumption         |
| Use cases         | Non-idempotent actions (processing payments...) | IoT data ingestion, streaming data, mobile app backends |

### Execution Model Express
- Async:
  - don't wait Workflow complete (get results from CloudWatch Logs)
  - In case don't need an immediate response (messaging services)
  - must manage idempotence
- Sync:
  - Wait Workflow complete
  - Need an immediate response (e.g., orchestrate microservices)
  - Can be invoked from API Gateway or Lambda function

## AppSync overview
- A managed service that uses GraphQL
- Includes combining data from one or more sources
  - NoSQL data stores, relational databases, HTTP APIs...
  - Integrate with DynamoDB, Aurora, OpenSearch, and others
  - Custom sources with AWS Lambda
- Retrieve data in real-time with WebSocket or MQTT on WebSocket
- For mobile apps: local data access and data synchronization
- All starts with uploading one GraphQL schema
### AppSync Security
- Four ways to authorize apps to interact with AWS AppSync GraphQL API:
  - API_KEY
  - AWS_IAM: IAM users / roles / cross-account access
  - OPENID_CONNECT: OpenID Connect provider / JWT tokens
  - AMAZON_COGNITO_USER_POOLS
- for custom domain & HTTPS, use CloudFront in front of AppSync

## AWS Amplify
- set of tools to get started with creating mobile, web applications
- Must-have features such as data storage, authentication, storage, and machine-learning, all powered by AWS
- Front-end libs with ready-to-use components for React, Vue, JavaScript, iOS, Android, Flutter, ...
- Incorporates AWS best practices to for reliability, security, scalability
- Build and deploy with the Amplify CLI or Amplify Console

### Important features
- Authentication: `amplify add auth`
  - Leverages AWS Cognito
  - User registration, authentication, account recovery, and other operations
  - Support MFA, Social Sign-in...
  - Pre-buildt UI components
  - Fine-grained authorization
- Data Store: `amplify add api`
  - Leverages Amazon AppSync and DynamoDB
  - Work with local data and have auto-sync to the cloud without complex code
  - Powered by GraphQL
  - Offline and real-time capabilities
  - Visual data modeling with Amplify Studio
- Hosting: `amplify add hosting`
  - build and host modern web apps
  - CICD
  - Pull Request Previews
  - Custom domain
  - Monitoring
  - Redirect and Custom headers
  - Password protection

### End-to-End (E2E) Testing
- Run e2e tests in the test phase in Amplify
- Catch regressions before pushing code to production
- Use test step to run any test commands at build time (amplify.yml)
- Integrated with the Cypress testing framework
  - Allows generating a UI report for tests



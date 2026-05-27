# AWS Monitoring & Audit: CloudWatch, X-Ray and CloudTrail

## Monitoring overview

Things user care:

- App latency  
- App outages  
- Good outcome  
- Troubleshooting and remediation

Internal monitoring:

- Prevent issues before they happen  
- Performance and cost  
- Trends (scaling patterns)  
- Learning and improvement

### Monitoring in AWS

- AWS CloudWatch  
  - Metrics: collect and track key metrics  
  - Logs: collect, monitor, analyze and store log files  
  - Events: send notifications when events happen  
  - Alarms: react in real-time to metrics / events  
- AWS X-Ray:  
  - Troubleshooting app performance and errors  
  - Distributed tracing of microservices  
- AWS CloudTrail:  
  - Internal monitoring of API calls being made  
  - Audit changes to AWS Resources by users

## CloudWatch Metrics

- CloudWatch provides metrics for every services in AWS  
- Metric is a variable to monitor (CPUUtilization, NetworkIn,...)  
- Metrics belong to namespaces  
- Dimension is an attribute of a metric (instance id, environment,..)  
- Up to 30 dimensions per metric  
- Metrics have timestamps

### EC2 detailed monitoring

- EC2 instance metrics have metrics every 5min  
- With detailed monitoring if want to scale faster (for a cost) \-\> get data every 1min  
- Free tier allows to have 10 detailed monitoring metrics  
- Note: EC2 Memory usage is by default not pushed(must pushed from inside instance as custom metric)

### Custom metrics

- Define and send custom metrics  
- Example: RAM usage, disk space, logged in users,..  
- Use API PutMetricData  
- Use dimensions (attribute) to segment metrics  
  - Instance.id  
  - Environment.name   
- Metric resolution (StorageResolution API parameter \- 2 possible value):  
  - Standard: 1min   
  - High resolution: 1/5/10/30 s \- higher cost  
- **Important:** Accepts metric data points 2 weeks ago and 2h in the future (make sure configure EC2 instance time correctly) 

## CloudWatch Logs

- Log groups: arbitrary name, representing app  
- Log stream: instances within app / log files / containers  
- Can define log expiration policies (never expire, 1d to 10y,..)  
- Logs can send to:  
  - S3 (export)  
  - Kinesis Data Streams  
  - Kinesis Data Firehose  
  - AWS Lambda  
  - OpenSearch  
- Logs are encrypted by default  
- Can setup KMS-based encryption with own keys

### Sources

- SDK, CloudWatch Logs Agent, CloudWatch Unified Agent  
- Elastic Beanstalk: collection of logs from app  
- ECS: collection from containers  
- AWS LambdaL collection from function logs  
- VPC Flow Logs: VPC specific logs  
- API Gateway  
- CloudTrail based on filter  
- Route 53: Log DNS queries

### CloudWatch Logs Insights

- Search and analyze log data stored in CloudWatch Logs  
- Example: find a specific IP inside a log, count occurrences of “ERROR” in logs  
- Provides a purpose-built query language  
  - Auto discovers fields from AWS services and JSON log event  
  - Fetch desired event fields, filter based on conditions, calculate aggregate statistics, sort events, limit number of events.,.  
  - Save queries, add to CloudWatch Dashboards  
- Query multiple Log Groups in different AWS accounts  
- It’s query engine, not real-time engine

### S3 export

- Take up to 12 hours become available for export  
- API is CreateExportTask  
- Not near real-time or real-time, use Logs Subscriptions instead 

### CloudWatch Logs Subscription

- Get real-time log events from CloudWatch Logs for processing and analysis  
- Send to Kinesis Data Streams, Kinesis Data Firehose, or Lambda  
- Subscription Filter  
- Cross-account Subscription \- send log events to resources in different account (KDS, KDF)

## CloudWatch Agent & CloudWatch Logs Agent

- By default, no logs from EC2 machine go to CloudWatch  
- Run CloudWatch agent on EC2 to push the log files  
- Make sure IAM permissions are correct  
- CloudWatch log agent can be setup on-premises too

### CloudWatch Logs Agent & Unified Agent

- For virtual servers(EC2, on-premise server,...)  
- **CloudWatch Logs Agent**  
  - Old version of the agent  
  - Can only send to CloudWatch Logs  
- **CloudWatch Unified Agent**  
  - Collect additional system-level metrics (RAM, processes,..)  
  - Collect logs to send to CloudWatch Logs  
  - Centralized configuration using SSM parameter store

### CloudWatch Unified Agent \- Metric

- Collected directly on Linux server / EC2  
- CPU (active, guest, idle, system, user, steal)  
- Disk metrics (free, used, total), Disk IO (writes, read, bytes, iops)  
- RAM (free, inactive, used, total, cached)  
- Netstat (number of TCP and UDP connections, net packets, bytes)  
- Processes (total, dead, bloqued, idle, running, sleep)  
- Swap Space (free, used, used %)  
- Reminder: out of the box metrics for EC2 \- disk, CPU, Network (high level)

## CloudWatch Log \- Metric Filters

- CloudWatch Logs can use filter expressions  
  - Find specific IP  
  - Count occurrences of “ERROR”  
  - Trigger alarms  
- Filters only publish metric data points for events that happen after filter was created  
- Ability to specify up to 3 dimensions for Metric Filter (optional)

## CloudWatch Alarms

- Trigger notifications for any metric  
- Various options (sampling, %, max , min, ..)  
- Alarm states:  
  - OK  
  - INSUFFICIENT\_DATA  
  - ALARM  
- Period  
  - Length of time in seconds to evaluate the metric  
  - High resolution custom metrics: 10s, 30s, multiples of 60s

### Target

- Stop, Terminate, Reboot, or Recover EC2  
- Trigger Auto Scaling Action  
- Send notification to SNS

### Composite Alarms

- CloudWatch Alarms are on single metric  
- Composite Alarms are monitoring the states of multiple other alarms  
- AND and OR conditions  
- Helpful to reduce “alarm noise”

### EC2 Instance Recovery

- Status check:  
  - Instance status \= check the EC2 VM  
  - System status \= check the underlying hardware  
  - Attached EBS status \= check attached EBS volumes  
- Recovery: same private, public, elastic IP, meta data, placement group

### Good to know

- Alarms can be created based on CloudWatch Logs Metrics Filters  
- To test alarms and notifications, set alarm state to Alarm using CLI

## CloudWatch Synthetics

- Configurable script monitor APIs, URLs, Web,...  
- Reproduce what customers do to find issues  
- Checks availability and latency, store load time data and screenshots of the UI  
- Integration with CloudWatch Alarms  
- Scripts written in Node.js or Python  
- Headless Google Chrome browser  
- Run once or on regular schedule

### CloudWatch Synthetics Canary Blueprints

- Heartbeat Monitor \- load URL, store screenshot and an HTTP archive file  
- API Canary \- test read write APIs  
- Broken Link Checker \- check all links  
- Visual Monitoring \- compare screenshot with baseline screenshot  
- Canary Recorder \- record actions on web and auto generates script  
- GUI Workflow Builder \- verifies actions can taken from web

## Amazon EventBridge

- Schedule: cron jobs (script)  
- Event Pattern: Event rules to react to a service doing something  
- Trigger lambda, send SQS/SNS messages

### Rules

Sample source: 

- EC2(start instance)  
- CodeBuild(failed build)  
- S3 (upload)

Setup schedule or cron and filter source then send to EventBridge  
Sample destination: 

- Compute(Lambda, AWS Batch, ECS task)  
- Integration (SQS,SNS,Kinesis Data Streams)  
- Orchestration(Step Functions, CodePipeline, CodeBuild)  
- Maintenance(SSM, EC2 Actions)

### Event buses

-  Can be accessed by other AWS accounts using Resource-based Policies  
- Can archive events (all/filter) sent to event bus (indefinitely or set period)  
- Ability to replay archived events

### Schema Registry

- EventBridge can analyze events in bus and infer schema  
- Schema Registry allows generate code for app, to know how data is structured in bus  
- Schema can be versioned

### Resource-based Policy

- Manage permissions for specific Event Bus  
- allow/deny events from another AWS account or AWS region  
- Use case: aggregate all events from AWS Organization in a single account or region

## Amazon EventBridge \- Multi-Account Aggregation

Create resource policy on the event bus of the central account to accept events from other accounts

## X-Ray overview

Problems:

- Debugging in Production:  
  - Test locally  
  - Add log everywhere  
  - Re-deploy in production  
- Log formats differ between apps, analytics is hard  
- Hard to debugging distributed service  
- No common views

X-Ray: visual analysis of app

### Advantages

- Troubleshooting performance(bottlenecks)  
- Understand dependencies in microservice architecture  
- Pinpoint service issues  
- Review request behavior  
- Find errors and exceptions  
- Where throttled?  
- Identify impacted users

### Compatibility

- Lambda  
- Beanstalk  
- ECS  
- ELB  
- API Gateway  
- EC2 or any app server

### Leverages tracing

- Tracing: end to end way to following a “request”  
- Each component dealing with request adds its own “trace”  
- Tracing is made of segments (+ sub segments)  
- Annotations to provide extra-info  
- Be able to trace every request or sample request(% or rate per min)  
- X-Ray security: IAM for authorization, KMS for encryption at rest

### How to enable

1. Code(Java,Python,Go,Node.js,.NET) must import AWS X-Ray SDK  
- Little code modification  
- App SDK will capture:  
  - Calls to AWS services  
  - HTTP/HTTPS request  
  - DB calls  
  - Queue calls  
2. Install X-Ray daemon or enable X-Ray AWS Integration  
- X-Ray daemon works as a low level UDP packet interceptor(Linux/Windows/Mac,..)  
- AWS Lambda / other AWS services already run X–Ray daemon  
- Each app must have IAM rights to write data to X-Ray

### Magic

- Collect data from all different services  
- Service map is computed from all segments and traces  
- Graphical, non-tech can help troubleshoot

### Troubleshooting

- If not working on EC2  
  - IAM Role  
  - Instance is running X-Ray daemon  
- Enable on Lambda  
  - IAM Role with policy (AWSX-RayWriteOnlyAccess)  
  - X-Ray is import in the code  
  - Enable Lambda X-Ray Active Tracing

## X-Ray: Instrumentation and Concepts

- Instrumentation : measure product’s performance, diagnose errors, write trace information  
- To instrument app code \-\> X-Ray SDK  
- Modify app code to customize and annotation data SKD sends to X-Ray using interceptors, filters, handlers, middleware..

### X-Ray Concepts

- Segments: each app / service will send  
- Subsegments: details in segment  
- Trace: segments collected together to form an end-to-end trace  
- Sampling: decrease amount of requests sent to X-Ray, reduce cost  
- Annotations: key-value pairs used to index traces and use with filters  
- Metadata: key-value pairs, not indexed, not used for searching  
- The X-Ray daemon / agent has a config to send traces cross account:  
  - Make sure IAM permissions are correct \- agent will assume the role  
  - Allows to have a central account for all app tracing

### X-Ray Sampling Rules

- Control amount of data record  
- Modify sampling rules without changing code  
- By default, X-Ray SDK records **first request each second**, and 5% of any additional requests  
- One request per second is the ***reservoir***, which ensures that at least one trace is recorded each second  
- Five percent is the rate at which additional requests beyond the reservoir size are sampled

## X-Ray: Sampling Rules

- Can create rules with the reservoir and rate  
- Don’t have to restart app or SDK when change sampling rules in X-Ray console 

## X-Ray APIs

### Write APIs (used by X-Ray daemon)

- PutTraceSegments: uploads segment documents to AWS X-Ray  
- PutTelemetryRecords: Used by AWS X-Ray daemon to upload telemetry (SegmentsReceivedCount, SegmentsRejectedCounts, BackendConnectionErrors,..)  
- GetSamplingRules: Retrieve all sampling rules  
- GetSamplingTarges & GetSamplingStatisticSummaries  
- X-Ray daemon needs to have IAM policy authorizing the correct API calls to function 

### Read APIs

- GetServiceGraph: main graph  
- BatchGetTraces: list of traces specified by ID, each trace is collection of segment documents that originates from single request  
- GetTraceSummaries: IDs and annotations for traces available for specified time frame using optional filter. To get full traces, pass trace to BatchGetTraces  
- GetTraceGraph: service graph for one or more specific trace IDs

## X-Ray with Beanstalk

- Beanstalk include X-Ray daemon  
- Run daemon by setting option in Beanstalk console or with configuration file (.ebeextensions/xray-daemon.config)  
- Give instance correct IAM permissions to X-Ray daemon  
- App code must instrumented with X-Ray SDK  
- X-Ray daemon is not provided for Multicontainer Docker

## X-Ray & ECS

### Integration options

- ECS Cluster X-Ray Container as a Daemon  
- ECS Cluster X-Ray Container as a “Side Car”  
- Fargate Cluster X-Ray Container as a “Side Car”

## AWS Distro for OpenTelemetry

- AWS support Distribution of open-source OpenTelemetry  
- Provides a single set of APIs, libraries, agents, and collector services  
- Collects distributed traces and metrics from app  
- Collects metadata from AWS resources and services  
- Auto-instrumentation Agents to collect traces without changing code  
- Send traces and metrics to multiple AWS services and partner solutions (X-Ray, CloudWatch, Prometheus..)  
- Instrument apps running on AWS (EC2,ECS,EKS,Fargate,Lambda) as well as on-premises  
- Migrate from X-Ray to ADOT if standardized with open-source APIs from Telemetry or send traces to multiple destinations simultaneously

## CloudTrail

- Provides governance, compliance and audit for AWS account  
- CloudTrail is enabled by default  
- Get an history of events / API calls made within AWS account by (Console, SDK, CLI, AWS Services)  
- Put logs from CloudTrail into CloudWatch Logs or S3  
- A trail can be applied to All Regions (default) or single Region  
- If a resource is deleted in AWS, investigate CloudTrail first

### CloudTrail Events

- Management events:  
  - Operations that are performed on resources in AWS account  
  - Examples  
    - Config security  
    - Config rules for routing data  
    - Setting up logging  
  - By default, trails are configured to log management events  
- Data events:  
  - By default, data events are not logged(high volume operations)  
  - Amazon S3 object-level activity (GetObject, DeleteObject, PutObject)   
  - Lambda function execution activity (Invoke API)  
- CloudTrail Insights Events  
  - Enable to detect unusual activity  
    - Inaccurate resource provisioning  
    - Hitting service limits  
    - Bursts AWS IAM actions  
    - Gaps periodic maintenance activity  
  - Analyzes normal management events to create baseline  
  - Continuously analyzes write events to detect unusual patterns  
    - Anomalies appear in CloudTrail console  
    - Event is sent to S3  
    - An EventBridge event is generated

### CloudTrail Events Retention

- Stored for 90d  
- To keep events beyond period, log to S3 and use Athena

## CloudTrail \- EventBridge Integration

Flow  
User trigger some API \-\> CloudTrail logs and send event \-\> EventBridge \-\> SNS

## CloudTrail vs CloudWatch vs X-Ray

- CloudTrail  
  - Audit API calls made by users / services / AWS console  
  - Useful to detect unauthorized calls or root cause of changes  
- CloudWatch  
  - CW Metrics over time for monitoring  
  - CW Logs for storing app log  
  - CW Alarms to send notifications in case of unexpected metrics  
- X-Ray  
  - Automated Trace Analysis & Central Service Map Visualization  
  - Latency, Errors And Fault analysis  
  - Request tracking across distributed systems
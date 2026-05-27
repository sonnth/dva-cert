# AWS Integration & Messaging: SQS, SNS & Kinesis

## Introduction to Messaging

- Inevitably need to communicate with another  
- Two patterns of app communication: synchronous and asynchronous(event based)  
- Synchronous problem: sudden spikes of traffic.Sample: suddenly encode 1000 videos but usually it’s 10 \-\> overwhelmed  
  \=\> Better decouple apps  
- SQS: queue model  
- SNS: pub/sub model  
- Kinesis: real-time streaming model

These services can scale independently 

## SQS \- Standard Queues Overview

- Oldest offering (over 10 years old)  
- Fully managed service, used to decouple apps  
- Attributes:  
  - Unlimited throughput, unlimited number of messages in queue  
  - Default retention of messages: 4d, max 14d  
  - Low latency (\< 10ms on publish and receive)  
  - Limitation of 1024KB per message sent  
- Can have duplicate messages  
- Can have out of order messages

### Producing messages

- Produced to SQS using SDK (SendMessage API)  
- Message is persisted in SQS until a consumer deletes it

### Consumer messages

- Consumers(ec2 instances, servers, aws lambda,..)  
- Poll SQS for messages (up to 10 messages at a time)  
- Process the messages  
- Delete messages using DeleteMessage API

### Multiple EC2 Instances Consumers

- Consumers receive and process messages in parallel  
- At least once delivery  
- Best-effort message ordering  
- Consumers delete messages after processing them  
- Scale consumers horizontally to improve throughput of processing

### SQS with Auto Scaling Group (ASG)

- ASG poll for message from SQS Queue  
- CloudWatch Metric \- Queue Length set alarm for breach  
- CloudWatch Alarm \-\> scale instances in ASG

### SQS \- Security

- Encryption:  
  - In-flight encryption using HTTPS API  
  - At-rest encryption using KMS keys  
  - client-side encryption if client wants to perform encryption/decryption itself  
- Access Controls: IAM policies to regulate access to SQS API  
- SQS Access Policies (similar to S3 bucket policies)  
  - Useful for cross-account access to SQS queues  
  - Useful for allowing other services(SNS, S3,..) to write to SQS queue

## SQS Queue Access Policy

- Cross account access  
- Publish S3 Event Notifications to SQS Queue

## SQS \- Message Visibility Timeout

- After message polled by one consumer, it becomes invisible to other consumers  
- Default message visibility timeout is 30s (message has 30s to process)  
- After visibility timeout is over, message is “visible” in SQS  
- If message is not processed within visibility timeout, it will be processed twice  
- A consumer could call **ChangeMessageVisibility** API to get more time  
- If visibility timeout is high, and consumer crashes, re-processing will take time  
- If visibility timeout is low, we may get duplicates

## SQS \- Dead Letter Queues

- Set a threshold of how many times a message can go back to the queue  
- After MaximumReceives threshold is exceeded, message goes into dead letter queue (DLQ)  
- Useful for debugging  
- DLQ of a FIFO queue must be a FIFO queue, and also Standard queue  
- Make sure process message in the DLQ before they expire (good to set 14d in DLQ)

### Redrive to Source

- Feature to help consumer messages in DLQ to understand what wrong  
- When code is fixed then redrive messages from DLQ back into the source queue (or other queue) in batches without writing custom code

## SQS \- Delay Queues

- Delay a message (consumer don’t see immediately) up to 15mins  
- Default is 0  
- Can set default at queue level  
- Override default on send using DelaySeconds param

## SQS \- Certified Developer concepts

### Long Polling

- Consumer can optionally “wait” for messages to arrive if there are none in the queue, this called Long Polling  
- LongPolling decreases number of API calls made to SQS, increase efficiency and decrease latency of app  
- Wait time between 1s to 20s (20s preferable)  
- Long Polling is preferable to Short Polling  
- Long Polling can be enabled at queue level or at API level using ReceiveMessageWatiTimeSeconds

### SQS Extended Client

- Using SQS Extended Client (Java Library) to send large messages  
- Idea is using S3 as a repo for large data and send small metadata message to SQS

### Must know API

- CreateQueue(argument MessageRetentionPeriod to set how long message should be kept in queue), DeleteQueue(delete queue and all messages)  
- PurgeQueue: delete all messages  
- SendMessage(DelaySeconds), ReceiveMessage(polling), DeleteMessage  
- MaxNumberOfMessages: default 1, max 10(ReceiveMessage API)  
- ReceiveMessageWaitTimeSeconds: Long polling  
- ChangeMessageVisibility: change the message timeout  
- Batch APIs for SendMessage, DeleteMessage, ChangeMessageVisibility helps decrease costs

## SQS \- FIFO Queues

- Message send and receive in order  
- Limited throughput: 300 msg/s without batching, 3000 msg/s with  
- Exactly-once send capability (remove duplicates using Deduplication ID)  
- Messages are processed in order  
- Ordering by Message Group ID (all messages in the same group are ordered) \- mandatory parameter

### Deduplication

- De-duplication interval is 5 min  
- Two methods:  
  - Content-based: do SHA-256 hash the message body  
  - Provide Message Deduplication ID

### Message Grouping

- If specify the same MessageGroupID then only have one consumer, all messages in order  
- Ordering at level of subset of messages \-\> specify different values for MessageGroupID  
  - Messages order within group  
  - Each Group ID can have different consumer (parallel processing)  
  - Ordering across groups not guaranteed

## Amazon SNS

Send one message to many receivers \-\> pub/sub pattern

- The “event producer” only sends message to one SNS topic  
- Many “event receivers” (subscriptions) listen to SNS topic notifications  
- Each subscriber will get all messages  
- Up to 12.5m sub per topic  
- 100k topics limit  
- Many AWS services can send data directly to SNS for notifications

### Publish

- Topic publish (SDK)  
  - Create topic  
  - Create subscription  
  - Publish to topic  
- Direct publish (for mobile apps SDK)  
  - Create platform app  
  - Create platform endpoint  
  - Publish to platform endpoint  
  - Works with Google GCM, Apple APNC, Amazon ADM,..

### Security

- Encryption  
  - In-flight HTTPS API  
  - At-rest using KMS keys  
  - Client-side if encrypt/decrypt itself  
- Access Controls: IAM policies to regulate access to SNS API  
- SNS Access Policies (similar S3 bucket policies)  
  - Cross-account access  
  - Allow other services 

## SNS and SQS \- Fan Out Pattern

### Fan Out

- Push once in SNS, receive in all SQS queues that subscribed  
- Fully decoupled, no data loss  
- SQS allows for: data persistence, deploy processing and retries of work  
- Ability to add more SQS subscribers over time  
- Cross-region Delivery: works with SQS Queues in other regions

### S3 events to multiple queues

- Only have one S3 Event rule  
- Use fan-out to send the same S3 event to many SQS queues

### Application: SNS to S3 through Kinesis Data Firehose

SNS sends to Kinesis(for analysis, process,..) and then store S3 or any supported KDF destination

### SNS \- FIFO topic

- Similar as SQS FIFO  
  - Message Group ID  
  - Deduplication  
- Can have SQS Standard and FIFO queues as subscribers  
- Limited throughput (same as SQS FIFO)

### SNS FIFO \+ SQS FIFO: Fan out

- In case you need fan out \+ ordering \+ deduplication

### Message Filtering

- JSON policy used to filter messages sent to SNS topic’s subscriptions  
- If doesn’t have filter then receives all

## Amazon Kinesis Data Streams

- Collect and store streaming data in real-time  
- Real-time data \-\> Producers(Kinesis agent) \-\> Kinesis Data Streams \-\> Consumers(Lambda, amazon data firehose, apache flink,...)

### Kinesis data streams

- Retention between up to 365d  
- Ability to reprocess (replay) data by consumers  
- Data can’t be deleted from Kinesis (until it expires)  
- Data up to 10MB(use case is lot of small real-time data)  
- Data ordering guarantee for data with the same “Partition ID”  
- At-rest KMS encryption, in-flight HTTPS  
- Kinesis Producer Library (KPL) to write an optimized producer app  
- Kinesis Client Library (KCL) to write an optimized consumer app

### Capacity modes

- Provisioned mode  
  - Choose number of shards  
  - Each shard gets 1MB/s in (or 1k records per second)  
  - Each shard gets 2MB/s out  
  - Scale manually to increase or decrease the number of shards  
  - Pay per shard provisioned per hour  
- On-demand mode  
  - No need to provision or manage capacity  
  - Default (4MB/s in or 3k records per second)  
  - Scales auto based on observed throughput peak during last 30d  
  - Pay per stream per hour & data in/out per GB


## Amazon Data Firehose

- Fully managed service  
  - Support Amazon Redshift / S3 / OpenSearch Service  
  - 3rd party: Splunk / MongoDB / Datadog / NewRelic /…  
  - Custom HTTP Endpoint  
- Auto scaling, serverless, pay for use  
- Near real-time with buffering capability based on size / time  
- Support CSV, JSON, Parquet, Avro, Raw Text, Binary data  
- Conversions to Parquet / ORC, compressions with gzip / snappy  
- Custom data transformations using AWS Lambda (ex: CSV to JSON)

| Kinesis Data Streams | Amazon Data Firehose |
| :---- | :---- |
| Streaming data collection Producer & Consumer code Real-time Provisioned / On-Demand mode Data storage up to 365d Replay capability | Load streaming data into S3/Redshift/OpenSearch/3rd party/custom HTTP Near real-time Auto scaling No data storage Doesn’t support replay capability |

## Amazon Managed Service for Apache Flink

- Flink(Java, Scala or SQL) is a framework for processing data streams  
- Run any Apache Flink application on a managed cluster on AWS  
  - Provisioned compute resources, parallel computation, auto scaling  
  - App backups (implemented as checkpoints and snapshots)  
  - Use any Apache Flink programming features to transform data  
  - Important: Flink does not read from Amazon Data Firehose

## SQS vs SNS vs Kinesis

| SQS | SNS | Kinesis |
| :---- | :---- | :---- |
| Consumer pull data Data is deleted after consumed Many consumers as we want No need provision throughput Order only on FIFO queues Individual message delay capability | Push data to many subscribers Up to 12.5m subscribers Data not persisted (lost if not delivered) pub/sub Up to 100k topics No need to provision throughput Integrates with SQS for fan-out architecture pattern FIFO capability for SQS FIFO | Standard: pull data 2MB per shard Enhanced fan-out push data: 2MB per shard per consumer Possibility to relay data Meant for real-time big data, analytics and ETL Ordering at shard level Data expires after X days Provisioned mode or on-demand capacity mode |


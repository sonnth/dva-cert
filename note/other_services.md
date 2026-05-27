# AWS Other Services
## AWS SES

- Send emails to people using
  - SMTP interface
  - Or AWS SDK
- Ability to receive email. Integrates with
  - S3
  - Lambda
  - SNS
- Integrated with IAM for allowing to send emails

## Amazon OpenSearch Service

- Amazon OpenSearch is the successor to Amazon Elasticsearch Service
- In DynamoDB, queries only exist by primary key or indexes...
- With OpenSearch, can search any field, even partially matches
- It's common to use OpenSearch as a complement to another database
- Two modes: managed cluster or severless cluster
- Does not natively support SQL (can be enabled via a plugin)
- Ingestion from Kinesis Data Firehose, AWS IoT, and CloudWatch Logs
- Security through Cognito & IAM, KMS encryption, TLS
- Comes with OpenSearch Dashboards (visualization)

## Amazon Athena

- Serverless query service to analyze data stored in S3
- Uses standard SQL language to query the files (built on Presto)
- Supports CSV, JSON, ORC, Avro, and Parquet
- Pricing: $5.00 per TB of data scanned
- Commonly used with Amazon QuickSight for reporting/dashboards
- Use cases: Build intelligence / analytics / reporting, analyze & query VPC Flow Logs, ELB Logs, CloudTrail trails, etc.
- Exam Tip: analyze data in S3 using Serverless SQL, use Athena

### Performance Improvements

- Use columnar data for cost-savings (less scan)
  - Apache Parquet or ORC is recommended
  - Huge performance improvement
  - Use Glue to convert data to Parquet or ORC
- Compress data for smaller retrievals (bzip2, gzip, snappy, lz4, zlip, zstd...)
- Partition datasets in S3 for easy querying on virtual columns
- Use larger files (> 128MB) to minimize overhead

### Federated Query

- Allows running SQL queries across data stored in relational, non-relational, object, and custom data sources (AWS or on-premises)
- Uses Data Source Connectors that run on AWS Lambda to run Federated Queries (e.g., CloudWatch Logs, DynamoDB, RDS,...)
- Store the results back in S3

## Amazon MSK (Managed Streaming for Kafka)

- Alternative to Amazon Kinesis
- Fully managed Apache Kafka on AWS
  - Allow create/update/delete clusters
  - MSK creates and manages Kafka brokers nodes & Zookeeper nodes
  - Deploy MSK cluster in VPC, multi-AZ (up to 3 for HA)
  - Auto recovery from common Apache Kafka failures
  - Data is stored on EBS volumes for as long as you want
- MSK Serverless
  - Run Apache Kafka on MSK without managing the capacity
- MSK auto provisions resources and scales compute & storage

| Kinesis Data Streams      | Amazon MSK                                  |
|:--------------------------|:--------------------------------------------|
| 1 MB message size limit   | 1MB default, configure for higher (ex: 10MB |
| Data Streams with Shard   | Kafka Topics with Partitions                |
| Shard Splitting & Merging | Can only add partitions to a topic          |
| TLS In-flight encryption  | PLAINTEXT or TLS In-flight Encryption       |
| KMS at-rest encryption    | KMS at-rest encryption                      |

## Amazon Certificates Manager (ACM)

- Service to manage SSL/TLS certificates
- Used to provide in-flight encryption for websites (HTTPS)
- Free of charge for public TLS certificates
- Automatic TLS certificate renewal
- Integrations with (load TLS certificates on)
  - Elastic Load Balancers
  - CloudFront Distributions
  - APIs on API Gateway

## ACM Private CA

- managed service allows creating private Certificate Authorities (CA), including root and subordinaries CAs
- Can issue and deploy end-entity X.509 certificates
- Certificates are trusted only by Organization (not the public Internet)
- Works for AWS services that are integrated with ACM
- Use cases:
  - Encrypted TLS communication, Cryptographically signing code
  - Authenticate users, computers, API endpoints, and IoT devices
  - Enterprise customers building a Public Key Infrastructure (PKI)

## Amazon Macie

- fully managed data security and data privacy service that uses machine learning and pattern matching to discover and protect sensitive data in AWS
- it helps identify and alert sensitive data, such as personally identifiable information (PII)

## AWS AppConfig

- configure, validate, deploy dynamic configurations
- deploy dynamic configuration changes to apps independently of any code deployments
  - don't need restart app
- Feature flags app tuning, allow/block listing...
- use with apps on EC2, Lambda, ECS, EKS...
- Gradually deploy the configuration changes and rollback if issues occur
- Validate configuration changes before deployment using
  - JSON schema (syntactic check)
  - Lambda Function: run code to perform validation (semantic check)
# DynamoDB

## Overview

### Traditional Architecture:

- leverage RDBMS, have SQL query language
- Horizontal Scalability: add more servers
- Vertical Scalability: add more resources to existing server

### NoSQL databases:

- non-relational and distributed database service
- include MongoDB, DynamoDB,..
- not support query joins (or just limited support)
- all data query is present in one row
- don't perform aggregations such as SUM, AVG,..
- scale horizontally

### DynamoDB:

- fully managed service, automatic scaling and high availability, multi AZs
- millions of requests per second, trillions of row, 100s of TB of storage
- fast and consistent performance, single digit millisecond latency
- Integrated with IAM for security, authorization and administration
- Enables event driven programming with DynamoDB Streams
- Low cost and auto scaling capabilities
- Standard & Infrequent Access (IA) Table

### DynamoDB Basics

- DynamoDB is made of Tables, each table has primary key
- Tables can have infinite number of Items = rows
- Each item can have Attributes (columns), up to 400KB
- Data types supported:
  - Scalar types: String, Number, Binary, Boolean, Null
  - Set types: String Set, Number Set, Binary Set
  - Document types: List, Map

### DynamoDB Primary Key

- Option 1: Partition Key (Hash Key) only
  - Partition key must be unique for each item
  - Partition key must be "diverse" so that items are distributed evenly across partitions
- Option 2: Partition Key (Hash Key) + Sort Key (Range Key)
  - The combination must be unique for each item
  - Data is grouped by partition key

## DynamoDB WCU & RCU - Throughput

- Control table's capacity for read/write throughput
- Provisioned Mode:
  - Specify read and write per second
  - Need to plan capacity beforehand
  - Pay for provisioned read & write capacity units
- On-Demand Mode:
  - Pay for what you consume
  - No need to plan capacity beforehand
  - Pay for read & write requests, more expensive
- Can switch between provisioned and on-demand mode once per 24 hours.

### R/W Capacity Modes - Provisioned

- table must have provisioned read and write capacity units
- Read Capacity Units (RCU) - throughput for reads
- Write Capacity Units (WCU) - throughput for writes
- option to set up auto-scaling of throughput to meet demand
- throughput can be exceeded temporarily using "Burst Capacity"
- if Burst Capacity has been consumed -> ProvisionedThroughputExceededException
- advised to do an exponential backoff retry

### Write Capacity Units (WCU)

- 1 WCU represents one write per second for an item up to 1 KB in size
- if item larger than 1KB, more WCUs are consumed (rounded to the upper KB)
  - 10 items per second, size 2KB -> 20WCUs
  - 6 items per second, size 4.5KB (rounded 5KB) -> 30WCUs
  - 120 items per minute, size 2KB -> 4 WCUs

### Strongly Consistent Read vs Eventually Consistent Read

- Eventually Consistent Read (default)
  - If read just after a write, possible get some stale data because of replication
- Strongly Consistent Read
  - if read just after write, get the correct data
  - set "ConsistenRead" parameter to True in API calls (GetItem, BatchGetItem, Query, Scan)
  - consume twice RCU

### Read Capacity Units (RCU)

- 1 RCU represents 1 Strongly Consistent Read per second, or 2 Eventually Consistent Reads per second, for an item up to 4KB
- if items larger than 4KB, more RCU are consumed\

  - 10 Strongly Reads per second, size 4KB -> 10RCUs
  - 16 Eventually Reads per second, size 12KB -> 24RCUs
  - 10 Strongly Reads per second, size 6KB -> 20RCUs

### Partitions Internal

- data is stored in partitions
- partition keys go through the hashing algorithm to know which partition they go to
- to compute number of partitions
  - by capacity = (RCUs total/ 3000) + (WCUs total/1000)
  - by size = total size/ 10GB
  - result = ceil(max (by capacity, by size))
- WCUs and RCUs are spread evenly across partitions

### Throttling

- if exceed provisioned RCUs or WCUs -> ProvisionedThroughputExceededException
- reasons:
  - Hot Keys - 1 partition key is being read many times (popular item)
  - Hot Partititons
  - Very large items, RCU and WCU depends on size of items
- solutions:
  - exponential backoff when exception is encountered (already in SDK)
  - distribute partition keys as much as possible
  - if RCU issue, use DynamoDB Accelerator (DAX)

### R/W Capacity Modes - On-demands

- Read/writes auto-scale up/down with workloads
- no capacity planning needed (WCU/RCU), unlimited WCU & RCU, no throttle
- charged for reads/writes used in terms of RRU(Read Request Units) and WRU(Write Request Units)
- 2.5x more expensive than provisioned capacity
- use cases: unknown workloads, unpredictable app traffic,

## Basic Operations

### Write Data
- PutItem
  - Creates a new item or fully replaces an old item (same Primary Key)
  - Consumes WCUs
- UpdateItem
  - Edits an existing item's attributes or adds a new item if it does not already exist
  - Can be used to implement Atomic Counters - a numeric attribute that's unconditionally incremented
- Conditional Writes
  - Accept a write/update/delete only if conditions are met, otherwise retuns error
  - helps with concurrency access to items
  - no performance impact

### Read Data
- GetItem
  - Read based on Primary Key
  - Primary Key can be HASH or HASH + RANGE
  - Eventually Consistent Read by default, optionally Strongly Consistent Read (longer, more RCU)
  - ProjectionExpression can be specified to return only specific attributes
- Query
  - Query returns items based on:
    - KeyConditionExpression
      - Parititon Key value (must be = operator) - required
      - Sort Key value (=,<=,>=,BETWEEN,IN,BEGINS_WITH) - optional
    - FilterExpression
      - additional filtering after Query operation (before data returned)
      - use only with non-key attributes (does not allow HASH or RANGE attributes)
  - Returns:
    - number of items specified by Limit
    - Or up to 1MB data
  - Ability to do pagination on the results
  - Can query table, a Local Secondary Index, or a Global Secondary Index
- Scan
  - scan entire table and then filter out data (inefficient)
  - returns up to 1MB data - use pagination to keep on reading
  - consumes a lot of RCU
  - Limit impact using Limit or reduce result's size and pause
  - For faster performance, use Parrallel Scan
    - multiple workers scan multiple data segments at the same time
    - increase the throughput and RCU consumed
    - limit impact of parallel scans like Scans
  - Can us ProjectionExpression & FilterExpression (no changes to RCU)

### Delete Data
- DeleteItem
  - Delete individual item
  - ability to perform a conditional delete
- DeleteTable
  - delete table and all items in it
  - much quicker deletion than DeleteItem on all items

### Batch Operations
- allow to save latency by reducing API calls
- operations are done in parallel for better efficiency
- Part of a batch can fail; in which case need to try again for failed items
- BatchWriteItem
  - up to 25 PutItem and/or DeleteItem in one call
  - up to 16MB data written, up to 400KB per item
  - can't update items (use UpdateItem)
  - UnprocessedItems for failed write operations (exponential backoff or add WCU)
- BatchGetItem
  - return items from one or more tables
  - up to 100 items, 16MB data
  - items are retrieved in parallel to minimize latency
  - UnprocessedKeys for failed read operations (exponential backoff or add RCU)

### PartiQL
- SQL-compatible query language for DynamoDB
- allow select, insert, update, delete using SQL
- run queries across multiple tables
- run PartiQL queries from:
    - AWS Management Console
    - NoSQL Workbench for DynamoDB
    - DynamoDB APIs
    - AWS CLI
    - AWS SDKs

## Conditionally Writes

- For PutItem, UpdateItem, DeleteItem, and TransactWriteItems
- Can specify a Condition expression to determine which items should be modified
  - attribute_exists
  - attribute_not_exists
  - attribute_type
  - contains(string)
  - begins_with(string)
  - ProductCategory IN () and Price between and
  - size (string length)
- Note: Filter Expression filters the results of read queries, while Condition Expressions are for write operations

## Indexes (GSI +LSI)

### Local Secondary Index (LSI)
- Alternative Sort Key for table (same Partition Key as that of base table)
- Sort Key consists of one scalar attribute (String, Number, or Binary)
- Up to 5 LSI per table
- Defined at table creation time
- Attribute Projections - can contain some or all attributes of base table (KEYS_ONLY, INCLUDE, ALL)

### Global Secondary Index (GSI)
- Alternative Primary Key (HASH or HASH + RANGE) from the base table
- Speed up queries on non-key attributes
- The Index Key consists of scalar attributes (String, Number, or Binary)
- Attribute Projections - some or all attributes of the base table (KEYS_ONLY, INCLUDE, ALL)
- Must provision RCUs & WCU for index
- Can be added/modified after table creation

### Indexes and Throttling

- GSI
  - If writes are throttled on GSI, then main table will be throttled
  - Even if WCU on the main tables are fine
  - Choose GSI partition key carefully
  - Assign WCU capacity carefully
- LSI
  - Uses WCUs and RCUs of main table
  - No special throttling considerations

## PartiQL

- use a SQL-like syntax to manipulate DynamoDB tables
- Supports some (not all) statements:
  - INSERT
  - UPDATE
  - SELECT
  - DELETE
- It supports Batch operations

## Optimistic Locking

- DynamoDB has a feature called "Condition Writes"
- A stategy to ensure an item hasn't chagned before update/delete
- Each item has an attribute that acts as a version number

## DynamoDB Accelerator (DAX)

- fully-managed, highly available, seamless in-memory cache for DynamoDB
- microseconds latency,for cached reads & queries
- doesn't require app logic
- solves "Hot Key" problem (too many reads)
- 5m TTL for cache (default)
- up to 10 nodes in the cluster
- Multi-AZ (3 nodes minimum recommended for production)
- Secure (Encruption at rest with KMS, VPC, IAM, CloudTrail, ...)

### DAX vs ElastiCache

| DynamoDB Accelerator                                | ElastiCache              |
|-----------------------------------------------------|--------------------------|
| - individual objects cache<br/>- query & scan cache | store aggregation result |


## DynamoDB Streams

- ordered stream of item-level modifications(create, update, delete) in a DynamoDB table
- Stream records can be:
  - Sent to Kinesis Data Streams
  - Read by AWS Lambda
  - Read by Kinesis Client Library app (KCL)
- Data Retention for up to 24 hours
- Use cases:
  - react to changes in real-time (welcome email to users)
  - Analytics
  - Insert into derivative tables
  - Insert into OpenSearch service

- Ability to choose information that will be written to the stream
  - KEY_ONLY – only the key attributes of the modified item
  - NEW_IMAGE – the entire item, as it appears after the modification,
  - OLD_IMAGE – the entire item, as it appeared before the modification,
  - NEW_AND_OLD_IMAGES – both the new and old images of the item
- DynamoDB Streams are made of shards, just like Kinesis Data Streams
- Don't provision shards, AWS automates this
- Records are not retroactively populated in a stream after enabling it

### DynamoDB Streams & AWS Lambda
- Define an Event Source Mapping to read from a DynamoDB Stream
- Ensure Lambda function has the appropriate permissions
- Lambda function is invoked synchronously

## Dynamo TTL

- automatically delete items after an expiry timestamp
- no extra cost, doesn't consume WCUs
- TTL attribute must be a "Number" type with a "Unix Epoch timestamp" value
- Expired items are deleted within a few days of expiration
- Expired items that haven't been deleted appear in reads/queries/scans (filter if not wanted)
- Expired items are deleted from both LSIs and GSIs
- Delete operation for each expired item enters DynamoDB Streams (help recover)
- Use cases: reduce stored data, adhere to regulatory obligations, ...

## CLI
- `--projection-expression`: one or more attributes to retrieve
- `--filter-expression`: filter items before returning
- General AWS CLI Pagination options (e.g., DynamoDB, S3, ...):
  - `--page-size`: retrieves full list items but with a larger number of API calls instead of one API call (defulat 1000)
  - `--max-items`: max number of items to return (returns NextToken if more items)
  - `--starting-token`: specify the last NextToken to retrieve the next set of items

## Transactions

- Coordinated, all-or-nothing operations (add/update/delete) to multiple items across one or more tables
- Provides Atomicity, Consistency, Isolation, Durability (ACID) guarantees
- Read Modes – Eventual Consistent, Strongly Consistent, Transactional
- Write Modes – Standard, Transactional
- Consumes 2x WCUs & RCUs
  - performs 2 operations for every item (prepare & commit)
- Two operations:
  - TransactGetItems: one or more GetItem operations
  - TransactWriteItems: one or more PutItem, UpdateItem, DeleteItem operations
- Use cases: financial transactions, managing orders, multiplayer games, ...

### Capacity Computations
- Example 1: 3 transactional writes per second, size = 5KB
  - Need 3*(5KB/1KB)*2(transactional cost) = 1.5RCUs
- Example 2: 5 transactional reads per second, size = 5KB
  - Need 5*(8KB/4KB)*2(transactional cost) = 20RCUs

## DynamoDB as Session State Cache

- Common use DynamoDB to store session state
- vs Elasticache:
  - Elasticache is in-memory, DynamoDB is serverless
  - Both are key/value stores
- vs EFS:
  - EFS must be attached to EC2 instances as network drive
- vs EBS & Instance Store:
  - EBS & Instance store can only be used for local caching, not shared caching
- vs S3:
  - S3 is higher latency and not meant for small objects

## Partitioning Strategies

### Write sharding

- allows better distribution of items evenly across partitions
- add a suffix to Partition Key value
- two methods:
  - sharding using Random Suffix
  - sharding using Calculated Suffix

## Conditional Writes, Concurrent Writes & Atomic Writes

### Write types
- Concurrent writes:
  - overwrite the previous value of an item
- Conditional writes:
  - write-only if the item exists and the attribute value matches the specified condition
- Atomic writes:
  - both writes succeed or both fail
- Batch writes:
  - write/update many items at a time

## Patterns with S3

### Large Objects Pattern
- Application stores large objects in S3 
- DynamoDB stores metadata about the S3 object
- Application gets metadata from DynamoDB then gets a file from S3

### Indexing S3 Objects Metadata
- store object's metadata in DynamoDB
- client can filter objects by metadata

## DynamoDB Operations

- Table Cleanup
  - option 1: scan + DeleteItem
    - Very slow, consumes RCU & WCU, expensive
  - option 2: Drop Table + Recreate Table
    - Fast, efficient, cheap
- Copy a DynamoDB Table
  - option 1: Using AWS Backup
  - option 2: Using AWS Glue
  - option 3: Scan + PutItem or BatchWriteItem
    - Write code

## DynamoDB Security & Other

- Security
  - VPC Endpoints are available to access DynamoDB without using the Internet
  - Access fully controlled by IAM
  - Encryption at rest using AWS KMS and in-transit using SSL/TLS
- Backup and Restore feature available
  - Point-in-time recovery (PITR) like RDS
  - No performance impact
- Global Tables
  - Multi-region, multi-active, fully replicated, high performance
- DynamoDB Local
  - Develop and test apps locally without accessing the DynamoDB web service (without Internet)
- AWS Database Migration Service (AWS DMS) can be used to migrate to DynamoDB (from MonggoDB, Oracle, S3, ...)

### Users Interact with DynamoDB Directly
- Identity Provider (IdP)
  - AWS Cognito
  - Auth0
  - Okta
  - Google
  - Facebook
  - ...

### Fine-grained Access Control
- Using Web Identity Federation (WIF) or Cognito Identity Pools, each user gets AWS credentials
- IAM policies can be attached to IAM users with a Condition to limit API access
- LeadingKeys – limit row-level access for users on the Primary Key
- Attributes – limit specific attributes the user can see

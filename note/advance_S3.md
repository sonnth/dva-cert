# Advance Amazon S3

## Lifecycle Rules

- Transition Actions \- configure objects to transition to another storage class  
  - Move objects to Standard IA class 60 days after creation  
  - Move to Glacier for archiving after 6 months  
- Expiration actions \- configure objects to expire (delete) after some time  
  - access log files can be set to delete after 365 days  
  - Delete old versions of files  
  - Delete incomplete Multi-Part uploads  
- Rules can apply for a certain prefix (s3://mybucket/mp3/\*)  
- Rules can apply for certain objects Tag (Department: Finance)

#### Scenario 1

App on EC2: create images thumbnails after profile photos upload S3. Thumbnails easily recreated, need to be kept for 60d. Source images need to be retrieved immediately for these 60d, afterwards users can wait up to 6h. **Solution:**

- source images \-\> Standard and then Glacier after 60d  
- thumbnails \-\> One-Zone IA and then expire after 60d

#### Scenario 2

Recover deleted S3 objects immediately for 30d (rarely). After this time, and for up to 365d, deleted objects should be recoverable within 48h **Solution:**

- enable S3 Versioning, deleted objects hidden by "delete marker" and can be recovered  
- Transition the "noncurrent versions" of the objects to Standard IA  
- Transition afterwards the "noncurrent versions" to Glacier Deep Archive

### Storage Class Analysis

- Decide when to transition objects to right storage class  
- Recommendations for Standard and Standard IA (doesn't work for One-Zone IA or Glacier)  
- Report updated daily  
- 24 to 48h to start seeing data analysis

## Event Notifications

- S3:ObjectCreated, S3:ObjectRemoved, S3:ObjectRestore, S3:Replication,...  
- Object name filtering possible (\*.jpg)  
- Use case: generate thumbnails of images uploaded to S3  
- Can create as many "S3 events" as desired

### IAM permissions

Event notifications targets \-\> SNS topic, on SQS queue or on Lambda function

- Must define resource access policies on notifications targets Events \-\> S3 bucket \-all events-\> Amazon EventBridge \-rules-\> over 18 AWS services as destinations  
- advanced filtering options with JSON rules (metadata, object size, name,...)ß  
- muliple destinations \- Step Functions, Kinesis Streams, Firehose,...  
- EventBridge Capabilities \- Archive, Replay Events, Reliable delivery

## Performance

- S3 auto scales to high request rates, latency 100-200ms  
- achieve at least 3500 PUT/COPY/POST/DELETE or 5500 GET/HEAD requests per second prer prefix in a bucket  
- no limits number of prefixes in a bucket  
- if 4 prefixes \-\> 22000 requests per second for GET and HEAD

### Optimize

- **multi-part upload**(parallelize uploads): recommended for files \> 100MB, must use for \> 5GB ß  
- **transfer acceleration**: increase transfer speed by transferring fil4 to AWS edge location which forward data to the S3 bucket in the target region. It compatible with multi-part upload Example: file in USA upload to edge location USA and then transfer over the private AWS network to S3 Bucket Australia (minimize public and maximize private network)

### Byte-Range Fetches

- Parallelize GETs by requesting specific byte ranges  
- Better resilience in case of failures Can be used to speed up downloads or retrieve only partial data(head of a file)

## Object Tags & Metadata

- User-Defined Object Metadata  
  - name-value(key-value) pairs  
  - user-defined metadata names must begin with "x-amz-meta-"  
  - store keys in lowercase  
  - can retrieved while retrieving the object  
- Object Tags  
  - key-value pairs for objects in S3  
  - useful for fine-grained permissions  
  - useful for analytics purpose (S3 Analytics. to group by tags)  
- Can not search the object metadata or object tags  
- Instead, must use external DB as a search index such as DynamoDBß


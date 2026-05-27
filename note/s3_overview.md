# S3
## S3 overview

Use cases:

- Backup & storage

- disaster recovery

- archive

- hybird cloud storage

- application hosting

- media hosting

- data lakes & big data analytics

- software delivery

- static web

### S3 - Buckets

- S3 allows people to store objects(file) in "buckets" (directories)

- Buckets must have a globally unique name (across all regions all accounts)

- Buckets defined at region level

- S3 look like global service but buckets are created in a region

- naming convension: No uppercase, underscore, 3-36 characters long, not an IP, must start with lowercase letter or number, must not start with the prefix xn--, must not end with the suffix -s3alias

### S3 - Objects

- objects (files) have a Key

- The key is the FULL path:

+ s3://mybucket/my_file.txt

+ s3://mybucket/folder_1/my_file.txt

- The key is composed of prefix + object name

- There's no concept of "directories" within buckets

- Just keys with very long names that contains slashes "/"

- Object value are the content of the body: max 5TB, if more than 5GB then must use  "multi-part upload"

- Metadata (list of text key/value pairs - system or user metadata)

- Tags (Unicode key/value pair - up to 10) - useful for security/ lifecycle

- Version ID (if versioning is enabled)

### S3 - Security
- User based
	- IAM policies - which API calls should be allowed for user from IAM
	
- Resource-based
	- Bucket policies - bucket wide rules from S3 console - allow across account
	- Object Access Controll List (ACL) - finer grain (can be disabled)
	- Bucket Access Controll List (ACL) - less common (can be disabled)
- Note: can access if: 
	- IAM permission ALLOW OR resource policy ALLOW
	- AND  there's no explicit DENY
- Encryption: using encryption keys 

#### S3 Bucket Policies
- JSON based policies: Resources, Effect, Actions, Principal
- Use S3 bucket for policy to: Grant public access to the bucket, Force objects to be encrypted at upload, grant access to other account
```
Public access: -> S3 Bucket Policy allow public access
User access to S3 -> IAM permissions
EC2 instance access -> Use IAM Roles with IAM permissions
Cross-Account access -> Bucket Policy Allows Cross-Account
```
- Block Public Access
 - Prevent company data leak
 - if bucket should never public -> leave these on (default)
 - Can be set at account level
 
### S3 - Versioning
- verison files in S3
- enabled at bucket level
- savem key overwrite will change version: 1,2,3...
- best practice to version buckets:
	- protect against unintended deletes (able to restore)
	- easy roll back to previous
- note: file not versioned -> have version "null", suspending versioning does not delete the previous versions
- notice about delete marker, when delete the delete marker, object is recover
### S3 - Replication (CRR & SRR)
- must enable Versioning in source and destination buckets
- Cross-Region Replication (CRR), 2 regions must be different
- Same Replication (SRR), 2 regions are the same
- possible in different accounts, copying is asynchronous
- must give proper IAM permissions to S3
Usecase:
- CRR - compliance, lower latency access, replication across accounts
- SRR - long aggregation, live replication between production and test accounts
Note:
- after enable replication, only new objecss are replicated
- can replicate exsisting objects using S3 Batch Replication (for existing objects and failed replication)
- for DELETE: can replicate delete markers from source to target, deletions with a version ID are not replicated (avoid malicious deletes)
- no "chaining" of replication. if bucket 1 has replication into bucket 2, bucket 2 has replication into bucket 3 => objects created in bucket 1 are not replicated to bucket 3
### S3 Storage Classes
- Amazon S3 standard - general purpose
- Amazon S3 standard-infrequent access (IA)
- Amazon S3 one zone-infrequent access
- Amazon S3 glacier instant retrieval
- Amazon S3 glacier flexible retrieval
- Amazon S3 glacier deep archive
- Amazon S3 intelligent tiering
Can move objects between classes manually or using S3 Lifecycle configurations
#### S3 durability and availabitily
Durability

- high durability (11 9's), if store 10m objects with S3, average expect to incur a loss of single object once every 10k years
- same for all storage classes

Availabitily

- measures how readily available a service is
- Varies depending on storage classes
Example: S3 standard has 99.99% availability = not available 53 minutes a year

**S3 Standard**: general purpose
- 99.99% availability
- for frequently accessed data
- low latency, high throughput
- sustain 2 concurrent facility failures
Use cases: big data analytics, mobile & gaming app, content distribution
**S3 storage classes - infrequent access**
- for less frequently accessed, but requires rapid access when needed
- lower cost than S3 standard
- 2 types: 
  - S3 Standard-Infrequent Access (S3 Standard-IA)
  	99.9 availability, for disaster recovery, backups
  - S3 One Zone-Infrequent access (S3 One Zone-IA)
  	high durability (99.999999999%) in a single AZ, data lost when AZ is destroyed, 99.5% availability, for storing secondary backup copies of on-premise data or data can recreate

**S3 Glacier Storage**

- low-cost, for archiving, backup
- pricing: storage + object retrieval cost
- 3 types:
Amazon S3 glacier instant retrieval:

	- retrieval ms unit, greate for data accessed once a quarter
Amazon S3 glacier flexible retrieval
	
	- expedited (1-5mins), standard (3-5h), bulk (5-12h) - free
	- minimum storage duration of 90days
Amazon S3 glacier deep archive

	- standard (12h), bulk (48h)
	- minimum storage duration of 180 days

**S3 Intelligent-Tiering**
- small monthly monitoring, auto-tiering fee
- move objects auto between access tiers based on usage
- no retrieval charge





























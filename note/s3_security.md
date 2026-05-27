# S3 Security
## Encryption
4 methods:
- Server-side encryption (SSE)
	- Amazon S3-Managed Keys (SSE-S3) - default enabled
	- KMS Keys stored in AWS KMS (SSE-KMS)
	- Customer-Provided Keys (SSE-C)
- Client-Side Encryption
### SSE-S3
- keys handled, managed, owned by AWS
- encryption type is AES-256
- must set header "x-amz-server-side-encryption":"AES256"
- enabled by default for new buckets & new objects
### SSE-KMS
- keys handled, managed by AWS KMS(key management service)
- KMS advantages: user control + audit key usage using CloudTrail
- must set header "x-amz-server-side-encryption":"aws:kms"

Limitation:

- using SSE-KMS may impacted by KMS limit
- when upload, it calls GenerateDataKey KMS API. when download, it calls Decrypt KMS API
- count towards the KMS quota per second (5500, 10000, 30000 req/s based on region)
- can request a quota using Service Quota Console
### SSE-C
- keys fully managed by customer, AWS not store key
- HTTPS must be used
- Encryption key must provided in HTTP headers, for every HTTP request made
### Client-Side Encryption
- use client lib such as Amazon S3 Client-Side Encryption Library
- Clients must encrypt data before sending to S3
- Clients must decrypt data after retrieving to S3
- Customer fully manages keys and encryption cycle

### Encryption in transit (SSL/TLS)
encryption in flight also called SSL/TLS
- exposes 2 endpoints: HTTP(non encrypted), HTTPS(encryption in flight)
- HTTPS is recommended and it is mandatory for SSE-C
Can force encryption in transit by using bucket policy (aws:SecureTransport)

## Default Ecryption vs Bucket Policies
- SSE-S3 encryption is auto applied to new objects stored in S3 bucket
- Optionally, can "force encryption" using a bucket policy and refuse any API call to PUT an S3 object wihtout encryption headers (SSE-KMS or SSE-C)
Note: Bucket Policies are evaluated before "Default Encryption"

## CORS
Cross-Origin Resource Sharing (CORS)
- Origin = scheme (protocol) + host(domain) + port
- Web Browser based mechnism to allow requests to other origins while visiting the main origin
- The requests won't be fulfilled unless the other origin allows for the requests, using CORS Headers (example: Access-Control-Allow-Origin)
- If client makes a cross-origin request on S3 bucket, we need to enable the correct CORS headers
- Can allow for a specific origin or for * (all)
Web browser -> resource A -> resource B
=> resource B (Access-Controll-Allow-Origin must be "resource A") if A want to get resource from B

## MFA Delete
force users to generate a code on a device before doing important operations on S3

- MFA required to:
	- permanently delete an object version
	- suspend versioning on the bucket

- MFA won't be required to:
	- enable versioning
	- list deleted versions

- to use MFA Delete, Versioning must be enabled on the bucket

- Only bucket owner (root account) can enable/disable MFA Delete

## Access Logs
- any request made to S3, from any acc, authorized or denied, will be logged into another S3 bucket
- Data can be analyzed using data analysis tools(Amazon Athena,...)
- Target logging bucket must be in the same AWS region
#### Warning
- Not set logging bucket to be the monitored bucket (logging loop, bucket will grow exponentially)

## Pre-Signed URLs
Generate pre-signed URLs using S3 Console, AWS CLI or SDK
- URL expiration
	- S3 Console: 1-720 mins (72h)
	- AWS CLI: configure with expires-in parameter in seconds(default 3600s, max 604800s ~ 168h)
- Users given a pre-signed URL inherit the permissions of the user that generated the URL for GET/PUT
- Example:
	- allow only logged-in users to download file from S3 bucket
	- allow an ever-changing list of users to download files by generating URLs dynamically
	- allow temporarily a user to upload a file to a precise location in S3 bucket
	
## Access Points
- Simplify security management for S3 Buckets
- Each access point has:
	- its own DNS name (Internet Origin or VPC Origin)
	- an access point policy (similar to bucket policy) - manage security at scale
	
### VPC Origin
- define access point to be accessible only from within VPC
- must create a VPC Endpoint to access the Access Point (Gateway or Interface Endpoint)
- The VPC Endpoint Policy must allow access to the target bucket and Access Point

## S3 Object Lambda
- AWS Lambda Function to change the object before it is retrieved by the caller app
- Only one S3 bucket is needed, on top of which we create S3 Access Point and S3 Object Lambda Access Points
- Use cases:
	- Redacting personally identifiable information for analytics or non-production environments
	- Converting accross data formats, XML -> JSON,...
	- Resizing and watermarking images on the fly using caller-specific details, such as the user who requestsed object
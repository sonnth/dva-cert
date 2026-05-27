# AWS CloudFront
## Overview
- Content Delivery Network (CDN)
- improves read performance, content is cached at the edge
- improves users experience
- hundreds of Points of Presence globally (edge locations, caches)
- DDoS protection (because worldwide), integration with Shield, AWS Web Application Firewall
### CloudFront - Origins
- S3 buckets
	- for distributing files and caching them at the edge
	- for uploading files to S3 through CloudFront
	- Secured using Origin Acccess Control (OAC)
- VPC Origin
	- for apps hosted in VPC private subnets
	- private ALB/NLB/EC2 instances
- Custom Origin (HTTP)
	- S3 web (must first enable the bucket as a static S3 web)
	- Any public HTTP backend (Example: Public ALB)
### CloudFront high level
Client <-> CloudFront Edge Location(Local Cache) <-> Origin (S3 or HTTP)

## S3 as an Origin
- Edge Location in specific location will connect to S3 through private AWS network 
- Using Origin Access Control + S3 bucket policy to secure
#### CloudFront vs S3 Cross Region Replication
- CloudFront
	- Global Edge network
	- Files are cached for a TTL (maybe a day)
	- Greate for static content that must be available everywhere
- S3 Cross Region Replication:
	- Must be setup for each region
	- File are updated in near real-time
	- Read only
	- Greate for dynamic content that needs to be available at low-latency in few regions
	
## Caching & Caching Policies
- Cache key by default consists of hostname + resource portion of the URL
- Can add other elements (HTTP header, cookies, query strings) to Cache Key using CloudFront Cache Policies
- Cache based on:
	- HTTP Headers: None – Whitelist
	- Cookies: None – Whitelist – Include All-Except – All
	- Query Strings: None – Whitelist – Include All-Except – All
- TTL (0 seconds to 1 year), can be set by the origin using the Cache-Control header, Expires header...
- All HTTP headers, cookies, and query strings that you include in the  Cache Key are automatically included in origin requests
### Origin Request Policy
- specify included values in origin request without cache key (no duplicated cached content)
- can included:
	- HTTP headers
	- Cookies
	- Query Strings
### Cache invalidation
force entire or partial cache refresh by performing a CloudFront Invalidation
### Behavior
- configure different settings for a give URL path pattern
- route to different kind of origin, origin groups based on content type or path pattern (/images/*, /api/*, /*(default))
- when adding Cache Behavior, the Default Cache Behavior (/*) is always the last to be processed
### Maximize cache hits by separating static and dynamic distributions
- Dynamic: cache based on correct headers and cookie
- Static: no headers/session caching rules
## ALB/EC2 as an Origin
### Using VPC Origins
- deliver content from app hosted in VPC private subnets(ALB, NLB, EC2 instances)
### Using Public Network
- allow public IP of Edge Locations
## Geo Restriction
- Retrict who can access distribution
	- allowlist
	- blocklist
- Use case: Copyright Laws to control access to content
## Signed URL/Cookies
distribute paid shared content to premium users
- attach policy with:
	- includes URL expiration
	- includes IP ranges to access
	- Trusted signers
- how long should URL valid
	- shared content: make it short(few minutes)
	- private content: make it last for years
- signed URL = access to individual files (1 URL/1 file)
- singed Cookies = access to multiple files (1 cookie/many files)
### CloudFront signed URL vs S3 Pre-Signed URL
#### CloudFront signed URL
- allow access path, no matter origin
- account wide key-pair, only root manage
- filter IP, path, date, expiration
- leverage caching features
#### S3 Pre-Signed URL
- Issue a request as the person who pre-signed the URL
- Uses the IAM key of the signing IAM principal
- Limited lifetime
### CloudFront Signed URL Process
- Two types of signers:
	- Either a trusted key group (recommended)
		- Can leverage APIs to create and rotate keys (and IAM for API security)
	- An AWS Account that contains a CloudFront Key Pair
		- Need to manage keys using the root account and the AWS console
		- Not recommended because you shouldn’t use the root account for this
- In your CloudFront distribution, create one or more trusted key groups
- You generate your own public / private key
	- The private key is used by your applications (e.g. EC2) to sign URLs
	- The public key (uploaded) is used by CloudFront to verify URLs

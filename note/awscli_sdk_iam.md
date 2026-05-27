# AWS CLI, SDK, IAM Roles & Policies
## AWS EC2 Instance Metadata
- AWS EC2 Instance Metadata (IMDS) allows instances to learn about themselves without using an IAM Role
- URL is http://169.254.169.254/latest/meta-data
- can retrieve IAM Role name but can not retrieve IAM Policy
- Metadata = ec2's info, Userdata = launch script
- IMDS v2 vs v1, v2 is more secure by using session token

## AWS CLI, SDK
### AWS CLI profile
- manage account using profile:
```bash
aws configure --profile other-account
aws s3 ls --profile other-account
```
### MFA with CLI
- to use MFA with CLI, must create a temporary session
- to do so, must run the STS GetSessionToken API call
API use to generate temporary session tokens
- **aws sts get-session-token** --serial-number arn-of-the-mfa-device --token-code code-from-token --duration-seconds 3600
### AWS Limits (Quotas)
- API rate limits:
	- DescribleInstances API for EC2 has a limit of 100 calls per second
	- GetObject on S3 has lmit of 5500 GET per second per prefix
	- For Intermittent Errors: implement Exponential Backoff
	- For Consistent Errors: request an API throttling limit increase
- Service Quotas (Service Limits):
	- running on-demand standard instances: 1152 vCPU
	- can request a service limit increase by opening a ticket
	- can request a service quota increase by using the Service Quotas API
#### Exponential Backoff (any AWS service)
- if get ThrottlingException intermittently, use exponential backoff
- retry mechanism already included in AWS SDK API calls
- must implement yourself if using AWS API:
	- only retry on 5xx server error and throttling
	- not implement on 4xx client errors
## AWS Credentials Provider & Chain
#### CLI look for credentials in this order:
1. Command line options: --regison, --output, --profile
2. Environment variables: AWS_ACCESS_KEY_ID, AWS_SECRET_ACCESS_KEY, AWS_SESSION_TOKEN
3. CLI credentials file: aws configure (~/.aws/credentials)
4. CLI configuration file: aws configure (~/.aws/config)
5. Container credentials: for ECS tasks
6. Instance profile credentials: EC2 instance profile
#### AWS SDK Default Credential Provider Chain
The Java SDK (example) will look for credentials in this order
1. Java system properties: aws.accessKeyId and aws.secretKey
2. Environment variables: AWS_ACCESS_KEY_ID and AWS_SECRET_ACCESS_KEY
3. The default credential profiles file: ~/.aws/credentials
4. Amazon ECS container credentials: ECS containers
5. Instance profile credentials: EC2 instances

#### AWS Credentials best practices
- overall, never store aws credentials in your code
- best practice is for credentials to be inherited from the credentials chain
- if using working within AWS, use IAM Roles:
	- EC2 instances roles for ec2 instances
	- ECS roles for ECS tasks
	- Lambda roles for Lambda functions
- if working outside of AWS, use environment variables/named profiles
## AWS signature
- when call AWS HTTP API, sign the request so AWS can identity
- some requests don't need to be signed
- if use SDK or CLI, HTTP requests are signed for you
- should sign an AWS HTTP using Signature v4 (SigV4)
- 2 ways to sign: authentication header and query string

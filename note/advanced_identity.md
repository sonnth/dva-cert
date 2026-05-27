# Advanced Identity

## STS Overview
- Allows granting limited and temporary access to AWS resources (up to 1 hour)
- AssumeRole: Assume roles within your account or cross-account
- AssumeRoleWithSAML: return credentials for users logged with SAML
- AssumeRoleWithWebIdentity
  - return creds for users logged with an IdP (Facebook Login, Google Login, OIDC compatible…)
  - AWS recommends using Cognito Identity Pools instead of calling this API
- GetSessionToken: for MFA, from a user or AWS account root user
- GetFederationToken: get temporary creds for a federated user
- GetCallerIdentity: return details about the IAM user or role used in the API call
- DecodeAuthorizationMessage: decode an error message when an AWS API is denied

### Using STS to Assume a Role
- Define an IAM Role within or cross-account, define which principals can access it
- Use AWS STS to retrieve credentials and impersonate IAM Role has access to (AssumeRole API)
- Temporary credentials can be valid between 15m to 1h

### STS with MFA
- Use GetSessionToken from STS
- Appropriate IAM policy using IAM Conditions
- aws:MultiFactorAuthPresent:true condition key
- Reminder, GetSessionToken return Access ID, Secret Key, Session Token, and Expiration time

## Advanced IAM

### Authorization Model. Evaluation of Policies, simplified

1. If there's an explicit DENY, end decision and DENY
2. If there's an ALLOW, end decision with ALLOW
3. Else DENY
Have both ALLOW and DENY, DENY takes precedence

### IAM Policies & S3 Bucket Policies
- IAM Policies are attached to users, roles, groups
- S3 Policies are attached to buckets
- when evaluating an IAM Principal can perform an operation X on a bucket, the union of its assigned IAM Policies and S3 Bucket Policies is evaluated will be evaluated
- Example 1:
  - IAM Role attached to EC2 instance, authorizes RW to "my bucket"
  - No S3 Bucket Policy attached
  ⇒ EC2 instance can read and write to "my bucket"
- Example 2:
  - IAM Role attached to EC2 instance, authorizes RW to "my bucket"
  - S3 Bucket Policy attached, explicit deny to IAM Role
  ⇒ EC2 instance cannot read and write to "my bucket"
- Example 3:
  - IAM Role attached to EC2 instance, no S3 bucket permissions
  - S3 Bucket Policy attached, explicit RW allow to IAM Role
  ⇒ EC2 instance can read and write to "my bucket"
- Example 4:
  - IAM Role attached to EC2 instance, explicitly DENY S3 bucket permissions
  - S3 Bucket Policy attached, explicit RW allow to IAM Role
  ⇒ EC2 instance cannot read and write to "my bucket"

### Dynamic Policies with IAM
- Assign each user a folder in S3: 
  - option 1: create one policy per user
  - create one dynamic policy with IAM, then leverage the special variable ${aws:user}

### Inline vs. Managed Policies
- AWS Managed Policies:
  - Maintained by AWS
  - Good for power users and administrators
  - Updated in case of new services / new APIs
- Customer Managed Policy:
  - Best Practice, re-usable, can be applied to many principals
  - Version Controlled and rollback, central change management
- Inline
  - Strict one-to-one relationship between policy and principal
  - Policy is deleted if delete IAM principal

## Granting a User Permissions to Pass a Role to an AWS Service
- to configure many AWS services, must pass an IAM role to service (happens only once during setup)
- the service will later assume the role an perform actions
- Example:
  - To an EC2 instance
  - To a Lambda function
  - To an ECS Task
  - To CodePipeline to allow it to invoke other AWS services
- Need the IAM permission iam:PassRole
- It often comes with iam:GetRole to view the role being passed
- Roles can only be passed to what their trust allows
- A trust policy for the role that allows the service to assume the role

## AWS Directory Service (AWS DS)
### Microsoft Active Directory (AD)
- On Windows Server with AD Domain Services
- Database of objects: User Accounts, Computers, Printers, File Shares, Security Groups
- Centralized security management, create account, assign permissions
- Objects are organized in trees
- A group of trees is a forest

### AWS Directory Services
- AWS Managed Microsoft AD
  - create AD in AWS, manage users locally, support MFA
  - Establish "trust" connections with on-premise AD
- AD Connector
  - Directory Gateway (proxy) to redirect to on-premise AD supports MFA
  - Users are managed on the on-premise AD
- Simple AD
  - AD-compatible managed directory on AWS
  - Cannot be joined with on-premise AD
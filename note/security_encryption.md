# AWS Security and Encryption: KMS, Encryption SDK, SSM Parameter Store, IAM & STS

## Encryption 101

### Encryption in flight (TLS/SSL)

- Data is encrypted before sending and decrypted after receiving.
- TLS certificates help with encryption (HTTPS)
- ensure no MITM (man in the middle) attacks

### Server-side encryption at rest

- Data is encrypted after being received by the server
- Data is decrypted before being sent
- It is stored in an encrypted form
- The encryption / decryption keys must be managed somewhere, and the server must have access to it

### Client-side encryption

- Data is encrypted by the client and never decrypted by the server
- Data will be decrypted by a receiving client
- The server should not be able to decrypt the data
- Could leverage Envelope Encryption

## KMS (Key management service)

- mostly KMS is used for encryption AWS services
- AWS manages encryption keys
- Fully integrated with IAM for authorization
- Easy way to control access to data
- able to audit KMS Key usage using CloudTrail
- Seamlessly integrated into most AWS services (S3, EBS, RDS, SSM, etc.)
- Never ever store secrets in plaintext, especially in code
  - KMS Key Encryption is also available through API calls (SDK, CLI)
  - Encrypted secrets can be stored in the code / environment variables

### KMS Keys Types

- KMS Keys is the new name of KMS Customer Master Key
- Symmetric (AES-256 keys)
  - Single encryption key used to Encrypt and Decrypt
  - AWS Services that are integrated with KMS use Symmetric CMKs
  - Never get access to KMS Key unencrypted (must call KMS API to use)
- Asymmetric (RSA & ECC key pairs)
  - Public (Encrypt) and Private (Decrypt) pair
  - Used for Encrypt / Decrypt, or Sign / Verify operations
  - The public key is downloadable, but can't access the Private Key unencrypted
  - Use case: encryption outside AWS by users who can't call KMS API
- Types of KMS Keys:
  - AWS Owned Keys (free): SSE-S3, SSE-SQS, SSE-DDB (default key)
  - AWS Managed Key: free (aws/service-name, example: aws/rds or aws/ebs)
  - Customer managed keys created in KMS: $1 / month
  - Customer managed keys imported: $1 / month
  - - pay for API call to KMS ($0.03 / 10000 calls)
- Automatic Key rotation:
  - AWS-managed KMS Key: automatic every 1 year
  - Customer-managed KMS Key: (must be enabled) automatic & on-demand
  - Imported KMS Key: only manual rotation possible using alias

### Copying Snapshots across regions

- Snapshots are decrypted by key in a source region and re-encrypted by a new key in the destination region

### KMS Key Policies

- Control access to KMS keys, "similar" to S3 bucket policies
- Difference: cannot control access without them
- Default KMS Key Policy:
  - Created if you don't provide a specific KMS Key Policy
  - Complete access to the key to the root user = entire AWS Account
- Custom KMS Key Policy:
  - Define users, roles that can access the KMS Key
  - Define who can administer the key
  - Useful for cross-account access of KMS Key

### Copying Snapshots across accounts

1. Create a Snapshot, encrypted with KMS Key (Customer Managed Key)
2. Attach a KMS Key Policy to authorize cross-account access
3. Share the encrypted Snapshot
4. (source) Create a copy of the Snapshot, encrypt with a CMK in account
5. Create a volume from the Snapshot

## KMS Encryption Patterns and Envelope Encryption

### API Encrypt and Decrypt (< 4KB)

- Encrypt API: Service -> call encrypt API -> KMS (check IAM permissions), perform encryption -> send encrypted -> Service

- Decrypt API: Service -> call decrypt API -> KMS (check IAM permissions), perform decryption -> send decrypted -> Service

### Envelope Encryption ( > 4KB)

- KMS Encryption API call has a limit 4KB
- The main API that will help us is the GenerateDataKey API

- For the exam: anything over 4KB of data needs to be encrypted must use the Envelope Encryption == GenerateDataKey API

### GenerateDataKey API

- Call GenerateDataKey API
- KMS check IAM permissions
- KMS generate data key
- Send plaintext data key
- Client side encryption by plaintext data key
- KMS send encrypted data key (for decrypt the key)
- wrap encrypted file and data encrypted key to an envelop

### Decrypt envelop data

- Call Decrypt API (< 4KB data)
- Check IAM permissions
- Decrypt data encrypted key using CMK
- Send plaintext data key to Client
- Client side decryption using DEK

### Encryption SDK

- AWS Encryption SDK implemented Envelop Encryption for us
- Encryption SDK also exiswts as CLI tool we can install
- implementations for Java, Python, C, JavaScript
- Feature - Data Key Caching:
  - re-use data keys instead of creating new ones for each encryption
  - Helps with reducing number of calls to KMS with security trade-off
  - Use LocalCryptoMaterialsCache (max age, max bytes, max number of messages)

### KMS Symmetric - API Summary

- Encrypt: encrypt up to 4 KB of data through KMS
- GenerateDataKey: generates a unique symmetric data key (DEK)
  - returns a plaintext copy of the data key
  - AND a copy that is encrypted under the CMK that you specify
- GenerateDataKeyWithoutPlaintext
  - Generate a DEK to use at some point (not immediately)
  - DEK that is encrypted under the CMK that you specify (must use Decrypt later)
- Decrypt: decrypt up to 4 KB of data (including Data Encryption Keys)
- GenerateRandom: Returns a random byte string

## KMS Limits

### KMS Request Quotas

- When exceed a request quota, get ThrottlingException
- To respond, use exponential backoff (backoff and retry)
- For cryptographic operations, they share a quota
- This includes requests made by AWS on behalf (ex: SSE-KMS)
- For GenerateDataKey, consider using DEK caching from the Encryption SDK
- Can request a Request Quotas increase through API or AWS support

## S3 Bucket Key for SSE-KMS encryption

- The new setting to decrease:
  - number of API calls made to KMS from S3 by 99%
  - Costs of overall KMS encryption with Amazon S3 by 99%
- Leverages data keys
  - A "S3 bucket key" is generated
  - That key is used to encrypt KMS objects with new data keys
- less KMS CloudTrail events in CloudTrail

## KMS Key Policies & IAM Principals

- Principal options in IAM Policies
  - AWS Account and Root User

    ```json
    "Principal": { "AWS": "12312312321" }
    "Principal": { "AWS": "arn:aws:iam::123341243124:root" }
    ```

  - IAM Roles

    ```json
    "Principal": { "AWS": "arn:aws:iam::123341243124:role/role-name" }
    ```

  - IAM Role Sessions

    ```json
    "Principal": { "AWS": "arn:aws:sts::12312312321:assumed-role/role-name/role-session-name" }
    "Principal": { "Federated": "cognito-identity.amazonaws.com" }
    "Principal": { "Federated": "arn:aws:iam::123341243124:saml-provider/provider-name" }
    ```

  - IAM Users

    ```json
    "Principal": { "AWS": "arn:aws:iam::123341243124:user/user-name" }
    ```

  - Federated User Sessions

    ```json
    "Principal": { "AWS": "arn:aws:sts::12312312321:federated-user/user-name" }
    ```

  - AWS Services

    ```json
    "Principal": { "Service": [
        "ecs.amazonaws.com",
        "elasticloadbalancing.amazonaws.com"
      ] 
    }
    ```

  - All principals

    ```json
    "Principal": "*"
    "Principal": { "AWS": "*" }
    ```

## CloudHSM Overview

- KMS ⇒ AWS manages the software for encryption
- CloudHSM => AWS provisions encryption hardware
- Dedicated Hardware (HSM = Hardware Security Module)
- Self-managed encryption keys entiredly (not AWS)
- HSM device is tamper-resistant FIPS 140-2 Level 3 compliance
- Supports both symmetric and asymmetric encryption (SSL/TLS keys)
- No free tier available
- must use the CloudHSM Client Software
- Redshift supports CloudHSM for database encryption and key management
- Good option to use with SSE-C encryption

### Cloud HSM - High Availability

- CloudHSM clusters are spread across Multi AZ (HA)
- Great for availability and durability

### Integration with AWS Services

- Through integration with AWS KMS
- Configure KMS Custom Key Store with CloudHSM

| Feature                        | AWS KMS                                                                                   | AWS CloudHSM                                                                 |
|:-------------------------------|:------------------------------------------------------------------------------------------|:-----------------------------------------------------------------------------|
| **Tenancy**                    | Multi-Tenant                                                                              | Single-Tenant                                                                |
| **Standard**                   | FIPS 140-2 Level 3                                                                        | FIPS 140-2 Level 3                                                           |
| **Master Keys**                | • AWS Owned CMK<br/>• AWS Managed CMK<br>• Customer Managed CMK                           | Customer Managed CMK                                                         |
| **Key Types**                  | • Symmetric<br>• Asymmetric<br>• Digital Signing                                          | • Symmetric<br>• Asymmetric<br>• Digital Signing & Hashing                   |
| **Key Accessibility**          | Accessible in multiple AWS regions (can't access keys outside the region it's created in) | • Deployed and managed in a VPC<br>• Can be shared across VPCs (VPC Peering) |
| **Cryptographic Acceleration** | None                                                                                      | • SSL/TLS Acceleration<br>• Oracle TDE Acceleration                          |
| **Access & Authentication**    | AWS IAM                                                                                   | You create users and manage their permissions                                |
| **High Availability**          | AWS Managed Service                                                                       | Add multiple HSMs over different AZs                                         |
| **Audit Capability**           | • Cloud Trail<br/>• CloudWatch                                                            | • Cloud Trail<br/>• CloudWatch<br/> • MFA support                            |
| **Free Tier**                  | Yes                                                                                       | No                                                                           |


## SSM Parameter Store Overview

- secure storage for configuration and secrets
- optional Seamless Encryption using KMS
- Serverless, scalable, durable, easy SDK
- Version tracking of configurations / secrets
- Security through IAM
- Notification with Amazon EventBridge
- Integration with CloudFormation
- SSM Parameter Store Hierarchy (get by path)

### Standard and advanced parameter tiers
- Standard: free, up to 4KB of data, no more than 10,000 parameters per account. Parameter policies are not available.
- Advanced: $0.05 / parameter / month, up to 8KB of data, 10,000 parameters per account. Parameter policies are available.

### Parameter Policies (for advanced parameters)
- Allow assigning a TTL to a parameter to force updating or deleting sensitive data such as passwords.
- Can assign multiple policies at a time.
  - Expiration
  - ExpirationNotification (EventBridge)
  - NoChangeNotification (EventBridge)

## Secrets Manager Overview

- Newer service, meant for storing secrets
- Ability to force rotation of secrets every X days
- Automate generation of secrets on rotation (uses Lambda)
- Integration with Amazon RDS (MySQL, PostgreSQL, Aurora)
- Secrets meant for RDS integration

### Multi-Region Secrets
- Replicate Secrets across multiple regions
- Secrets Manager keeps read replicas in sync with the primary Secret
- Ability to promote a read replica Secret to a standalone Secret
- Use cases: multi-region apps, disaster recovery strategies, multi-region DB...

## SSM Parameter Store vs. Secrets Manager

- Secret Manager ($$$)
  - Automatic rotation of secrets with AWS Lambda
  - Lambda function is provided for RDS, Redshift, DocumentDB
  - KMS encryption is mandatory
  - Can integrate with CloudFormation
- SSM Parameter Store ($)
  - Simple API
  - No secret rotation (can enable rotation using Lambda triggered by EventBridge)
  - KMS encryption is optional
  - Can integrate with CloudFormation
  - Can pull a Secrets Manager secret using the SSM Parameter Store API

### SSM Parameter Store vs. Secrets Manager Rotation

- Both invoke Lambda function and change the password in RDS
- SSM Parameter Store must change value manually in the Parameter Store after Lambda rotation, while Secrets Manager does it automatically

## CloudFormation – Secrets Manager & SSM Integration

- Reference external values stored in System Manager Parameter Store and Secrets Manager within Cloudformation templates
- CloudFormation retrieves the value of the specified reference during create/update/delete operations
- For example, retrieve RDS DB Instance master password from Secrets Manager
- Supports
  - ssm: for plaintext values stored in SSM Parameter Store
  - ssm-secure: for secure strings stored in SSM Parameter Store
  - secretsmanager: for secret values stored in Secrets Manager
  - the syntax is: `{{resolve:service-name:reference-key}}`

## CloudWatch Logs Encryption

- Can encrypt CloudWatch logs with KMS keys
- Encryption is enabled at the log group level, by associating a CMK with a log group, either when create the log group or after it exists
- Cannot associate a CMK with a log group using the CloudWatch console
- Must use the CloudWatch Logs API:
  - associate-kms-key: if the log group already exists
  - create-log-group: if the log group doesn't exist yet

## CodeBuild Security

- to access resources in VPC, make sure to specify a VPC configuration for CodeBuild
- Secrets in CodeBuild:
- Don't store as plaintext in environment variables
- Instead...
  - Environment variables can reference parameter store parameters
  - Environment variables can reference secrets manager secrets

## AWS Nitro Enclaves

- process highly sensitive data in an isolated compute environment
  - Personally Identifiable Information (PII), healthcare, financial,...
- Fully isolated virtual machines, hardened, and highly constrained
  - Not a container, no persistent storage, no interactive access, no external networking
- Helps reduce attack surface for sensitive data processing apps
  - Cryptographic Attestation: only authorized code can be running in Enclave
  - Only Enclaves can access sensitive data (integration with KMS)
- Use cases: securing private keys, processing credit cards, secure multi-party computation...
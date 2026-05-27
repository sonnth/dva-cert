# Cognito: Cognito User Pools, Cognito Identity Pools & Cognito Sync

## Overview

- give users an identity to interact with our webapp
- Cognito User Pools:
  - sign in functionality for app users
  - integrate with API Gateway & ALB
- Cognito Identity Pools (Federated Identity)
  - provide AWS credentials to users so they can access AWS services directly
  - integrate with Cognito User Pools as an identity provider
- Cognito vs IAM: notify keywords: "hundreds of users", "mobile users", "authenticate with SAML"

## Cognito User Pools (CUP)
### User Features
- create a serverless database of user for web-app
- simple login: username/password
- password reset
- email & phone number verification
- multifactor authentication (MFA)
- federated identities: user from Facebook, Google, SAML...
- feature: block users if their credentials are compromised
- login sends back a JSON Web Token (JWT)

### Diagram
Web-app ↔ Cognito User Pools ↔ Database of users or Third-party identity provider (IdP)

### Integrations
- API Gateway
  - Cognito User Pools as an identity provider (authenticate and evaluate users before allowing access to API Gateway)
- Application Load Balancer
  - authenticate users at the ALB level before allowing access to the web-app

### Lambda Triggers
- CUP can invoke the Lambda function synchronously on these triggers:

| User Pool Flow            | Operation                           | Description                                                   |
|:--------------------------|:------------------------------------|:--------------------------------------------------------------|
| **Authentication Events** | Pre Authentication Lambda Trigger   | Custom validation to accept or deny the sign-in request       |
|                           | Post Authentication Lambda Trigger  | Event logging for custom analytics                            |
|                           | Pre Token Generation Lambda Trigger | Augment or suppress token claims                              |
| **Sign-Up**               | Pre Sign-up Lambda Trigger          | Custom validation to accept or deny the sign-up request       |
|                           | Post Confirmation Lambda Trigger    | Custom welcome messages or event logging for custom analytics |
|                           | Migrate User Lambda Trigger         | Migrate a user from an existing user directory to user pools  |
| **Messages**              | Custom Message Lambda Trigger       | Advanced customization and localization of messages           |
| **Token Creation**        | Pre Token Generation Lambda Trigger | Add or remove attributes in Id tokens                         |

### Hosted Authentication UI
- Hosted authentication UI to handle sign-up and sign-in workflows
- Can customize with a custom logo and custom CSS
### Hosted UI Custom Domain
- Must create an ACM certificate in us-east-1
- The custom domain must be defined in the “App Integration” section

### Adaptive Authentication
- Block sign-ins or require MFA if logic appears suspicious
- Cognito examines each sign-in attempt and generates a risk score (low, medium, high) for how likely the sign-in request is to be from a malicious attacker
- Users are prompted for a second MFA only when risk is detected
- Risk score is based on different factors (device, location, IP address)
- Checks for compromised credentials, account takeover protection, and phone and email verification
- Integrates with CloudWatch Logs (sign-in attempts, risk score, failed challenges...)

### Decoding a ID Token JWT
- issues JWT Token (Base64 encoded) with header, payload, signature
- The signature must be verified to ensure JWT can be trusted
- Libraries verify JWT by Cognito User Pools
- Payload contains user information (sub UUID, given_name, email, phone_number, attributes...)

## Application Load Balancer – User Authentication
- ALB can securely authenticate users
  - offload the work of authenticating users to ALB
  - apps can focus on business logic
- Authenticate users through
  - Identity Provider (IdP): OpenID Connect (OIDC) compliant
  - Cognito User Pools
    - social IdPs, such as Amazon, Facebook, Google
    - Corporate identities using SAML, LDAP, or Microsoft AD
- Must use an HTTPS listener to set authenticate-oidc & authenticate-cognito rules
- OnAuthenticatedRequest: authenticate (default), deny, allow
### ALB – Auth through Cognito User Pools
- Create CUP, Client, and Domain
- Make sure ID token is returned
- Add social or Corporate Idp if needed
- URL redirections are necessary
- Allow CUP Domain on IdP app's callback URL

### ALB – Auth through an Identity Provider (IdP), compliant with OIDC
- Configure a Client ID & Client Secret
- Allow redirect from OIDC to ALB DNS name (AWS provided) and CNAME (DNS Alias of app)

## Cognito Identity Pools (Federated Identities)
- Get identities for "users" so they obtain temporary AWS credentials
- Identity pool (identity source) can include
  - Public Providers (Login with Amazon, Facebook, Google...)
  - Users in an Amazon Cognito User Pool
  - OpenID Connect Providers & SAML Providers
  - Developer AUthenticated Identities (custom authentication system)
  - Cognito Identity Pools allo for unauthenticated (guest) access
- Users can then access AWS services directly or through API Gateway 
  - The IAM policies applied to the credentials are defined in Cognito 
  - They can be customized based on the user_id for fine-grained control

### Cognito Identity Pools – IAM Roles
- Default IAM roles for authenticated and guest users
- Define rules to choose the role for each user based on the user’s ID
- You can partition your users’ access using policy variables
- IAM credentials are obtained by Cognito Identity Pools through STS with AssumeRoleForWebIdentity STS API Call
- The roles must have a “trust” policy of Cognito Identity Pools

## User pools vs. Identity pools

- Cognito User Pools (for authentication = identity verification)
  - Database of users for your web and mobile application
  - Allows federating logins through Public Social, OIDC, SAML…
  - Can customize the hosted UI for authentication (including the logo)
  - Has triggers with AWS Lambda during the authentication flow
  - Adapt the sign-in experience to different risk levels (MFA, adaptive authentication, etc…)
- Cognito Identity Pools (for authorization = access control)
  - Get AWS credentials for your users
  - Users can login through Public Social, OIDC, SAML & Cognito User Pools
  - Users can be unauthenticated (guests)
  - Users are mapped to IAM roles & policies, can leverage policy variables
- CUP + CIP = authentication + authorization
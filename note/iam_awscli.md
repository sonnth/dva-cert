# IAM and AWS CLI
## Users, Groups, Policies
Users or Groups can be assigned permissions (IAM policy) JSON document
Inline policy which has a policy that's only attached to a user (not group)
### IAM policy structure: 
- version: policy language version
- id: an identifier for the policy (optional)
- statement: one or more individual statements (required)
### Statement:
- sid: an identifier for the statement (optional)
- effect: Allow, Deny
- Principal: account/user/role to which this policy applied to
- Action: List of actions this policy allows or denies
- Resource: list of resources to which the actions applied to
- Condition: conditions for when this policy is in effect (optional)

Audit permissions of your account using IAM Credentials Report & IAM Access Advisor
Access Advisor (Last Acess): which services were accessed and when
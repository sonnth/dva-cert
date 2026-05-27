# API Gateway

## Overview

- AWS Lambda + API Gateway = Serverless
- API versioning (v1, v2, etc.)
- different environments (dev, staging, prod)
- security (authentication, authorization, CORS)
- create API keys, handle request throttling
- Swagger / OpenAPI import
- Transform and validate requests and responses
- Generate SDKs and API specifications
- Cache API responses

### Integrations High Level
- Lambda function
  - invoke
  - expose REST API 
- HTTP
  - expose HTTP API
  - example: internal HTTP API on premise, Application Load Balancer, etc.
  - why need? add rate limiting, caching, user authentication, API keys, etc.
- AWS Service
  - expose any AWS API
  - example: start an AWS Step Functions workflow, post a mesesage to SQS
  - why need? Add authentication, deploy publicly, rate control, etc.

### Endpoint Types
- Edge-Optimized (default): for global clients
  - requests are routed though CloudFront Edge locations (improves latency)
  - API Gateway lives in only one region
- Regional
  - for clients within a region
  - Could manually combine with CloudFront (more control over caching stategies and distribution)
- Private
  - only accessible from within a VPC using an interface VPC endpoint (ENI)
  - use resource policy to define access

### Security

- User AUthentication through
  - IAM Roles (useful for internal app)
  - Cognito (identity for external users)
  - Custom Auth 

- Custom Domain Name HTTPS security through integration with AWS Certificate Manager (ACM)
  - if using Edge-Optimized endpoint, then the certificate must be in us-east-1
  - if using Regional endpoint, then the certificate must be in the same region as the API Gateway
  - must set up CNAME or A-alias record in Route 53

## API Gateway Stages and Deployments

### Deployment Stages
- Need to create "deployment" to make API Gateway "live" 
- Changes are deployed to "Stages", each has its own configuration parameters
- Stages can be rolled back as a history of deployment is kept

### Stage Variables
- like environment variables for API Gateway
- used in:
  - Lambda function ARN
  - HTTP Endpoint
  - Parameter mapping templates
- Use cases:
  - Configure HTTP endpoints of stages (dev, staging, prod, etc.)
  - Pass configuration parameters to AWS Lambda through mapping templates
- Stage variables are passed to the "context" object in AWS Lambda
- Format: ${stageVariables.variableName}

### Stage Variables & Lambda Aliases
- create a stage variable to indicate the corresponding Lambda alias
- Gateway will invoke the alias instead of the default version

## API Gateway Canary Deployments

- Possibility to enable canary deployments for any stag (usually prod)
- Choose % of traffic the canary channel receives
- Metrics & Logs are separate (for better monitoring)
- Possibility to override stage variables for canary
- this is a blue / green deployment with AWS Lambda & API Gateway

## API Gateway Integration Types & Mappings

### Integration Types
- Mock
  - without sending a request to the backend
- HTTP / AWS (Lambda, AWS service)
  - configure both integration request and response
  - set up data mapping using mapping templates for request and response
- AWS_PROXY (Lambda Proxy Integration)
  - the incoming request from the client is the input to Lambda
  - function responsible for the logic of request / response
  - No mapping template, headers, query string parameters, ... are passed as arguments
- HTTP_PROXY 
  - No mapping template
  - The HTTP request is passed to the backend
  - The HTTP response from the backend is forwarded to the client
  - Possibility to add HTTP headers if need be (ex: API key)

### Mapping Templates (AWS & HTTP integration)
- used to modify request / responses
- Rename / Modify query string parameters, modify body content, add headers
- Use Velocity Template Language (VTL) for loop, if, etc.
- Filter output results (remove unnecessary data)
- Content-Type can be set to application/json or application/xml

## API Gateway OpenAPI

- Import existing OpenAPI 3.0 spec to API Gateway
  - Method
  - Method request
  - Integration request
  - Method response
  - more AWS extensions for API Gateway and set up every single option
- can export the current API as OpenAPI spec
- OpenAPI specs can be written in YAML or JSON
- Using OpenAPI can generate SDK for apps

### Request Validation
- perform basic validation of an API request before proceeding with integration request
- when validation fails, API Gateway immediately fails the request
  - return 400 error response
- reduce unnecessary calls to backend
- checks
  - request parameters in URI, query string, and headers of incoming request are included an non-blank
  - applicable request payload adheres to configured JSON Schema request model of method

## API Gateway Caching

### Caching API responses
- reduce number of calls to backend
- default TTL: 300s (min 0s, max 3600s)
- Caches are defined per stage
- Possible to override cache settings per method
- cache encryption option
- cache capacity between 0.5GB to 237GB
- cache is expensive, makes sense in production, not in dev, test

### Cache Invalidation
- able to flush entire cache immediately
- clients can invalidate cache with header "Cache-Control: max-age=0"(with proper IAM authorization)
- if don't impose an InvalidateCache policy (or choose the Require authorization check box in the console), any client can invalidate the API cache

## API Gateway Usage Plans & API Keys

- in case make an API available as an offering to customers
- Usage Plan:
  - who can access one or more deployed API stages and methods
  - how much and how fast they can access
  - uses API keys to identify API clients and meter access
  - configure throttling limits and quota limits that are enforced on individual client
- API Keys:
  - alphanumeric string values to distribute to clients
  - use with usage plans to control access
  - Quotas limit is the overall number of maximum requests
### Correct Order for API keys
- to configure a usage plan
1. Create API keys, configure methods to require API key, and deploy to stage
2. Generate or import API keys to distribute to app developers(who using API)
3. Create usage plan with desired throttle and quota limits
4. Associate API stages and API keys with usage plan

- Callers of API must supply an assigned API key in the x-api-key header in requests to the API.

## API Gateway Monitoring, Logging, and Tracing

### Logging & Tracing

- CloudWatch Logs
  - contains information about the request / response body
  - enable CloudWatch logging at the Stage level (with Log Level – ERROR, DEBUG, INFO)
  - can override settings on per API basis
- X-Ray
  - enable tracing to get extra information about requests in API Gateway
  - X-Ray API Gateway + AWS Lambda 

### CloudWatch Metrics

- Metrics are by stage, Possibility to enbale detailed metrics
- CacheHitCount & CacheMissCount efficiency of the cache
- Count: the total number API requests in a given period
- IntegrationLatency: the time between when API Gateway relays a request to the backend and when it receives a response from the backend
- Latency: the time between when API Gateway receives a request from a client and when it returns a response to the client. The latency includes the integration latency and other API Gateway overhead.
- 4XXErrorCount (client-side) & 5XXErrorCount (server-side)

### API Gateway Throttling
- account limit
  - API Gateway throttles requests at 10000 rps across all API
  - soft limit that can be increased upon request
- in case of throttling => 429 Too Many Requests (retriable error)
- can set Stage limit & method limit to improve performance
- can define usage plan to throttle per client
- like lambda concurrency, one API  that is overloaded, if not limited, can cause other APIs to be throttled

### Errors
- 4xx means client error
  - 400: bad request
  - 403: access denied, WAF filtered
  - 429: quota exceeded, throttled
- 5xx means server error
  - 502: bad gateway, usually for an incompatible output returned from Lambda proxy integration and occasionally for out-of-order invocations due to heavy load
  - 503: service unavailable exception
  - 504: integration failure, usually for timeout of backend integration

## API Gateway CORS

- CORS must be enabled when receive API calls from another domain
- the OPTIONS pre-flight request must contain the following headers:
  - Access-Control-Allow-Methods
  - Access-Control-Allow-Headers
  - Access-Control-Allow-Origin
- CORS can be enabled through the console

## API Gateway Authentication and Authorization

### IAM Permissions
- create an IAM policy authorization and attach to User / Role
- Authentication = IAM. Authorization = IAM policy
- Good to provide access within AWS (EC2, Lambda, IAM user, etc.)
- Leverage "Sig v4" capability where IAM credential is in headers

### Resource Policies
- similar to Lambda Resource Policy (JSON format to define who and what can access the API)
- Allow for Cross Account Access (combined with IAM security)
- Allow for a specific source IP address
- Allow for VPC endpoints

### Cognito User Pools
- fully manages user lifecycle, token expires automatically
- API Gateway verifies identity automatically from AWS Cognito
- No custom implementation required
- Authentication = Cognito. Authorization = API Gateway Methods

### Lambda Authorizer (formerly Custom Authorizers)
- Token-based authorizer (bearer token). Ex: JWT or OAuth
- A request parameter-based Lambda authorizer (headers, query string, stage var)
- Lambda must return an IAM policy for the user, result policy is cached
- Authentication = External. Authorization = Lambda function

### Summary
- IAM:
  - great for users / roles already within AWS account, resource policy for cross account
  - handle authentication and authorization within AWS
  - leverage Signature Version 4 (Sig v4)
- Custom Authorizer:
  - great for third party tokens,
  - very flexible in terms of what IAM policy can be returned
  - pay per Lambda invocation, result policy is cached
- Cognito:
  - manage user pool (can be backed by Facebook, Google login, etc.)
  - No need to write any custom code
  - must implement authorization in backend

## API Gateway REST API vs HTTP API

- HTTP APIs
  - low-latency, cost-effective AWS Lambda proxy, HTTP proxy APIs, and private integration (no data mapping)
  - support OIDC and OAuth 2.0 authorization, and build-in support for CORS
  - no usage plans and API keys
- REST APIs
  - all features of HTTP APIs except Native OpenID Connet / Oauth 2.0

## API Gateway WebSocket API

### Summary
- two-way interactive communication between client and server
- server can push information to the client
- enables a stateful applications use case
- often used in real-time applications (chat app, collaboration platform, multiplayer games, and financial trading platform)
- it works with AWS services (Lambda, DynamoDB) or HTTP endpoints

### Connect to the API
- WebSocket URL: wss://[some-uniqueid].execute-api.[region].amazonaws.com/[stage-name]
- ConnectionID is re-used
- when Lambda function sendMessage, it calls HTTP POST (IAM Sig v4) 

### Connection URL Operations
- Connection URL: wss://[some-uniqueid].execute-api.[region].amazonaws.com/[stage-name]/@connections/connectionId
- POST: sends a message from the server to the connected WS client
- GET: get the latest connection status of the connected WS client
- DELETE: disconnect the connected client from the WS client

### Routing
- incoming JSON messages are routed to different backend
- if no routes → sent to $default
- must request a route selection expression to select the field on JSON to route from
- sample expression: $request.body.action
- the result is evaluated against the route keys available in the API gateway
- the route key is used to route the message to the correct backend

## API Gateway – Architecture

- create a single interface for all microservices
- use API endpoints with various resources
- apply a simple domain name and SSL certificates
- can apply forwarding and transformation rules at API Gateway level


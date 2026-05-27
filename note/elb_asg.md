# AWS Fundamentals: ELB + ASG
## High Availability and Scalability
- Vertical scalability

increase size of instance (e.g., from t2.micro to t2.large)

common for non distributed systems (database)

sample services: RDS, ElastiCache

- Horizontal Scalability

increasing number of instances/ systems for app

for distributed system

common for webapp, modern app

- High availability

hand in hand with horizontal scaling

means run app/system in at least 2 data centers (AZ)

goal is to survive if a center loss

can be passive (RDS Multi AZ for example)

can be actice (for horizontal scaling)

#### Summary:

- vertival: increase size (scale up/down)

- horizontal: increase number of instances (scale out/in)

- high availability: run same app across multi AZ.

## Elastic Load Balancing (ELB) Overview
Elastic Load Balancer

a managed load balancer

- costs less to setup

- integrated with many AWS services: ec2, ec2 auto scaling groups, ecs, acm, cloudwatch, route 52, waf, aws global accelerator
Types of load balancer on AWS: 4 kinds

- Classic Load Balancer (CLB) 2009

- Application Load Balancer (ALB) 2016

- Network Load Balancer (NLB) 2017

- Gateway Load Balancer (GWLB) 2020

recommend to use newer for more features

can be setup as internal or external

## Application Load Balancer (ALB)
Application Load Balancer

fit for micro services & container-based app

Target Groups

- ec2 instances (can be managed by an Auto Scaling Group)

- ECS tasks

- Lambda Function

- IP Address

ALB can route to multiple target groups, health checks are at target group level

Instance can know client's ip through header: X-Forwarded-For

## Network Load Balancer (NLB)
Network Load Balancer (layer 4)

used for extreme performance, TCP or UDP traffic

- forward TCP & UDP traffic to your instances

- handle millions request per seconds

- ultra-low latency

- NLB has 1 static IP per AZ

NLB - Target Groups

ec2 instances

IP addresses (must be private IPs)

health checks support TCP, HTTP and HTTPS protocols

## Gateway Load Balancer (GWLB)
Gateway Load Balancer

operate at layer 3 (network layer) IP packets

combines:

+ transparent network gateway: single entry/exit for all traffic

+ load balancer: distributes traffic to virtual app

use the GENEVE protocol on port 6081

Target Groups

ec2 instances

IP Addresses (private IPs)

## Elastic Load Balancer - Sticky Sessions
Sticky Sessions

- possible to implement stickinesss (same client -> same instance)

- works for Classic LB, Application LB, Network LB

- the cookie used for stickiness has an expired date

- use case: user doesn't lose session data

- may imbalance to load over EC2 instances

## Elastic Load Balancer - Cross Zone Load Balancing
Cross-Zone Load Balancing
With cross zone load balancing

each LB instance distributes evenly accross all registered in all AZ

Without cross zone load balancing

distributed in the instances of the node of ELB

ALB:

- enabled by default (can be disabled at Target Group level)

- no charges for inter AZ data

NLB & GLB

- disabled by default

- pay charges for inter AZ data if enabled

CLB

- disable by default

- no chareges for inter AZ data if enabled

## Elastic Load Balancer - SSL Certificates
SSL/TLS

- allows traffic between clients and LB encrypted in transit

- SSL: Secure Sockets Layer -> encrypt connections

- TLS: Transport Layer Security -> newer version

- public SSL cert issued by CA (Certificate Authorities)

Load Balancer- SSL cert

- LB uses an X.509 cert (SSL/TLS server certificate)

- manage certs using ACM (AWS Certificate Manager)

Server Name Indication (SNI)

solves the problem of loading multiple SSL certs onto one web server (serve multiple websites)

it's a newer protocol, and requires client to indicate hostname of the target server in the initial SSL handshake

server find correct cert, or return default one

Note:

only works for ALB & NLB (newer generation), CloudFront

not work for CLB

## Elastic Load Balancer - Connection Draining
Connection Draining

- feature naming

+ connection draining - for CLB

+ deregistration delay - for ALB & NLB

- time to comple "in-flight requests" while instance is de-registering or unhealthy

- stops sending new requests to ec2 instance which is de-registering

- between 1 - 3600s (default 300s)

- can be disabled (set to 0)

- set to low value if request are short

## Auto Scaling Groups (ASG) Overview
Auto Scaling Group

load on websites and app can change

goals of ASG:

- scale out (add instances)

- scale in (remove instances)

- ensure minimum and maximum number of instances running

- auto register new instances to a LB

- re-create instance in case a previous one is terminated (ex: unhealthy)

ASG are free (only pay for underlying instances)

ASG attributes:

- a launch template

+ AMI + Instance Type

+ EC2 User Data

+ EBS Volumes
+ Security Groups
+ SSH Key Pair

+ IAM roles for ec2

+ network + subnets infor

+ LB infor

- Min/ Max/ Initital capacity

- scaling policy

CloudWatch Alarms & Scaling

- possible to scale an ASG based on CloudWatch alarms

- alarm monitor a metric (average cpu, or custom metric)

- metric are computed for overall ASG instances

## Auto Scaling Groups - Scaling Policies
Auto Scaling Group - scaling policies

- dynamic scaling

+ target tracking scaling -> the average ASG CPU stay around 40%

+ simple/step scaling -> when CloudWatch alarm is triggered then add or remove instance

- scheduled scaling

+ anticipate a scaling based on known usage patterns -> increate min to 10 at 5pm on Fridays

- predictive scaling: continuously forecast load and schedule scaling ahead

Metrics to scale on:

- CPUUtilization: average CPI accross instances

- RequestCountPerTarget

- Average Network In/Out

- Any custom metric (CloudWatch)

Scaling cooldowns

- after a scaling activity happens, then colldown period (default 300s)

- during cooldown, ASG not launch or terminate additional instances (to allow metrics to stabilize)

- advice: use a ready-to-use AMI to reduce configuration time in order to serving request fasters and reduce the cooldown period

## 76. Auto Scaling Groups - Instance Refresh
Instance refresh

- goal: update launch template and then re-creating all ec2 instances

- setting minimum healthy percentage

- specify warm-up time (how long until instance ready to use)
# ECS, ECR & Fargate
## Overview
- platform to deploy apps
- apps packaged in containers, run on anyOS
- use cases: microservices, lift-and-shift apps,..
- images stored in docker repositories (Docker Hub, ECR, ECR Public Gallery)
- Amazon Elastic Container Service (Amazon ECS): Amazon's own container platform
- Amazon Elastic Kubernetes Service (Amazon EKS): Amazon's managed Kubernetes (open source)
- AWS Fargate: Amazon's own Serverless container platform, work with ECS, EKS
- Amazon ECR: store container images
## ECS
### Introduction

#### EC2 launch type
- launch Docker containers on AWS = launch ECS Tasks on ECS clusters
- EC2 launch type: must provision & maintain the infrastructure(EC2 instances)
- Each EC2 instance must run ECS Agent to register ECS Cluster, AWS takes care of start/stop containers

#### Fargate Launch Type
- launch Docker containers on AWS
- Do not provision the infrastructure (no EC2 instances to manage)
- It's all Serverless
- Just create task definitions
- AWS just runs ECS Tasks based on CPU/RAM needed
- To scale, just increase number of tasks

#### IAM Roles for ECS
- EC2 Instance Profile (EC2 launch type)
	- Used by ECS agent
	- Makes API calls to ECS service
	- Send container logs to CloudWatch Logs
	- Pull Docker image from ECR
	- Reference sensitive data in Secrets Manager or SSM Parameter Store
- ECS Task Role:
	- Allows each task to have a specific role
	- Use different roles for different ECS Services
	- Task Role is Defined in the task definition

#### Load Balancer Integrations
- Application Load Balancer supported and works for most use cases
- Network Load Balancer recommended only for high throughput/high performance use cases, or to pair it with AWS Private Link
- Classic Load Balancer supported but not recommended

#### Data Volumes (EFS)
- Mount EFS file systems onto ECS tasks
- Works for both EC2 and Fargate launch types
- Tasks running in any AZ will share the same data in the EFS file system
- Fargate + EFS = Serverless
- Use cases: persistent multi-AZ shared storage for containers
Note: S3 cannot be mounted as a file system

### Auto Scaling
- Auto increase/decrease the desired number of ECS tasks
- Amazon ECS Auto Scaling uses AWS Application Auto Scaling
	- CPU
	- Memory
	- Request count per target
- Target Tracking - based on target value for specific CloudWatch metric
- Step Scaling - based on a specified CloudWatch Alarm
- Scheduled Scaling - based on a specified date/time
- ECS Service Auto Scaling (task level) # EC2 Auto Scaling (EC2 instance level)


- Accomodate ECS Service Scaling by adding underlying EC2 Instances
- Auto Scaling Group Scaling
	- Scale ASG based on CPU
	- Add Ec2 instances over time
- ECS Cluster Capacity Provider
	- Used to automatically provision and scale the infra for ECS Tasks
	- Capacity Provider paired with an ASG
	- Add EC2 Instances when missing capacity (CPU, RAM,..)

### Rolling updates
- control how many tasks can be started and stopped, and which in over
- Sample ECS rolling update 
	- Min 50%, Max 100%. Starting: 4 tasks v1 -> 2 tasks v1, 2 tasks v2 -> 4 tasks v2
	- Min 100%, Max 150%. Starting: 4 tasks v1 -> 4 tasks v1, 2 tasks v2 -> 2 tasks v1, 2 tasks v2 -> 2 tasks v1, 4 tasks v2 -> 4 tasks v2 
	
### Solutions Architectures
#### ECS tasks invoked by Event Bridge
- Client upload object to S3 bucket
- S3 Bucket send event to EventBridge(Already set rule to run ECS Task)
- ECS Task get object from S3 Bucket, process and save result to Amazon DynamoDB
#### ECS tasks invoked by Event Bridge Schedule
- EventBridge create a schedule
- Run ECS Task to process batch,.. by schedule
#### ECS - SQS Queue Example
- ECS tasks poll for messages from SQS Queue
- Process message
#### ECS - Intercept Stopped Tasks using EventBridge
- Intercept events from within ECS Cluster
- Tasks exiting or starting in Cluster
- Send event to EventBridge
- Alert SNS
- Send email to Administrator

### Task definitions
- The metadata in JSON form to tell ECS how to run a Docker container
- upto 10 containers in a Task Definition
- It contains:
	- Image name
	- Port Binding
	- Memory and CPU required
	- Environment variables
	- Network information
	- IAM Role
	- Logging configuration (ex CloudWatch)
#### Load Balancing (EC2 launch type)
- Dynamic Host Port Mapping if define only the container port in the task definition
- The ALB finds the right port on EC2
- Must allow on the EC2 instance's Security Group any port from the ALB's Security Group
#### Load Balancing (Fargate launch type)
- Each task has a unique private IP
- Only define the container port(host port is not applicable)
- Example:
	- ENS ENI Security Group: allow port 80 from the ALB
	- ALB Security Group: allow port 8/443 from web
#### ECS One IAM Role per Task Definition
- ECS Task A Role -> Service A -> S3
- ECS Task B Role -> Service B -> DynamoDB
#### Env
- Hardcoded
- SSM Parameter Store(API keys, share configs,..)
- Secrets Manager(DB pass,..)
- Env file (bulk) - S3
#### Data Volumes (Bind Mounts)
- share data between containers have the same task
- works for both EC2 and Fargate tasks
- EC2 Tasks - using EC2 instance storage(data tied to EC2 instance)
- Fargate Tasks - using ephemeral storage(data tied to containers)
- Use cases:
	- Share ephemerral data between containers
	- "Sidecar" container pattern, send metrics/logs to other destinations

### Task Placements
- ECS must determine where to place container, with the constraints of CPU, Mem, port
- Similarly, when scales in, ECS needs to determine which task to terminate
- Define task placement strategy and task placement constraints
- Note: this is only for ENS with EC2
#### Task placement process
- Identify instances satisfy the CPU, mem, port requirements in the task definition
- Identify instances satisfy the task placement constraints
- Identify instances satisfy the task placement strategies
- Select instances for task placement
#### Task placement strategies
- Binpack
	- based on the least available amount of CPU, mem
	- minimizes number of instances in use(cost savings)
- Random
	- Place task randomly
- Spread
	- Place task evenly based on the specified value
	- Example: instanceId, attribute:ecs.availability-zone
Can mix task placement strategies
#### Task Placement Constraints
- distinctInstance: place each task on different container instance
- memberOf: places task on instances that satisfy an expression
	- uses the Cluster Query Language (advanced)
	
## ECR
- Elastic Container Registry, store and manage Docker images on AWS\
- Private and Public repository(Amazon ECR Public Gallery)
- Fully integrated with ECS, backed by Amazon
- Access is controlled through IAM(permission errors => policy)
- Supports image vulnerability scanning, versioning, image tags, image lifecycle,...

## AWS Copilot
- CLI tool (not a service) to build, release, and operate produciton-ready containerized apps
- Run apps on AppRunner, ECS, and Fargate
- Help focus on building apps rather than setup infra
- Provisions all required infra for containerized apps (ECS, VPC, ELB, ECR,..)
- Automated deployments with one command using CodePipeline
- Deploy to multiple env
- Troubleshooting, logs, health status,..

## EKS
### Overview
- Amazon Kubernetes Service, it's a way to launch managed Kubernetes clusters on AWS
- open-source system for auto deployment, scaling, management containerized ap
- It similar goal to ECS but different API
- EKS support EC2 if want to deploy worker nodes or Fargate to deploy serverless containers
- Use case: The company already using Kubernetes on-premises or in another cloud, and wants to migrate to AWS using Kubernetes
- Kubernetes is cloud-agnostic(can be used in any cloud - Azure, GCP...)
### Node Types
- Managed Node Groups
	- Creates and manages Nodes (EC2 instances)
	- Nodes are part of an ASG managed by EKS
	- Supports On-Demand or Spot Instances
- Self-Managed Nodes
	- Creates by you and registered to the EKS cluster and managed by an ASG
	- Use prebuilt AMI - Amazon EKS Optimized AMI
	- Supports On-Demand or Spot Instances
- AWS Fargate
	- No maintenance required, no nodes managed
### Data Volumes
- Need to specify StorageClass manifest on EKS cluster
- Leverages a Container Storage Interface (CSI) compliant driver
- Support for: EBS, EFS(works with Fargate), FSx for Lustre, FSx for NetwApp ONTAP

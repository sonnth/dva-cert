# EC2 fundamentals
## AWS budget setup
Activate billing for iam user: Account > IAM user and role access to Billing information
Setup budget and alarm: Billing and Cost Management > Budgets > Create a budget > Zero spend budget
Notified when:

- actual spend reaches 85%
- actual spend reaches 100%
- forecasted spend is expected to reach 100%

## EC2 basics
EC2 (elastic compute cloud) Infrastructure as a Service

- renting virtual machines (ec2 instance)
- storing data on virtual drives (ebs)
- distributing load across machines (elb)
- scaling services using an auto-scaling group (asg)

### configuration

- os (linux, windows, macos)
- cpu
- ram
- storage: network-attached (ebs & efs), hardware (ec2 instance store)
- netword card
- firewall rules: security group
- bootstrap script: configure at first launch


script is only run at the instance first start and runs with root user

some usecase for bootstrap script:

- install update
- install software
- download common files from internet

## EC2 Instance Types Basics
aws naming convention for instance type:

sample: m5.2xlarge

- m: instance class

- 5: generation (AWS improve hardware over time)

- 2xlarge: size within instance class (the more the size, the more the memory CPU)


General purpose:

- balance: compute, memory, networking

- for: web servers or code repositories

Compute optimized

- compute-intensive task high performance processors

- for: batch processing, media transcoding, high performance web servers, computing, scientific modeling & machine learning, gaming servers

- C name (C5, C6, ... )

Memory optimized

- process large data sets in memory

- for relational/non-relational databases, distributed web scale cache stores, im-memory databases optimized for BI (business intelligence), real-time processing of big unstructured data

- R series (R5, R4,...)

Storage optimized

- sequential read and write, accessing a lot of data sets on local storage

- for OLTP systems (online transaction processing), relational & nosql databases, cache for in-mem databases (redis), data warehousing apps, distributed file system.

- start with I, D, H1


## Security Groups & Classic Ports Overview
Security Groups

- control traffic is allowed into instance

- contain allow rules (what is allowed to go in and to go out)

- reference by IP or other security group

- act as firewall on ec2 instance

- regulate: ports, IP ranges, inbound network, outbound network

- one SG can attach to many ec2 instances and and vice versa.

- if switch other region -> create new SG/VPC

- it live outside ec2 instance

- if app is not accessible(timeout) -> SG issue

- if app gives connection refused -> app error or not launched

- default status: all inbound traffic is blocked and all outbound traffic is authorised

## EC2 Instance Purchasing Options

ec2 instances purchasing options:

- on-demand: short workload, predictable pricing, pay by second

- reserved (1&3 years): 

+ reserved instances: long workloads

+ convertible reserved instances: long workloads with flexible instances

- savings plan (1&3 years): commitment to an amount of usage

- spot instances: short workload, cheap, can lose instanes (less reliable)

- dedicated hosts: book an entire physical server, control instance placement

- dedicated instances: no other customers will share your hardware

- capacity reservations: reserve capacity in a specific AZ for any duration

### On demand:

- pay for what use

- highest cost, no upfront payment

- no long-term commitment

recommended for short-term and un-interrupted workloads

### Reserved instances:

- up to 72% discount compared to on-demand

- reserve specific attributes (instance type, region, tenancy, OS)

- period: 1y (+discount) or 3y(+++discount)

- payment options: no upfront (+), partial upfront(++), all upfront(+++)

- reserved instance's scope: regional or Zonal (capacity in AZ)

- recommend for steady-state (database,..)

- can buy and sell in the reserved instance marketplace

### Convertible reserved instances:

- can chagne ec2 instance type, instance family, OS, scope and tenancy

- up to 66% discount

### Savings plan:

- discount based on long-term usage (to 72% - same as Reserved instances)

- commit to certain type of usage (10$/house for 1 or 3 years)

- usage beyond is billed at on-demand price

- locked to specific family & region (e.g.. M5 in us-east-1)

- flexible across:

+ size (e.g., m5.xlarge, m5.2xlarge)

+ os (e.g., linux, windows)

+ tenancy (host, dedicated, default)

### Spot instances

- discount up to 90% compared to on-demand

- lose instance at any point of time if max price is less than current spot price

- most cost-efficient

- useful for resilient to failure workloads (batch jobs, data analysis, image processing, distributed , flexible start and end time)

- not suitable for critical jobs or databases

### Dedicated hosts:

- physical server

- allows to use existing server-bound software licenses (per-socket, per-core, pe-VM software licenses)

- purchasing options:

+ on-demand: pay per second for active Dedicated Host

+ reserved 1 or 3 y (no upfront, partial upfront all upfront)

- the most expensive option

### Dedicated instances:

- instances run on hardware that's dedicated to you

- may share hardware with other instances in same account

- no control over instance placement

### Capacity reservations:

- reserve on-demand capacity in specfic AZ for any duration

- always have access ec2 capacity when needed

- no time commitment, no billing discounts

- combine with Regional Reserved and Savings plans to benefit from billing discounts

- charged at on-demand rate whether run instances or not

- for short-term, uninterrupted workloads in specific AZ
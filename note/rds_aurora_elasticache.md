# RDS, Aurora, ElastiCache

	
## Amazon RDS Overview
RDS overview

RDS vs  DB on EC2

RDS managed service:

- automated provisioning, OS patching

- backup, restore continuous (point in time restore)

- monitoring dashboards

- read replicas for performance

- multi AZ setup for Disaster Recovery

- Maintenance windows for upgrade

- Scaling capability (vertical and horizontal)

- Storage backend by EBS

can't ssh into instances

Storage Auto Scaling

- increase storage dynamically

- have to set Maximum Storage Threshold (limit for storage)

- automatically modify if:

+ free less than 10% of allocated

+ low-storage lasts at least 5 minutes

+ 6 hours have passed since last modification

- useful for unpredictable workloads

- support all RDS engines

## RDS Read Replicas vs Multi AZ
RED Read Replicas

up to 15 replicas within, cross AZ, Region

use cases: run other workload without affect to exists

network cost when different region

for disaster recovery -> standby instancse (no read-write). Note that read replicas be setup as multi AZ

from single to multi AZ -> create snapshot -> restore in new AZ -> synchronization is established

## Amazon Aurora
Amazon Aurora

from AWS (not open sourced)

support both Postgres and MySQL

AWS cloud optimized, 5x performance than MySQL on RDS, 3x than Postgres on RDS

auto grows storage 10GB, up to 256TB

have up to 15 replicas, the replication faster than MySQL (sub 10ms replica lag)

HA native

cost more than RDS (20% more), but efficent

Aurora HA and Read scaling

6 copies of data across 3AZ

one aurora instance takes writes (master)

auto failover for master less than 30 seconds

support cross region replication

Aurora DB Cluster

writer endpoint -> master node

reader endpoint  -> load balacing -> auto scaling reader node

## RDS & Aurora Security
RDS & Aurora Security

Encryption:

- Master & replicas using AWS KMS

- if master not encrypted -> read replicas can not be encrypted

- to encrypt an un-encrypted database -> snapshot & restore as encrypted

IAM authentication instead of username/password

Security group to controll network access

No SSH available except on RDS custom

Audit logs can be enabled and sent to CloudWatch

## RDS Proxy
RDS proxy:

- pool and share connection (useful for lambda function)

- reduce failover time

- enforce IAM Authentication for DB

- never publicly accessible (must accessed from VPC)

## ElastiCache Overview
ElastiCache

RDS -> relational databases

ElastiCache -> Redis or Memcached

in-memory databases with high performance, low latency

reduce load off of databases for read intensive workloads

make app stateless

AWS take care OS maintenance/patching, optimizations, setup, configuration, monitoring, failure recovery and backups

ElastiCache solution architecture

DB cache

- app queries elasticache, if not avaialble -> get from RDS and store elasticache

- helps relieve load in RDS

- cache must have an invalidation strategy to make sure only the most current data is used in there

User session store

write session data into ElastiCache -> any instances can retrieves data -> application is stateless

ElastiCache - Redis vs Memcached

Redis

- multi AZ with auto-failover

- read replicas to scale reads and have high availability

- Data durability using AOF persistence

- Backup & Restore

- Supports Sets and Sorted Sets

Memcached

- multi-node for partitioning (sharding)

- no high availability (replication)

- non persistent

- Backup & Restrore (serverless)

- multi-threaded architecture

## ElastiCache Strategies
ElastiCache Strategies

Considering

- is it safe to cache (may be out of date, eventually consistent)

-is it effective

+ pattern: data changing slowly, few keys are frequently needed

+ anti-pattern: data changing rapidly, all large key space frequently needed

- is data structured well for caching (key value, aggregations results)

### Lazy loading/ cache-aside/lazy population strategy

pros:

- only request data is cached

- node faillures are not fatal

cons:

- delay (3 round trips) app -> cache -> db -> cache -> app

### Write through - add or update cache when database is updated

pros:

- cache's data never stale, reads are quick

- write penalty vs read penalty (each write requires 2 calls)

cons:

- missing data until it is added/updated in DB

- cache churn - lot of data will never be  read

### Cache evictions and TTL

occur in three ways:

- delete item in the cache

- memory is full and it's not recently used (LRU)

- set time-to-live (TTL)

TTL helpfult for  any kind of data: leaderboards, comments, activity streams

TTL range from few seconds to hours or days

If too many evictions happen due to memory -> should scale up or out

## Amazon MemoryDB for Redis - Overview
Amazon MemoryDB for Redis

- redis-compatible, durable, in-mem database

- ultra-fast performance with over 160m requests/second

- multi-AZ transactional log

- scale seamlessly from 10s GBs to 100s TBs of storage

- use cases: web, mobile apps, online gaming, media streaming,..
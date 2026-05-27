# EC2 instance storage
## EBS Overview
EBS Volume:

elastic block store

- network drive(might be a bit of latency), attach to instances while running

- allow instances persist data, event after termination

- mount to one instance at a time

- bound to specific AZ (EBS Volume in us-east-1a can't be attached to us-east-1b)
same way as ec2 instances, both are bound to an AZ

possible to create EBS volumes and leave them unattached

EBS behaviour when ec2 terminates

Default

- root EBS volume is delted

- any other attached EBS volume is not deleted

This can be controlled by aws cli

## EBS Snapshots
EBS Snapshot

- make a backup at a point in time

- not necessary to detach volume to do snapshot, but recommend

- can copy snapshots across AZ or Region

EBS Snapshots features:

- archive (75% cheaper, takes 24-72h for restoring)

- recycle bin (setup rule to retain deleted, specify retention 1d-1y)

- fast snapshot restore (FSR): force full initalization of snapshot to have no latency

## AMI Overview
AMI stands for Amazon Machine Image

- customization of an EC2 instance

- build for specific region (can be copied accross regions)

- launch from:

+ public: aws provided

+ your own: make and maintain yourself

+ aws marketplace AMI: potentially sells

## EC2 Instance Store
EC2 instance store:

high-performance hardware disk with:

- better I/O performance

- lose storage if stopped

- risk of data loss if hardware fails

- backups, replication are your responsibility

## EBS Volume Types
EBS volume types:

- gp2/gp3 (ssd) general purpose, balance price and performance

- io1/io2 Block Express (ssd) highest-performance

- st1(hdd) low cost, for frequently accessed

- sc1 lowest cost, for less frequently

only gp2/gp3 nad io1/io2 used as boot volumes

## EBS Multi-Attach
EBS multi-attach

for io1/io2 family in the same AZ

up to 16 ec2 instances at a time

use case: achieve higher app availability in clustered, app must manage concurrent write

must use a file system that's cluster-aware (not XFS, EXT4,..)

## Amazon EFS
EFS - elastic file system

- managed NFS (network file system) that can be mounted on many EC2

- works with ec2 instances in multi-AZ

-highly available, scalable, expensive (3x than gp2), pay per use

- use cases: content management, web serving, data sharing, wordpress

- uses NFSv4.1 protocol

- uses security group to controll access to EFS

- compatible with Linux based AMI

- encryption ata rest using KMS

- file system scales auto, no capacity planning, pay-per-use

Performance & Storage classes

- EFS scale

+ 1000s of concurrent NFS clients, 10GB+ /s throughput

+ grow to Petabyte-scale network file system, auto

- Performance mode (set when create)

+ General purpose (default)

+ Max I/O

- Thoughput Mode

+ Bursting

+ Provisioned

+ Elastic

- Storage tiers (lifecycle management feature - move file after N days)

+ Standard

+ Infrequent access (EFS-IA)

+ Archive

+ implement lifecycle policies to move files between storage tiers

- Avaiability and durability

+ Standard: multi-az, for prod

# VPC fundamentals
	
## VPC

Things should know:

- VPC, Subnets, Internet Gateways & NAT Gateways

- Security Groups, Network ACL (NACL), VPC Flow Logs

- VPC Peering, VPC Endpoints

- Site to Site VPN & Direct Connect

## VPC, Subnets, IGW and NAT
Internet Gateway & NAT Gateways

- Internet Gateway helps VPC instances connect with the internet

- Public subnets have a route to the internet gateway

- NAT Gateways (AWS-managed) & NET Instances (self-managed) allow instances in Private Subnets to access the internet while remaining private

## NACL, SG, VPC Flow Logs
Network ACL & Security Groups

NACL (Network ACL)

- A firewall controls traffic from and to subnet

- Can have ALLOW and DENY rules

- Attached at Subnet level

- Rules only include IP addresses

Security Groups

- A firewall controls traffic fto and from an ENI/EC2

- only ALLOW rules

- Rules include IP addresses and other security groups

## NACL, SG, VPC Flow Logs
VPC flow logs

- capture information about IP traffic going to interface

+VPC flow logs

+Subnet flow logs

+Elastic Network Interface flow logs

- Helps to monitor & troubeshoot connectivity issues:

+ Subnets to internet

+ Subnets to subnets

+ Internet to Subnets

- Captures network information from AWS managed (ELB, ElastiCache, RDS, Aurora, ...)

- VPC flow logs data can go to S3, CloudWatch, Kinesis Data Firehose

## VPC Peering, Endpoints, VPN, DX
VPC peering

- Connect 2 VPC, privately using AWS's network

- Make them behave as if they were in the same network

- Must not have overlapping CIDR (IP address range)

- VPC Peering connection is not transitive (must be established for each VPC that need to communicate with one another) A <=> B, A <=> C, must be establish B <=> C so B cannot connect to C

## VPC Peering, Endpoints, VPN, DX
VPC endpoints
Connect AWS service using private network
=> security, low latency

Site to site VPN
- connect on-premises VPN to AWS
- connect go over the internet, auto encrypted
Direct connect (DX)
- physical connection
- fast, private, secure. Go over private network
- take a month to establish
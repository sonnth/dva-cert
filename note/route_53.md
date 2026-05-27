# Route 53
	
## What is a DNS?
### DNS

Domain name system -> Translates hostnames into ip addresses

www.google.com => 172.217.18.36

DNS uses hierarchical naming structure

### DNS terminologies:

Domain Registrar: Amazon Route 53, GoDaddy,..

DNS record: A, AAAA. CNAME, NS, ...

Zone file: contains DNS records

Name Server: resolves DNS queries (Authoritative or Non-Authoritative)

Top Level Domain (TLD): .com, .us, .in, .gov, .org,. ..

Second Level Domain (SLD): amazon.com, google.com,...

http://api.www.example.com. -> URL

the last . -> Root

.com. -> TLD

.example.com. -> SLD

.www.example.com. -> Sub Domain

api.www.example.com. -> FQDB (full qualified Domain Name)

http -> protocol

### How DNS works

Local DNS Server look up local DNS resolver to see if it already knows the IP

by recursive asking Root DNS Server, TLD DNS Server, SLD DNS Server

after find the corresponding IP address -> cache to local DNS resolver



## Amazon Route 53

- A high available, scalable, fully managed and Authoritative DNS

- also domain registar

- ability to check health of resources

- only AWS service which provides 100% availability SLA

- 53 is a reference to the traditional DNS port

### Route 53 - records

- route traffic for a domain

- each record contains:

+ domain/subdomain name - example.com

+ record type - A or AAAA

+ value - 12.34.56.78

+ routing policy - how Route 53 responds to queries

+ TTL - amount of time the record  cached at DNS Resolvers

- Route 53 supports the following DNS record types:

+ (mustknow) A/ AAAA/ CNAME/ NS

+ (advanced) CAA/ DS/ MX/ NAPTR/ PTR/ SOA/ TXT/ SPF/ SRV

### Route 53 - record types

A - maps a hostname to IPv4

AAAA - map a hostname to IPv6

CNAME - maps a hostname to another hostname

+ target hostname must have an A or AAAA record

+ can't create CNAME for top node of DNS namespace (Zone Apex)

+ Example: can't create for example.com, but can create for www.example.com

- NS - Name Servers for the Hosted Zone

### Route 53 - hosted zones

a container for records -> define how to route traffic to domain and its subdomains

- Public hosted zones: specify how to route traffic on the Internet (public domain names)

- Private hosted zones: specify how to route traffic within one or more VPCs (private domain names)

pay 0.5$ per month per hosted zone

## Route 53 - TTL
Route 53 - Records TTL (time to live)

- High TTL - e.g., 24hr

+ less traffic on Route 53

+ Possibly oudated records

- Low TTL - e.g., 60s

+ more traffic on Route 53 ($$)

+ records are outdated for less time

+ easy to change records

- Except for Alias records. TTL is mandatory for each DNS record

## Route 53 CNAME vs Alias
CNAME vs Alias

- AWS resources (Load Balancer, CloudFront...) expose an AWS hostname

- CNAME (only root domain): points hostname to any other hostname. E.g,. app.domain.com => anything.com

- Alias (for root domain or non-root domain): points a hostname to an AWS Resource (app.domain.com => amazonaws.com),

+ it free of charge.

+ auto recognizes change resource's IP

+ always type A/AAAA for AWS Resource (IPv4/IPv6)

+ can't set the TTL

+ Target: ELB, CloudFront, API Gateway, Elastic Beanstalk, S3 Web, VPC, Global Accelerator, Route 53 Record, can not set ALIAS record for EC2 DNS name

## Routing Policy - Simple
Route 53 - routing policies

- define how route 53 responds to DNS queries

- this routing from DNS perspective, DNS only respond to the DNS queries 

- Routing policies: simple, weighted, failover, latency based, geolocation, multi-value answer, geoproximity

### Routing Policies - Simple

- to a single resource

- can specify multiple values in the same record

- if multiple values -> random one is chosen by the client

- when Alias enabled, specify only one AWS resource

- can't be associated with Health Checks

### Routing Policies - Weighted

- control the % of the requests that go to each specific resource

- assign each record a relative weight

+ traffic (%) = weight for a specific record / sum of all

+ weights don't need to sum up to 100

- DNS records must have the same name as type

- can be associated with Health Checks

- Use cases: load balancing between regions, testing new app versions

- assign a weight of 0 to a record to stop sending traffic to a resource

- if all weight = 0, then all will be returned equally

### Routing Policies - Latency-based

- redirect to the resource that has the least latency close to us

- helpful when latency for users is a priority

- Latency is based on traffic between users and AWS Regions

- Can be associated with Health Checks (has a failover capability)

## Route 53 - Health Checks

- HTTP Health Checks are only for public resource

- Health Chek => Automated DNS Failover

+ monitor an endpoint (app, server, AWS resource)

+ monitor other health check

+ monitor CloudWatch Alarms (throttles of DynamoDB, alarms on RDS, custom metrics,..)

- Health Checks are integrated with CW metrics

### Monitor an endpoint

- about 15 global health checkers

+healthy/unhealthy threshold - 3 (default)

+ interval - 30 sec (can set to 10 sec - higher cost)

+ if > 18% healthy then healthy, otherwise unhealthy

+ ability to choose which locations route 53 to use

- pass only when endpoint with 2xx and 3xx status codes

- pass/fail based on the text in the first 5120 bytes of the response]

- configure router/firewall to allow incoming requests from Route 53 Health Checkers

### Route 53 - Calculated Health Checks

- combine result of multiple to single health check

- can use OR, AND, NOT

- up to 256 child health checks

- specify number of health checks need to pass

- Usage: perform maintenance without causing all heal checks to fail

### Health Checks - Private Hosted Zones

- Route 53 health checkers are outside the VPC

- can't access private endpoints

- able to create CloudWatch Metric and associate a CloudWatch Alarm, then create a Health Check that check the alarm itself

## Routing Policy
### Routing Policy - Failover
Routing Policy - Failover

- using health check (mandatory), if health then Primary else Secondary(disaster recovery) instance

- only be one primary and one secondary

### Routing Policy - Geolocation
Routing policy - Geolocation

- based on user location

- specify location by Continent, Country or by US State

- should create a "Default" record (case non match location)

- Use cases: web localization, retric content distribution, load balancing,...

- can be associated with Health Checks

### Routing Policy - Geoproximity
Geoproximity Routing Policy

- Route traffic to your resources based on the geographic location of users and resources

- ability to shift more traffic to resources based on the defined bias

+ expand (1 to 99) - more traffic to the resource

+ shrink (-1 to -99) - less traffic to the resource

- Resource can be: AWS(specify region) or Non-AWS (specify Latitude and Longtitude)

- Must use Route 53 traffic flow (advanced) to use this feature

### Routing Policy - Traffic Flow & Geoproximity Hands On
Route 53 - Traffic Flow

- Simplify the process of creating and maintaining records in large and complex configurations

- Visual editor to manage complex routing decision trees

### Routing Policy - IP-based
Routing Policy - IP-based Routing

- based on clients' IP addresses

- provide list of CIDRs for clients and corresponding endpoints/locations

- use cases: optimize performance, reduce network costs

- Example: route end users from a particular ISP to a specific endpoint

### Routing Policy - Multi Value
Routing policies - multi-value

- use when routing traffic to multiple resources

- Route 53 return multiple values/resources

- can be associated with health checks (return only values for healthy resources)

- up to 8 records are returned

- Multi-Value is not a substitute for having an ELB
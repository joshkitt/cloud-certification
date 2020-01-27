# AWS Certified Solutions Architect Associate

## Index

- [Compute](#compute)
    - [EC2](#ec2)   
    - [ELB](#elb)
    - [Auto Scaling Group](#auto-scaling-group)
    - [ECR](#ecr)
    - [ECS](#ecs)
    - [Fargate](#fargate)
    - [EKS](#eks)
    - [EBS](#ebs)
    - [Lambda](#lambda)
    - [Lambda@Edge](#lambdaedge)
    - [AWS Step Functions](#aws-step-functions)
    - [Elastic Beanstalk](#elastic-beanstalk)
    - [EBS](#ec2-and-ebs)
- [Storage](#storage)
    - [S3](#s3)
    - [EFS](#efs)
    - [Storage Gateway](#storage-gateway)
- [Database](#database)
    - [RDS](##rds)
    - [Amazon Aurora](#amazon-aurora)
    - [DynamoDB](#dynamodb)
    - [ElastiCache](##elasticache)
    - [Neptune](#neptune)
    - [Amazon Redshift](##redshift)
- [Application Integration](#application-integration)
    - [SNS](#sns)
    - [SQS](#sqs)
    - [SWF](#swf)
- [Networking and Content Delivery](#networking-and-content-delivery)
    - [VPC](#vpc)
    - [CloudFront](#cloudfront)
    - [Route 53](#route53)
    - [Amazon API Gateway](#api-gateway)
    - [Direct Connect](#direct-connect)
    - [Bastian Hosts](#bastion-hosts)
- [Analytics](#analytics)
    - [Athena](#athena)
    - [EMR](#emr)
    - [Kinesis](#kinesis)
    - [Glue](#glue)
- [Management and Governance](#management-and-governance)
    - [CloudWatch](#cloudwatch)
    - [CloudFormation](#cloudformation)
    - [CloudTrail](#cloudtrail)
    - [OpsWorks](#opsworks)
- [Security, Identity, & Compliance](#security-identity--compliance)
    - [IAM](#iam)
    - [Cognito](#cognito)
    - [Secrets Manager](#secrets-manager)
    - [Certificate Manager](#certificate-manager)
    - [Directory Service]
    - [KMS](#kms)
    - [CloudHSM](#cloudhsm)
    - [WAF & Shield](#waf-and-shield)
- [Migration and Transfer](#migration-and-transfer)
    - [Snowball](#snowball)
- [Other](#other)
    - [Shared Responsibility Model](#shared-responsibility-model)
    - [Well-Architected Framework](#well-architected-framework)
    - [Support](#support)
    - [Parameter Store]

# Compute
### EC2
https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/concepts.html

Purchase Options
- On-demand (default) - pay by the second
- Reserved - purchase at discount for 1-3 years
    - Use if you always need a server
- Spot - purchase unused capacity at discount
    - Batch processing, process can start and stop without affecting a job

User Data - you can store information for an instance to run on startup

Placement Groups
- Cluster - Can put EC2 instances close to each other to minimize latency
    - great network speed, increased risk of failure
    - can cover multiple Azs and different hardware - want to maximize resiliency
- __Spread can only have 7 running instances per AZ__
- Partitioned - cluster EC2 instances, but spread out the clusters, up to 7 partitions per AZ

Hypervisor: Nitro and XEN

Store data in 2 ways
- Instance store - ephemeral, not persistent, lose data if stopped, but not if restarted
- Elastic Block Store - persistent

Secured by a VPC security group and a keypair for logging into the server

Scaling
- Vertical scaling - scale up instance type with additional resources (downtime)
- Horizontal scaling - add more instances to handle more demand

AMI - Amazon Machine Image (eg. Amazon Linux 2)
- Pre-built from Amazon
- Marketplace for AMIs from other people
- Custom built AMI
    - Can pre-install packages
        - Faster boot time
        - Don't need to use User Data to run things
- You can't copy an AMI with an associated billingProduct code that was shared with you from another account. Instead, to copy, launch an EC2 instance and create AMI from instance
- Defines processor, memory, and storage type
- Cannot be changed without downtime
- Instance Types
    - T - General purpose, burstable
    - M - General purpose, medium duty 
    - C - Compute
    - R - RAM - Memory optimized
    - I - Storage optimized
    - P - Accelerated Computing
    - G - GPU optimized

### ELB
Elastic Load Balancer
- Distributes traffic across multiple targets
- Route users to different locations
- Integrates with EC2, ECS, and Lambda
- Supports one or more Azs in a region
- Any load balancer has a static hostname, do not resolve underlying IP
- Put in front of an auto-scaling group
- Like an F5
- Intelligent routing
- SSL termination
- __Access Logs can be enabled to capture detailed inforamtion about requests (disabled by default)__
- HTTPS listener
    - __clients can use Server Name Indication (SNI) to specify the hostname they reach__
- Types
    - ALB - Application Load Balancer
        - HTTP Layer 7
        - Called by DNS name
        - Hostname, path, redirects, dynamic host port mapping for ECS
        - Good fit for Docker 
        - Performs health checks to route to healthy nodes
        - Stickiness cookie to send traffic back to same server for HTTP (1 sec. to 7 days)
            - Can cause imbalance across servers
        - __Passes along client info in X-Forwarded-For, etc. header__
        - Uses "Target group" for routing target
        - __Weighted Target Groups routing__
    - NLB - Network Load Balancer
        - TCP Layer 4
        - Static IP per AZ
        - Public facing - must attach Elastic IP
        - Private facing - gets random private IP
        - Higher performance than ALB
        - Low-latency
        - NLB does not use X-Forwarded-For header, you can see source IP
    - Classic Load Balancer
        - original AWS service
        - For older configurations
        - cheaper
        - not as intelligent
- Security groups
    __Load balancer security group can be allowed in to the EC2 instance security group__
        - Used to provide security by limiting direct EC2 access (must go through ELB first)
-  SSL / TLS Certificates
    - __AWS Certificate Manager - used to manage certificates__
    - Can use many certificates for different domains
    - __Server Name Indication (SNI) - clients can specify the hostname__

Health checks - to monitor healthy nodes before routing
	- Out of service vs In service

### Auto Scaling Group
- Scale in or out based on CloudWatch alarms - built-in metrics and custom metrics
- Free - just pay for resources being launched
- Will terminate unhealthy instances and recreate them
- Termination policy default (scale in)
    1. Will choose the AZ with the most instances first
    2. Will terminate the oldest launch configuration first
    3. __Will terminate the instance that is closest to the next billing hour__
- Scaling cooldown
    - Can change cooldown and threshold settings to optimize the amount of scaling and minimize thrashing
    - If scaling up and down too much, modify the cooldown timer and CloudWatch alarm period that triggers the scale-in
- Launch configuration defines instance template for the group
- Defines min, max, and desired number of instances
- Performs health checks on each instance
- Includes scaling policies that define behavior
- Exists within 1 ore more AZs within a single region
- Application Load Balancer - communicates with Auto-Scaling Group to know to route traffic to healthy nodes

### ECR
https://docs.aws.amazon.com/AmazonECR/latest/userguide/what-is-ecr.html

### ECS
https://docs.aws.amazon.com/AmazonECS/latest/developerguide/Welcome.html

- Containerized apps - Docker - Container orchestration service
- Manages a fleet of Docker containers
- Tasks are how we execute a service

### Fargate
Enables containerized apps without managing servers / Kubernetes

### EKS
- Amazon ECS for Kubernetes 
- Manages k8s cluster

### EBS
Types
- Filesystem for EC2 instances
    - __SSD can be used for bootable volume (HDD CAN NOT)__
    - __SSD is good for small, random I/O__
    - __HDD is good for large, sequential I/O__
- Types 
    - General purpose (SSD) - default, less expensive
        - __Max 16,000 IOPS (3x disk size, up to max)__
    - Provisioned IOPS (SSD) - pay more for higher level of performance - most expensive
        - __IO1 more than 16,000 IOPS (outdated - new max of 32,000)__
        - __Maximum ratio of provisioned IOPS volume size (GB) is 50:1 (ex. 500 IOPS / 10 GB)__
            - 64,000 IOPS on Nitro instances
            - 32,000 IOPS on other instances 
    - Throughput optimized HDD - chatty database use-case, cheaper
    - Cold HDD - cheap
    - Magnetic - cheapest

Snapshots - point in time snapshot, gets stored in S3 behind the scenes (same durability)
- Run snapshots during downtime, slow time
- __Can still use instance while taking a snapshot__  

Encryption

Only attached to one instance at a time - like a USB drive - Not shared across instances
- Locked to AZ (have to back-up/snapshot to copy to different AZ)

HDD is better for sequential, SSD is better for scatter

RAID
- RAID 0 - High performance IOPS, NO fault tolerance
- RAID 1 - Fault tolerance - redundant copies, 2x network
- Other options not recommended
- Configure in OS

Can not attach a drive across Azs
- Must snapshot, create volume from snapshot

Disk IO is high => increase volume size (for gp2)

### Lambda
Lambda
- Serverless
    - Run code without provisioning any infrastructure
- Only charged for usage execution time
- Can configure from 128 MB to 3008 MB (64 MB increments)
- Concurrency limits: 1000
- Scales out automatically
- Timeout up to 5 minutes (outdated: new limit is 15 minutes)
- Integrates with services
- Enables event-driven workflows
- Primary service for serverless architecture
- Triggered from other services too (CloudWatch, SNS, S3, API Gateway)

### Lambda@Edge
- Run globally at the edge
- Can process / modify
    - User request/response
    - Origin request/response
- Use cases
    - Security / privacy
    - Dynamic web app at edge

### AWS Step Functions
- JSON State Machine
- Lambda
- __Serverless (as opposed to SWF)__

### Elastic Beanstalk
- Leverages other AWS services
- Only pay for the other AWS services you use
- Handles provisioning, load balancing, scaling, and monitoring

# Storage

### S3
https://docs.aws.amazon.com/AmazonS3/latest/dev/Welcome.html

| | Availability | Availability SLA | Durability | __Limitation__ | Retrieval |
| --- | --- | --- | --- | --- | --- |
| S3 Standard             | 99.99% | 99.9% | 99.999999999% (11 9’s) |
| S3 Intelligent-Tiering* | __99.9%__  | __99%__  | 99.999999999% (11 9’s) |
| S3 Standard-IA          | __99.9%__  | __99%__  | 99.999999999% (11 9’s) | __30 days__ |
| S3 One Zone-IA†         | __99.5%__  | __99%__  | 99.999999999% (11 9’s) | __30 days__ |
| S3 Glacier              | 99.99% | 99.9% | 99.999999999% (11 9’s) | __Can not specify GLACIER at time of creation__ | Standard: 3 - 5 hours, Expedited: 1 - 5 mins, Bulk: 12 hours |
| S3 Glacier Deep Archive | 99.99% | 99.9% | 99.999999999% (11 9’s) |
- Lifecycle
    - __Objects must be stored at least 30 days before you can transition them to to the IA classes__
- Versioning
   - Set at the bucket level
   - Any file that is not versioned prior to enabling versioning will be version "null"
- Stores data across a minimum of 3 Azs
- Enables url access to files (based on permissions)
- Can provide upload __transfer acceleration__ using AWS Edge Locations
- Configurable rules for data lifecycle
- MFA Delete 
- Encryption
- Use "multi-part upload" for files over 5GB
- Static website hosting
  - Need to enable public read policy on bucket
- CORS - cross origin resource sharing
- Consistency model
    - PUT (new) - read after write
        - New object can be retrieved immediately (unless you already asked for it and it was cached null
    - PUT (existing) and DELETE - eventually consistent
- Cross Region replication
    - Must set up for each region
    - Files are updated in near-real-time

S3 Glacier
- Designed for archiving of data within S3
- Configurable retrieval times
- Different storage classes
    - S3 Glacier - for archival data - minimum of 90 days
        - Pay a retrieval charge
        - Can retrieve in minutes or hours
        - 5x less expensive than S3 standard
    - S3 Glacier deep archive- minimum of 180 days
        - 23x less expensive
- Expedited retrieval - for a cost can get it back in 10 mins
- Bulk retrieval

### EFS
- Managed NFS
- Works with EC2 in Multi-AZ
- NFSv4.1 protocol
- Block store that can be shared across instances
- Attached to instances as a filesystem
- Use-case for content management, web serving, data sharing, wordpress
- Linux AMI, Not Windows
- EFS file sync - to sync from on-premise file system to EFS

### Storage Gateway
- File gateway
    - NFS mount
    - Backed by S3
    - NFS or SMB
    - Cache - most recently used data is cached in the file gateway
- Volume gateway
    - Block storage
    - Cached volume - low latency for recently used data
    - Stored volume - data is on premise, backed-up to S3
    - iSCSI protocol
- Tape gateway
    - Backup
    - Virtual Tape Library backed by S3 and Glacier
    - iSCSI

Deployment options - hardware device, virtual device

Exam tips:
- On premise data to the cloud -> Storage Gateway
- File access / NFS -> File Gateway
- Volumes / Block storage/ iSCSI -> Volume Gateway
- Tape solution -> Tape Gateway


# Database

### RDS
- Relational Database Service
- Managed service for relational databases (patches, backup, monitoring)
- Platforms
    - MySQL
    - PostgreSQL
    - MariaDB
    - Oracle
    - SQL Server
    - Aurora
- Multi-AZ - for Disaster Recovery (not scaling)
    - __Synchronous replication to standby in different region__
    - Increases availability
    - Automatic failover via single DNS
    - Not used for scaling
- Read Replicas (for scalability)
    - Some platforms support read replicas
    - Up to 5 read replicas
    - Used for scaling reads
    - ASYNC replication within AZ, cross-AZ, and cross-region
    - Replicas can be promoted to their own db
    - __Applications must update connection string to leverage read replicas__
        - read connection and write connection
- Backups - automatically daily stored - default 7 days, can go to 35
- Snapshots - manually triggered
- Maintenance window - patches and upgrades get applied
    - Major version upgrades may need to be specified
- Encryption
    - RDS Encryption at rest with AWS KMS
    - SSL certificates to encrypt data in transit
    - To enforce SSL:
        - PostgreSQL: rds.force_ssl=1
        - MySQL: "GRANT USAGE ON *.* TO 'mysqluser… REQUIRE SSL"
    - Provide SSL trust certificate - download from AWS
    - Provide SSL options when connecting
- Usually deployed within a private subnet, not public
- Security
    - Leverages security groups
    - IAM policies control who can manage
- Authentication
    - Username / password login, or IAM for MySQL, PostgreSQL
    - IAM Authentication
        - __MySQL, PostgreSQL only__
        - 15 minute token
- TDE - Transparent Data Encryption
  - __Oracle or SQL Server only__ (on top of KMS - may affect performance)
- No SSH access (managed service)
- __Manage DB engine configuration through Parameter Groups__

### Amazon Aurora

- Proprietary
    - MySQL (5x performance increase)
    - PostgreSQL (3x performance increase)
- Storage automatically grows in increments of 10GB up to 64TB
- Read replicas
    - Up to 15 read replicas
    - Faster replication than RDS
    - Autoscaling replicas
    - Reader endpoint - connects automatically to replicas
    - Load balancing happens at the connection level
    - cross-region
- Writer endpoint - single via DNS
- HA native
    - 6 copies of data stored across 3 AZ
    - Self healing peer-to-peer replication
    - Storage is striped across 100s of volumes
- Costs more (20%) but is more efficient
- Multi-AZ by default
- Multi-Region available
- Encryption at rest via KMS
- __IAM authentication__
- No SSH
- DR
    - One primary region
    - One DR region
    - DR region can be used for lower latency reads
- Global Database - spans multiple regions and enables DR
    - One primary region, one DR region
    
Aurora Serverless
- No need to choose an instance size
- Only supports MySQL 5.6 and Postgres (beta)
- Can migrate from Aurora cluster to aurora serverless
- Measured in capacity units


### DynamoDB

- Low latency at any scale (trillions of requests/day, millions/second)
- Key/Value store
- Document store
- Tables with primary key
- Items are like a rows
- Attribute is like a column
- Item max is 400kb

Provisioned Throughput
- Read Capacity Units
- Write Capacity Units
- Throughput can be exceeded if you have burst credits

Dynamo DB Accelerator (DAX)
- Cache for DynamoDB (5 mins default)
- Writes go through to Dynamo

DynamoDB Stream
- Changes in Dynamo can end up in Stream
- Stream can be read by Lambda
    - React to changes
    - Analytics
    - Insert into ElasticSearch
- 24 hours of data retention

Transactions (Nov 2018)

On Demand (Nov 2018)
- No capacity planning needed
- More expensive than provisioned

Global tables
- Multi region, replicated, high performance
    
Local DynamoDB available for dev


### ElastiCache
- In-memory database - microseconds to milliseconds (10x faster)
- Types:
    - Memcached
        - Memcached still available but offers fewer features than Redis
    - Redis
        - Redis is good for gaming leaderboards because it has sorting
        - Redis AUTH (username/password) (requires SSL)
- Does not support IAM
- Multi-AZ with auto failover
- Write scaling
- Read scaling
- Relieves load from DB / share state
- Must have an invalidation strategy (App code, etc.)
- Low latency
- Scaling and replicas
- Use-case: Database caching, session storage

### Neptune
- Graph database
- Use when relationship is more important

## Amazon Redshift
- Scalable data warehouse service
- Petabyte scale
- Columnar storage
- Encryped
- Redshift spectrum can handle exabytes

# Application Integration

### SNS
https://docs.aws.amazon.com/sns/latest/dg/welcome.html

- Pub / Sub messaging service
- Topics - Producer sends message to a single SNS topic
- Consumers subscribe to topics
- Subscribers get all the messages (new feature to filter messages - not on exam)
- Create decoupled apps
- Integrates across multiple AWS services
- 10,000,000 subscriptions per topic
- 100,000 topics limit
- Integrates with lots of Amazon products
-Protocols
    - Http
    - Https
    - Email
    - Email Json
    - Amazon SQS
    - AWS Lambda

### SQS
https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/welcome.html

- Message queue service
- Decoupled / fault tolerant apps
- 256 KB payload
- Store messages for up to 14 days
    - __Default is 4 days__
- Visibility Timeout
    - Message is invisible to other consumers for a defined period
    - ChangeMessageVisibility API can be used to change visibility while processing a message
    - Maximum is 12 hours
- DeleteMessage - client will delete message
- Dead letter queue - after certain retries, message will go to dead letter queue
- Long Polling - can be configured from 1 second to 20 seconds (preferred)
    - Can save money because of fewer API calls - increase efficiency
    - __Set ReceiveMessageWaitTimeSeconds to a number greater than zero__
- FIFO Queue
    - Offers deduplication by id
    - Name must end with .fifo
    - Reduced throughput (significantly reduced)

### SWF
Simple Workflow Service
- Coordinates steps in an application (code, web service calls, human actions, etc.)
- Used by Amazon in warehouse
- Workflow executions can last up to 1 year (14 days for SQS)
- Task-oriented API
- __A task is assigned only once and never duplicated__
- Keeps track of all the tasks and events in an application
- Deciders - control the flow of activity tasks
- Workers - carry out the activity tasks
- Domain - collection of related tasks

# Networking and Content Delivery

### VPC

A logically isolated section of the AWS Cloud where you can launch AWS resources in a virtual network that you define
 
- Enables virtual networks (IPv4 and IPv6)
    - IP address range, subnets, route tables, network gateways
- Supports public and private subnets
- Can utilize NAT for private subnets
- Enables a connection to your own datacenter
- Can connect to others
- VPC Endpoint - can add a private endpoint between services to keep traffic in the VPC and not over the public internet
    - Supports private connections to many AWS services (VPC endpoints)
    - Types 
        - __Gateway VPC Endpoint - for DynamoDB and S3 only__
        - Interface VPC Endpoint - for other services 
- Security Groups - (stateful) applied to services, used as a firewall
- Network Access Control Lists - (stateless) - more like a traditional firewall
    - Only associated with subnets (not services)
- Virtual Private Network (VPN)
- Elastic IPs - static IP address that won't change that can be assigned to services. Can scale (elastic).
- Internet Gateways - Elastic scalable gateway to the internet
    - Unlimited bandwidth
- NAT Gateway - allows you to remain private in a private subnet but get certain internet traffic (patch updates, etc)
    - Has a hard bandwidth limit and is limited by EC2 instance type/size used.
- Route Table - a simple route table
- Public Subnet - has an Internet Gateway routed in the Route Table
- Flow Logs - gives you logs for network
- __AWS reserves 5 IP addresses for their own use__
    - ex. If you need 29 IP addresses for EC2, you can't choose a Subnet of size /27 (only 32 IP)
        - You need at least Subnet size /26 for 64 IP  
- __A VPC can only have 1 Internet Gateway__
- __Egress only Internet Gateway is for IPV6 only__
- __VPC Peering can connect 2 VPCs together (not transitive)__
    - For each VPC subnet you want to peer, you have to change the route table on both sides
    - __CIDRs must not overlap__
 
NACL
- STATELESS - traffic will be evaluated against rules on both ingress and egress
- Controls inbound and outbound traffic for subnets within the VPC
- VPCs are automatically given a default NACL with allows all outbound and inbound traffic
- Each subnet within a VPC must be associated with a NACL - either explicit or implicit
- Subnets can only be associated with 1 NACL at a time
- Rule can either ALLOW or DENY traffic (SecurityGroups can only ALLOW)
- When you create a NACL, it will deny all traffic by default (this is different than when AWS Default VPC creates NACL)

Security Group
- STATEFUL
- All traffic is not allowed by default
- There are no DENY rules, only allow rules
- Multiple EC2 instances across multiple subnets can belong to a Security Group
    - Can be attached to multiple instances
- Cannot block a specific IP Address (need NACL for that)
- firewall-like controls for resources within the VPC
- Locked down to a region/vpc combination
 
VPC Flow logs - captures the information around traffic within the VPC


### CloudFront
- Supports static and dynamic content
- Utilizes AWS edge locations
- Cache (has an upstream origin for content source)
- Includes advanced security features
    - AWS Shield for protection agains DDoS attacks
    - AWS WAF
- Restrict Bucket Access - can only access s3 content via CloudFront
- Origin Access Identity
    - To connect to S3
- Signed URLs - use SDK to generate signed urls for CloudFront
- GEO Restriction - determined via 3rd party GEO-IP database
    - Whitelist countries
    - Blacklist countries
    - Use-case: copyright laws

### Route53
- DNS service (connect domain name to ip address / servers)
- Highly available, can resolve to a different region if needed
- Global resource routing depending on latency or geo location (proximity)
- Routing policy types
    - Weighted routing - eg 60% traffic to one IP, 40% to the other
    - Latency routing - based on health check
    - Geolocation routing - based on location
    - Geoproximity routing - visual diagram of routing
- Record Sets
    - A RECORD - Points URL to IP address
    - CNAME - points a URL to any other URL - only for non-root domain
    - ALIAS - points a URL to an AWS Resource - works for root domain and non root domain
        - A custom AWS extension to DNS

### Amazon API Gateway
- Fully managed API management service
- Directly integrates with multiple AWS services
- Provides monitoring and metrics on API calls
- Supports VPC and on-prem private apps
- Track client consumption of API
- Can sit in front of microservices with disparate backends
- Versioning - maintain multiple versions
- Environments - dev, test, stage, prod, etc.
- Create API keys
- __Throttle requests to prevent attacks / abuse__
- Transform requests / responses
- Can use custom domain
- Supports AWS Certificate Manager (free)
- __API Caching - TTL__ to improve performance
- Scales automatically
- __Need to enable CORS on API Gateway__
- __HTTPS ONLY (does not support unencrypted HTTP)
    - __Uses the API Gateway certificate, or your custom domain / certificate__

Security:
- Authz and Authn
    - API keys can be associated with a client / user
- IAM permissions - client passes Sig v4 header
- Lambda Authorizer (formerly Custom Authorizers)
    - Validate token passed in header
    - Cache result of authentication
    - Used for 3rd party Oauth / SAML
    - Lambda must return IAM policy
- Cognito User Pools
    - Fully manages user lifecycle
    - API gateway verifies identity
    - Only helps with authentication, not authorization
 
### Direct Connect
Establish a private, dedicated network connection from your private data center to AWS
- IPSECTunnel

### Bastion Hosts
- "jump box"
- Put one in your public subnet to authenticate and access your other servers
- Make sure it only has port 22 traffic from your own IP you need, not from the security groups of your other instances


# Analytics

### Athena
https://docs.aws.amazon.com/athena/latest/ug/what-is.html
- Fast, cost-effective, interactive query service that makes it easy to analyze petabytes of data in S3 with no data warehouses or clusters to manage.
- SQL to query
- JDBC/ODBC driver available
- Charged per query and per amount of data scanned
- CSV, JSON, ORC, AVRO, and Parquet
- __Exam tip: Use Athena to analyze data directly on S3__

### EMR
- Elastic Map Reduce
- Big data analytics for Spark, Hadoop, etc.

### Kinesis
- Managed alternative to Apache Kafka
- Streaming tool
- Logs, metrics, IoT, click streams, stock prices, gaming data
- Data is replicated to 3 AZs
- Choose a partition key that is high distributed (prevent hot partition)
- Consumer
    - KCL - Kinesis Client Library - uses DynamoDB to track workers
- Service Types:
    - Kinesis Streams
        - Data is stored in Shards, you can add more Shards to scale
        - Stores data for 24 hours, up to 7 days
        - Consumers can store data in DynamoDB, RDS, S3, etc.
    - Kinesis Firehose
        - Data is analyzed as it streams in (Lambda)
        - Data is not stored but can be output to S3, ElasticSearch, etc.
    - Kinesis Analytics
        - Can analyze the data on the fly

### Glue
- Fully-managed ETL
- Prep data for analytics
- Serverless
- Automated code generation
- Glue Data Catalog: metadata of the source tables


# Management and Governance

### CloudWatch
- Monitoring and management service
- Logs, metrics, events for most AWS services
- Alarms based on metrics (status codes, etc.)
- Provides visualization capabilities for metrics (response time over a week, etc.)
- ustom dashboards based on metrics

### CloudFormation
- Infrastructure as code
- Managed service for provisioning infrastructure based on templates
- YAML or JSON
- Infrastructure as code (best practice for configuring)
- Manages dependencies between resources
- Creates services in the correct order
- Provides drift detection to find changes
- Rollback triggers

### CloudTrail

- Enables logging of all actions taken within your AWS account
- __CloudTrail event log files are encrypted by default with S3 SSE (can use optional KMS key)__

### OpsWorks
- Managed Chef / Puppet
- Help managing configuration as code
- Consistent deployments
- Automate user accounts, cron jobs, etc.
- Leverage Recipes or Manifests

# Security, Identity, & Compliance

### IAM
- Identity & Access Management
- Service that controls access to resources
- Free
- Manages both authn and authz
- Supports identity federation (active directory, etc)
- Password polices (strength, rotation policies, etc.)
- Integrates with services

Users
- Root account users - can set up support plan or delete account
    Don't use on daily basis
- IAM user - can attach permissions for what that user can do
- Can assume a Role

Groups
- Manage permissions for a group of IAM users

Roles 
- Enables an AWS service to assume permissions for a task

Policy
- JSON document that defines permissions for AWS IAM identity principal
- Defines services that identity can access and what actions can be taken on that service
- Can be either customer managed or managed by AWS
- __Can be used with Tags to restrict based on tags__

Best Practices
- Multi-factor authentication provides additional security
- Least Privilege Access - only grant the access that is required


### Cognito
- Web Identity Federation (Amazon, Facebook, Google)
- Use case is a huge amount of users (50 Million) you don't want to manage in IAM
- Can get temporary AWS credentials, mapped to an IAM role, to use AWS services
- JWT - JSON Web Token
- Sign-up and Sign-in to apps
- Identity Broker
- Recommended for all mobile applications
- User Pools - Registration, Users can sign in directly, username/password are stored in Cognito
- Identity Pools - Authorizes access to your AWS resources

### Secrets Manager
A step beyond KMS

### Certificate Manager
Integrated with services like Load Balancers

### KMS
- Key Management Service - create, rotate, enable, disable
- Generate and store keys for encryption
- Can only encrypt up to 4KB of data per call
    - If need more than that use ENVELOPE Encyprtion

### CloudHSM
Physical hardware device, encryption, for complex cryptographic needs
__If you get locked out, you will lose your keys__

### WAF & Shield
- AWS WAF - protects your app from common exploits (SQL injection, etc.)
- AWS Shield - provides detection and mitigation of DDoS attacks

# Migration and Transfer
### Snowball
- AWS Snowball - service to physically migrate petabytes of data
- AWS Snowmobile - service to physically migrate exabyte scale data onto AWS

# Other

Encryption
- S3 can do in-place encryption
- Migration required
    - EBS
    - RDS
    - ElastiCache
    - EFS
    
Cost Explorer - tool that enables you to view and analyze your costs and usage.

Amazon Guard​Duty - Protect your AWS accounts and workloads with intelligent threat detection and 
continuous monitoring

AWS Budgets - gives you the ability to set custom budgets that alert you when your costs or usage exceed
(or are forecasted to exceed) your budgeted amount.

### Parameter Store

- Secure storage for configuration and secrets
- Free for standard parameters

### Shared Responsibility Model

### Well-Architected Framework
https://aws.amazon.com/architecture/well-architected/

Five pillars of the Well-Architected Framework
1. Operational Excellence 
2. Security
3. Reliability
4. Performance Efficiency
5. Cost Optimization

### Support
- AWS Basic Support
    - Access to Trusted Advisor - gives insight and suggestions to optimize costs, etc.
        - Trusted Advisor scans your AWS infrastructure, compares it to AWS best practices in five categories
         and provides recommended actions. Cost, Performance, Security, Fault tolerance, Service limits
    - 24x7 access to support, whitepapers, forumns, etc.
    - Personal health dashboard
    - Free
- AWS Developer Support
    - Business hours access to support engineers
    - Limited to 1 primary contact (not targeted at organizations)
    - $29/month but can go up based on usage
- AWS Business Support
    - Full set of Trusted Advisor checks
    - 24x7 phone, email, chat with support engineers
    - Unlimited contacts (anyone can submit a support request)
    - $100/month but goes up based on usage
- AWS Enterprise Support
    - Includes Technical Account Manager
    - Concierge Support Team
    - $15,000/month but goes up based on usage
- Response time goes down for each level

- Reliability - Solution is performing tasks and working as expected
- Fault tolerance - Being able to support the failure of components
- High Availability - Entire solution running despite issues that may occur
- Disaster Recovery
    - Terms
        - RPO - Recovery Point Objective - Time between recovery point and disaster results in data loss
        - RTO - Recovery Time Objective - When you recover (RTO - disaster = downtime )
    - Backup and restore (High RPO)
    - Pilot Light - minimal resources setup in AWS to support a DR event (ex. main database)
    - Warm Standby - full system running in AWS (not scaled), ready to go and scale up to meet demand
    - Multi-Site - (Low RTO) (Active / Active) - actively sending users to AWS and your own data center

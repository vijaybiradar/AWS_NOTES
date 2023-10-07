# AWS Regions

- AWS Regions are clusters of data centers
- Example of regions: us-east-1, eu-west-1, etc.
- Most services provided by AWS are regions scoped. This means a data for a service used in one region is not replicated in another region

## AWS Availability Zones (AZ)

- AWS Regions are divided into availability zones
- Each region has between 2 and 6 AZs. Usually they have 3
- Each AZ is one or more discrete data center with redundant power, networking an connectivity
- All AZs from a region are geographically separated from each other
- Even if they are separated, they have high bandwidth connection between them


## 3 AWS service categories 

# IaaS (Infrastructure as a Service):

- Components: Hardware + Operating System
- Example: Amazon EC2 (Elastic Compute Cloud)
- Explanation: IaaS provides virtualized hardware resources (e.g., virtual machines) and an operating system. Users are responsible for managing the OS, applications, and configurations on these virtual machines.

# PaaS (Platform as a Service):

- Components: Hardware + Operating System + Framework
- Example: AWS Elastic Beanstalk
- Explanation: PaaS builds upon IaaS by adding a framework layer. It abstracts not only the hardware and OS but also provides a platform or runtime environment for developing, deploying, and managing applications. Users focus on coding, and the platform handles infrastructure and framework components.

# SaaS (Software as a Service):

- Components: Hardware + Operating System + Framework + Application
- Example: AWS Lambda
- Explanation: SaaS encompasses the entire stack, including hardware, OS, framework, and a fully functional application. Users consume the application as a service, typically over the internet, without needing to manage any infrastructure or application development.


# EC2

- It mainly consists of the following capabilities:
    - Renting virtual machines in the cloud (EC2)
    - Storing data on virtual drives (EBS)
    - Distributing load across multiple machines (ELB)
    - Scaling the services using an auto-scaling group (ASG)

## Introduction to Security Groups (SG)

- Security Groups are the fundamental of networking security in AWS
- They control how traffic is allowed into or out of EC2 machines
- Basically they are firewalls

## Security Groups Deep Dive

- Security groups regulate:
    - Access to ports
    - Authorized IP ranges - IPv4 and IPv6
    - Control of inbound and outbound network traffic
- Security groups can be attached to multiple instances
- They are locked down to a region/VPC combination
- They do live outside of the EC2 instances - if traffic is blocked the EC2 instance wont be able to see it
- *It is good to maintain one separate security group for SSH access*
- If the request for the application times out, it is most likely a security group issue
- If for the request the response is a "connection refused" error, then it means that it is an application error and the traffic went through the security group
- By default all inbound traffic is **blocked** and all outbound traffic is **authorized**
- A security group can allow traffic from another security group. A security group can reference another security group, meaning that it is no need to reference the IP of the instance to which the security group is attached

## Elastic IP

- When an EC2 instance is stopped and restarted, it may change its public IP address
- In case there is a need for a fixed IP for the instance, Elastic IP is the solution
- An Elastic IP is a public IP the user owns as long as the IP is not deleted by the owner
- With Elastic IP address, we can mask the failure of an instance by  rapidly remapping the address to another instance 
- AWS provides a limited number of 5 Elastic IPs (soft limit)
- Overall it is recommended to avoid using Elastic IP, because:
    - They often reflect pool arhcitectural decisions
    - Instead, us e a random public IP and register a DNS name to it

## EC2 User Data

- It is possible to bootstrap (run commands for setup) an EC2 instance using EC2 User data script
- The user data script is only run once at the first start of the instance
- EC2 user data is used to automate boot tasks such as:
    - Installing update
    - Installing software
    - Downloading common files from the internet
    - Any other start-up task
- THe EC2 user data scripts run with root user privileges

## EC2 Instance Launch Types

- On Demand Instances: short workload, predictable pricing
- Reserved: known amount of time (minimum 1 year). Types of reserved instances:
    - Reserved Instances: recommended long workloads
    - Convertible Reserved Instances: recommended for long workloads with flexible instance types
    - Scheduled Reserved Instances: instances reserved for a longer period used at a certain schedule 
- Spot Instances: for short workloads, they are cheap, but there is a risk of losing the instance while running
- Dedicated Instances: no other customer will share the underlying hardware
- Dedicated Hosts: book an entire physical server, can control the placement of the instance

### EC2 On Demand

- Pay for what we use, billing is done per second after the first minute
- Hast the higher cost but it does not require upfront payment
- Recommended for short-term and uninterrupted workloads, when we can't predict how the application will behave

### EC2 Reserved Instances

- Up to 75% discount compared to On-demand
- Pay upfront for a given time, implies long term commitment
- Reserved period can be 1 or 3 years
- We can reserve a specific instance type
- Recommended for steady state usage applications (example: database)
- **Convertible Reserved Instances**:
    - The instance type can be changed
    - Up to 54% discount
- **Scheduled Reserved Instances**:
    - The instance can be launched within a time window
    - It is recommended when is required for an instance to run at certain times of the day/week/month

### EC2 Spot Instances

- We can get up to 90% discount compared to on-demand instances
- It is recommended for workloads which are resilient to failure since the instance can be stopped by the AWS if our max price is less then the current spot price
- Not recommended for critical jobs or databases
- Great combination: reserved instances for baseline performance + on-demand and spot instances for peak times

### EC2 Dedicated Hosts

- Physical dedicated EC2 server
- Provides full control of the EC2 instance placement
- It provides visibility to the underlying sockets/physical cores of the hardware
- It requires a 3 year period reservation
- Useful for software that have complicated licensing models or for companies that have strong regulatory compliance needs

### EC2 Dedicated Instances

- Instances running on hardware that is dedicated to a single account
- Instances may share hardware with other instances from the same account
- No control over instance placement
- Gives per instance billing

## EC2 Spot Instances - Deep Dive

- With a spot instance we can get a discount up to 90%
- We define a max spot price and get the instance if the current spot price < max spot price
- The hourly spot price varies based on offer and capacity
- If the current spot price goes over the selected max spot price we can choose to stop or terminate the instance within the next 2 minutes
- Spot Block: block a spot instance during a specified time frame (1 to 6 hours) without interruptions. In rare situations an instance may be reclaimed
- Spot request - with a spot request we define:
    - Maximum price
    - Desired number of instances
    - Launch specifications
    - Request type:
        - One time request: as soon as the spot request is fulfilled the instances will be launched an the request will go away
        - Persistence request: we want the desired number of instances to be valid as long as the spot request is active. In case the spot instances are reclaimed, the spot request will try to restart the instances as soon as the price goes down
- Cancel a spot instance: we can cancel spot instance requests if it is in open, active or disabled state (not failed, canceled, closed)
- Canceling a spot request does not terminate the launched instances. If we want to terminate a spot instance for good, first we have to cancel the spot request and the we can terminate the associated instances, otherwise the spot request may relaunch them

### Spot Fleet

- Spot Fleet is a set of spot instances and optional on-demand instances
- The spot fleet will try to meet the target capacity with price constraints
- AWS will launch instances from a launch pool, meaning we have to define the instance type, OS, AZ for a launch pool
- We can have multiple launch pools from within the best one is chosen
- If a spot a fleet reaches capacity or max cost, no more new instances are launched
- Strategies to allocate spot instances in a spot fleet:
    - **lowerPrice**: the instances will be launched from the pool with the lowest price
    - **diversified**: launched instances will be distributed from all the defined pools
    - **capacityOptimized**: launch with the optimal capacity based on the number of instances

## EC2 Instance Types

- R: applications that needs a lot of RAM - in-memory cache
- C: applications that need good CPU -  compute/database
- M: applications that are balanced - general / web app
- I: applications that need good local I/O - databases
- G: applications that need GPU - video rendering / ML
- T2/T3 - burstable instances
- T2/T3 unlimited: unlimited burst

### Bustable Instances (T2/T3)

- Overall the performance of the instance is OK
- When the machine needs to process something unexpected (a spike load), it can burst and CPU can be very performant
- If the machine bursts, it utilizes "burst credits"
- If all the credits are gone, the CPU becomes bad
- If the machine stops bursting, credits are accumulated over time
- Credit usage / credit balance of a burstable instance can be seen in CloudWatch
- CPU credits: bigger the instance the faster credit is earned
- T2/T3 Unlimited: extra money can be payed in case the burst credits are used. There wont be any performance loss

## AMI

- AWS comes with lots of base images
- Images can be customized ar runtime with EC2 User data
- In case of more granular customization AWS allows creating own images - this is called an AMI
- Advantages of a custom AMI:
    - Pre-install packages
    - Faster boot time (on need for the instance to execute the scripts from the user data)
    - Machine configured with monitoring/enterprise software
    - Security concerns - control over the machines in the network
    - Control over maintenance
    - Active Directory out of the box
- An AMI is built for a specific region (NOT GLOBAL!)

### Public AMI

- We can leverage AMIs from other people
- We can also pay for other people's AMI by the hour, basically renting the AMI form the AWS Marketplace
- Warning: do not use AMI which is not trustworthy!

### AMI Storage

- An AMI takes space and they are stored in S3
- AMIs by default are private and locker for account/region
- We can make our AMIs public and share them with other people or sell them on the Marketplace

### Cross Account AMI Sharing

- It is possible the share AMI with another AWS account
- Sharing an AMI does not affect the ownership of the AMI
- If a shared AMI is copied, than the account who did the copy becomes the owner
- To copy an AMI that was shared from another account, the owner of the source AMI must grant read permissions for the storage that backs the AMI, either the associated EBS snapshot or an associated S3 bucket
- Limits:
    - An encrypted AMI can not be copied. Instead, if the underlying snapshot and encryption key where shared, we can copy the snapshot while re-encrypting it with a key of our own. The copied snapshot can be registered as a new AMI
    - We cant copy an AMI with an associated **billingProduct** code that was shared with us from another account. This includes Windows AMIs and AMIs from the AWS Marketplace. To copy a shared AMI with **billingProduct** code, we have to launch an EC2 instance from our account using the shared AMI and then create an AMI from source

## Placement Groups

- Sometimes we want to control how the EC2 instances are placed in the AWS infrastructure
- When we create a placement group, we can specify one of the following placement strategies:
    - **Cluster** - cluster instances into a low-latency group in a single AZ
    - **Spread** - spread instances across underlying hardware (max 7 instances per group per AZ)
    - **Partition** - spread instances across many different partitions (which rely on different sets of racks) within an AZ. Scale to 100s of EC2 instances per group (Hadoop, Cassandra, Kafka)

### Placement Groups - Cluster

- Pros: Great network (10Gbps bandwidth between instances)
- Cons: if the rack fails, all instances fail at the time
- Use cases:
    - Big data job that needs to complete fast
    - Application that needs extremely low latency and high network throughput

### Placement Groups - Spread

- Pros:
    - Can span across multiple AZs
    - Reduces risk for simultaneous failure
    - EC2 instances are on different hardware
- Cons:
    - Limited to 7 instances per AZ per placement group
- Use case:
    - Application that needs to maximize high availability
    - Critical applications where each instance must be isolated from failure

### Placement Groups - Partitions

- Pros:
    - Up to 7 partitions per AZ
    - Can have hundreds of EC2 instances per AZ
    - The instances in a partition do not share racks with the instances from other partitions
    - A partition failure can effect many instances but they wont affect other partitions
    - Instances get access to the partition information as metadata
- Use cases: HDFS, HBase, Cassandra, Kafka

## Elastic Network Interfaces - ENI

- Logical component in a VPC that represents a virtual network card
- An ENI can have the following attributes:
    - Primary private IPv4 address, one or more secondary IPv4 addresses
    - One Elastic IP (IPv4) per private IPv4
    - One Public IPv4
- ENI instances can be created independently from an EC2 instance
- We can attach them on the fly to an EC2 instances or move them from one to another (useful for failover)
- ENIs are bound to a specific available zone
- ENIs can have security group attached to them
- EC2 instances usually have a primary ENI (eth0). In case we attach a secondary ENI, eth1 interface will be available. The primary ENI can not be detached.

## EC2 Hibernate

- We can stop or terminate EC2 instances:
    - If an instance is stopped: the data on the disk (EBS) is kept intact
    - If an instance is terminated: any root EBS volume will also gets destroyed
- On start, the following happens in case of an EC2 instance:
    - Fist start: the OS boots and EC2 User data script is executed
    - Following starts: the OS boots
    - After the OS boot the applications start, cache gets warmed up, etc. which may take some time
- EC2 Hibernate:
    - All the data from RAM is preserved on shut-down
    - The instance boot is faster
    - Under the hood: the RAM state is written to a file in the root EBS volume
    - The root EBS volume must be encrypted
- Supported instance types for hibernate: C3, C4, C5, M3, M4, M5, R3, R4, R5
- Supported OS types: Amazon Linux 1 and 2, Windows
- Instance RAM size: must be less then 150 GB
- Bare metal instances do not support hibernate
- Root volume: must be EBS, encrypted, not instance store. And it must be large enough
- Hibernate is available for on-demand and reserved instances
- An instance can not hibernate for more than 60 days

## EC2 for Solution Architects

- EC2 instances are billed by the second, t2.micro is free tier
- On Linux/Mac we can use SSH, on Windows Putty or SSH
- SSH is using port 22, the security group must allow our IP to be able to connect
- In cas of a timeout, it is most likely a security group issue
- Permission for SSH key => chmod 0400
- Security groups can reference other security groups instead of IP addresses
- EC2 instance can be customized at boot using EC2 User Data
- 4 EC2 launch modes:
    - On-demand
    - Reserved
    - Spot
    - Dedicated hosts
- We can create AMIs to pre-install software
- An AMI can be copied through accounts and regions
- EC2 instances can be started in placement groups:
    - Cluster
    - Spread
    - Partition


# Elastic Load Balancers

## Scalability and High Availability

- Scalability means that an a system can handle greater loads by adapting
- We can distinguish two types of scalability strategies:
    - **Vertical Scalability** (scale up/down)
        - Increase the size of the current instance (ex. from a t2.micro instance migrate to a a t2.large one)
        - Vertical scalability is common for non distributed systems, such as databases
        - RDS, ElastiCache are services that can scale vertically
    - **Horizontal Scalability** (scale out/in)
        - Increase the number of instances on which the application runs
        - Horizontal scaling implies having a distributed system
        - It's easy to scale horizontally thanks to could offerings such as EC2
- High Availability means running our application in at least 2 data centers (AZs)
    - The goal of high availability is to survive a data center loss
- High Availability can be:
    - Passive
    - Active
- Load balancers can scale but not instantaneously - contact AWS for a "warm-up"
- Troubleshooting:
    - 4xx errors are client induced errors
    - 5xx errors are application induced errors (server side errors)
    - Error 503 means that the load balancer is at capacity or no registered targets can be found
    - If the load balancer can't connect to the application, it most likely means that the security group blocks the connection
- Monitoring:
    - ELB access logs will log all the access requests to the LB
    - CloudWatch Metrics will give aggregate statistics (example: connections counts)

## Load Balancing Basics

- Load balancers are servers that forward internet traffic to multiple other servers (most likely EC2 instances)
- Why use load balances?
    - Spear load across multiple downstream instances
    - Expose a single point of access (DNS) to the application
    - Seamlessly handle failures of downstream instances (by using health checks)
    - Do regular health checks to registered instances
    - Provide SSL termination (HTTPS) for the website hosted on the downstream instances
    - Enforce stickiness for cookies
    - High availability across availability zones (load balancer can be spread across multiple AZs, not regions!!!)
    - Cleanly separate public traffic from private traffic
- An ELB (Elastic Load Balancer) is a **managed load balancer** which means:
    - AWS guarantees that it will be working
    - AWS takes care of upgrades, maintenance and high availability
    - An ELB provides a few configuration options for us also
    - It costs less to setup our custom load balancer, but it will be a lot more effort to maintain on the long run
    - An ELB is integrated with many AWS offering/services, it will be more flexible than a custom LB

## Health Checks

- They enable for a LB to know if an instance for which traffic is forwarded is available to reply to requests
- The health checks is done using a port and a route (usually /health)
- If the response is not 200, then the instance is considered unhealthy

## Types of Load Balancers on AWS

- AWS provides 4 type of load balancers:
    - Classic Load Balancer (v1 - old generation): supports HTTP, HTTPS and TCP
    - Application Load Balancer (v2 - new generation): supports HTTP, HTTPS and WebSockets
    - Network Load Balancer (v2 - new generation): supports TCP, TLS (secure TCP) and UDP
    - Gateway Load Balancer (new generation - see VPC section of the notes)
- It is recommended to use the new versions
- We can setup **internal** (private) and **external** (public) load balancers on AWS

### Classic Load Balancers (CLB)

- They support 2 types of connections: TCP (layer 4) and HTTP(S) (layer 7)
- Health checks are either TCP or HTTP based
- CLBs provide a fixed hostname: XXX.region.elb.amazonaws.com

### Application Load Balancers (ALB)

- They are a layer 7 type load balancers (only HTTP or HTTPS)
- They allow load balancing to multiple HTTP applications across multiple machines (target groups). Also they allow to load balance to multiple applications on the same EC2 instance (useful in case of containers)
- They have support for HTTP2 and WebSockets.
- They support redirects, example for HTTP to HTTPS
- They provide routing tables to different target groups:
    - Routing based on path in URL
    - Routing based on the hostname
    - Routing based on query strings an headers
- ALBs are great fit for micros-services and container based applications
- ALBs have port mapping features to redirect to dynamic ports in case ECS
- Target groups can contain:
    - EC2 instances (can be managed by an Auto Scaling Group)
    - ECS tasks (managed by ECS itself)
    - Lambda Functions -  HTTP request is translated to a JSON event
    - IP Addresses - must be private IP addresses
- ALBs also provide a fixed hostname (same as CLBs): XXX.region.elb.amazonaws.com
- The application servers behind the LB can not see the IP of the client who accessing them directly, but they can retrieve for **X-Forwarded-For** header. The port can be fetched from **X-Forwarded-Port** and the protocol from **X-Forwarded-Proto**

### Network Load Balancers (NLB)

- Network load balancers (layer 4) allow to:
    - Forward TCP and UDP traffic to the registered instances
    - Handle millions of requests per second
    - Less latency ~100ms (vs 400ms for ALB)
- NLBs have **one static IP per AZ** and supports Elastic IPs (can be used when whitelisting is necessary)
- Use case for NLBs: NLBs are used for extreme performance in case of TCP or UDP traffic (example: video games)
- Instances behind an NLB don't see traffic coming from the load balancer, they see traffic as it was coming from the outside world => no security group is attached to LB => security group attached to the target EC2 instance should be changed to allow traffic from the outside (example: 0.0.0.0/0, on port 80)

## Stickiness

- It is possible to implement stickiness in case of CLB and ALB load balancers
- Stickiness means that the traffic from the same client will be forwarded to the same target instance
- Stickiness works by adding a cookie to the request which has an expiration date for controlling the
stickiness period
- Possible use case for stickiness: we have to make sure that the user does not lose his session data
- Enabling stickiness may bring imbalance to the load over the downstream target instances

## Cross-Zone Load Balancing

- With Cross-Zone Load Balancing enabled each LB instance distributes traffic evenly across multiple AZs
- Otherwise, ech LB node distributes requests evenly only in the AZ where it is registered
- Classic Load Balancer: 
    - Cross-zone load balancing is disabled by default
    - No additional charges for cross zone load balancing if the feature is enabled
- Application Load Balancer: 
    - Cross-zone load balancing is always on, can not be disabled
    - No charges applied for cross zone load balancing
- Network Load Balancer:
    - Cross-zone load balancing is disabled by default
    - Additional charges apply if the feature is enabled

## SSL/TLS Certificates

- An SSL certificate allows traffic to be encrypted between the clients and the load balancers. This ia called encryption in transit or in-flight encryption
- SSL - Secure Socket Layer
- TLS (newer version of SSL) - Transport Layer Security
- Nowadays TLS are mainly used, but we are still referring to it as SSL
- Public SSL certificates are issued by a Certificate Authority
- SSL certificates have an expiration dates and they must be renewed
- SSL termination: client can talk with a LB using HTTPS but internal traffic can be routed to a target using HTTP
- Load balancer can load an X.509 certificate (which is a SSL/TLS server certificate)
- We can manage certificates in AWS using ACM (AWS Certificate Manager)
- HTTPS Listener:
    - We must specify a default certificate
    - We can add an optional list of certificates to support multiple domains
    - Clients can use SNI (Server Name Indication) to specify which hostname want to reach
    - Ability to specify a security policy to support older versions of SSL/TLS (for legacy clients like Internet Explorer 5 lol:) )

### SNI - Server Name Indication

- SNI solves the problem of being able to load multiple SSL certificates onto one web server
- There is a newer protocol which requires the client to indicate the hostname of the target server in the initial SSL handshake
    - In case of AWS this only works for ALB, NLB and CloudFront (no CLB!)

## ELB - Connection Draining

- Feature naming:
    - In case of a CLB is called Connections Draining
    - If we have a target group: (ALB, NLB) it is called Deregistration Delay
- Connection draining is the time to complete in-flight requests while the instance is de-registering or unhealthy. Basically it allows the instance to terminate whatever it was doing
- The LB will stop sending new requests to the target instance which is in progress of de-registering
- The time period of the connection draining can be set between 1 seconds to 3600 seconds
- It also can be disabled (set the period to 0 seconds)


# Amazon S3

- Amazon S3 is one of the main building blocks of AWS being advertised as "infinitely scaling" storage

## S3 Buckets

- Amazon S3 allows to store objects (files) in "buckets"
- Buckets must have a **globally unique name**
- Buckets are defined at the region level
- Bucket naming conventions:
    - No uppercase
    - No underscore
    - 3-63 character long names
    - Name must no be an IP
    - Must start with a lowercase letter or number

## S3 Objects

- In a bucket an object is a file
- Objects do have a **key** as an identifier, which is basically the full path of the file:
    - s3://my-bucket/**my_file.txt**
    - s3://my-bucket/**my_folder/another_folder/my_file.txt**
- The key is composed of *the prefix* + **object name**
    - s3://my-bucket/*my_folder/another_folder*/**my_file.txt**
- In S3 there is no concept of directories within the buckets, just keys with very long names that contain slashes ("/")
- Object values are the content of the body:
    - Max object size in S3 is 5TB
    - In case of a upload bigger than 5GB, we must use **multi-part upload**
- Each object can have metadata: list of text key/value pairs - system or user added
- Each object can have tags: unicode key/value par, useful for security, lifecycle. A bucket can have up to 10 tags
- If versioning is enabled each object has a version ID

## S3 Versioning

- Files can have a version in S3, this should be enabled at the bucket level
- If a files is uploaded with the same key (same filename) the version of the file will be changed, the existing file wont be overridden, we will have both files available with different versions
- It is best practice to version the files, because:
    - The files will be protected against unintended deletes
    - The files can be rolled back to previous versions
- Notes:
    - Any file that is not versioned prior to enabling versioning will have the version "null"
    - Suspending versioning does not delete the previous versions of the file
- Deleting versioned files:
    - When deleting a versioned file adds a delete marker to the file, but the file wont be deleted
    - The file can be restored by deleting the delete marker
    - Deleting the delete marker and the file together is a **permanent delete**, meaning the file wont be able to be restored

## S3 Security

### Encryption at Rest

- AWS provides 4 methods of encryption for objects in S3
    1. SSE-S3: encrypts S3 objects using keys handled and managed by AWS
    2. SSE-KMS: leverage AWS Key Management Service to manage encryption keys
    3. SSE-C: the encryption keys are managed by the user
    4. Client Side Encryption
- SSE-S3:
    - Keys used for encryption is managed by Amazon S3
    - Objects are encrypted in the server side
    - It uses AES-256 encryption
    - In order to have SSE-S3 encryption clients must set a header, which is **x-amz-server-side-encryption": "AES256"**
- SSE-KMS:
    - Encryption keys are handled and managed by KMS
    - KMS allows the manage which keys will be used for the encryption, moreover it has audit trails in order to be able to see who was using the KMS key
    - Objects are encrypted in the server-side
    - In order to have SSE-S3 encryption clients must set a header, which is **x-amz-server-side-encryption": "aws:kms"**
- SSE-C:
    - Server side encryption using data keys provided by the user from the outside of AWS
    - Amazon S3 does not store the encryption key provided by the user
    - Data should be transmitted through HTTPS, because a key is sent the AWS
    - Encryption key must be provided in the header of the request for every request
    - When retrieving the object, the same encryption key must be provided in the header
- Client Side Encryption:
    - Data must be encrypted before sending it to S3
    - This is usually accomplished by using a third party encryption library, like Amazon S3 Encryption Client
    - The user is solely responsible for encryption-decryption
    - The keys and the encryption cycle is managed by the user

### Encryption in transit (SSL/TLS)

- Amazon S3 exposes:
    - HTTP endpoint for non-encrypted data
    - HTTPS endpoint for encryption in flight which relies on SSL/TLS
- Most clients for S3 will use HTTPS by default
- In case of SSE-C encryption HTTPS in mandatory

### S3 Security Overview

- User based security:
    - Accomplished by using IAM policies: specified which calls should be allowed for a specified sued from IAM console
- Resource based security:
    - Accomplished by using bucket policies which are bucket wide rules from the S3 console. These rules may allow cross account access to the bucket
    - We also have Object Access Control Lists (ACL) and Bucket Access Control Lists which allow finer grain control over the bucket
Note: an IAM principle can access an S3 object if:
    - The user IAM permission allows it or the resource policy allows it
    - The is no explicit deny

### S3 Bucket Policies

- Bucket policies are JSON based documents
- They can be applied to both buckets and objects in buckets
- The effect of a statement in the bucket policy can be either allow or deny
- The principle in the policy represents the account or the user for which the policy applies to
- Common use cases for S3 bucket policies:
    - Grant public access to the bucket
    - Force objects to be encrypted at the upload
    - Grant access to another account (cross account access)

### Bucket Settings for Block Public Access

- Relatively new settings that was created to block public access to buckets and objects if the account has some restrictions:
- S3 provides 4 different kind of block public access settings:
    - new access control lists
    - any access control lists
    - new public bucket or access point policies
    - block public and cross-account access to buckets and objects through any public bucket or access point policies
- These settings were created to prevent company data leaks

### S3 Other Security Features

- Networking:
    - S3 supports VPC Endpoints (for instances in VPC without internet access)
- Logging and Audit:
    - S3 Access Logs can be stored in other S3 buckets
    - API calls can be logged in AWS CloudTrail
- User Security:
    - MFA Delete can be required in versioned buckets in order to protect for accidental deletions
    - Pre-Signed URLs: ULRs that are valid only for a limited time

## S3 Websites

- S3 can host static websites and have them accessible from the internet
- The website URL will be something like this:
    - `<bucket-name>.s3-website-<AWS-region>.amazonaws.com`
    - `<bucket-name>.s3-website.<AWS-region>.amazonaws.com`
- In case of `403` errors we have to make sure that the bucket policy allows public reads

### CORS

- An origin is a scheme (protocol), host (domain) or port
- CORS: Cross Origin Resource Sharing
- CORS is a web browser based mechanism to allow requests to other origins while visiting the main one
- Same origin example: http://example.com/app1 and http://example.com/app2
- Different origins: http://example.com and http://otherexample.com
- The request wont be fulfilled unless the other origin allows for the request, using CORS headers (example: Access-Control-Allow-Origin, Access-Control-Allow-Method)

### S3 CORS

- If a client does a cross-origin request on an S3 bucket, the correct CORS headers need to be enabled in order for the request to succeed
- Request can be allowed for a specified origin (by specifying the URL of the origin) or for all origins (by using *)

## S3 Consistency Model

- S3 is strong read-after-write consistent


# Networking - VPC

## CIDR - Classless Inter-Domain Routing

- CIDR is used for Security Group rules and AWS networking in general
- It is used to define an IP address range
    - ww.xx.yy.zz/32 - one IP address
    - 0.0.0.0/0 - all possible IP addresses
    - 192.168.0.0/26 - range of 192.168.0.0 - 192.168.0.63 64 IP addresses
- CIDR has to component:
    - The base IP: represents an IP from a given the range
    - The subnet mask: defines how many bits can change in the IP
- The subnet mask can take 2 forms:
    - 255.255.255.0 - less common
    - /24 - more common
- The subnet mask basically allows a part of the base IP to get additional next values:
    - /32 allows for 1 IP => 2^0
    - /31 allows for 2 IPS => 2^1
    - /30 allows for 4 IPs => 2^2
    - /29 allows for 8 IPs => 2^3
    ...
    - /0 allows for 2^31 IPs, all IPs
- Most used subnet masks:
    - /32 - no IP number can change
    - /24 - last IP number can change (256 IP addresses)
    - /16 - last 2 IP numbers can change (65536 IP addresses)
    - /8 - last three IP numbers can change
    - /0 - all the IP numbers can change
- To calculate CIDR, use: https://www.ipaddressguide.com/cidr

## Private vs Public IP Addresses (IPv4)

- Private IP can only allow certain values:
    - 10.0.0.0 - 10.255.255.255 (10.0.0.0/8) - used for big networks
    - 172.16.0.0 - 172.31.255.255 (172.16.0.0/12) - default AWS private VPC
    - 192.168.0.0 - 192.168.255.255 (192.168.0.0/16) - home networks
- All the rest of the IP addresses are considered public

## Default VPC

- All new accounts have a default VPC
- New instances are launched into default VPC if no subnet is specified
- Default VPCs have internet connectivity and all instances have public IP address
- We also get a public and a private DNS name

## VPC - Virtual Private Cloud

- We can have multiple VPCs in a region (max 5 per region - soft limit)
- Max CIDR per VPC is 5, for each CIDR:
    - Min size is /28 = 16 IP addresses
    - Max size is /16 = 65K IP addresses
- VPC is private => only the private address ranges are allowed
- **A newly created VPC CIDR should not overlap with an existing one used another VPC**


## VPC Subnets

- Subnets are tied to specific AZs
- AWS reservers 5 IP addresses (first 4 and the last 1 address) in each subnet
- These 5 IP addresses are not available and can not be assigned to an instance
- Example, in case of a CIDR block 10.0.0.0/24 the reserver IPs are the following:
    - 10.0.0.0 - Network address
    - 10.0.0.1 - Reserved by AWS for the VPC router
    - 10.0.0.2 - Reserved by AWS for mapping to an Amazon provided DNS
    - 10.0.0.3 - Reserved for future use
    - 10.0.0.255 - Network broadcast address. AWS does not supports broadcast in a VPC, therefore the address is reserved

## IGW - Internet Gateways

- Internet gateways helps a VPC instance to connect to the internet
- It scales horizontally and is HA and redundant
- Must be created separately from a VPC
- One VPC can have only one attached internet gateway, one internet gateway can have only one VPC
- Internet Gateway is also a NAT for the instances that have a public IPv4
- Internet Gateways on their one do not allow internet access, route tables also must be configured

## NAT Instances - Network Address Translation (outdated)

- Allow instances in private subnets to connect to the internet
- NAT instances must be launched in a public subnet to have internet access
- *Source/Destination Check* flag for EC2 must be disabled
- NAT instanced should have an Elastic IP attached to them
- Route tables must be configured to route traffic from the private subnet to the NAT instance
- In order to set up we must use a pre-configured Amazon Linux AMI
- It is not HA/resilient out of the box => We would need to set up an ASG in multi AZ + resilient user data script
- Internet traffic bandwidth depends on the EC2 instance performance

## NAT Gateway

- AWS managed NAT, higher bandwidth, better availability, no administration
- It is pay by hour for usage and bandwidth
- NAT is created in a specific AZ and it uses an Elastic IP
- It can not be used by an instance from the same subnet as the NAT
- Required an IGW
- 5Gbps of bandwidth with automatic scaling up to 45Gbps
- There is no security group to manage
- NAT Gateway is resilient within a single AZ
- We must create multiple NAT Gateways in multiple AZs for fault-tolerance

## Comparison between NAT Instance and NAT Gateway

- AWS comparison: https://docs.aws.amazon.com/vpc/latest/userguide/vpc-nat-comparison.html

## DNS Resolution in VPC

-DNS Resolutions settings:
    - **enableDnsSupport**:
        - By default is set to true
        - Helps decide if DNS resolution is supported for the VPC
        - If true, queries the AWS DNS server at 169.254.169.253
    - **enableDNSHostName**:
        - By default is set to false for an user created VPC, true for the Default VPC
        - Won't do anything unless enableDnsSupport=true
        - If true, the VPC assigns public host names to EC2 instances
- If we use custom DNS domain names in a private zone in Route53, we must set both of these attributes to true

## Network Access Control Lists (NACL) and Security Groups (SG)

- NACL is a like a firewall which controls traffic which goes outside of comes inside a subnet
- The default NACL allows everything outbound and everything inbound
- We define one NACL per subnet, new subnets are assigned the default NACL
- We can define NACL rules:
    - Rules have a number (1 - 32766) which defines the precedence
    - Rules with lower number have higher precedence
    - Rule with higher precedence wins at the time of evaluation
    - Last rule is has the precedence of asterisk (*) and denies all the requests. This rule is not editable
    - AWS recommends adding rules by increment of 100
- Newly created created NACL will deny everything (last rule)
- NACLs are great way of blocking a specific IP address at the subnet level

| Security Group                                           |  Network ACL                                                            |
|----------------------------------------------------------|-------------------------------------------------------------------------|
| Operates at the instance level                           | Operates at the subnet level                                            |
| Supports allow rules only                                | Supports allow rules and deny rules                                     |
| It is stateful: return traffic is automatically allowed  | It is stateless: return traffic must be explicitly allowed by the rules |
| All rules are evaluated before deciding to allow traffic | Rules have a precedence when deciding whether to allow or deny traffic  |
| It is associated to an instance inside of a VPC          | Automatically applies to all instances in the subnet                    |

- Example of NACL with ephemeral ports: https://docs.aws.amazon.com/vpc/latest/userguide/vpc-network-acls.html

## VPC Peering

- VPC Peering allows to connect two VPCs privately using AWS network. This will make them behave as if they were the same network
- Networks must have non-overlapping CIDRs
- VPC Peering connections is not transitive, which means it must be established for each VPC that needs to communicate with other
- VPC Peering can be done form one AWS account to another
- We must update the route tables in each VPC subnet to ensure instances can communicate with each other
- VPC peering can work inter-region, cross-account. We can reference a security group of a peered VPC (this works cross account also)


## VPC Endpoints

- Allow connection to AWS services from VPCs using a private network instead of the public internet
- They scale horizontally
- They remove the need of IGW, NAT, etc. to access AWS services
- There are 2 kinds of VPC Endpoints:
    - **Interface**: provisions an ENI (private IP address) as an entry point. A security group must be attached to it. Most AWS services use this method
    - **Gateway**: provisions a target and must be used in a route table. Gateway is used for S3 and DynamoDB
- In case of issues:
    - We have to check DNS settings resolution in the VPC
    - We have to check the route tables

## VPC FLow Logs

- Flow logs help capture information about the IP traffic going into the interfaces
- There are 2 kinds of Flow Logs:
    - VPC Flow Logs
    - Subnet Flow Logs
    - Elastic Network Interface Flow Logs
- Flow logs help monitor and troubleshoot connectivity issues
- Flow logs data can go to S3/CloudWatch logs
- It can capture network information from some of the AWS managed interface too: ELB, RDS, ElastiCache, Redshift, AWS Workspaces

### Flow Log Syntax

```
<version> <account-id> <interface-id> <srcaddr> <dstaddr> <srcport> <dstport> <protocol> <packets> <bytes> <start> <end> <action> <log-status>
```
- Srcaddr, dstaddr help identify problematic IP addresses
- Scrport, dstport help identify problematic ports
- Action: success or failure of the request due to SG/NACL
- Flow logs can be used for analytics on usage patterns or malicious behavior
- We can query VPC flow logs using Athena on S3 or CloudWatch Logs Insights

## Bastion Hosts

- Bastion hosts are used to SSH into instances from private subnets
- The bastion host is sitting in the public subnet which is connected to the private subnets
- Bastion host SG must be really strict
- We have to make sure the bastion host only has port 22 traffic from the IP we need, not from the security group of other instances

## Site to Site VPN

- Virtual Private Gateway:
    - VPN concentrator on the AWS side of the VPN connection
    - Virtual Private Gateway (VPG) is created and attached to the VPC from which we want to create the site-to-site VPN connection
    - Possibility to customize the ASN
- Customer Gateway:
    - Software application or physical device on customer side of the VPN connection
    - IP Address: use static, internet-routable IP address for the customer device. If it is behind a NAT (with NAT-T enabled), use the public IP address of the NAT

## Direct Connect (DX)

- Direct Connect provides a dedicated private connection from a remote network to the AWS VPC
- Dedicated connection must be setup between the client's DC and AWS Direct Connect locations
- In AWS we still need to setup a Virtual Private Gateway in the VPC
- DX will allow access to public resources (S3) and private (EC2) on the same connection
- Use cases:
    - Increase bandwidth throughput
    - More consistent network experience - application using real-time data feeds
    - Hybrid Environment
- DX supports both IPv4 and IPv6
- Direct Connect Gateway: if we want to setup a Direct Connect to one or more VPC in many different regions (but in the same account), we must use a Direct Connect Gateway
- Direct Connect Gateway does not replace VPC Peering
- Direct Connect has two connections types:
    - Dedicated Connections: 1 Gbps and 10 Gbps capacity
        - Physical ethernet port dedicated to a customer
        - In order to get this a request has to be made to AWS first, then completed by AWS Direct Connect Partners
    - Hosted Connections: 50 Mbps, 500 Mbps, up to 10 GBps
        - Connection requests are made via AWS Direct Connect Partners
        - Capacity can be added or removed on demand
- Lead times are often longer than 1 month

### Direct Connect Encryption

- Data in transit is not encrypted but is private
- In case there is need for encryption we can combine DX with VPN which provides IPsec-encrypted private connection

## Egress Only Internet Gateway

- Egress only Internet Gateway works only for IPv6 
- Similar function as a NAT, but for IPv6 (NAT is for IPv4)
- All IPv6 addresses are public addresses, therefore all our instances are publicly accessible
- Egress Only Internet Gateway gives our IPv6 instances access to the internet, but they wont be directly reachable by the internet
- After creating an Egress Only Internet Gateway we have to edit the route tables

## Exposing Services in a VPC to Other VPC

- Option 1: make it public
    - Goes through the public internet
    - Tough to manage access
- Option 2: VPC peering
    - Must create many peering relations
    - Opens the whole internal network to the other network
- Option 3: AWS Private Link (VPC Endpoint Services)
    - Most secure and scalable way to expose a service to multiple other VPCs
    - The solution does not require VPC peering, IGW, NAT, route tables
    - Requires a network load balancer in the service VPC and an ENI in the customer VPC

## EC2-Classic and AWS ClassicLink (deprecated)

- EC2-Classic: instances are running in a single network shared with other customers
- Amazon VPC: instances are running logically isolated in an AWS account
- ClassicLink allows to link EC2-Classic instances to a VPC inside an account. In order to have a link between a VPC and an EC2-Classic instance we have to:
    - Create a ClassicLink and associate a security group to it which will enable private communication
    - ClassicLink removes the necessity to have IPv4 public addresses for this type of communication

## AWS VPN CloudHub

- Provides secure communication between multiple customer networks in case we have multiple VPN connections
- Low cost hub-and-spoke model for primary or secondary network connectivity between locations

## Transit Gateway

- It is a star connection between all the VPCs and the on-premises infrastructure
- It is for having transitive peering between thousands of VPCs and on-premises and hub-and-spoke connections
- Transit Gateway is a regional resource, but it can work cross-region
- It supports cross-account sharing using Resource Access Manager (RAM)
- We can peer Transit Gateways across regions
- Route tables can limit which VPC can talk with what other VPC
- Works with Direct Connect Gateway, VPN connections
- Supports **IP Multicast** (not supported by any other AWS services)

## VPC Summary

- CIDR: IP Range
- VPC: Virtual Private Cloud => we define a list of IPv4 and IPv6 CIDRs
- Subnets: tied to an AZ, we define it by CIDRs
- Internet Gateway: at the VPC level, provides IPv4 and IPv6 internet access
- Route tables: must be edited to add routes from subnets to the IGW, VPC peering connections, VPC Endpoints, etc.
- NAT Instance: gives internet access to instances in a private subnets. Deprecated, must be setup in a public subnet, Source/Destination flag should be disabled
- NAT Gateway: managed by AWS, provides scalable internet access to private instances, only for IPv4
- Private DNS + Route 53: enables DND Resolution + DNS hostname inside a VPC
- NACL: Stateless, subnet level routes, used for filter inbound and outbound traffic
- Security Groups: Stateful, operate at the instance level
- VPC Peering: connect to VPC with non overlapping CIDRs, non transitive
- VPC Endpoints: provide private access to AWS Services
- VPC FLow Logs: can be configured at VPC/Subnet/ENI level, used for monitoring traffic
- Bastion Hosts: public instances to SSH into, used for connecting to other private instances via SSH
- Site to Site VPN: used for setup a Customer Gateway on DC, a Virtual Private Gateway on VPC and site-to-site VPN over public internet
- Direct Connect: used for setup a VIrtual Private Gateway on VPC and establish a direct private connection to an AWS Direct Connect Location
- Direct Connect Gateway: setup Direct Connect to many VPCs in different regions
- Internet Gateway Egress: similar to NAT, but for IPv6
- Private Link (VPC Endpoint Services): connect services from one VPC to another
- ClassicLink: connect EC2-Classic instances to VPCs
- VPN CloudHub: hub-and-spoke VPN model to connect sites
- Transit Gateway: transitive peering connection for VPC, VPN and DX

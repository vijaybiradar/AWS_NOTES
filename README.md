# AWS_NOTES

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

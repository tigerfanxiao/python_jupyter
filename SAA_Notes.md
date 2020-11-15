课程

Linux Academy 老课程， 实验多

https://linuxacademy.com/cp/modules/view/id/341

Linux Academy 新课程

https://linuxacademy.com/cp/modules/view/id/630

# Questions

1. 为什么说DynamoDB的strongly consistence 需要开发
2. 什么是 SQS 里的 receivemessageWatitimeSeconds
3. SQS本质是上起到什么作用, 在开源的框架中对应什么系统 kafka

# Glossary

### ACID

Atomicity, consistency, isolation, durability

Atomicity: A transaction is all or nothing. If any part of the transaction fails, the entire transaction fails

Consistency: Any transaction will bring a database from on valid state to another

Isolation: Ensure that concurrent transactions executed result in a stat of the system similar to if transaction were executed serially

Durability: Ensure that once a transaction is committed, it remains so, irrespective of a database crash, power or other errors.

### AWS premium support

* Basic: free

* Developer: 29$/month 12-24hours local business hours

* Business: 100$/month, 24x7 support, 1hour response

* Enterprise: 15,000$/mon, 15-minute response - TAM Techinal Account Manager

# EC2

### Reasons for EC2 terminated Abnomally

* The AMI is missing a required part
* The snapshot is corrupt
* You have reached your volume limit

# S3

S3 provide developers and IT teams with secure, durable, highly-scalable object storage. Amazon S3 is easy to use, with a simple web services interface to store and retrieve any amount of data from anywhere on the web

* S3 is object-based
* File can be from 0 to 5TB
* There is unlimited storage
* Files are stored in Buckets
* S3 is a universal namespace. this is , names must be unique globally

### MFA Delete

### 99.999999999% durability

Standard, Standard-Infrequent Access, One Zone-Infrequent Access

### Amazon Glacier

Amazon Glacier, which ebables long-term storage of mission-critcal data, has add Vault Lock. This new feature allows you to lock your vault with a variety of compliance controls that are designed to support such long-term records retention. 

### S3-Storage Class

1. S3 Standardd, availability 99.99%, durability 99.999999999%
2. S3-IA: infrequetly Accessed, availability 99.9%
3. S3 One Zone-IA, availability 99.5%
4. S3-Intelligent Tiering, AI
5. S3 Glacier, retrieval times from minutes to hours
6. S3 Glacier Deep Archive, retrieval time 12 hours

# Database

## DynamoDB

### Read data

* Scan
* Query

DynamoDB is using json to save data

- provisioning read and write capacity and the storage of data will be charged
- DynamoDB is distributed across three geographically distinct datacentres by default
- The ability to perform operations by using a user-defined primary key
- The primary key can either be a single-attribute or a composite
- DynamoDB supports *eventually consistent* and *strongly consistent* reads

**Eventually Consistent Reads**

When you read data from a DynamoDB table, the response might not reflect the results of a recently completed write operation. The response might include some stale data. If you repeat your read request after a short time, the response should return the latest data.

**Strongly Consistent Reads**

When you request a strongly consistent read, DynamoDB returns a response with the most up-to-date data, reflecting the updates from all prior write operations that were successful. However, this consistency comes with some disadvantages:

- A strongly consistent read might not be available if there is a network delay or outage. In this case, DynamoDB may return a server error (HTTP 500).
- Strongly consistent reads may have higher latency than eventually consistent reads.
- Strongly consistent reads are not supported on global secondary indexes.
- Strongly consistent reads use more throughput capacity than eventually consistent reads. 

# AWS AutoScaling

* Cross AZ, but not region

### Cooldown

if an Auto Scaling group is launching more than one instance, the cool down period for each instance starts after that instanc is launched.

### Step Sacling policies

* A lower bound
* An upper bound
* The adjustment type
* The amount by which to increase the desired capacity

### Target Scaling Policies

### Scheduled Scaling Policies

### Launch Configuration VS Launch Template

Launch Template is newer than Launch Configuration. Once you create a launch configuration, you cannot modify it. Launch Templates are also versioned, allow you to change them after creation. 

# Instance Store

An instance store provides temporary block level storage for you instance. 

# CloudWatch

AWS can see the instance, but not inside the instance to what it is doing. AWS can see that have Memory, but how much of the memory is being used cannot be seen by AWS. In the case of CPU AWS can see how much CPU are using, but cannot see what you are using it for. 

# Amazon Kinesis

Kinesis organizes records into shards that contain data along with critical meta information. If one shard hits a limit, the workload can be further distributed by subdividing it into multiple shards.
As it’s highly scalable and fully managed, Kinesis streams maintain near-real-time performance and, by extension, can have a significant impact on overall data processing performance  

## Kinesis Data Streams

## Kinesis Data Firehose

Key components of Kinesis Data Firehose are: delivery streams, records of data and destinations.

# AWS ECS

Amazon ECS contains the following components: 

A cluster is a logical grouping of container instances that you can place tasks on

A container instance is an Amazon EC2 instance that is running the Amazon ECS agent and has been registered into a cluster. 

A task definition is a descriptin of an application that contains one or more container definitions. 

A Scheduler is the method used for placing tasks on container instance

A Service is an Amazon ECS service that allows you to run and maintain a specified number of instances of a task definition simultaneously. 

# AWS WAF

AWS WAF is a web application firewall that lets you monitor the HTTP and HTTPS requests that are forwarded to Amazon CloudFront, and Applicaition Load Balancer or API Gateway.

AWS WAF helps protect your web application from common web exploits that could affect application availability, compromise security, or consume excessive resources.

You can configure conditions such as what IP addresses are allowed to make this request or what query string parameters need to be passed for the request to be allowed.

Then the application load balancer or CloudFront or API Gateway will either allow this content to be received or to give a HTTP 403 Status Code.

At it most basic level, AWS WAF allows 3 different behaviours. 

1. Allow all request except the ones you specify
2. Block all requests except the ones you specify
3. Count the requests that match the properties you specify

### conditions you could specify in WAF

* IP address that requests originate from 
* Country that requests orginiate from
* Values in request headers
* Strings that appear in requests, either specific strings or string that match regular expression patterns. 
* Length of requests
* Presence of SQL code that is likely to be malicious (know as SQL injuction)

# AWS EFS

When you need distributed, highly resilient stroage for Linux instance and Linux-based applicaitons.

A managed NAS tier for EC2 instance based on Network File System (NFS) version 4, only work for Unix and Linux.

# FSx

Amazon FSx for Windows File Server provides a fully managed native Microsoft Windows file system so you can easily move your Windows-based applications that require shared file storage to AWS.

### Difference between FSx and EFS

EFS is only for Unix and Linux, but FSx for windows can work for windows servers. 

### Amazon FSx for Windows

When you need centralised storage for windows-based applciations such as Sharepoint, Microsoft SQL Server, Workspaces. IIS Web Server or any other native Microsoft Application

### Amazon FSx for Lustre

When you need high-speed, high-capacity distributd storage. This will be for applications that do High Performance Compute (HPC), financial modelling etc.Remember that FSx for lustre can store data directly on S3.



# Amazon DataSync

* Used to move large amounts of data from on-premises to AWS
* Used with NFS- and SMB-compatible file system
* Replication can be done hourly, daily, or weekly
* Install the DataSync agent to start the replication
* Can be used to replicate EFS to EFS

# AWS Storage Gateway

AWS Storage Gateway provides software gateway appliances (based on VMware ESXi, Microsoft Hyper-V, or EC2 images) with multiple virtual connectivity interfaces.   

Storage Gateway is desiged to simplify backing up archives to the AWS cloud. it is not for shareing files

# AWS Elastic Beanstalk

With Elastic Beastalk, you can quickly deploy and manage applications in the AWS Cloud without worring about the infrastructure that runs those applications. You simply upload you application, and Elastic Beanstalk automatcially handles the details of capacity provising, load balancing, scaling, and application health monitoring.

Elastic Beanstalk provides platforms for programming languages (Go, Java, Node. js, PHP, Python, Ruby), but there are many more applications that can work on AWS.

# AWS DMS

The AWS Database Migrations Service is the best choice for conventional data migrations.

# AWS Secrets Manager

with Secrets Manager, you can deliver the most recent credentials to applications on request. The manager will even automatically take care of credential rotation  

# Security Pillar

Identities (including users, groups, and roles) can be authenticated
using a number of AWS services, including Cognito, Managed
Microsoft AD, and single sign-on. 

Authentication secrets are managed by services such as AWS Key Management Service (KMS), AWS Secrets Manager, and AWS CloudHSM  

### Service Control Policy

Service Control polices offer cnetral control over the maximum available permissions for all accounts in your organizations, allowing you to ensure your accoutns stay within your organization's access guidelines. 

# SQS

Simple Queue Service allows for event-driven messaging within distributed system that can decouple while coordinating the discrete steps of a larger process.

The Standard SQS message queue does not preserve message order nor guarantee messages are delivered only once - these are features of a FIFO message queue. Since Standard SQS message queues are designed to be massively scalable using a highly distributed architecture, receiving messages in the exact order they are sent is not guaranteed. Standard queues provide at-least-once delivery, and in some circumstances, duplicates can occur.



### Long Polling vs Short Polling

Amazon SQS provides short polling and long polling to receive messages from a queue. By default, queues use short polling.

Amazon **SQS long polling** is a way to retrieve messages from your Amazon **SQS** queues. While the regular **short polling** returns immediately, even if the message queue being polled is empty, **long polling** doesn't return a response until a message arrives in the message queue, or the **long poll** times out.

Short polling occurs when the `WaitTimeSeconds` parameter of a `ReceiveMessage` request is set to `0` in one of two ways:

- The `ReceiveMessage` call sets `WaitTimeSeconds` to `0`.
- The `ReceiveMessage` call doesn’t set `WaitTimeSeconds`, but the queue attribute [`ReceiveMessageWaitTimeSeconds`](https://docs.aws.amazon.com/AWSSimpleQueueService/latest/APIReference/API_SetQueueAttributes.html) is set to `0`.

# VPC

### CIDR

AWS will take first four IP address, and last one for broadcast

### Flow Log

flow logs can be set up for a VPC, subnet, or individual network interface. The data can be published to CloudWatch Logs or Amazon S3

# Route53

DNS record types

| Type  |                                             |      |
| ----- | ------------------------------------------- | ---- |
| A     | Define one hostname as an alias for another |      |
| CNAME | name server record                          |      |
| MX    | mail exchange record                        |      |
| NS    | name server record                          |      |
| SOA   | Start of authority record                   |      |



**FQDN** stands for fully qualified Domain Name

aws.amazon.com

aws = subdomain

amazon = SLD

com = TLD



# Glossary

### Geolocation VS Geoproximity

* Geolocation: geopolitical boundaries
* Geoproximity: longitude/latitude geographic areas

### API Gateway

API Gateway is used to generate custom client SDKs for your API to connect your backend  system to mobile, web and server application or services. 
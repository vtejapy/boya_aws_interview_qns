# AWS Interview Questions & Answers - Complete Guide

## Table of Contents
- [EC2 (Elastic Compute Cloud)](#ec2-elastic-compute-cloud)
- [S3 (Simple Storage Service)](#s3-simple-storage-service)
- [RDS (Relational Database Service)](#rds-relational-database-service)
- [VPC (Virtual Private Cloud)](#vpc-virtual-private-cloud)
- [Lambda (Serverless Computing)](#lambda-serverless-computing)
- [IAM (Identity and Access Management)](#iam-identity-and-access-management)
- [CloudWatch (Monitoring)](#cloudwatch-monitoring)
- [Auto Scaling](#auto-scaling)
- [Load Balancers](#load-balancers)
- [Route 53 (DNS)](#route-53-dns)
- [CloudFront (CDN)](#cloudfront-cdn)
- [DynamoDB](#dynamodb)
- [SNS & SQS (Messaging)](#sns--sqs-messaging)
- [CloudFormation](#cloudformation)
- [Elastic Container Service (ECS)](#elastic-container-service-ecs)
- [Elastic Kubernetes Service (EKS)](#elastic-kubernetes-service-eks)
- [API Gateway](#api-gateway)


---

## EC2 (Elastic Compute Cloud)

### Q1: What is EC2 and what are its key features?
**Answer:** EC2 is a web service that provides resizable compute capacity in the cloud. Key features include:
- **Virtual servers** called instances with various configurations
- **Elastic scaling** - scale up/down based on demand
- **Multiple instance types** optimized for different use cases
- **Amazon Machine Images (AMIs)** for quick instance launch
- **Flexible storage options** (EBS, instance store)
- **Security groups** for network-level security
- **Key pairs** for secure SSH access

### Q2: Explain the difference between stopping and terminating an EC2 instance.
**Answer:**
- **Stopping**: Instance shuts down but EBS root volume is preserved. Instance can be restarted later with same configuration
- **Terminating**: Instance is permanently destroyed along with instance store volumes. EBS root volumes are deleted by default (unless configured otherwise)

### Q3: What are the different EC2 instance types and their use cases?
**Answer:**
- **General Purpose (T3, M5)**: Balanced CPU, memory, networking - web servers, small databases
- **Compute Optimized (C5)**: High-performance processors - CPU-intensive applications
- **Memory Optimized (R5, X1)**: High memory-to-CPU ratio - in-memory databases, big data processing
- **Storage Optimized (I3, D2)**: High sequential read/write - distributed file systems, data warehouses
- **Accelerated Computing (P3, G4)**: GPU instances - machine learning, high-performance computing

### Q4: How do you secure EC2 instances?
**Answer:**
- Use **IAM roles** instead of storing credentials on instances
- Configure **security groups** as firewalls (stateful)
- Use **Network ACLs** for additional subnet-level security (stateless)
- Keep instances in **private subnets** when possible
- Regular **patch management** and updates
- **Encrypt EBS volumes** and snapshots
- Use **AWS Systems Manager** for secure access without SSH keys
- Enable **CloudTrail** for API logging

### Q5: What are Spot Instances and when would you use them?
**Answer:**
Spot Instances use spare EC2 capacity at up to 90% discount compared to On-Demand prices.
**Use cases:**
- Batch processing jobs
- Data analysis workloads
- Background processing
- Fault-tolerant applications
- Development/testing environments

**Best practices:**
- Use with Auto Scaling Groups for high availability
- Implement graceful shutdown handling
- Use multiple instance types and AZs

---

## S3 (Simple Storage Service)

### Q6: What is S3 and what are its storage classes?
**Answer:** S3 is object storage service with unlimited capacity. Storage classes include:
- **S3 Standard**: Frequently accessed data, low latency
- **S3 Standard-IA**: Infrequently accessed data, lower cost than Standard
- **S3 One Zone-IA**: Non-critical infrequently accessed data in single AZ
- **S3 Intelligent-Tiering**: Automatically moves data between tiers based on access patterns
- **S3 Glacier Instant Retrieval**: Archive data with millisecond access
- **S3 Glacier Flexible Retrieval**: Archive data with 1-12 hour retrieval
- **S3 Glacier Deep Archive**: Lowest cost for rarely accessed data, 12+ hour retrieval

### Q7: How do you secure S3 buckets?
**Answer:**
- **Block Public Access**: Enable by default to prevent accidental exposure
- **Bucket Policies**: JSON-based resource policies for fine-grained access control
- **IAM Policies**: User/role-based permissions
- **Access Control Lists (ACLs)**: Object and bucket level permissions
- **Server-Side Encryption**: SSE-S3, SSE-KMS, or SSE-C
- **Versioning**: Protect against accidental deletion/modification
- **MFA Delete**: Require MFA for permanent deletion
- **Access Logging**: Track requests for audit purposes

### Q8: Explain S3 Cross-Region Replication (CRR).
**Answer:**
CRR automatically replicates objects across different AWS regions.
**Requirements:**
- Versioning enabled on both source and destination buckets
- IAM role with permissions to replicate objects
- Different regions for source and destination

**Use cases:**
- Compliance requirements
- Disaster recovery
- Reduce latency for global users
- Data sovereignty requirements

### Q9: What is S3 Transfer Acceleration?
**Answer:**
Transfer Acceleration uses CloudFront edge locations to accelerate uploads to S3. Data is uploaded to the nearest edge location and then transferred to S3 over AWS's optimized network paths.

**Benefits:**
- Faster uploads from distant locations
- Consistent transfer speeds
- No changes to application code required

---

## RDS (Relational Database Service)

### Q10: What is RDS and what database engines does it support?
**Answer:** RDS is a managed relational database service that supports:
- **MySQL**
- **PostgreSQL**
- **MariaDB**
- **Oracle Database**
- **Microsoft SQL Server**
- **Amazon Aurora** (MySQL and PostgreSQL compatible)

**Key features:**
- Automated backups and snapshots
- Multi-AZ deployments for high availability
- Read replicas for scaling
- Automated patching
- Monitoring and metrics

### Q11: Explain RDS Multi-AZ vs Read Replicas.
**Answer:**
**Multi-AZ Deployment:**
- **Synchronous replication** to standby instance in different AZ
- **Automatic failover** during planned maintenance or outages
- **Same endpoint** - no application changes needed
- **Higher availability** with minimal downtime
- **Higher cost** due to standby instance

**Read Replicas:**
- **Asynchronous replication** to read-only copies
- **Scale read operations** across multiple instances
- **Different endpoints** for each replica
- **Can be in same region or cross-region**
- **Lower cost** - pay only for additional instances

### Q12: How do you backup and restore RDS databases?
**Answer:**
**Automated Backups:**
- Enabled by default with 7-day retention (configurable up to 35 days)
- Full daily snapshots + transaction logs
- Point-in-time recovery within retention period

**Manual Snapshots:**
- User-initiated snapshots
- Retained until manually deleted
- Can be shared across accounts
- Can be copied to other regions

**Restore Process:**
- Creates new RDS instance from backup
- Cannot restore over existing instance
- Can restore to any point within backup retention period

---

## VPC (Virtual Private Cloud)

### Q13: What is VPC and its main components?
**Answer:** VPC is a virtual network dedicated to your AWS account.

**Main Components:**
- **Subnets**: Segments of VPC IP address range
- **Route Tables**: Control traffic routing between subnets
- **Internet Gateway**: Connects VPC to internet
- **NAT Gateway/Instance**: Allows outbound internet access for private subnets
- **Security Groups**: Instance-level firewalls (stateful)
- **Network ACLs**: Subnet-level firewalls (stateless)
- **VPC Endpoints**: Private connectivity to AWS services

### Q14: Difference between Security Groups and NACLs?
**Answer:**
**Security Groups:**
- **Instance level** firewall
- **Stateful** - return traffic automatically allowed
- **Allow rules only** - everything denied by default
- **Evaluate all rules** before allowing traffic
- **Apply to instances** when launched

**Network ACLs:**
- **Subnet level** firewall
- **Stateless** - return traffic must be explicitly allowed
- **Allow and deny rules** supported
- **Process rules in order** (lowest number first)
- **Apply to all instances** in associated subnets

### Q15: How do you connect VPCs together?
**Answer:**
**VPC Peering:**
- One-to-one connection between VPCs
- Same or different regions
- No transitive routing
- Non-overlapping CIDR blocks required

**Transit Gateway:**
- Central hub connecting multiple VPCs
- Supports thousands of connections
- Transitive routing supported
- Cross-region peering available

**VPN Connection:**
- Encrypted tunnel over internet
- Connect on-premises to VPC
- Virtual Private Gateway on AWS side

**Direct Connect:**
- Dedicated network connection to AWS
- Consistent network performance
- Private connectivity to VPC

---

## Lambda (Serverless Computing)

### Q16: What is AWS Lambda and its key characteristics?
**Answer:** Lambda is a serverless compute service that runs code in response to events.

**Key Characteristics:**
- **No server management** required
- **Automatic scaling** from zero to thousands of concurrent executions
- **Pay per request** and compute time
- **Event-driven** execution model
- **Supports multiple languages** (Python, Node.js, Java, Go, .NET, Ruby)
- **Maximum execution time** of 15 minutes
- **Built-in monitoring** with CloudWatch

### Q17: How do you optimize Lambda performance?
**Answer:**
**Memory Allocation:**
- More memory = more CPU power
- Test different memory settings for optimal price/performance

**Cold Start Optimization:**
- Use **Provisioned Concurrency** for consistent performance
- Initialize connections outside handler function
- Minimize deployment package size
- Use Lambda layers for common dependencies

**Code Optimization:**
- Reuse database connections
- Use connection pooling
- Implement proper error handling
- Use environment variables for configuration

### Q18: What are Lambda layers and their benefits?
**Answer:** Lambda layers allow sharing code and dependencies across multiple functions.

**Benefits:**
- **Code reuse** across multiple functions
- **Smaller deployment packages**
- **Easier dependency management**
- **Version control** for shared components
- **Separation of concerns** between business logic and dependencies

**Use Cases:**
- Common libraries and dependencies
- Runtime environments
- Configuration and utilities
- Third-party integrations

---

## IAM (Identity and Access Management)

### Q19: What is IAM and its core components?
**Answer:** IAM manages access to AWS services and resources securely.

**Core Components:**
- **Users**: Individual people or applications
- **Groups**: Collections of users with common permissions
- **Roles**: Temporary credentials for AWS services or cross-account access
- **Policies**: JSON documents defining permissions
- **Identity Providers**: External identity systems (SAML, OIDC)

### Q20: Explain IAM policy types and evaluation logic.
**Answer:**
**Policy Types:**
- **Identity-based policies**: Attached to users, groups, or roles
- **Resource-based policies**: Attached to resources (S3 buckets, KMS keys)
- **Permission boundaries**: Set maximum permissions for identity-based policies
- **Service Control Policies (SCPs)**: Applied at organization/account level

**Evaluation Logic:**
1. **Explicit Deny** always wins
2. **SCPs** set maximum permissions
3. **Resource-based policies** can grant access
4. **Identity-based policies** within permission boundaries
5. **Default Deny** if no explicit allow

### Q21: IAM best practices?
**Answer:**
- **Principle of least privilege** - grant minimum required permissions
- **Use roles instead of users** for applications and services
- **Enable MFA** for all users, especially privileged accounts
- **Rotate credentials regularly**
- **Use IAM Access Analyzer** to review and refine permissions
- **Implement permission boundaries** for maximum security
- **Regular access reviews** to remove unused permissions
- **Use AWS managed policies** when possible
- **Don't embed credentials** in code

---

## CloudWatch (Monitoring)

### Q22: What is CloudWatch and its main features?
**Answer:** CloudWatch is a monitoring and observability service.

**Main Features:**
- **Metrics**: Collect and track metrics from AWS services and applications
- **Logs**: Centralized log management and analysis
- **Alarms**: Automated notifications based on metric thresholds
- **Events/EventBridge**: React to system events and schedule tasks
- **Dashboards**: Visualize metrics and operational data
- **Insights**: Query and analyze log data
- **X-Ray**: Distributed tracing for applications

### Q23: How do you create effective CloudWatch alarms?
**Answer:**
**Alarm Configuration:**
- Choose appropriate **metrics and thresholds**
- Set **evaluation periods** and **datapoints to alarm**
- Configure **comparison operators** (greater than, less than, etc.)
- Handle **missing data** appropriately

**Notification Setup:**
- Use **SNS topics** for multi-channel notifications
- Implement **escalation procedures** for critical alarms
- Configure **auto-scaling actions** for infrastructure alarms
- Set up **Lambda functions** for automated remediation

**Best Practices:**
- Start with AWS recommended alarms
- Monitor both infrastructure and application metrics
- Use composite alarms for complex conditions
- Regularly review and tune alarm thresholds

### Q24: Explain CloudWatch Logs and log analysis.
**Answer:**
**CloudWatch Logs Features:**
- **Log Groups**: Organize logs by application or service
- **Log Streams**: Individual log files within groups
- **Retention policies**: Control log storage duration
- **Metric filters**: Extract metrics from log data
- **Subscription filters**: Stream logs to other services

**Log Analysis:**
- **CloudWatch Insights**: SQL-like queries for log analysis
- **Real-time monitoring** with log streams
- **Export to S3** for long-term storage and analysis
- **Integration with ElasticSearch** for advanced search capabilities

---

## Auto Scaling

### Q25: What is Auto Scaling and its types?
**Answer:** Auto Scaling automatically adjusts capacity to maintain performance and minimize costs.

**Types:**
- **EC2 Auto Scaling**: Scale EC2 instances in Auto Scaling Groups
- **Application Auto Scaling**: Scale other AWS services (ECS, DynamoDB, Lambda)
- **AWS Auto Scaling**: Unified scaling across multiple services

**Components:**
- **Launch Template/Configuration**: Defines instance specifications
- **Auto Scaling Group**: Manages collection of instances
- **Scaling Policies**: Define when and how to scale
- **Health Checks**: Monitor instance health

### Q26: Explain different scaling policies.
**Answer:**
**Target Tracking Scaling:**
- Maintain specific metric target (e.g., 70% CPU utilization)
- Automatically creates CloudWatch alarms
- Simplest and most commonly used

**Step Scaling:**
- Scale by different amounts based on alarm breach size
- More granular control than simple scaling
- Supports multiple scaling steps

**Simple Scaling:**
- Single scaling action when alarm triggers
- Cooldown period prevents rapid scaling
- Less sophisticated than other methods

**Scheduled Scaling:**
- Scale based on predictable time patterns
- Useful for known traffic patterns
- Can be combined with other scaling types

### Q27: Auto Scaling best practices?
**Answer:**
- **Use multiple AZs** for high availability
- **Configure appropriate health checks** (EC2 and ELB)
- **Set up proper monitoring** and alerting
- **Use lifecycle hooks** for custom initialization
- **Implement warm-up periods** for new instances
- **Right-size instances** before scaling
- **Test scaling policies** under load
- **Use launch templates** instead of launch configurations

---

## Load Balancers

### Q28: What are the types of load balancers in AWS?
**Answer:**
**Application Load Balancer (ALB):**
- **Layer 7** (HTTP/HTTPS) load balancing
- **Advanced routing** based on path, host, headers
- **WebSocket and HTTP/2 support**
- **Integration with WAF** for security
- **SSL termination** and certificates

**Network Load Balancer (NLB):**
- **Layer 4** (TCP/UDP) load balancing
- **Ultra-low latency** and high throughput
- **Static IP addresses** and Elastic IP support
- **Preserve source IP** address
- **Handle millions of requests** per second

**Classic Load Balancer (CLB):**
- **Legacy** load balancer (not recommended for new applications)
- **Basic Layer 4 and Layer 7** features
- **Being phased out** in favor of ALB and NLB

### Q29: How do you configure health checks for load balancers?
**Answer:**
**Health Check Parameters:**
- **Protocol**: HTTP, HTTPS, TCP, SSL
- **Port**: Target port for health checks
- **Path**: HTTP path to check (for HTTP/HTTPS)
- **Interval**: Time between health checks (15-300 seconds)
- **Timeout**: Time to wait for response (2-120 seconds)
- **Healthy threshold**: Consecutive successful checks for healthy status
- **Unhealthy threshold**: Consecutive failed checks for unhealthy status

**Best Practices:**
- Use lightweight health check endpoints
- Check actual application functionality
- Monitor health check CloudWatch metrics
- Use appropriate timeouts for your application

### Q30: When would you use each type of load balancer?
**Answer:**
**Use ALB when:**
- HTTP/HTTPS applications
- Microservices architecture
- Container-based applications
- Need advanced routing features
- SSL termination required

**Use NLB when:**
- TCP/UDP applications
- Ultra-low latency requirements
- High throughput needs
- Gaming applications
- IoT applications
- Need static IP addresses

**Avoid CLB for:**
- New applications (use ALB or NLB instead)
- Advanced routing requirements
- Modern application architectures

---

## Route 53 (DNS)

### Q31: What is Route 53 and its routing policies?
**Answer:** Route 53 is AWS's DNS web service with various routing policies.

**Routing Policies:**
- **Simple**: Route to single resource
- **Weighted**: Distribute traffic based on assigned weights
- **Latency-based**: Route to lowest latency endpoint
- **Failover**: Active-passive failover setup
- **Geolocation**: Route based on user's geographic location
- **Geoproximity**: Route based on location with bias adjustment
- **Multivalue**: Return multiple healthy IP addresses

### Q32: How do health checks work in Route 53?
**Answer:**
**Health Check Types:**
- **HTTP/HTTPS**: Check specific web pages
- **TCP**: Check if port is reachable
- **Calculated**: Combine multiple health checks with boolean logic

**Health Check Features:**
- **Global network** of health checkers
- **Customizable intervals** and failure thresholds
- **SNS notifications** for health check failures
- **CloudWatch metrics** integration
- **String matching** for HTTP/HTTPS checks

**Failover Configuration:**
- Primary and secondary resources
- Automatic DNS updates based on health
- Works with all routing policies except simple

### Q33: How do you implement DNS failover with Route 53?
**Answer:**
**Setup Process:**
1. Create health checks for primary and secondary resources
2. Create Route 53 records with failover routing policy
3. Set primary record as "Primary" and backup as "Secondary"
4. Configure health check associations

**Best Practices:**
- Use health checks that test actual application functionality
- Set appropriate failure thresholds
- Monitor health check metrics in CloudWatch
- Test failover scenarios regularly
- Consider using latency-based routing for global applications

---

## CloudFront (CDN)

### Q34: What is CloudFront and its benefits?
**Answer:** CloudFront is AWS's Content Delivery Network (CDN) service.

**Benefits:**
- **Global edge locations** for low latency content delivery
- **Caching** static and dynamic content
- **DDoS protection** with AWS Shield
- **SSL/TLS encryption** with custom certificates
- **Integration** with other AWS services
- **Real-time metrics** and logging
- **Cost reduction** by reducing origin server load

### Q35: How do you configure CloudFront caching?
**Answer:**
**Cache Behaviors:**
- **Path patterns** to define which content gets cached
- **TTL settings** (minimum, maximum, default)
- **Query string and header** forwarding configuration
- **Compression** settings for text-based content

**Caching Strategies:**
- **Static content**: Long TTL (days/weeks)
- **Dynamic content**: Short TTL (minutes/hours) or no caching
- **API responses**: Based on cacheable endpoints
- **User-specific content**: Use signed URLs/cookies

**Cache Invalidation:**
- **Invalidate objects** when content changes
- **Versioning strategy** to avoid invalidation costs
- **Wildcard invalidations** for bulk updates

### Q36: What are CloudFront security features?
**Answer:**
**Security Features:**
- **Origin Access Control (OAC)**: Restrict S3 bucket access to CloudFront only
- **Signed URLs/Cookies**: Control access to private content
- **AWS WAF integration**: Protection against web attacks
- **SSL/TLS encryption**: End-to-end encryption support
- **Geographic restrictions**: Block content by country
- **Custom headers**: Add authentication headers to origin requests

**Security Best Practices:**
- Use OAC instead of OAI for S3 origins
- Implement signed URLs for sensitive content
- Configure WAF rules for common attacks
- Use custom SSL certificates for branded domains
- Monitor security metrics and logs

---

## DynamoDB

### Q37: What is DynamoDB and its key features?
**Answer:** DynamoDB is a fully managed NoSQL database service.

**Key Features:**
- **Serverless** with automatic scaling
- **Single-digit millisecond** latency
- **Global tables** for multi-region replication
- **Point-in-time recovery** and on-demand backups
- **DynamoDB Streams** for change data capture
- **Fine-grained access control** with IAM
- **Built-in security** with encryption at rest and in transit

### Q38: Explain DynamoDB primary keys and partition key design.
**Answer:**
**Primary Key Types:**
- **Partition Key**: Single attribute that uniquely identifies items
- **Composite Key**: Partition key + sort key combination

**Partition Key Design Principles:**
- **High cardinality**: Many distinct values
- **Uniform access**: Avoid hot partitions
- **Scalable**: Support future growth patterns

**Anti-patterns:**
- Sequential keys (timestamps, auto-incrementing IDs)
- Low cardinality keys (status, type fields)
- Celebrity problem (few items accessed frequently)

**Best Practices:**
- Use composite keys for better distribution
- Add random prefix/suffix to sequential keys
- Monitor CloudWatch metrics for throttling

### Q39: What's the difference between DynamoDB on-demand and provisioned capacity?
**Answer:**
**On-Demand Mode:**
- **Pay per request** pricing model
- **Automatic scaling** up and down
- **No capacity planning** required
- **Higher cost** per request
- **Good for**: Unpredictable workloads, new applications, serverless

**Provisioned Mode:**
- **Pre-specified** read/write capacity units
- **Lower cost** per request with reserved capacity
- **Auto Scaling** available to adjust capacity
- **Predictable billing**
- **Good for**: Predictable workloads, cost optimization

**When to Choose:**
- Start with on-demand for flexibility
- Move to provisioned when traffic patterns become predictable
- Use reserved capacity for additional cost savings

---

## SNS & SQS (Messaging)

### Q40: What's the difference between SNS and SQS?
**Answer:**
**SNS (Simple Notification Service):**
- **Pub/Sub messaging** pattern
- **Push-based** delivery to multiple subscribers
- **Fan-out** scenarios (one message to many receivers)
- **Real-time** message delivery
- **Supported endpoints**: Email, SMS, HTTP, Lambda, SQS

**SQS (Simple Queue Service):**
- **Point-to-point** messaging pattern
- **Pull-based** consumption by consumers
- **Queue-based** message storage
- **Decoupling** between producers and consumers
- **Message retention** up to 14 days

### Q41: How do you implement reliable message processing?
**Answer:**
**Message Durability:**
- SQS **stores messages redundantly** across multiple AZ
- Configure appropriate **message retention** period
- Use **dead letter queues** for failed messages

**Error Handling:**
- Implement **exponential backoff** for retries
- Set **visibility timeout** based on processing time
- Use **message deduplication** for FIFO queues
- Handle **poison messages** with dead letter queues

**Scaling Patterns:**
- Use **long polling** to reduce empty receives
- Implement **batch processing** for efficiency
- Scale consumers based on **queue depth metrics**
- Use **SQS Extended Client** for large messages

### Q42: Explain SQS FIFO queues vs Standard queues.
**Answer:**
**Standard Queues:**
- **At-least-once delivery** (may receive duplicates)
- **Best-effort ordering** (messages may arrive out of order)
- **Unlimited throughput**
- **Lower cost**

**FIFO Queues:**
- **Exactly-once delivery** (no duplicates)
- **Strict ordering** within message groups
- **Limited throughput** (3,000 messages/second with batching)
- **Higher cost**
- **Message deduplication** and **content-based deduplication**

**When to Use FIFO:**
- Order of messages is critical
- Duplicate processing would cause issues
- Banking transactions, order processing

---

## CloudFormation

### Q43: What is CloudFormation and its benefits?
**Answer:** CloudFormation is Infrastructure as Code (IaC) service for AWS resources.

**Benefits:**
- **Version control** for infrastructure
- **Repeatable deployments** across environments
- **Rollback capabilities** for failed deployments
- **Template reusability** and parameterization
- **Dependency management** between resources
- **Cost tracking** by stack
- **Compliance** and governance

### Q44: Explain CloudFormation template structure.
**Answer:**
**Template Sections:**
```yaml
AWSTemplateFormatVersion: '2010-09-09'
Description: 'Template description'
Metadata: # Additional information
Parameters: # Input values
Mappings: # Static lookup tables
Conditions: # Conditional resource creation
Transform: # Macros and transforms
Resources: # AWS resources (required)
Outputs: # Return values
```

**Key Concepts:**
- **Intrinsic functions**: Ref, GetAtt, Join, Sub, etc.
- **Pseudo parameters**: AWS::Region, AWS::AccountId
- **Cross-stack references**: Export/ImportValue
- **Custom resources**: Lambda-backed resources

### Q45: CloudFormation best practices?
**Answer:**
**Template Organization:**
- Use **nested stacks** for modularity
- Implement **cross-stack references** for shared resources
- **Parameterize** templates for flexibility
- Use **mappings** for environment-specific values

**Deployment Strategies:**
- Use **change sets** to preview changes
- Implement **stack policies** to protect critical resources
- Use **stack sets** for multi-account deployments
- Configure **rollback triggers** for automatic rollback

**Security:**
- Use **IAM roles** for CloudFormation execution
- Apply **least privilege** principles
- Use **AWS Secrets Manager** for sensitive data
- Implement **resource-level permissions**

---

## Elastic Container Service (ECS)

### Q46: What is ECS and its launch types?
**Answer:** ECS is a container orchestration service for Docker containers.

**Launch Types:**
**Fargate:**
- **Serverless** container hosting
- **No EC2 instance management**
- **Pay for vCPU and memory** used
- **Automatic scaling** and patching
- **Higher cost** per container

**EC2:**
- **Run containers on EC2 instances**
- **More control** over underlying infrastructure
- **Lower cost** with reserved instances
- **Manual cluster management**
- **Custom AMIs** and instance types

### Q47: How do you design ECS services for high availability?
**Answer:**
**Multi-AZ Deployment:**
- Deploy tasks across **multiple availability zones**
- Use **Application Load Balancer** for traffic distribution
- Configure **target group health checks**

**Auto Scaling:**
- **Service auto scaling** based on metrics (CPU, memory, ALB requests)
- **Cluster auto scaling** for EC2 launch type
- **Target tracking** and **step scaling** policies

**Service Discovery:**
- **ECS Service Discovery** with Route 53
- **Private DNS** for service-to-service communication
- **Load balancer integration** for external access

### Q48: ECS security best practices?
**Answer:**
**Task Security:**
- Use **IAM roles for tasks** instead of EC2 instance roles
- Implement **least privilege** access
- Use **AWS Secrets Manager** for sensitive data
- **Scan container images** for vulnerabilities

**Network Security:**
- Use **private subnets** for containers
- Configure **security groups** appropriately
- Implement **VPC endpoints** for AWS service access
- Use **AWS PrivateLink** for third-party services

**Runtime Security:**
- Use **read-only root filesystems** when possible
- **Run containers as non-root** users
- Implement **resource limits** (CPU, memory)
- Enable **logging** to CloudWatch

---

## Elastic Kubernetes Service (EKS)

### Q49: What is EKS and when should you use it?
**Answer:** EKS is a managed Kubernetes service that runs Kubernetes control plane.

**When to Use EKS:**
- **Kubernetes expertise** in the team
- **Multi-cloud** or hybrid deployments
- **Complex orchestration** requirements
- **Rich ecosystem** of Kubernetes tools
- **Microservices** architecture

**EKS vs ECS:**
- **EKS**: More complex, Kubernetes-native, portable
- **ECS**: Simpler, AWS-native, easier to learn

### Q50: How do you secure EKS clusters?
**Answer:**
**Cluster Security:**
- **Private endpoint** access for API server
- **IAM authentication** with RBAC
- **Network policies** for pod-to-pod communication
- **Pod security standards** and policies

**Node Security:**
- **Managed node groups** for automatic updates
- **Custom AMIs** with security hardening
- **Instance metadata service v2** (IMDSv2)
- **Container runtime security**

**Workload Security:**
- **IAM roles for service accounts** (IRSA)
- **Kubernetes secrets** management
- **Image scanning** with ECR
- **Network segmentation** with security groups

---

## API Gateway

### Q51: What is API Gateway and its types?
**Answer:** API Gateway is a managed service for creating, publishing, and managing APIs.

**API Types:**
**REST API:**
- **Full-featured** API management
- **Custom domain names** and SSL certificates
- **API keys and usage plans**
- **Request/response transformations**
- **Caching and throttling**

**HTTP API:**
- **Lower cost** and **higher performance**
- **Simpler** feature set
- **JWT authorization** built-in
- **CORS support**

**WebSocket API:**
- **Real-time** bidirectional communication
- **Connection management**
- **Route-based** message handling

### Q52: How do you implement API Gateway security?
**Answer:**
**Authentication Methods:**
- **IAM authentication** with Signature Version 4
- **Cognito User Pools** for user authentication
- **Lambda authorizers** for custom authentication
- **API Keys** for simple access control

**Authorization:**
- **Resource-based policies** for fine-grained access
- **CORS configuration** for web applications
- **Throttling** to prevent abuse

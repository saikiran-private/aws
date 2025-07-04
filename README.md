# AWS Cloud Computing Course Syllabus

## Course Overview
This 16-week course provides hands-on experience with AWS cloud services, building from fundamental concepts to advanced architectures. Students will develop a real-world sentiment analysis application while learning AWS best practices, security, and operational excellence.

**Prerequisites:** Basic computer networking knowledge, familiarity with command line interfaces, and fundamental programming concepts.

**Course Goals:**
- Master core AWS services and their integration
- Build a production-ready cloud application
- Understand AWS security and compliance best practices
- Develop operational skills for cloud environments
- Learn Infrastructure as Code (IaC) principles

---

## Week 1: Kick-off & Guard-rails
**Duration:** 2 hours  
**Core Topics:** Course rules, budgets, billing alarms, MFA, shared responsibility model

### Learning Objectives
- Understand AWS shared responsibility model
- Set up billing alarms and cost monitoring
- Implement multi-factor authentication (MFA)
- Establish course infrastructure and guardrails

### Detailed Content

#### AWS Shared Responsibility Model
- **AWS Responsibility:** Security OF the cloud
  - Hardware and global infrastructure
  - Regions, Availability Zones, Edge locations
  - Compute, storage, database, networking infrastructure
  - Managed services (RDS, Lambda, etc.)

- **Customer Responsibility:** Security IN the cloud
  - Operating system configuration and updates
  - Application security and data encryption
  - Network and firewall configuration
  - Identity and access management (IAM)

#### Cost Management & Billing
- **AWS Pricing Models:**
  - On-demand: Pay for what you use
  - Reserved Instances: 1-3 year commitments for discounts
  - Spot Instances: Use spare capacity at up to 90% discount
  - Savings Plans: Flexible pricing for compute usage

- **Cost Optimization Strategies:**
  - Right-sizing instances
  - Scheduling non-production resources
  - Using appropriate storage classes
  - Monitoring and alerting on spending

#### Lab: Create Cost & Usage Alarm (30 minutes)
**Objective:** Set up proactive cost monitoring

**Steps:**
1. Navigate to AWS Billing Dashboard
2. Create a billing alarm with $50 threshold
3. Set up email notifications
4. Configure cost allocation tags
5. Review cost explorer and budget reports

**Teaching Notes:**
- Emphasize the importance of cost monitoring from day one
- Show students how to set up multiple alarm thresholds
- Discuss cost allocation strategies for organizations

---

## Week 2: EC2 Refresher
**Duration:** 2 hours  
**Core Topics:** EC2 families, pricing, user data, AMI basics

### Learning Objectives
- Understand EC2 instance families and use cases
- Launch instances with user data scripts
- Create and manage AMIs
- Implement basic instance configuration

### Detailed Content

#### EC2 Instance Families
- **General Purpose (M-series):** Balanced compute, memory, and networking
  - Use cases: Web servers, small databases, development environments
  - Examples: m5.large, m6g.xlarge (Graviton2)

- **Compute Optimized (C-series):** High-performance processors
  - Use cases: Batch processing, gaming servers, high-traffic web servers
  - Examples: c5.2xlarge, c6g.xlarge

- **Memory Optimized (R-series):** High memory-to-vCPU ratio
  - Use cases: Large databases, in-memory caching, real-time big data analytics
  - Examples: r5.xlarge, r6g.2xlarge

- **Storage Optimized (I/D-series):** High storage I/O
  - Use cases: Data warehousing, distributed file systems, high-frequency trading
  - Examples: i3.xlarge, d2.xlarge

#### User Data and Instance Metadata
- **User Data:** Scripts that run when instance launches
  - Bash scripts for Linux instances
  - PowerShell scripts for Windows instances
  - Base64 encoding requirements
  - 16KB size limit

- **Instance Metadata Service (IMDS):**
  - Access instance metadata at http://169.254.169.254/latest/meta-data/
  - Instance ID, instance type, security groups, etc.
  - IMDSv2 for enhanced security

#### Lab: Launch t2.micro with Nginx (30 minutes)
**Objective:** Deploy a web server using user data

**Steps:**
1. Launch t2.micro instance in default VPC
2. Configure security group for HTTP (port 80)
3. Add user data script to install and start Nginx
4. Verify web server is accessible
5. Document instance details for project

**User Data Script Example:**
```bash
#!/bin/bash
yum update -y
yum install -y nginx
systemctl start nginx
systemctl enable nginx
echo "<h1>Hello from AWS!</h1>" > /var/www/html/index.html
```

**Teaching Notes:**
- Explain why t2.micro is free tier eligible
- Discuss burstable performance concept
- Show how to monitor instance metrics
- Introduce CloudWatch for monitoring

---

## Week 3: IAM + CLI + SSM
**Duration:** 2 hours  
**Core Topics:** Users/groups, roles, policies, CLI profiles, Session Manager

### Learning Objectives
- Implement least privilege access with IAM
- Configure AWS CLI with multiple profiles
- Use Systems Manager Session Manager for secure access
- Remove SSH access for enhanced security

### Detailed Content

#### IAM Fundamentals
- **Users:** Long-term credentials for human access
  - Username and password for console access
  - Access keys for programmatic access
  - MFA requirement for sensitive operations

- **Groups:** Collections of users with shared permissions
  - Easier permission management
  - Best practice: Assign permissions to groups, not users
  - Common groups: Administrators, Developers, ReadOnly

- **Roles:** Temporary credentials for AWS services or users
  - No long-term credentials
  - Assumed by services or users as needed
  - Cross-account access capabilities

- **Policies:** JSON documents defining permissions
  - Effect: Allow or Deny
  - Action: Specific API calls
  - Resource: AWS resources the policy applies to
  - Condition: Optional constraints

#### AWS CLI Configuration
- **Profiles:** Named collections of settings
  - Default profile and custom profiles
  - Stored in ~/.aws/credentials and ~/.aws/config
  - Environment variables override profile settings

- **Best Practices:**
  - Use named profiles for different accounts/environments
  - Rotate access keys regularly
  - Use IAM roles when possible instead of access keys

#### Systems Manager Session Manager
- **Benefits over SSH:**
  - No inbound ports required
  - Centralized access control
  - Session logging and auditing
  - Works through AWS Systems Manager

- **Prerequisites:**
  - SSM Agent installed on instances
  - IAM role with SSM permissions
  - VPC endpoints or internet access for SSM

#### Lab: Secure EC2 Access (30 minutes)
**Objective:** Replace SSH with Session Manager

**Steps:**
1. Create IAM role for SSM access
2. Attach role to EC2 instance
3. Install SSM Agent (if needed)
4. Connect via Session Manager
5. Remove SSH security group rule
6. Test secure access

**Teaching Notes:**
- Emphasize security benefits of Session Manager
- Show how to configure session preferences
- Discuss port forwarding capabilities
- Explain IAM role attachment process

---

## Week 4: Storage for Compute
**Duration:** 2 hours  
**Core Topics:** EBS types & snapshots, EFS shared storage

### Learning Objectives
- Choose appropriate EBS volume types
- Create and manage EBS snapshots
- Implement EFS for shared file storage
- Mount EFS on multiple instances

### Detailed Content

#### EBS Volume Types
- **General Purpose SSD (gp2/gp3):**
  - Balanced price and performance
  - 3 IOPS per GB for gp2, 3000 baseline for gp3
  - Use cases: Boot volumes, development workloads

- **Provisioned IOPS SSD (io1/io2):**
  - High-performance, low-latency
  - Up to 64,000 IOPS per volume
  - Use cases: Critical business applications, databases

- **Throughput Optimized HDD (st1):**
  - Low-cost HDD for frequently accessed data
  - Cannot be boot volumes
  - Use cases: Big data, data warehouses, log processing

- **Cold HDD (sc1):**
  - Lowest cost HDD for infrequently accessed data
  - Cannot be boot volumes
  - Use cases: File servers, backup storage

#### EBS Snapshots
- **Features:**
  - Point-in-time backups
  - Incremental backups (only changed blocks)
  - Cross-region and cross-account copying
  - Encryption support

- **Best Practices:**
  - Regular snapshot schedules
  - Lifecycle policies for automatic cleanup
  - Cross-region backup for disaster recovery

#### Amazon EFS (Elastic File System)
- **Characteristics:**
  - Fully managed NFS file system
  - Automatic scaling
  - Multi-AZ availability
  - On-premises access via Direct Connect

- **Performance Modes:**
  - General Purpose: Latency-sensitive workloads
  - Max I/O: Higher throughput, higher latency

- **Throughput Modes:**
  - Bursting: Baseline with burst capability
  - Provisioned: Consistent performance

#### Lab: EFS Shared Storage (30 minutes)
**Objective:** Create shared file storage for multiple instances

**Steps:**
1. Create EFS file system
2. Configure mount targets in multiple subnets
3. Launch two EC2 instances
4. Install EFS client on both instances
5. Mount EFS on both instances
6. Upload and share poster images
7. Test file sharing between instances

**Teaching Notes:**
- Explain NFS protocol basics
- Show EFS mount options and performance tuning
- Discuss EFS pricing model
- Compare EFS vs EBS for different use cases

---

## Week 5: Custom VPC 101
**Duration:** 2 hours  
**Core Topics:** 2-AZ design, subnets, route tables, IGW, NAT GW, SG vs NACL, Flow Logs

### Learning Objectives
- Design a multi-AZ VPC architecture
- Configure subnets, route tables, and gateways
- Implement security groups and network ACLs
- Enable VPC Flow Logs for monitoring

### Detailed Content

#### VPC Design Principles
- **Availability Zone Distribution:**
  - Minimum 2 AZs for high availability
  - Public subnets in each AZ
  - Private subnets in each AZ
  - Database subnets in each AZ (optional)

- **CIDR Block Planning:**
  - Choose appropriate VPC CIDR (e.g., 10.0.0.0/16)
  - Subnet CIDR planning (e.g., 10.0.1.0/24, 10.0.2.0/24)
  - Avoid overlapping CIDR blocks

#### VPC Components
- **Internet Gateway (IGW):**
  - Provides internet access for public subnets
  - One IGW per VPC
  - Attached to VPC, not specific subnets

- **NAT Gateway:**
  - Allows private instances to access internet
  - One NAT Gateway per AZ
  - Requires Elastic IP address
  - Charges apply for data processing

- **Route Tables:**
  - Control traffic flow between subnets
  - Main route table (default)
  - Custom route tables for specific subnets

#### Security Groups vs Network ACLs
- **Security Groups (Stateful):**
  - Operate at instance level
  - Allow rules only (implicit deny)
  - Return traffic automatically allowed
  - Can reference other security groups

- **Network ACLs (Stateless):**
  - Operate at subnet level
  - Allow and deny rules
  - Explicit rules for return traffic
  - Numeric rule ordering

#### Lab: Build Custom VPC (30 minutes)
**Objective:** Create a production-ready VPC architecture

**Steps:**
1. Create VPC with 10.0.0.0/16 CIDR
2. Create 2 public subnets (10.0.1.0/24, 10.0.2.0/24)
3. Create 2 private subnets (10.0.3.0/24, 10.0.4.0/24)
4. Create Internet Gateway and attach to VPC
5. Create NAT Gateway in public subnet
6. Configure route tables for public and private subnets
7. Move existing EC2 instance to public subnet
8. Test internet connectivity

**Teaching Notes:**
- Explain subnet routing logic
- Show how to calculate subnet CIDR blocks
- Discuss NAT Gateway costs and alternatives
- Demonstrate VPC Flow Logs for traffic monitoring

---

## Week 6: Advanced VPC (Optional)
**Duration:** 2 hours  
**Core Topics:** VPC Endpoints, VPC peering, Transit Gateway

### Learning Objectives
- Implement VPC endpoints for private AWS service access
- Understand VPC peering and Transit Gateway
- Optimize network traffic and reduce costs
- Enhance security with private connectivity

### Detailed Content

#### VPC Endpoints
- **Gateway Endpoints:**
  - S3 and DynamoDB only
  - No additional charges
  - Route table configuration required
  - No security groups

- **Interface Endpoints:**
  - All AWS services
  - ENI in your VPC
  - Security group configuration
  - Hourly charges apply

- **Benefits:**
  - Private connectivity to AWS services
  - Reduced data transfer costs
  - Enhanced security (no internet traffic)

#### VPC Peering
- **Characteristics:**
  - Direct connection between VPCs
  - Same or different AWS accounts
  - Same or different regions
  - Non-transitive (A→B→C not possible)

- **Use Cases:**
  - Resource sharing between environments
  - Cross-account VPC connectivity
  - Regional VPC connectivity

#### Transit Gateway
- **Features:**
  - Central hub for VPC connectivity
  - Supports multiple VPCs and VPN connections
  - Transitive routing
  - Route table management

- **Use Cases:**
  - Large-scale VPC connectivity
  - Hub-and-spoke architecture
  - Multi-region connectivity

#### Lab: S3 Gateway Endpoint (30 minutes)
**Objective:** Enable private S3 access from VPC

**Steps:**
1. Create S3 Gateway Endpoint
2. Configure route table for private subnets
3. Test S3 access from private instances
4. Verify traffic stays within AWS network
5. Review cost implications

**Teaching Notes:**
- Explain endpoint vs internet routing differences
- Show VPC endpoint policy configuration
- Discuss when to use different endpoint types
- Demonstrate cost savings with private connectivity

---

## Week 7: Load Balancer & Auto Scaling
**Duration:** 2 hours  
**Core Topics:** ALB, target groups, ASG, golden AMI

### Learning Objectives
- Configure Application Load Balancer (ALB)
- Implement Auto Scaling Groups (ASG)
- Create launch templates and golden AMIs
- Build highly available front-end infrastructure

### Detailed Content

#### Application Load Balancer (ALB)
- **Features:**
  - Layer 7 load balancing
  - Path-based and host-based routing
  - SSL/TLS termination
  - Health checks and target group management

- **Target Groups:**
  - EC2 instances, IP addresses, Lambda functions
  - Health check configuration
  - Load balancing algorithms (round-robin, least connections)

- **Listeners:**
  - HTTP (port 80) and HTTPS (port 443)
  - Default actions and rules
  - SSL certificate management

#### Auto Scaling Groups
- **Scaling Policies:**
  - Target tracking (CPU, memory, custom metrics)
  - Simple scaling (scale up/down by fixed amount)
  - Step scaling (scale based on breach thresholds)

- **Configuration:**
  - Minimum, desired, and maximum capacity
  - Launch template or launch configuration
  - Health check grace period
  - Cooldown periods

#### Golden AMI Strategy
- **Benefits:**
  - Consistent instance configuration
  - Faster instance launch times
  - Reduced configuration drift
  - Security patch management

- **Creation Process:**
  - Launch base instance
  - Install and configure applications
  - Apply security patches
  - Create AMI from instance
  - Test AMI in staging environment

#### Lab: HA Front-end (30 minutes)
**Objective:** Deploy highly available web application

**Steps:**
1. Create golden AMI from existing web server
2. Create launch template with AMI
3. Configure Application Load Balancer
4. Create target group and health checks
5. Set up Auto Scaling Group
6. Test auto-scaling by terminating instances
7. Verify load balancing functionality

**Teaching Notes:**
- Explain health check configuration importance
- Show scaling policy configuration
- Discuss instance refresh strategies
- Demonstrate cross-zone load balancing

---

## Week 8: Observability & Alerts
**Duration:** 2 hours  
**Core Topics:** CloudWatch metrics/logs, SNS, ASG policies

### Learning Objectives
- Configure CloudWatch monitoring and logging
- Set up SNS notifications
- Create Auto Scaling policies
- Implement operational monitoring

### Detailed Content

#### CloudWatch Metrics
- **Built-in Metrics:**
  - EC2: CPU, network, disk I/O
  - RDS: CPU, connections, storage
  - ALB: request count, target response time
  - Custom metrics from applications

- **Metric Types:**
  - Standard resolution (1-minute)
  - High resolution (1-second)
  - Basic monitoring vs detailed monitoring

#### CloudWatch Logs
- **Features:**
  - Centralized log collection
  - Log groups and log streams
  - Metric filters for log analysis
  - Log retention policies

- **Integration:**
  - CloudWatch Logs agent
  - AWS SDK integration
  - Lambda function logging

#### Simple Notification Service (SNS)
- **Components:**
  - Topics: Communication channels
  - Subscriptions: Endpoints (email, SMS, HTTP/HTTPS)
  - Publishers: Sources of messages

- **Use Cases:**
  - Alert notifications
  - Application notifications
  - System event notifications

#### Lab: Monitoring & Alerting (30 minutes)
**Objective:** Set up comprehensive monitoring

**Steps:**
1. Configure CloudWatch agent on EC2 instances
2. Set up log streaming for Nginx logs
3. Create CloudWatch alarms for 5xx errors
4. Configure SNS topic and email subscription
5. Test alarm notifications
6. Create Auto Scaling policy based on CPU
7. Monitor scaling events

**Teaching Notes:**
- Explain metric aggregation and statistics
- Show log filtering and searching
- Discuss alarm state management
- Demonstrate dashboard creation

---

## Week 9: RDS & Secrets Manager
**Duration:** 2 hours  
**Core Topics:** RDS MySQL, Multi-AZ, backups, Secrets Manager

### Learning Objectives
- Deploy and configure RDS MySQL database
- Implement Multi-AZ deployment for high availability
- Configure automated backups and snapshots
- Store and retrieve database credentials securely

### Detailed Content

#### Amazon RDS
- **Supported Engines:**
  - MySQL, PostgreSQL, MariaDB
  - Oracle, SQL Server
  - Aurora (MySQL and PostgreSQL compatible)

- **Deployment Options:**
  - Single-AZ: Development and testing
  - Multi-AZ: Production workloads
  - Read Replicas: Read scaling

- **Instance Classes:**
  - Memory-optimized (R-series)
  - General purpose (M-series)
  - Burstable performance (T-series)

#### Multi-AZ Deployment
- **Benefits:**
  - Automatic failover
  - Synchronous replication
  - Enhanced availability
  - Maintenance window management

- **Failover Process:**
  - Automatic detection of primary failure
  - DNS endpoint update
  - Application reconnection

#### Backup and Recovery
- **Automated Backups:**
  - Daily backups during maintenance window
  - Point-in-time recovery
  - Configurable retention period (1-35 days)

- **Manual Snapshots:**
  - User-initiated backups
  - Indefinite retention
  - Cross-region copying

#### AWS Secrets Manager
- **Features:**
  - Automatic rotation of credentials
  - Encryption with KMS
  - Fine-grained access control
  - Integration with RDS

- **Supported Secret Types:**
  - RDS database credentials
  - API keys
  - SSH keys
  - Custom secrets

#### Lab: Database Deployment (30 minutes)
**Objective:** Deploy production-ready database

**Steps:**
1. Create RDS MySQL instance in private subnet
2. Configure Multi-AZ deployment
3. Set up security group for database access
4. Create Secrets Manager secret for database credentials
5. Test database connectivity from backend EC2
6. Configure automated backups
7. Test failover scenario

**Teaching Notes:**
- Explain database parameter groups
- Show connection pooling concepts
- Discuss backup and restore procedures
- Demonstrate secret rotation

---

## Week 10: Backend & Model Tiers
**Duration:** 2 hours  
**Core Topics:** Private-subnet ASGs, internal ALB, SG chaining

### Learning Objectives
- Deploy backend application tier in private subnets
- Configure internal load balancer
- Implement security group chaining
- Complete end-to-end application architecture

### Detailed Content

#### Multi-Tier Architecture
- **Front-end Tier:**
  - Public subnets
  - Internet-facing load balancer
  - Stateless application servers

- **Backend Tier:**
  - Private subnets
  - Internal load balancer
  - Application logic and business rules

- **Data Tier:**
  - Private subnets
  - Database servers
  - Data storage and persistence

#### Internal Load Balancer
- **Characteristics:**
  - No internet-facing endpoint
  - Internal DNS resolution
  - Same features as internet-facing ALB
  - Enhanced security

#### Security Group Chaining
- **Pattern:**
  - Front-end SG allows traffic from internet
  - Backend SG allows traffic from front-end SG
  - Database SG allows traffic from backend SG
  - No direct internet access to private tiers

#### Lab: Complete Architecture (30 minutes)
**Objective:** Deploy full three-tier application

**Steps:**
1. Deploy backend application in private subnets
2. Configure internal Application Load Balancer
3. Set up security group chaining
4. Deploy model tier (sentiment analysis service)
5. Configure end-to-end connectivity
6. Test sentiment analysis API calls
7. Monitor application performance

**Teaching Notes:**
- Explain microservices architecture concepts
- Show API gateway integration options
- Discuss service discovery patterns
- Demonstrate distributed tracing

---

## Week 11: Serverless & Queueing (Stand-alone)
**Duration:** 2 hours  
**Core Topics:** Lambda, SQS, event-driven architecture

### Learning Objectives
- Create and deploy Lambda functions
- Implement SQS queues for asynchronous processing
- Understand event-driven architecture patterns
- Build serverless workflows

### Detailed Content

#### AWS Lambda
- **Characteristics:**
  - Serverless compute service
  - Event-driven execution
  - Automatic scaling
  - Pay-per-request pricing

- **Supported Runtimes:**
  - Node.js, Python, Java, Go
  - .NET Core, Ruby, Custom runtimes
  - Container images

- **Triggers:**
  - API Gateway, S3, DynamoDB
  - CloudWatch Events, SQS
  - Custom events

#### Amazon SQS
- **Queue Types:**
  - Standard: At-least-once delivery, best-effort ordering
  - FIFO: Exactly-once processing, strict ordering

- **Features:**
  - Message retention (1-14 days)
  - Visibility timeout
  - Dead letter queues
  - Message attributes

#### Event-Driven Architecture
- **Benefits:**
  - Loose coupling between services
  - Scalability and resilience
  - Real-time processing
  - Cost optimization

#### Lab: Serverless Processing (30 minutes)
**Objective:** Build event-driven image processing

**Steps:**
1. Create S3 bucket for image uploads
2. Deploy Lambda function for thumbnail generation
3. Configure S3 trigger for Lambda
4. Set up SQS queue for processing status
5. Test end-to-end workflow
6. Monitor Lambda execution metrics
7. Review cost implications

**Teaching Notes:**
- Explain cold start considerations
- Show Lambda configuration options
- Discuss error handling and retries
- Demonstrate monitoring and debugging

---

## Week 12: S3 + KMS & Static Hosting
**Duration:** 2 hours  
**Core Topics:** S3 buckets, versioning, lifecycle, Glacier, SSE-KMS

### Learning Objectives
- Configure S3 buckets with appropriate settings
- Implement server-side encryption with KMS
- Set up static website hosting
- Migrate front-end to S3

### Detailed Content

#### Amazon S3
- **Bucket Configuration:**
  - Region selection
  - Block public access settings
  - Versioning and lifecycle policies
  - Object ownership

- **Storage Classes:**
  - S3 Standard: Frequently accessed data
  - S3 Intelligent-Tiering: Automatic cost optimization
  - S3 Standard-IA: Infrequently accessed data
  - S3 One Zone-IA: Single AZ storage
  - S3 Glacier: Long-term archival
  - S3 Glacier Deep Archive: Lowest cost storage

#### Server-Side Encryption
- **SSE-S3:** S3-managed keys
- **SSE-KMS:** AWS KMS-managed keys
- **SSE-C:** Customer-provided keys

#### Static Website Hosting
- **Features:**
  - Website endpoint (bucket-name.s3-website-region.amazonaws.com)
  - Index and error document configuration
  - Redirect rules
  - No server-side processing

#### Lab: S3 Static Hosting (30 minutes)
**Objective:** Migrate front-end to S3

**Steps:**
1. Create S3 bucket for static website
2. Configure bucket for static website hosting
3. Upload HTML, CSS, and JavaScript files
4. Set up bucket policy for public read access
5. Configure server-side encryption with KMS
6. Test website functionality
7. Update DNS to point to S3 endpoint

**Teaching Notes:**
- Explain S3 bucket naming requirements
- Show CORS configuration for API calls
- Discuss cost optimization strategies
- Demonstrate versioning and lifecycle policies

---

## Week 13: CDN, WAF & Route 53
**Duration:** 2 hours  
**Core Topics:** CloudFront, caching, WAF, Route 53, ACM

### Learning Objectives
- Configure CloudFront distribution for global content delivery
- Implement WAF rules for security
- Set up Route 53 for DNS management
- Secure with SSL/TLS certificates

### Detailed Content

#### Amazon CloudFront
- **Features:**
  - Global content delivery network
  - Edge locations worldwide
  - Caching and compression
  - Origin failover

- **Behaviors:**
  - Path-based routing
  - Cache key customization
  - TTL configuration
  - Origin access control

#### AWS WAF
- **Web ACL Components:**
  - Rules and rule groups
  - Rate limiting
  - IP reputation lists
  - Custom rules

- **Managed Rule Sets:**
  - AWS managed rules
  - Marketplace rule sets
  - Common attack protection

#### Route 53
- **Record Types:**
  - A: IPv4 address
  - AAAA: IPv6 address
  - CNAME: Canonical name
  - Alias: AWS resource records

- **Routing Policies:**
  - Simple: Single resource
  - Weighted: Traffic distribution
  - Latency-based: Performance optimization
  - Geolocation: Geographic routing

#### AWS Certificate Manager (ACM)
- **Features:**
  - Free SSL/TLS certificates
  - Automatic renewal
  - Regional certificates
  - Wildcard certificates

#### Lab: Global Edge Infrastructure (30 minutes)
**Objective:** Deploy global content delivery

**Steps:**
1. Create CloudFront distribution for S3 website
2. Configure caching behaviors
3. Set up WAF Web ACL with basic rules
4. Request SSL certificate in ACM
5. Create Route 53 hosted zone
6. Configure A records with CloudFront alias
7. Test global access and caching

**Teaching Notes:**
- Explain CDN caching concepts
- Show invalidation procedures
- Discuss WAF rule configuration
- Demonstrate DNS propagation

---

## Week 14: Governance (Optional)
**Duration:** 2 hours  
**Core Topics:** CloudTrail, AWS Config, compliance

### Learning Objectives
- Implement comprehensive audit logging
- Configure compliance monitoring
- Set up automated remediation
- Establish governance controls

### Detailed Content

#### AWS CloudTrail
- **Features:**
  - API call logging
  - Management and data events
  - Cross-region logging
  - CloudTrail Lake for analytics

- **Event Types:**
  - Management events: Account management
  - Data events: S3 object-level operations
  - Insight events: Unusual activity

#### AWS Config
- **Components:**
  - Configuration recorder
  - Configuration rules
  - Configuration history
  - Remediation actions

- **Managed Rules:**
  - Security best practices
  - Compliance frameworks
  - Cost optimization
  - Operational excellence

#### Compliance Frameworks
- **Standards:**
  - SOC 1/2/3
  - PCI DSS
  - HIPAA
  - ISO 27001

#### Lab: Compliance Monitoring (30 minutes)
**Objective:** Set up governance controls

**Steps:**
1. Enable CloudTrail in all regions
2. Configure CloudTrail Lake
3. Set up AWS Config recorder
4. Create Config rules for security compliance
5. Configure automated remediation
6. Test compliance monitoring
7. Review audit reports

**Teaching Notes:**
- Explain compliance requirements
- Show CloudTrail event analysis
- Discuss Config rule customization
- Demonstrate remediation workflows

---

## Week 15: CloudFormation 101 (Optional)
**Duration:** 2 hours  
**Core Topics:** Stacks, parameters, outputs, drift detection

### Learning Objectives
- Understand Infrastructure as Code principles
- Create CloudFormation templates
- Manage stack lifecycle
- Implement drift detection

### Detailed Content

#### Infrastructure as Code
- **Benefits:**
  - Version control for infrastructure
  - Consistent deployments
  - Automated provisioning
  - Disaster recovery

#### CloudFormation Templates
- **Template Structure:**
  - AWSTemplateFormatVersion
  - Description
  - Parameters
  - Resources
  - Outputs

- **Resource Types:**
  - AWS service resources
  - Custom resources
  - Nested stacks

#### Stack Management
- **Operations:**
  - Create: Initial deployment
  - Update: Modify existing resources
  - Delete: Remove all resources
  - Rollback: Revert failed updates

#### Lab: Single-EC2 Stack (30 minutes)
**Objective:** Create basic CloudFormation template

**Steps:**
1. Create CloudFormation template for EC2 instance
2. Add parameters for instance type and key pair
3. Configure security group and IAM role
4. Deploy stack via console
5. Test stack update with parameter change
6. Enable drift detection
7. Review stack events and outputs

**Teaching Notes:**
- Explain template validation
- Show parameter constraints
- Discuss stack dependencies
- Demonstrate change sets

---

## Week 16: Capstone IaC
**Duration:** 2 hours  
**Core Topics:** Nested stacks, change sets, deletion policies

### Learning Objectives
- Deploy complete application using CloudFormation
- Implement nested stack architecture
- Manage complex deployments
- Perform smoke testing and cleanup

### Detailed Content

#### Nested Stacks
- **Benefits:**
  - Modular template design
  - Reusable components
  - Easier maintenance
  - Team collaboration

#### Change Sets
- **Features:**
  - Preview changes before execution
  - Understand resource impact
  - Validate template changes
  - Rollback capability

#### Deletion Policies
- **Options:**
  - Delete: Remove resource
  - Retain: Keep resource
  - Snapshot: Create snapshot before deletion

#### Lab: Full Architecture Deployment (30 minutes)
**Objective:** Deploy complete application infrastructure

**Steps:**
1. Create nested stack templates
2. Deploy VPC and networking stack
3. Deploy compute and database stacks
4. Deploy application and monitoring stacks
5. Run smoke tests
6. Document deployment process
7. Execute complete stack deletion

**Teaching Notes:**
- Explain stack dependencies
- Show cross-stack references
- Discuss deployment strategies
- Demonstrate troubleshooting techniques

---

## Course Assessment

### Project Deliverables
1. **Week 2-4:** Front-end prototype on EC2
2. **Week 5-7:** High-availability front-end
3. **Week 8-10:** Complete three-tier application
4. **Week 11-13:** Global edge infrastructure
5. **Week 14-16:** Infrastructure as Code deployment

### Evaluation Criteria
- **Technical Implementation (60%):**
  - Service configuration and integration
  - Security best practices
  - Performance optimization
  - Operational excellence

- **Documentation (20%):**
  - Architecture diagrams
  - Deployment procedures
  - Troubleshooting guides
  - Cost analysis

- **Presentation (20%):**
  - Technical demonstration
  - Architecture explanation
  - Lessons learned
  - Future improvements

### Final Project Requirements
- Complete sentiment analysis application
- Production-ready architecture
- Comprehensive monitoring and alerting
- Infrastructure as Code deployment
- Security and compliance implementation
- Cost optimization analysis

---

## Additional Resources

### AWS Documentation
- [AWS Well-Architected Framework](https://aws.amazon.com/architecture/well-architected/)
- [AWS Security Best Practices](https://aws.amazon.com/security/security-learning/)
- [AWS Cost Optimization](https://aws.amazon.com/cost-optimization/)

### Tools and Services
- AWS CLI and SDKs
- CloudFormation and CDK
- AWS Systems Manager
- AWS Config and CloudTrail
- AWS Cost Explorer and Budgets

### Best Practices
- Security by design
- Cost optimization
- Operational excellence
- Performance efficiency
- Reliability and availability

---

## Course Schedule Summary

| Week | Topic | Hands-on Lab | Project Milestone |
|------|-------|--------------|-------------------|
| 1 | Kick-off & Guard-rails | Cost monitoring setup | — |
| 2 | EC2 Refresher | Web server deployment | Front-end prototype |
| 3 | IAM + CLI + SSM | Secure access configuration | Security hardening |
| 4 | Storage for Compute | EFS shared storage | File sharing implementation |
| 5 | Custom VPC 101 | VPC architecture | Networking foundation |
| 6 | Advanced VPC | VPC endpoints | Private connectivity |
| 7 | LB & Auto Scaling | HA front-end | Scalable architecture |
| 8 | Observability & Alerts | Monitoring setup | Operational visibility |
| 9 | RDS & Secrets Manager | Database deployment | Data persistence |
| 10 | Backend & Model Tiers | Complete architecture | Full application |
| 11 | Serverless & Queueing | Lambda + SQS | Event-driven processing |
| 12 | S3 + KMS & Static Hosting | S3 website migration | Static hosting |
| 13 | CDN, WAF & Route 53 | Global edge infrastructure | Worldwide access |
| 14 | Governance | Compliance monitoring | Security controls |
| 15 | CloudFormation 101 | IaC basics | Infrastructure automation |
| 16 | Capstone IaC | Full deployment | Complete project |

This syllabus provides a comprehensive foundation for teaching AWS cloud computing, with practical hands-on experience and real-world project development throughout the course.

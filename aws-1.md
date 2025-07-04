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

**Cost Monitoring Importance:**
- Start with cost monitoring on day one to establish good habits
- Explain that AWS charges are real money and can accumulate quickly
- Share horror stories of unexpected bills (students love these!)
- Demonstrate how small configuration mistakes can lead to large charges
- Show the AWS Pricing Calculator to help students understand costs upfront

**Multiple Alarm Thresholds:**
- Set up tiered alarms: $25 (warning), $50 (alert), $100 (critical)
- Explain that different thresholds serve different purposes
- Show how to configure different notification channels for each threshold
- Demonstrate alarm history and trend analysis

**Cost Allocation Strategies:**
- Use resource tags for cost tracking (Environment: Dev/Prod, Project: Course, Student: Name)
- Explain how tags help identify which resources belong to which student/project
- Show cost allocation reports and how to interpret them
- Discuss organizational cost management best practices

**Additional Discussion Points:**
- Explain the difference between on-demand, reserved, and spot pricing
- Show how to use AWS Cost Explorer for historical analysis
- Demonstrate budget alerts vs. billing alarms
- Discuss the importance of regular cost reviews

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

**Free Tier Eligibility:**
- t2.micro is free tier eligible for 12 months (750 hours/month)
- Explain that this is perfect for learning and development
- Show students how to check their free tier usage in the AWS console
- Warn about exceeding free tier limits and unexpected charges
- Demonstrate how to set up billing alerts specifically for free tier usage

**Burstable Performance Concept:**
- Explain CPU credits and how they accumulate when instance is idle
- Show how burstable instances work differently from standard instances
- Demonstrate what happens when credits are exhausted (throttling)
- Use CloudWatch metrics to show CPU credit usage over time
- Explain when to use burstable vs. standard instances

**Instance Monitoring:**
- Show basic CloudWatch metrics: CPU, Network, Disk I/O
- Explain the difference between basic and detailed monitoring
- Demonstrate how to create custom dashboards
- Show how to set up alarms for high CPU usage
- Explain instance status checks vs. system status checks

**CloudWatch Introduction:**
- Explain CloudWatch as AWS's monitoring service
- Show the CloudWatch console and key features
- Demonstrate metric collection and storage
- Explain the relationship between CloudWatch and other AWS services
- Show how to use CloudWatch for troubleshooting

**Additional Discussion Points:**
- Explain the difference between instance types (t2, t3, m5, etc.)
- Show how to choose the right instance type for different workloads
- Demonstrate instance metadata service (IMDS) and its security implications
- Discuss instance lifecycle and termination protection

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

**Session Manager Security Benefits:**
- No inbound ports required (eliminates attack surface)
- All access is logged and auditable
- Centralized access control through IAM
- No need to manage SSH keys or passwords
- Works through AWS Systems Manager (no direct internet access needed)
- Explain how this follows the principle of least privilege

**Session Preferences Configuration:**
- Show how to configure session timeouts
- Demonstrate shell profile customization
- Explain session logging options (CloudWatch Logs)
- Show how to enable port forwarding for database access
- Demonstrate session sharing capabilities for troubleshooting

**Port Forwarding Capabilities:**
- Explain when port forwarding is useful (database access, web applications)
- Show how to forward local ports to remote services
- Demonstrate secure database connections through port forwarding
- Explain the security implications and best practices
- Show how to use port forwarding for debugging

**IAM Role Attachment Process:**
- Explain the difference between instance profiles and IAM roles
- Show how to create the required IAM role with SSM permissions
- Demonstrate the role attachment process during instance launch
- Show how to attach roles to existing instances
- Explain the trust relationship and permission policies

**Additional Discussion Points:**
- Compare Session Manager with traditional SSH access
- Show how to use Session Manager with AWS CLI
- Demonstrate session recording and playback
- Explain integration with CloudTrail for audit logging
- Discuss troubleshooting common Session Manager issues

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

**NFS Protocol Basics:**
- Explain Network File System (NFS) as a distributed file system protocol
- Show how NFS allows multiple clients to access the same file system
- Demonstrate NFS version differences (v3 vs v4)
- Explain the client-server architecture and mount process
- Show how NFS handles file locking and concurrent access

**EFS Mount Options and Performance Tuning:**
- Show different mount options for performance optimization
- Explain throughput modes (bursting vs provisioned)
- Demonstrate performance mode selection (general purpose vs max I/O)
- Show how to monitor EFS performance metrics
- Explain encryption in transit and at rest options

**EFS Pricing Model:**
- Explain pay-per-use pricing (no upfront costs)
- Show how storage pricing works (per GB per month)
- Demonstrate throughput pricing and burst credits
- Explain data transfer costs between AZs and regions
- Show cost comparison with EBS for different use cases

**EFS vs EBS Comparison:**
- EFS: Shared access, automatic scaling, NFS protocol
- EBS: Single instance access, manual scaling, block storage
- Show when to use each service based on requirements
- Demonstrate cost implications for different workloads
- Explain performance characteristics and limitations

**Additional Discussion Points:**
- Show EFS access points for application-specific access control
- Demonstrate EFS lifecycle management and cost optimization
- Explain EFS backup and disaster recovery options
- Show integration with AWS Backup for automated backups
- Discuss EFS security best practices and IAM integration

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

**Subnet Routing Logic:**
- Explain how route tables determine traffic flow
- Show the difference between main route table and custom route tables
- Demonstrate how routes are evaluated (most specific match wins)
- Explain the concept of implicit deny and explicit allow
- Show how to troubleshoot routing issues using route table analysis

**Subnet CIDR Block Calculation:**
- Review CIDR notation basics (/24 = 256 addresses, /16 = 65,536 addresses)
- Show how to calculate usable IP ranges for each subnet
- Demonstrate subnet planning for different tiers (public, private, database)
- Explain the importance of non-overlapping CIDR blocks
- Show tools for CIDR calculation and validation

**NAT Gateway Costs and Alternatives:**
- Explain NAT Gateway pricing (hourly charges + data processing)
- Show cost comparison with NAT Instances (EC2-based)
- Demonstrate when to use each option based on requirements
- Explain NAT Gateway limitations and considerations
- Show how to monitor NAT Gateway usage and costs

**VPC Flow Logs for Traffic Monitoring:**
- Explain what Flow Logs capture (source, destination, protocol, action)
- Show how to enable Flow Logs for VPC, subnet, and ENI levels
- Demonstrate Flow Log analysis using CloudWatch Logs
- Show how to create custom metrics from Flow Log data
- Explain Flow Log use cases for security and troubleshooting

**Additional Discussion Points:**
- Show VPC peering and Transit Gateway for multi-VPC connectivity
- Demonstrate VPC endpoints for private AWS service access
- Explain VPC sharing and resource access across accounts
- Show VPC security best practices and compliance considerations
- Discuss VPC design patterns for different application architectures

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

**Endpoint vs Internet Routing Differences:**
- Explain how VPC endpoints keep traffic within AWS network
- Show the difference in latency and bandwidth costs
- Demonstrate how endpoints bypass internet routing
- Explain security benefits (no exposure to internet threats)
- Show how to verify traffic routing using traceroute and flow logs

**VPC Endpoint Policy Configuration:**
- Explain endpoint policies as resource-based policies
- Show how to restrict access to specific S3 buckets or DynamoDB tables
- Demonstrate policy syntax and condition keys
- Show how to audit endpoint access using CloudTrail
- Explain integration with IAM policies for comprehensive access control

**Endpoint Type Selection:**
- Gateway endpoints: S3/DynamoDB only, no additional charges
- Interface endpoints: All AWS services, ENI-based, hourly charges
- Show when to use each type based on service and cost requirements
- Demonstrate endpoint availability and failover considerations
- Explain endpoint security group configuration for interface endpoints

**Cost Savings with Private Connectivity:**
- Show data transfer cost savings (no internet egress charges)
- Demonstrate bandwidth cost reduction for high-volume applications
- Explain how endpoints reduce NAT Gateway costs
- Show cost comparison examples for different workloads
- Explain how to calculate ROI for endpoint implementation

**Additional Discussion Points:**
- Show endpoint DNS resolution and troubleshooting
- Demonstrate endpoint monitoring and metrics
- Explain endpoint lifecycle management and cleanup
- Show integration with AWS PrivateLink for third-party services
- Discuss endpoint security best practices and compliance considerations

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

**Health Check Configuration Importance:**
- Explain how health checks determine instance availability
- Show the difference between healthy and unhealthy thresholds
- Demonstrate health check timeout and interval settings
- Explain how health checks affect load balancer routing
- Show how to troubleshoot health check failures

**Scaling Policy Configuration:**
- Explain target tracking vs. step scaling vs. simple scaling
- Show how to set appropriate scaling thresholds
- Demonstrate cooldown periods and their importance
- Explain how to avoid scaling thrashing
- Show how to monitor scaling activities and effectiveness

**Instance Refresh Strategies:**
- Explain the importance of keeping instances updated
- Show how to configure instance refresh policies
- Demonstrate rolling update strategies
- Explain health check grace periods during refresh
- Show how to handle failed instance refreshes

**Cross-Zone Load Balancing:**
- Explain how cross-zone load balancing distributes traffic
- Show the difference between zone-aware and cross-zone load balancing
- Demonstrate how to enable cross-zone load balancing
- Explain the cost implications of cross-zone data transfer
- Show how to monitor load distribution across zones

**Additional Discussion Points:**
- Show sticky sessions and session affinity configuration
- Demonstrate SSL/TLS termination and certificate management
- Explain access logs and request tracing
- Show integration with WAF for security
- Discuss load balancer monitoring and alerting best practices

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

**Metric Aggregation and Statistics:**
- Explain different aggregation methods (Average, Sum, Min, Max, Count)
- Show how aggregation periods affect metric resolution
- Demonstrate how to choose appropriate statistics for different metrics
- Explain the difference between basic and detailed monitoring
- Show how to create custom metrics from application data

**Log Filtering and Searching:**
- Explain CloudWatch Logs query syntax and patterns
- Show how to filter logs by time, severity, and content
- Demonstrate log group and log stream organization
- Show how to create metric filters from log data
- Explain log retention policies and cost implications

**Alarm State Management:**
- Explain alarm states: OK, ALARM, INSUFFICIENT_DATA
- Show how to configure alarm actions and notifications
- Demonstrate alarm history and trend analysis
- Explain how to avoid alarm fatigue with proper thresholds
- Show how to test alarms and verify notification delivery

**Dashboard Creation:**
- Show how to create custom CloudWatch dashboards
- Demonstrate widget types and configuration options
- Explain dashboard sharing and access control
- Show how to create operational dashboards for different teams
- Demonstrate dashboard automation and templating

**Additional Discussion Points:**
- Show CloudWatch Insights for advanced log analysis
- Demonstrate anomaly detection and forecasting
- Explain CloudWatch Contributor Insights for cost analysis
- Show integration with other monitoring tools (Grafana, etc.)
- Discuss monitoring best practices and operational excellence

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

**Database Parameter Groups:**
- Explain parameter groups as configuration containers for RDS
- Show how to create custom parameter groups for specific requirements
- Demonstrate parameter modification and application
- Explain the difference between static and dynamic parameters
- Show how to troubleshoot parameter group issues

**Connection Pooling Concepts:**
- Explain why connection pooling is important for database performance
- Show how to implement connection pooling in applications
- Demonstrate RDS Proxy for managed connection pooling
- Explain connection limits and how to monitor them
- Show how to optimize connection usage for cost and performance

**Backup and Restore Procedures:**
- Explain automated vs. manual backup differences
- Show how to create and restore from snapshots
- Demonstrate point-in-time recovery capabilities
- Explain cross-region and cross-account backup copying
- Show how to test backup and restore procedures

**Secret Rotation:**
- Explain the importance of regular credential rotation
- Show how Secrets Manager automates rotation for RDS
- Demonstrate rotation schedules and notification
- Explain how applications handle credential updates
- Show integration with Lambda for custom rotation logic

**Additional Discussion Points:**
- Show RDS performance insights and optimization
- Demonstrate read replicas for scaling read operations
- Explain Multi-AZ failover testing and procedures
- Show database monitoring and alerting best practices
- Discuss RDS security best practices and compliance

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

**Microservices Architecture Concepts:**
- Explain the difference between monolithic and microservices architectures
- Show how microservices enable independent scaling and deployment
- Demonstrate service boundaries and API design principles
- Explain the challenges of distributed systems (network latency, failure handling)
- Show how to design resilient microservices with circuit breakers and retries

**API Gateway Integration Options:**
- Explain API Gateway as a managed service for API management
- Show how to create RESTful and WebSocket APIs
- Demonstrate authentication and authorization integration
- Explain rate limiting and throttling capabilities
- Show how to monitor API usage and performance

**Service Discovery Patterns:**
- Explain the need for service discovery in distributed systems
- Show how to use AWS Cloud Map for service discovery
- Demonstrate DNS-based vs. API-based service discovery
- Explain health checking and load balancing in service discovery
- Show how to implement service mesh patterns

**Distributed Tracing:**
- Explain the importance of tracing in microservices
- Show how to implement distributed tracing with AWS X-Ray
- Demonstrate trace analysis and performance optimization
- Explain how to correlate traces across services
- Show integration with CloudWatch for comprehensive monitoring

**Additional Discussion Points:**
- Show event-driven communication patterns (SQS, SNS, EventBridge)
- Demonstrate container orchestration with ECS/EKS
- Explain database per service vs. shared database patterns
- Show how to implement CQRS and event sourcing
- Discuss microservices testing strategies and best practices

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

**Cold Start Considerations:**
- Explain what causes cold starts (first invocation, memory allocation)
- Show how to minimize cold start impact with provisioned concurrency
- Demonstrate the relationship between memory allocation and performance
- Explain cold start patterns for different runtimes (Python, Node.js, Java)
- Show how to monitor and analyze cold start frequency

**Lambda Configuration Options:**
- Explain memory allocation and its impact on CPU and network
- Show timeout configuration and its importance
- Demonstrate environment variables and their use cases
- Explain VPC configuration and networking considerations
- Show how to configure layers for shared code and dependencies

**Error Handling and Retries:**
- Explain Lambda's built-in retry mechanism for asynchronous invocations
- Show how to implement custom error handling in function code
- Demonstrate dead letter queues for failed message processing
- Explain how to handle partial failures in batch processing
- Show integration with CloudWatch for error monitoring and alerting

**Monitoring and Debugging:**
- Show CloudWatch metrics for Lambda functions (invocations, errors, duration)
- Demonstrate X-Ray tracing for Lambda function analysis
- Explain log analysis and troubleshooting techniques
- Show how to use CloudWatch Logs Insights for log querying
- Demonstrate local testing and debugging with SAM CLI

**Additional Discussion Points:**
- Show Lambda function versioning and aliases
- Demonstrate integration with other AWS services (API Gateway, S3, etc.)
- Explain Lambda function testing strategies and best practices
- Show cost optimization techniques and monitoring
- Discuss Lambda security best practices and compliance

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

**S3 Bucket Naming Requirements:**
- Explain global uniqueness requirement for bucket names
- Show naming conventions and best practices
- Demonstrate DNS compliance requirements
- Explain the relationship between bucket names and URLs
- Show how to check bucket name availability

**CORS Configuration for API Calls:**
- Explain Cross-Origin Resource Sharing (CORS) and its importance
- Show how to configure CORS policies for S3 buckets
- Demonstrate CORS configuration for different use cases
- Explain preflight requests and their handling
- Show how to troubleshoot CORS issues

**Cost Optimization Strategies:**
- Explain S3 storage classes and their cost implications
- Show how to use S3 Intelligent-Tiering for automatic optimization
- Demonstrate lifecycle policies for automatic transitions
- Explain data transfer costs and optimization techniques
- Show how to monitor and analyze S3 costs

**Versioning and Lifecycle Policies:**
- Explain S3 versioning and its benefits for data protection
- Show how to configure and manage versioned objects
- Demonstrate lifecycle policies for automatic cleanup
- Explain the relationship between versioning and lifecycle policies
- Show how to restore previous versions of objects

**Additional Discussion Points:**
- Show S3 access points for fine-grained access control
- Demonstrate S3 Object Lock for compliance requirements
- Explain S3 replication for disaster recovery
- Show integration with CloudFront for content delivery
- Discuss S3 security best practices and encryption options

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

**CDN Caching Concepts:**
- Explain how CloudFront caches content at edge locations
- Show the difference between cache hit and cache miss
- Demonstrate cache key customization and its impact
- Explain TTL (Time To Live) configuration and cache behavior
- Show how to monitor cache performance and hit ratios

**Invalidation Procedures:**
- Explain when and why to invalidate CloudFront cache
- Show how to create invalidation requests
- Demonstrate wildcard invalidation vs. specific path invalidation
- Explain invalidation costs and best practices
- Show how to use cache headers for cache control

**WAF Rule Configuration:**
- Explain WAF rule evaluation order and priority
- Show how to create custom rules for specific requirements
- Demonstrate managed rule sets and their benefits
- Explain rate limiting and IP reputation rules
- Show how to test WAF rules and monitor their effectiveness

**DNS Propagation:**
- Explain how DNS propagation works across the internet
- Show the difference between A records and CNAME records
- Demonstrate Route 53 health checks and failover
- Explain DNS caching and TTL settings
- Show how to troubleshoot DNS resolution issues

**Additional Discussion Points:**
- Show CloudFront functions for edge computing
- Demonstrate origin failover and multi-origin configurations
- Explain CloudFront security features and HTTPS enforcement
- Show integration with AWS Shield for DDoS protection
- Discuss CloudFront cost optimization and monitoring

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

**Compliance Requirements:**
- Explain common compliance frameworks (SOC, PCI DSS, HIPAA, ISO 27001)
- Show how AWS services help meet compliance requirements
- Demonstrate compliance reporting and audit trails
- Explain the shared responsibility model for compliance
- Show how to implement compliance controls in AWS

**CloudTrail Event Analysis:**
- Explain CloudTrail event structure and fields
- Show how to filter and search CloudTrail events
- Demonstrate CloudTrail Lake for advanced analytics
- Explain how to detect unusual activity and security events
- Show integration with SIEM tools for security monitoring

**Config Rule Customization:**
- Explain how to create custom Config rules using Lambda
- Show rule evaluation and compliance status tracking
- Demonstrate automated remediation with Config
- Explain rule scope and resource targeting
- Show how to test and validate custom rules

**Remediation Workflows:**
- Explain automated remediation vs. manual intervention
- Show how to configure remediation actions
- Demonstrate remediation execution and monitoring
- Explain remediation approval workflows and notifications
- Show how to handle remediation failures and rollbacks

**Additional Discussion Points:**
- Show AWS Security Hub for centralized security findings
- Demonstrate GuardDuty for threat detection
- Explain AWS Artifact for compliance documentation
- Show integration with third-party compliance tools
- Discuss governance best practices and organizational policies

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

**Template Validation:**
- Explain CloudFormation template syntax and structure
- Show how to validate templates before deployment
- Demonstrate common template errors and how to fix them
- Explain intrinsic functions and their usage
- Show how to use CloudFormation Designer for visual template creation

**Parameter Constraints:**
- Explain parameter types and their validation rules
- Show how to create custom parameter constraints
- Demonstrate parameter groups and labels for better UX
- Explain default values and allowed values
- Show how to use AWS Systems Manager Parameter Store for sensitive parameters

**Stack Dependencies:**
- Explain how CloudFormation handles resource dependencies
- Show explicit vs. implicit dependencies
- Demonstrate DependsOn attribute usage
- Explain circular dependency detection and resolution
- Show how to optimize stack deployment order

**Change Sets:**
- Explain the purpose and benefits of change sets
- Show how to create and review change sets
- Demonstrate change set execution and rollback
- Explain how to handle change set failures
- Show integration with CI/CD pipelines for automated deployments

**Additional Discussion Points:**
- Show CloudFormation macros for template customization
- Demonstrate custom resources for non-AWS resources
- Explain stack policies for update protection
- Show CloudFormation registry for third-party resources
- Discuss CloudFormation best practices and organizational policies

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

**Stack Dependencies:**
- Explain how nested stacks create parent-child relationships
- Show how to manage dependencies between nested stacks
- Demonstrate stack output values and their usage
- Explain circular dependency prevention in nested stacks
- Show how to optimize stack deployment order and parallelization

**Cross-Stack References:**
- Explain how to reference resources across different stacks
- Show Export/Import functionality for cross-stack communication
- Demonstrate stack output values and their limitations
- Explain how to handle cross-stack updates and dependencies
- Show best practices for organizing stack references

**Deployment Strategies:**
- Explain blue-green deployment with CloudFormation
- Show rolling update strategies for zero-downtime deployments
- Demonstrate canary deployments and traffic shifting
- Explain deployment monitoring and rollback procedures
- Show integration with AWS CodePipeline for automated deployments

**Troubleshooting Techniques:**
- Explain CloudFormation stack events and their interpretation
- Show how to use CloudFormation console for troubleshooting
- Demonstrate stack drift detection and resolution
- Explain common deployment failures and their solutions
- Show how to use AWS CLI for stack management and debugging

**Additional Discussion Points:**
- Show CloudFormation service roles and permissions
- Demonstrate stack set deployment across multiple accounts
- Explain CloudFormation cost optimization and cleanup
- Show integration with AWS Organizations for multi-account management
- Discuss CloudFormation governance and compliance considerations

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

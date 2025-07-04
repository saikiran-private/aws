# 🚀 AWS Comprehensive Training Guide
## Complete DevOps Journey with Movie Analyzer Project

---

## 📋 Table of Contents

1. [**Class 1: Kick-off & Guard-rails**](#class-1-kick-off--guard-rails)
2. [**Class 2: EC2 Refresher**](#class-2-ec2-refresher)
3. [**Class 3: IAM + CLI + SSM**](#class-3-iam--cli--ssm)
4. [**Class 4: Storage for Compute**](#class-4-storage-for-compute)
5. [**Class 5: Custom VPC 101**](#class-5-custom-vpc-101)
6. [**Class 6: Advanced VPC (Optional)**](#class-6-advanced-vpc-optional)
7. [**Class 7: Load Balancer & Auto Scaling**](#class-7-load-balancer--auto-scaling)
8. [**Class 8: Observability & Alerts**](#class-8-observability--alerts)
9. [**Class 9: RDS & Secrets Manager**](#class-9-rds--secrets-manager)
10. [**Class 10: Backend & Model Tiers**](#class-10-backend--model-tiers)
11. [**Class 11: Serverless & Queueing**](#class-11-serverless--queueing)
12. [**Class 12: S3 + KMS & Static Hosting**](#class-12-s3--kms--static-hosting)
13. [**Class 13: CDN, WAF & Route 53**](#class-13-cdn-waf--route-53)
14. [**Class 14: Governance (Optional)**](#class-14-governance-optional)
15. [**Class 15: CloudFormation 101 (Optional)**](#class-15-cloudformation-101-optional)
16. [**Class 16: Capstone IaC**](#class-16-capstone-iac)

---

## Class 1: Kick-off & Guard-rails

### 🎯 Learning Objectives
- Understand AWS shared responsibility model
- Set up billing alarms and cost monitoring
- Configure Multi-Factor Authentication (MFA)
- Establish budget constraints for Free Tier usage

### 📚 Theory: AWS Fundamentals & Cost Management

#### AWS Shared Responsibility Model
```mermaid
graph TB
    subgraph "AWS Responsibility"
        A[Hardware & Infrastructure]
        B[Managed Services]
        C[Global Infrastructure]
    end
    
    subgraph "Customer Responsibility"
        D[Data & Configuration]
        E[Identity & Access Management]
        F[Operating System Updates]
        G[Network Security]
    end
    
    A --> D
    B --> E
    C --> F
    C --> G
```

#### Free Tier Limits Overview
| Service | Free Tier Limit | Monthly Cost After Limit |
|---------|----------------|-------------------------|
| EC2 | 750 hours/month (t2.micro) | $0.0116/hour |
| RDS | 750 hours/month (t2.micro) | $0.017/hour |
| S3 | 5 GB storage | $0.023/GB |
| Lambda | 1M requests/month | $0.0000002/request |
| CloudWatch | 10 metrics | $0.30/metric |

### 🧪 Lab 1: Basic - Cost & Usage Alarm Setup

#### Via AWS Console:
1. **Navigate to AWS Billing Dashboard**
   - Sign in to AWS Console
   - Click on your account name → "Billing & Cost Management"
   - Screenshot placeholder: `[AWS_Console_Billing_Dashboard.png]`

2. **Create Budget Alert**
   - Click "Budgets" → "Create budget"
   - Choose "Cost budget"
   - Set monthly budget: $5 (Free Tier buffer)
   - Screenshot placeholder: `[Budget_Creation_Screen.png]`

#### Via AWS CLI:
```bash
# Install AWS CLI (if not installed)
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install

# Configure AWS CLI
aws configure
# Enter: Access Key, Secret Key, Region (us-east-1), Output format (json)

# Create budget
aws budgets create-budget \
    --account-id 123456789012 \
    --budget file://budget.json \
    --notifications-with-subscribers file://notifications.json
```

**budget.json:**
```json
{
  "BudgetName": "MovieAnalyzer-Budget",
  "BudgetLimit": {
    "Amount": "5.00",
    "Unit": "USD"
  },
  "TimeUnit": "MONTHLY",
  "BudgetType": "COST"
}
```

### 🧪 Lab 2: Intermediate - CloudWatch Billing Alarm

#### Console Method:
1. **Navigate to CloudWatch**
   - Services → CloudWatch → Alarms
   - Click "Create Alarm"
   - Choose "Billing" metric
   - Set threshold: $5.00

#### CLI Method:
```bash
# Create CloudWatch alarm
aws cloudwatch put-metric-alarm \
    --alarm-name "MovieAnalyzer-Billing-Alarm" \
    --alarm-description "Billing alarm for Movie Analyzer project" \
    --metric-name EstimatedCharges \
    --namespace AWS/Billing \
    --statistic Maximum \
    --period 86400 \
    --threshold 5.00 \
    --comparison-operator GreaterThanThreshold \
    --dimensions Name=Currency,Value=USD \
    --evaluation-periods 1 \
    --alarm-actions arn:aws:sns:us-east-1:123456789012:billing-alarm
```

### 🧪 Lab 3: Advanced - MFA Setup & Cost Optimization

#### Enable MFA:
1. **IAM Console Setup**
   - Navigate to IAM → Users → [Your Username]
   - Security credentials → Assign MFA device
   - Choose "Virtual MFA device"
   - Use Google Authenticator or Authy app

#### Cost Optimization Script:
```bash
#!/bin/bash
# cost-monitor.sh - Daily cost monitoring script

# Get current month's costs
aws ce get-cost-and-usage \
    --time-period Start=2024-01-01,End=2024-01-31 \
    --granularity MONTHLY \
    --metrics "UnblendedCost" \
    --group-by Type=DIMENSION,Key=SERVICE

# Get EC2 instances running
aws ec2 describe-instances \
    --filters "Name=instance-state-name,Values=running" \
    --query 'Reservations[].Instances[].{Instance:InstanceId,Type:InstanceType,State:State.Name}'

# Stop non-essential instances (example)
aws ec2 stop-instances --instance-ids i-1234567890abcdef0
```

---

## Class 2: EC2 Refresher

### 🎯 Learning Objectives
- Master EC2 instance families and pricing models
- Understand AMI creation and management
- Implement user data for automated configuration
- Deploy Movie Analyzer frontend prototype

### 📚 Theory: EC2 Deep Dive

#### EC2 Instance Families
```mermaid
graph TD
    A[EC2 Instance Types] --> B[General Purpose]
    A --> C[Compute Optimized]
    A --> D[Memory Optimized]
    A --> E[Storage Optimized]
    
    B --> B1[t2.micro - Web servers]
    B --> B2[t3.medium - Applications]
    C --> C1[c5.large - CPU intensive]
    D --> D1[r5.large - In-memory DB]
    E --> E1[i3.large - Distributed storage]
```

#### Pricing Models Comparison
| Model | Use Case | Cost Savings | Flexibility |
|-------|----------|--------------|-------------|
| On-Demand | Variable workloads | 0% | High |
| Reserved | Steady workloads | 75% | Medium |
| Spot | Fault-tolerant | 90% | Low |
| Dedicated | Compliance | -20% | Low |

### 🧪 Lab 1: Basic - Launch Movie Analyzer Frontend

#### Console Method:
1. **Launch EC2 Instance**
   - Navigate to EC2 → Instances → Launch Instance
   - Choose "Amazon Linux 2 AMI"
   - Select "t2.micro" (Free Tier eligible)
   - Screenshot placeholder: `[EC2_Launch_Instance.png]`

2. **Configure Security Group**
   - Create new security group: "MovieAnalyzer-Web-SG"
   - Add rules:
     - SSH (22) from My IP
     - HTTP (80) from Anywhere
     - Custom TCP (3000) from Anywhere

#### CLI Method:
```bash
# Create security group
aws ec2 create-security-group \
    --group-name MovieAnalyzer-Web-SG \
    --description "Security group for Movie Analyzer web servers"

# Add inbound rules
aws ec2 authorize-security-group-ingress \
    --group-name MovieAnalyzer-Web-SG \
    --protocol tcp \
    --port 22 \
    --cidr 0.0.0.0/0

aws ec2 authorize-security-group-ingress \
    --group-name MovieAnalyzer-Web-SG \
    --protocol tcp \
    --port 80 \
    --cidr 0.0.0.0/0

aws ec2 authorize-security-group-ingress \
    --group-name MovieAnalyzer-Web-SG \
    --protocol tcp \
    --port 3000 \
    --cidr 0.0.0.0/0

# Launch instance
aws ec2 run-instances \
    --image-id ami-0abcdef1234567890 \
    --count 1 \
    --instance-type t2.micro \
    --key-name MovieAnalyzer-KeyPair \
    --security-groups MovieAnalyzer-Web-SG \
    --user-data file://userdata.sh
```

### 🧪 Lab 2: Intermediate - User Data for Automated Setup

**userdata.sh:**
```bash
#!/bin/bash
# User data script for Movie Analyzer Frontend

# Update system
yum update -y

# Install Node.js and npm
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.0/install.sh | bash
source ~/.bashrc
nvm install 18
nvm use 18

# Install Git
yum install -y git

# Clone Movie Analyzer repository
cd /home/ec2-user
git clone https://github.com/your-repo/movie-analyzer.git
cd movie-analyzer/frontend

# Install dependencies
npm install

# Install PM2 for process management
npm install -g pm2

# Build application
npm run build

# Start application with PM2
pm2 start server.js --name "movie-frontend"
pm2 startup
pm2 save

# Configure auto-start
chkconfig pm2-ec2-user on
```

#### Advanced User Data with Environment Variables:
```bash
#!/bin/bash
# Advanced user data with environment configuration

# Create environment file
cat > /home/ec2-user/movie-analyzer/frontend/.env << EOF
NODE_ENV=production
PORT=3000
BACKEND_API_URL=http://localhost:8080
REACT_APP_API_URL=http://localhost:8080
EOF

# Set up log rotation
cat > /etc/logrotate.d/movie-analyzer << EOF
/home/ec2-user/.pm2/logs/*.log {
    daily
    rotate 30
    compress
    delaycompress
    missingok
    notifempty
    create 0644 ec2-user ec2-user
}
EOF

# Install CloudWatch agent
wget https://s3.amazonaws.com/amazoncloudwatch-agent/amazon_linux/amd64/latest/amazon-cloudwatch-agent.rpm
rpm -U ./amazon-cloudwatch-agent.rpm
```

### 🧪 Lab 3: Advanced - AMI Creation & Instance Templates

#### Create Custom AMI:
```bash
# Stop instance (optional for consistent state)
aws ec2 stop-instances --instance-ids i-1234567890abcdef0

# Create AMI
aws ec2 create-image \
    --instance-id i-1234567890abcdef0 \
    --name "MovieAnalyzer-Frontend-AMI" \
    --description "Movie Analyzer Frontend with Node.js and application pre-installed"

# Wait for AMI creation
aws ec2 wait image-available --image-ids ami-0987654321abcdef0

# Create launch template
aws ec2 create-launch-template \
    --launch-template-name MovieAnalyzer-Frontend-Template \
    --launch-template-data '{
        "ImageId": "ami-0987654321abcdef0",
        "InstanceType": "t2.micro",
        "KeyName": "MovieAnalyzer-KeyPair",
        "SecurityGroups": ["MovieAnalyzer-Web-SG"],
        "UserData": "base64-encoded-user-data"
    }'
```

#### Launch Template JSON:
```json
{
  "LaunchTemplateName": "MovieAnalyzer-Frontend-Template",
  "LaunchTemplateData": {
    "ImageId": "ami-0987654321abcdef0",
    "InstanceType": "t2.micro",
    "KeyName": "MovieAnalyzer-KeyPair",
    "SecurityGroups": ["MovieAnalyzer-Web-SG"],
    "IamInstanceProfile": {
      "Name": "MovieAnalyzer-EC2-Profile"
    },
    "UserData": "IyEvYmluL2Jhc2gKLy4uLg==",
    "TagSpecifications": [
      {
        "ResourceType": "instance",
        "Tags": [
          {
            "Key": "Name",
            "Value": "MovieAnalyzer-Frontend"
          },
          {
            "Key": "Environment",
            "Value": "Production"
          }
        ]
      }
    ]
  }
}
```

---

## Class 3: IAM + CLI + SSM

### 🎯 Learning Objectives
- Master IAM users, groups, roles, and policies
- Configure AWS CLI with profiles
- Implement Session Manager for secure access
- Eliminate SSH key dependencies

### 📚 Theory: Identity & Access Management

#### IAM Components Architecture
```mermaid
graph TB
    subgraph "IAM Structure"
        A[Users] --> B[Groups]
        B --> C[Policies]
        D[Roles] --> C
        E[EC2 Instances] --> D
        F[Applications] --> D
    end
    
    subgraph "Policy Types"
        G[AWS Managed]
        H[Customer Managed]
        I[Inline]
    end
    
    C --> G
    C --> H
    C --> I
```

#### Principle of Least Privilege
```mermaid
flowchart TD
    A[User Request] --> B{Identity Verified?}
    B -->|No| C[Deny Access]
    B -->|Yes| D{Policy Allows?}
    D -->|No| C
    D -->|Yes| E{Resource Available?}
    E -->|No| C
    E -->|Yes| F[Grant Access]
```

### 🧪 Lab 1: Basic - IAM Setup for Movie Analyzer

#### Console Method:
1. **Create IAM Group**
   - Navigate to IAM → Groups → Create Group
   - Group name: "MovieAnalyzer-Developers"
   - Attach policies: "AmazonEC2ReadOnlyAccess"

2. **Create IAM User**
   - Navigate to IAM → Users → Add User
   - Username: "movie-analyzer-dev"
   - Access type: "Programmatic access"
   - Add to group: "MovieAnalyzer-Developers"

#### CLI Method:
```bash
# Create IAM group
aws iam create-group --group-name MovieAnalyzer-Developers

# Create IAM user
aws iam create-user --user-name movie-analyzer-dev

# Add user to group
aws iam add-user-to-group \
    --group-name MovieAnalyzer-Developers \
    --user-name movie-analyzer-dev

# Attach policy to group
aws iam attach-group-policy \
    --group-name MovieAnalyzer-Developers \
    --policy-arn arn:aws:iam::aws:policy/AmazonEC2ReadOnlyAccess

# Create access keys
aws iam create-access-key --user-name movie-analyzer-dev
```

### 🧪 Lab 2: Intermediate - Custom IAM Policies & Roles

#### Create Custom Policy for Movie Analyzer:
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "ec2:DescribeInstances",
        "ec2:DescribeImages",
        "ec2:DescribeSecurityGroups",
        "ec2:RunInstances",
        "ec2:TerminateInstances"
      ],
      "Resource": "*",
      "Condition": {
        "StringEquals": {
          "aws:RequestedRegion": "us-east-1"
        }
      }
    },
    {
      "Effect": "Allow",
      "Action": [
        "s3:GetObject",
        "s3:PutObject"
      ],
      "Resource": "arn:aws:s3:::movie-analyzer-assets/*"
    }
  ]
}
```

#### Create IAM Role for EC2:
```bash
# Create trust policy
cat > trust-policy.json << EOF
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Service": "ec2.amazonaws.com"
      },
      "Action": "sts:AssumeRole"
    }
  ]
}
EOF

# Create role
aws iam create-role \
    --role-name MovieAnalyzer-EC2-Role \
    --assume-role-policy-document file://trust-policy.json

# Attach policies
aws iam attach-role-policy \
    --role-name MovieAnalyzer-EC2-Role \
    --policy-arn arn:aws:iam::aws:policy/AmazonSSMManagedInstanceCore

# Create instance profile
aws iam create-instance-profile \
    --instance-profile-name MovieAnalyzer-EC2-Profile

# Add role to instance profile
aws iam add-role-to-instance-profile \
    --instance-profile-name MovieAnalyzer-EC2-Profile \
    --role-name MovieAnalyzer-EC2-Role
```

### 🧪 Lab 3: Advanced - Session Manager Setup

#### Configure AWS CLI Profiles:
```bash
# Configure multiple profiles
aws configure --profile dev
aws configure --profile prod

# List profiles
aws configure list-profiles

# Use specific profile
aws ec2 describe-instances --profile dev
```

#### AWS CLI Config Files:
**~/.aws/config:**
```ini
[default]
region = us-east-1
output = json

[profile dev]
region = us-east-1
output = table

[profile prod]
region = us-west-2
output = json
```

**~/.aws/credentials:**
```ini
[default]
aws_access_key_id = AKIAIOSFODNN7EXAMPLE
aws_secret_access_key = wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY

[dev]
aws_access_key_id = AKIAI44QH8DHBEXAMPLE
aws_secret_access_key = je7MtGbClwBF/2Zp9Utk/h3yCo8nvbEXAMPLEKEY
```

#### Session Manager Connection:
```bash
# Install Session Manager plugin
curl "https://s3.amazonaws.com/session-manager-downloads/plugin/latest/linux_64bit/session-manager-plugin.rpm" -o "session-manager-plugin.rpm"
sudo rpm -i session-manager-plugin.rpm

# Connect to instance via Session Manager
aws ssm start-session --target i-1234567890abcdef0

# Port forwarding through Session Manager
aws ssm start-session \
    --target i-1234567890abcdef0 \
    --document-name AWS-StartPortForwardingSession \
    --parameters '{"portNumber":["3000"],"localPortNumber":["8080"]}'
```

#### Remove SSH Access:
```bash
# Remove SSH rule from security group
aws ec2 revoke-security-group-ingress \
    --group-name MovieAnalyzer-Web-SG \
    --protocol tcp \
    --port 22 \
    --cidr 0.0.0.0/0

# Verify security group rules
aws ec2 describe-security-groups \
    --group-names MovieAnalyzer-Web-SG
```

---

## Class 4: Storage for Compute

### 🎯 Learning Objectives
- Understand EBS volume types and performance characteristics
- Implement EBS snapshots and backup strategies
- Configure EFS for shared storage across multiple instances
- Store Movie Analyzer poster images on shared storage

### 📚 Theory: AWS Storage Services

#### EBS Volume Types Comparison
```mermaid
graph TB
    A[EBS Volume Types] --> B[General Purpose SSD]
    A --> C[Provisioned IOPS SSD]
    A --> D[Cold HDD]
    A --> E[Throughput Optimized HDD]
    
    B --> B1[gp3 - 3,000 IOPS baseline]
    B --> B2[gp2 - 3 IOPS per GB]
    C --> C1[io1 - Up to 64,000 IOPS]
    C --> C2[io2 - Up to 256,000 IOPS]
    D --> D1[sc1 - 250 IOPS max]
    E --> E1[st1 - 500 IOPS max]
```

#### Storage Performance & Cost Matrix
| Volume Type | Size Range | Performance | Cost/GB/Month | Use Case |
|-------------|------------|-------------|---------------|-----------|
| gp3 | 1 GB - 16 TB | 3,000-16,000 IOPS | $0.08 | General workloads |
| gp2 | 1 GB - 16 TB | 3-16,000 IOPS | $0.10 | Burst workloads |
| io1 | 4 GB - 16 TB | 100-64,000 IOPS | $0.125 | High IOPS |
| sc1 | 125 GB - 16 TB | 250 IOPS max | $0.015 | Cold storage |
| st1 | 125 GB - 16 TB | 500 IOPS max | $0.045 | Throughput intensive |

### 🧪 Lab 1: Basic - EBS Volume Management

#### Console Method:
1. **Create EBS Volume**
   - Navigate to EC2 → Volumes → Create Volume
   - Volume Type: General Purpose SSD (gp3)
   - Size: 20 GB
   - Availability Zone: Same as EC2 instance

2. **Attach Volume to Instance**
   - Select volume → Actions → Attach Volume
   - Choose instance → Device: /dev/sdf

#### CLI Method:
```bash
# Create EBS volume
aws ec2 create-volume \
    --size 20 \
    --volume-type gp3 \
    --availability-zone us-east-1a \
    --tag-specifications 'ResourceType=volume,Tags=[{Key=Name,Value=MovieAnalyzer-Data}]'

# Attach volume to instance
aws ec2 attach-volume \
    --volume-id vol-1234567890abcdef0 \
    --instance-id i-1234567890abcdef0 \
    --device /dev/sdf

# Wait for attachment
aws ec2 wait volume-in-use --volume-ids vol-1234567890abcdef0
```

#### Mount EBS Volume:
```bash
# Connect to instance
aws ssm start-session --target i-1234567890abcdef0

# Check available disks
lsblk

# Format the volume (first time only)
sudo mkfs -t ext4 /dev/xvdf

# Create mount point
sudo mkdir /mnt/movie-data

# Mount the volume
sudo mount /dev/xvdf /mnt/movie-data

# Make mount persistent
echo '/dev/xvdf /mnt/movie-data ext4 defaults,nofail 0 2' | sudo tee -a /etc/fstab

# Verify mount
df -h
```

### 🧪 Lab 2: Intermediate - EBS Snapshots & Backup Strategy

#### Create Snapshot:
```bash
# Create snapshot
aws ec2 create-snapshot \
    --volume-id vol-1234567890abcdef0 \
    --description "MovieAnalyzer data backup $(date +%Y-%m-%d)"

# List snapshots
aws ec2 describe-snapshots \
    --owner-ids self \
    --filters Name=volume-id,Values=vol-1234567890abcdef0

# Create volume from snapshot
aws ec2 create-volume \
    --size 20 \
    --snapshot-id snap-1234567890abcdef0 \
    --availability-zone us-east-1a
```

#### Automated Backup Script:
```bash
#!/bin/bash
# ebs-backup.sh - Automated EBS backup script

VOLUME_ID="vol-1234567890abcdef0"
RETENTION_DAYS=7

# Create snapshot
SNAPSHOT_ID=$(aws ec2 create-snapshot \
    --volume-id $VOLUME_ID \
    --description "MovieAnalyzer automated backup $(date +%Y-%m-%d-%H:%M)" \
    --query 'SnapshotId' --output text)

echo "Created snapshot: $SNAPSHOT_ID"

# Tag snapshot
aws ec2 create-tags \
    --resources $SNAPSHOT_ID \
    --tags Key=Name,Value=MovieAnalyzer-AutoBackup \
           Key=CreatedBy,Value=BackupScript \
           Key=RetentionDays,Value=$RETENTION_DAYS

# Delete old snapshots
OLD_SNAPSHOTS=$(aws ec2 describe-snapshots \
    --owner-ids self \
    --filters Name=tag:CreatedBy,Values=BackupScript \
    --query "Snapshots[?StartTime<='$(date -d "$RETENTION_DAYS days ago" --iso-8601)'].SnapshotId" \
    --output text)

for snapshot in $OLD_SNAPSHOTS; do
    echo "Deleting old snapshot: $snapshot"
    aws ec2 delete-snapshot --snapshot-id $snapshot
done
```

### 🧪 Lab 3: Advanced - EFS Shared Storage for Movie Posters

#### Create EFS File System:
```bash
# Create EFS file system
aws efs create-file-system \
    --creation-token movie-analyzer-efs-$(date +%s) \
    --performance-mode generalPurpose \
    --throughput-mode provisioned \
    --provisioned-throughput-in-mibps 100 \
    --tags Key=Name,Value=MovieAnalyzer-SharedStorage

# Get file system ID
EFS_ID=$(aws efs describe-file-systems \
    --query 'FileSystems[?Name==`MovieAnalyzer-SharedStorage`].FileSystemId' \
    --output text)

# Create mount targets (one per AZ)
aws efs create-mount-target \
    --file-system-id $EFS_ID \
    --subnet-id subnet-12345678 \
    --security-groups sg-12345678
```

#### Configure EFS Security Group:
```bash
# Create EFS security group
aws ec2 create-security-group \
    --group-name MovieAnalyzer-EFS-SG \
    --description "Security group for EFS access"

# Allow NFS traffic from EC2 security group
aws ec2 authorize-security-group-ingress \
    --group-name MovieAnalyzer-EFS-SG \
    --protocol tcp \
    --port 2049 \
    --source-group MovieAnalyzer-Web-SG
```

#### Mount EFS on Multiple Instances:
```bash
# Install EFS utilities
sudo yum install -y amazon-efs-utils

# Create mount point
sudo mkdir -p /mnt/movie-posters

# Mount EFS
sudo mount -t efs $EFS_ID:/ /mnt/movie-posters

# Make mount persistent
echo "$EFS_ID.efs.us-east-1.amazonaws.com:/ /mnt/movie-posters efs defaults,_netdev 0 0" | sudo tee -a /etc/fstab

# Verify mount
df -h | grep efs
```

#### Upload Movie Poster Images:
```bash
# Create directory structure
sudo mkdir -p /mnt/movie-posters/images/{shawshank,inception,interstellar,fight-club,gladiator,dark-knight}

# Set permissions
sudo chown -R ec2-user:ec2-user /mnt/movie-posters
chmod 755 /mnt/movie-posters

# Upload sample posters (placeholder)
cd /mnt/movie-posters/images
wget -O shawshank/poster.jpg "https://example.com/shawshank-poster.jpg"
wget -O inception/poster.jpg "https://example.com/inception-poster.jpg"
# ... continue for all movies

# Create image metadata
cat > /mnt/movie-posters/metadata.json << EOF
{
  "images": {
    "shawshank": {
      "title": "The Shawshank Redemption",
      "poster": "/images/shawshank/poster.jpg",
      "thumbnail": "/images/shawshank/thumb.jpg"
    },
    "inception": {
      "title": "Inception",
      "poster": "/images/inception/poster.jpg", 
      "thumbnail": "/images/inception/thumb.jpg"
    }
  }
}
EOF
```

#### Update Movie Analyzer to Use EFS:
```javascript
// frontend/src/config/storage.js
const STORAGE_CONFIG = {
  imageBasePath: '/mnt/movie-posters/images',
  metadataPath: '/mnt/movie-posters/metadata.json',
  uploadPath: '/mnt/movie-posters/uploads'
};

// Update image serving endpoint
app.get('/api/images/:movieId/:filename', (req, res) => {
  const { movieId, filename } = req.params;
  const imagePath = `${STORAGE_CONFIG.imageBasePath}/${movieId}/${filename}`;
  
  if (fs.existsSync(imagePath)) {
    res.sendFile(path.resolve(imagePath));
  } else {
    res.status(404).send('Image not found');
  }
});
```

---

## Class 5: Custom VPC 101

### 🎯 Learning Objectives
- Design and implement a custom VPC with multiple Availability Zones
- Configure subnets, route tables, and internet connectivity
- Implement NAT Gateway for private subnet internet access
- Understand Security Groups vs Network ACLs
- Deploy Movie Analyzer in a custom network architecture

### 📚 Theory: VPC Fundamentals

#### VPC Architecture Overview
```mermaid
graph TB
    subgraph "Custom VPC (10.0.0.0/16)"
        subgraph "Public Subnet A (10.0.1.0/24)"
            A1[Web Server 1]
            A2[Load Balancer]
        end
        
        subgraph "Private Subnet A (10.0.2.0/24)"
            B1[Backend Server 1]
            B2[Database]
        end
        
        subgraph "Public Subnet B (10.0.3.0/24)"
            C1[Web Server 2]
            C2[NAT Gateway]
        end
        
        subgraph "Private Subnet B (10.0.4.0/24)"
            D1[Backend Server 2]
            D2[Cache]
        end
    end
    
    E[Internet Gateway] --> A1
    E --> A2
    E --> C1
    C2 --> B1
    C2 --> B2
    C2 --> D1
    C2 --> D2
```

#### CIDR Block Planning
| Component | CIDR Block | Available IPs | Purpose |
|-----------|------------|---------------|---------|
| VPC | 10.0.0.0/16 | 65,536 | Main network |
| Public Subnet A | 10.0.1.0/24 | 256 | Web tier AZ-1 |
| Private Subnet A | 10.0.2.0/24 | 256 | App tier AZ-1 |
| Public Subnet B | 10.0.3.0/24 | 256 | Web tier AZ-2 |
| Private Subnet B | 10.0.4.0/24 | 256 | App tier AZ-2 |

### 🧪 Lab 1: Basic - Create Custom VPC

#### Console Method:
1. **Create VPC**
   - Navigate to VPC → Your VPCs → Create VPC
   - Name: MovieAnalyzer-VPC
   - IPv4 CIDR: 10.0.0.0/16
   - IPv6 CIDR: No IPv6
   - Tenancy: Default

2. **Create Subnets**
   - Navigate to VPC → Subnets → Create Subnet
   - Create 4 subnets as per the table above

#### CLI Method:
```bash
# Create VPC
VPC_ID=$(aws ec2 create-vpc \
    --cidr-block 10.0.0.0/16 \
    --query 'Vpc.VpcId' --output text)

# Tag VPC
aws ec2 create-tags \
    --resources $VPC_ID \
    --tags Key=Name,Value=MovieAnalyzer-VPC

# Enable DNS hostnames
aws ec2 modify-vpc-attribute \
    --vpc-id $VPC_ID \
    --enable-dns-hostnames

# Create subnets
PUBLIC_SUBNET_A=$(aws ec2 create-subnet \
    --vpc-id $VPC_ID \
    --cidr-block 10.0.1.0/24 \
    --availability-zone us-east-1a \
    --query 'Subnet.SubnetId' --output text)

PRIVATE_SUBNET_A=$(aws ec2 create-subnet \
    --vpc-id $VPC_ID \
    --cidr-block 10.0.2.0/24 \
    --availability-zone us-east-1a \
    --query 'Subnet.SubnetId' --output text)

PUBLIC_SUBNET_B=$(aws ec2 create-subnet \
    --vpc-id $VPC_ID \
    --cidr-block 10.0.3.0/24 \
    --availability-zone us-east-1b \
    --query 'Subnet.SubnetId' --output text)

PRIVATE_SUBNET_B=$(aws ec2 create-subnet \
    --vpc-id $VPC_ID \
    --cidr-block 10.0.4.0/24 \
    --availability-zone us-east-1b \
    --query 'Subnet.SubnetId' --output text)

# Tag subnets
aws ec2 create-tags --resources $PUBLIC_SUBNET_A --tags Key=Name,Value=MovieAnalyzer-Public-A
aws ec2 create-tags --resources $PRIVATE_SUBNET_A --tags Key=Name,Value=MovieAnalyzer-Private-A
aws ec2 create-tags --resources $PUBLIC_SUBNET_B --tags Key=Name,Value=MovieAnalyzer-Public-B
aws ec2 create-tags --resources $PRIVATE_SUBNET_B --tags Key=Name,Value=MovieAnalyzer-Private-B

# Enable auto-assign public IPs for public subnets
aws ec2 modify-subnet-attribute \
    --subnet-id $PUBLIC_SUBNET_A \
    --map-public-ip-on-launch

aws ec2 modify-subnet-attribute \
    --subnet-id $PUBLIC_SUBNET_B \
    --map-public-ip-on-launch
```

### 🧪 Lab 2: Intermediate - Internet Gateway & Route Tables

#### Create Internet Gateway:
```bash
# Create internet gateway
IGW_ID=$(aws ec2 create-internet-gateway \
    --query 'InternetGateway.InternetGatewayId' --output text)

# Tag internet gateway
aws ec2 create-tags \
    --resources $IGW_ID \
    --tags Key=Name,Value=MovieAnalyzer-IGW

# Attach to VPC
aws ec2 attach-internet-gateway \
    --internet-gateway-id $IGW_ID \
    --vpc-id $VPC_ID
```

#### Configure Route Tables:
```bash
# Create public route table
PUBLIC_RT_ID=$(aws ec2 create-route-table \
    --vpc-id $VPC_ID \
    --query 'RouteTable.RouteTableId' --output text)

# Tag public route table
aws ec2 create-tags \
    --resources $PUBLIC_RT_ID \
    --tags Key=Name,Value=MovieAnalyzer-Public-RT

# Add internet gateway route
aws ec2 create-route \
    --route-table-id $PUBLIC_RT_ID \
    --destination-cidr-block 0.0.0.0/0 \
    --gateway-id $IGW_ID

# Associate public subnets with public route table
aws ec2 associate-route-table \
    --route-table-id $PUBLIC_RT_ID \
    --subnet-id $PUBLIC_SUBNET_A

aws ec2 associate-route-table \
    --route-table-id $PUBLIC_RT_ID \
    --subnet-id $PUBLIC_SUBNET_B
```

### 🧪 Lab 3: Advanced - NAT Gateway & Private Routing

#### Create NAT Gateway:
```bash
# Allocate Elastic IP for NAT Gateway
EIP_ALLOC_ID=$(aws ec2 allocate-address \
    --domain vpc \
    --query 'AllocationId' --output text)

# Create NAT Gateway in public subnet
NAT_GW_ID=$(aws ec2 create-nat-gateway \
    --subnet-id $PUBLIC_SUBNET_A \
    --allocation-id $EIP_ALLOC_ID \
    --query 'NatGateway.NatGatewayId' --output text)

# Wait for NAT Gateway to be available
aws ec2 wait nat-gateway-available --nat-gateway-ids $NAT_GW_ID

# Tag NAT Gateway
aws ec2 create-tags \
    --resources $NAT_GW_ID \
    --tags Key=Name,Value=MovieAnalyzer-NAT-GW
```

#### Configure Private Route Tables:
```bash
# Create private route table
PRIVATE_RT_ID=$(aws ec2 create-route-table \
    --vpc-id $VPC_ID \
    --query 'RouteTable.RouteTableId' --output text)

# Tag private route table
aws ec2 create-tags \
    --resources $PRIVATE_RT_ID \
    --tags Key=Name,Value=MovieAnalyzer-Private-RT

# Add NAT Gateway route
aws ec2 create-route \
    --route-table-id $PRIVATE_RT_ID \
    --destination-cidr-block 0.0.0.0/0 \
    --nat-gateway-id $NAT_GW_ID

# Associate private subnets with private route table
aws ec2 associate-route-table \
    --route-table-id $PRIVATE_RT_ID \
    --subnet-id $PRIVATE_SUBNET_A

aws ec2 associate-route-table \
    --route-table-id $PRIVATE_RT_ID \
    --subnet-id $PRIVATE_SUBNET_B
```

#### Security Groups vs NACLs:
```bash
# Create web tier security group
WEB_SG_ID=$(aws ec2 create-security-group \
    --group-name MovieAnalyzer-Web-SG \
    --description "Security group for web servers" \
    --vpc-id $VPC_ID \
    --query 'GroupId' --output text)

# Allow HTTP/HTTPS from anywhere
aws ec2 authorize-security-group-ingress \
    --group-id $WEB_SG_ID \
    --protocol tcp \
    --port 80 \
    --cidr 0.0.0.0/0

aws ec2 authorize-security-group-ingress \
    --group-id $WEB_SG_ID \
    --protocol tcp \
    --port 443 \
    --cidr 0.0.0.0/0

# Create app tier security group
APP_SG_ID=$(aws ec2 create-security-group \
    --group-name MovieAnalyzer-App-SG \
    --description "Security group for application servers" \
    --vpc-id $VPC_ID \
    --query 'GroupId' --output text)

# Allow app port from web tier only
aws ec2 authorize-security-group-ingress \
    --group-id $APP_SG_ID \
    --protocol tcp \
    --port 8080 \
    --source-group $WEB_SG_ID
```

#### Deploy Movie Analyzer Frontend in Public Subnet:
```bash
# Launch frontend instance in public subnet
aws ec2 run-instances \
    --image-id ami-0abcdef1234567890 \
    --count 1 \
    --instance-type t2.micro \
    --key-name MovieAnalyzer-KeyPair \
    --security-group-ids $WEB_SG_ID \
    --subnet-id $PUBLIC_SUBNET_A \
    --user-data file://frontend-userdata.sh \
    --tag-specifications 'ResourceType=instance,Tags=[{Key=Name,Value=MovieAnalyzer-Frontend-Public}]'
```

#### VPC Flow Logs:
```bash
# Create IAM role for VPC Flow Logs
aws iam create-role \
    --role-name VPCFlowLogsRole \
    --assume-role-policy-document '{
        "Version": "2012-10-17",
        "Statement": [
            {
                "Effect": "Allow",
                "Principal": {
                    "Service": "vpc-flow-logs.amazonaws.com"
                },
                "Action": "sts:AssumeRole"
            }
        ]
    }'

# Attach policy
aws iam attach-role-policy \
    --role-name VPCFlowLogsRole \
    --policy-arn arn:aws:iam::aws:policy/CloudWatchLogsFullAccess

# Create VPC Flow Logs
aws ec2 create-flow-logs \
    --resource-type VPC \
    --resource-ids $VPC_ID \
    --traffic-type ALL \
    --log-destination-type cloud-watch-logs \
    --log-group-name VPCFlowLogs \
    --deliver-logs-permission-arn arn:aws:iam::123456789012:role/VPCFlowLogsRole
```

---

## Class 6: Advanced VPC (Optional)

### 🎯 Learning Objectives
- Implement VPC Endpoints for S3 and SSM
- Understand VPC Peering vs Transit Gateway
- Configure private communication with AWS services
- Optimize data transfer costs

### 📚 Theory: VPC Endpoints & Advanced Networking

#### VPC Endpoints Types
```mermaid
graph TB
    subgraph "VPC Endpoints"
        A[Gateway Endpoints] --> A1[S3]
        A --> A2[DynamoDB]
        B[Interface Endpoints] --> B1[SSM]
        B --> B2[EC2]
        B --> B3[ELB]
        B --> B4[Lambda]
    end
    
    subgraph "Benefits"
        C[Reduced NAT Costs]
        D[Enhanced Security]
        E[Improved Performance]
    end
    
    A1 --> C
    B1 --> D
    B2 --> E
```

### 🧪 Lab 1: Basic - S3 Gateway Endpoint

#### Create S3 Gateway Endpoint:
```bash
# Create S3 Gateway Endpoint
S3_ENDPOINT_ID=$(aws ec2 create-vpc-endpoint \
    --vpc-id $VPC_ID \
    --service-name com.amazonaws.us-east-1.s3 \
    --route-table-ids $PRIVATE_RT_ID \
    --query 'VpcEndpoint.VpcEndpointId' --output text)

# Tag S3 endpoint
aws ec2 create-tags \
    --resources $S3_ENDPOINT_ID \
    --tags Key=Name,Value=MovieAnalyzer-S3-Endpoint

# Verify endpoint
aws ec2 describe-vpc-endpoints \
    --vpc-endpoint-ids $S3_ENDPOINT_ID
```

### 🧪 Lab 2: Intermediate - SSM Interface Endpoint

#### Create SSM Interface Endpoint:
```bash
# Create security group for VPC endpoints
ENDPOINT_SG_ID=$(aws ec2 create-security-group \
    --group-name MovieAnalyzer-VPCEndpoint-SG \
    --description "Security group for VPC endpoints" \
    --vpc-id $VPC_ID \
    --query 'GroupId' --output text)

# Allow HTTPS from private subnets
aws ec2 authorize-security-group-ingress \
    --group-id $ENDPOINT_SG_ID \
    --protocol tcp \
    --port 443 \
    --cidr 10.0.2.0/24

aws ec2 authorize-security-group-ingress \
    --group-id $ENDPOINT_SG_ID \
    --protocol tcp \
    --port 443 \
    --cidr 10.0.4.0/24

# Create SSM endpoint
SSM_ENDPOINT_ID=$(aws ec2 create-vpc-endpoint \
    --vpc-id $VPC_ID \
    --service-name com.amazonaws.us-east-1.ssm \
    --route-table-ids $PRIVATE_RT_ID \
    --subnet-ids $PRIVATE_SUBNET_A $PRIVATE_SUBNET_B \
    --security-group-ids $ENDPOINT_SG_ID \
    --query 'VpcEndpoint.VpcEndpointId' --output text)
```

### 🧪 Lab 3: Advanced - Cost Optimization Analysis

#### Test Data Transfer Through Endpoints:
```bash
# Test S3 access through endpoint (from private instance)
aws s3 cp /tmp/test-file.txt s3://movie-analyzer-bucket/test-file.txt

# Monitor VPC Flow Logs for internal traffic
aws logs get-log-events \
    --log-group-name VPCFlowLogs \
    --log-stream-name $VPC_ID-all \
    --start-time $(date -d '1 hour ago' +%s)000
```

---

## Class 7: Load Balancer & Auto Scaling

### 🎯 Learning Objectives
- Configure Application Load Balancer (ALB) with target groups
- Implement Auto Scaling Groups (ASG) for high availability
- Create golden AMIs for consistent deployments
- Deploy Movie Analyzer with automatic scaling and healing

### 📚 Theory: Load Balancing & Auto Scaling

#### ALB vs Network Load Balancer
```mermaid
graph TB
    subgraph "Application Load Balancer"
        A[Layer 7 - HTTP/HTTPS]
        B[Content-based routing]
        C[SSL termination]
        D[WebSocket support]
    end
    
    subgraph "Network Load Balancer"
        E[Layer 4 - TCP/UDP]
        F[Ultra-low latency]
        G[Static IP addresses]
        H[Extreme performance]
    end
    
    I[Internet] --> A
    I --> E
    A --> J[Target Groups]
    E --> J
```

#### Auto Scaling Components
```mermaid
graph LR
    A[Launch Template] --> B[Auto Scaling Group]
    B --> C[Scaling Policies]
    C --> D[CloudWatch Alarms]
    D --> E[Scale Out/In]
    E --> F[Health Checks]
    F --> G[Replace Unhealthy]
```

### 🧪 Lab 1: Basic - Application Load Balancer Setup

#### Create Target Group:
```bash
# Create target group
TG_ARN=$(aws elbv2 create-target-group \
    --name MovieAnalyzer-Frontend-TG \
    --protocol HTTP \
    --port 3000 \
    --vpc-id $VPC_ID \
    --health-check-path /health \
    --health-check-interval-seconds 30 \
    --health-check-timeout-seconds 5 \
    --healthy-threshold-count 2 \
    --unhealthy-threshold-count 3 \
    --query 'TargetGroups[0].TargetGroupArn' --output text)

# Configure health check
aws elbv2 modify-target-group \
    --target-group-arn $TG_ARN \
    --health-check-path /health \
    --health-check-protocol HTTP \
    --health-check-port 3000
```

#### Create Application Load Balancer:
```bash
# Create ALB
ALB_ARN=$(aws elbv2 create-load-balancer \
    --name MovieAnalyzer-ALB \
    --subnets $PUBLIC_SUBNET_A $PUBLIC_SUBNET_B \
    --security-groups $WEB_SG_ID \
    --query 'LoadBalancers[0].LoadBalancerArn' --output text)

# Create listener
aws elbv2 create-listener \
    --load-balancer-arn $ALB_ARN \
    --protocol HTTP \
    --port 80 \
    --default-actions Type=forward,TargetGroupArn=$TG_ARN

# Get ALB DNS name
ALB_DNS=$(aws elbv2 describe-load-balancers \
    --load-balancer-arns $ALB_ARN \
    --query 'LoadBalancers[0].DNSName' --output text)

echo "ALB DNS: $ALB_DNS"
```

### 🧪 Lab 2: Intermediate - Auto Scaling Group

#### Create Launch Template:
```bash
# Create launch template
LT_ID=$(aws ec2 create-launch-template \
    --launch-template-name MovieAnalyzer-Frontend-LT \
    --launch-template-data '{
        "ImageId": "ami-0abcdef1234567890",
        "InstanceType": "t2.micro",
        "KeyName": "MovieAnalyzer-KeyPair",
        "SecurityGroupIds": ["'$WEB_SG_ID'"],
        "IamInstanceProfile": {
            "Name": "MovieAnalyzer-EC2-Profile"
        },
        "UserData": "'$(base64 -w 0 frontend-userdata.sh)'"
    }' \
    --query 'LaunchTemplate.LaunchTemplateId' --output text)

echo "Launch Template ID: $LT_ID"
```

#### Create Auto Scaling Group:
```bash
# Create Auto Scaling Group
aws autoscaling create-auto-scaling-group \
    --auto-scaling-group-name MovieAnalyzer-Frontend-ASG \
    --launch-template LaunchTemplateId=$LT_ID,Version=1 \
    --min-size 1 \
    --max-size 3 \
    --desired-capacity 2 \
    --target-group-arns $TG_ARN \
    --health-check-type ELB \
    --health-check-grace-period 300 \
    --vpc-zone-identifier "$PUBLIC_SUBNET_A,$PUBLIC_SUBNET_B" \
    --tags Key=Name,Value=MovieAnalyzer-Frontend-ASG,PropagateAtLaunch=true

# Verify ASG
aws autoscaling describe-auto-scaling-groups \
    --auto-scaling-group-names MovieAnalyzer-Frontend-ASG
```

### 🧪 Lab 3: Advanced - Golden AMI & Scaling Policies

#### Create Golden AMI:
```bash
# Launch instance for golden AMI
GOLDEN_INSTANCE_ID=$(aws ec2 run-instances \
    --image-id ami-0abcdef1234567890 \
    --count 1 \
    --instance-type t2.micro \
    --key-name MovieAnalyzer-KeyPair \
    --security-group-ids $WEB_SG_ID \
    --subnet-id $PUBLIC_SUBNET_A \
    --user-data file://golden-ami-userdata.sh \
    --query 'Instances[0].InstanceId' --output text)

# Wait for instance to be running
aws ec2 wait instance-running --instance-ids $GOLDEN_INSTANCE_ID

# Stop instance for consistent AMI
aws ec2 stop-instances --instance-ids $GOLDEN_INSTANCE_ID
aws ec2 wait instance-stopped --instance-ids $GOLDEN_INSTANCE_ID

# Create golden AMI
GOLDEN_AMI_ID=$(aws ec2 create-image \
    --instance-id $GOLDEN_INSTANCE_ID \
    --name "MovieAnalyzer-Golden-AMI-$(date +%Y%m%d-%H%M%S)" \
    --description "Golden AMI for Movie Analyzer Frontend" \
    --query 'ImageId' --output text)

echo "Golden AMI ID: $GOLDEN_AMI_ID"
```

#### Configure Scaling Policies:
```bash
# Create scale-up policy
SCALE_UP_POLICY_ARN=$(aws autoscaling put-scaling-policy \
    --auto-scaling-group-name MovieAnalyzer-Frontend-ASG \
    --policy-name MovieAnalyzer-ScaleUp-Policy \
    --policy-type TargetTrackingScaling \
    --target-tracking-configuration '{
        "TargetValue": 70.0,
        "PredefinedMetricSpecification": {
            "PredefinedMetricType": "ASGAverageCPUUtilization"
        },
        "ScaleOutCooldown": 300,
        "ScaleInCooldown": 300
    }' \
    --query 'PolicyARN' --output text)

# Create CloudWatch alarm for high CPU
aws cloudwatch put-metric-alarm \
    --alarm-name MovieAnalyzer-HighCPU \
    --alarm-description "High CPU usage alarm" \
    --metric-name CPUUtilization \
    --namespace AWS/EC2 \
    --statistic Average \
    --period 300 \
    --threshold 80 \
    --comparison-operator GreaterThanThreshold \
    --evaluation-periods 2 \
    --alarm-actions $SCALE_UP_POLICY_ARN \
    --dimensions Name=AutoScalingGroupName,Value=MovieAnalyzer-Frontend-ASG
```

#### Test Auto Healing:
```bash
# Get instance IDs from ASG
INSTANCE_IDS=$(aws autoscaling describe-auto-scaling-groups \
    --auto-scaling-group-names MovieAnalyzer-Frontend-ASG \
    --query 'AutoScalingGroups[0].Instances[*].InstanceId' --output text)

# Terminate one instance to test auto healing
aws ec2 terminate-instances --instance-ids $(echo $INSTANCE_IDS | cut -d' ' -f1)

# Monitor ASG activities
aws autoscaling describe-scaling-activities \
    --auto-scaling-group-name MovieAnalyzer-Frontend-ASG \
    --max-items 5
```

---

## Class 8: Observability & Alerts

### 🎯 Learning Objectives
- Configure CloudWatch metrics and custom metrics
- Set up log aggregation and monitoring
- Create SNS notifications for alerting
- Implement application performance monitoring

### 📚 Theory: Monitoring & Alerting

#### CloudWatch Architecture
```mermaid
graph TB
    subgraph "Data Sources"
        A[EC2 Instances]
        B[Application Logs]
        C[Custom Metrics]
        D[AWS Services]
    end
    
    subgraph "CloudWatch"
        E[Metrics]
        F[Logs]
        G[Alarms]
        H[Dashboards]
    end
    
    subgraph "Actions"
        I[SNS Notifications]
        J[Auto Scaling]
        K[Lambda Functions]
    end
    
    A --> E
    B --> F
    C --> E
    D --> E
    E --> G
    F --> G
    G --> I
    G --> J
    G --> K
```

### 🧪 Lab 1: Basic - CloudWatch Logs Setup

#### Configure Log Groups:
```bash
# Create log group for application logs
aws logs create-log-group \
    --log-group-name /aws/ec2/movie-analyzer/frontend

aws logs create-log-group \
    --log-group-name /aws/ec2/movie-analyzer/backend

aws logs create-log-group \
    --log-group-name /aws/ec2/movie-analyzer/access

# Set log retention
aws logs put-retention-policy \
    --log-group-name /aws/ec2/movie-analyzer/frontend \
    --retention-in-days 7
```

#### Install CloudWatch Agent:
```bash
# CloudWatch agent configuration
cat > /tmp/cloudwatch-config.json << EOF
{
  "logs": {
    "logs_collected": {
      "files": {
        "collect_list": [
          {
            "file_path": "/var/log/movie-analyzer/frontend.log",
            "log_group_name": "/aws/ec2/movie-analyzer/frontend",
            "log_stream_name": "{instance_id}-frontend"
          },
          {
            "file_path": "/var/log/nginx/access.log",
            "log_group_name": "/aws/ec2/movie-analyzer/access",
            "log_stream_name": "{instance_id}-access"
          }
        ]
      }
    }
  },
  "metrics": {
    "namespace": "MovieAnalyzer/Custom",
    "metrics_collected": {
      "cpu": {
        "measurement": [
          "cpu_usage_idle",
          "cpu_usage_iowait",
          "cpu_usage_user",
          "cpu_usage_system"
        ],
        "metrics_collection_interval": 60
      },
      "disk": {
        "measurement": [
          "used_percent"
        ],
        "metrics_collection_interval": 60,
        "resources": [
          "*"
        ]
      },
      "mem": {
        "measurement": [
          "mem_used_percent"
        ],
        "metrics_collection_interval": 60
      }
    }
  }
}
EOF

# Install and configure CloudWatch agent
sudo yum install -y amazon-cloudwatch-agent
sudo /opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-ctl \
    -a fetch-config \
    -m ec2 \
    -c file:/tmp/cloudwatch-config.json \
    -s
```

### 🧪 Lab 2: Intermediate - SNS Notifications

#### Create SNS Topic:
```bash
# Create SNS topic
SNS_TOPIC_ARN=$(aws sns create-topic \
    --name MovieAnalyzer-Alerts \
    --query 'TopicArn' --output text)

# Subscribe email to topic
aws sns subscribe \
    --topic-arn $SNS_TOPIC_ARN \
    --protocol email \
    --notification-endpoint your-email@example.com

# Confirm subscription (check email)
```

#### Create CloudWatch Alarms:
```bash
# High CPU alarm
aws cloudwatch put-metric-alarm \
    --alarm-name MovieAnalyzer-HighCPU \
    --alarm-description "High CPU utilization" \
    --metric-name CPUUtilization \
    --namespace AWS/EC2 \
    --statistic Average \
    --period 300 \
    --threshold 80 \
    --comparison-operator GreaterThanThreshold \
    --evaluation-periods 2 \
    --alarm-actions $SNS_TOPIC_ARN \
    --dimensions Name=AutoScalingGroupName,Value=MovieAnalyzer-Frontend-ASG

# High error rate alarm
aws cloudwatch put-metric-alarm \
    --alarm-name MovieAnalyzer-HighErrors \
    --alarm-description "High 5xx error rate" \
    --metric-name HTTPCode_Target_5XX_Count \
    --namespace AWS/ApplicationELB \
    --statistic Sum \
    --period 300 \
    --threshold 10 \
    --comparison-operator GreaterThanThreshold \
    --evaluation-periods 2 \
    --alarm-actions $SNS_TOPIC_ARN \
    --dimensions Name=LoadBalancer,Value=$(echo $ALB_ARN | cut -d'/' -f2-)
```

### 🧪 Lab 3: Advanced - Custom Metrics & Dashboard

#### Send Custom Metrics:
```javascript
// Custom metrics for Movie Analyzer
const AWS = require('aws-sdk');
const cloudwatch = new AWS.CloudWatch();

// Function to send custom metrics
async function sendMetrics(metricName, value, unit = 'Count') {
    const params = {
        Namespace: 'MovieAnalyzer/Application',
        MetricData: [
            {
                MetricName: metricName,
                Value: value,
                Unit: unit,
                Timestamp: new Date(),
                Dimensions: [
                    {
                        Name: 'Environment',
                        Value: 'Production'
                    },
                    {
                        Name: 'Service',
                        Value: 'Frontend'
                    }
                ]
            }
        ]
    };

    try {
        await cloudwatch.putMetricData(params).promise();
        console.log(`Metric ${metricName} sent successfully`);
    } catch (error) {
        console.error('Error sending metric:', error);
    }
}

// Usage in application
app.post('/api/reviews', async (req, res) => {
    try {
        // Process review
        const review = await processReview(req.body);
        
        // Send custom metrics
        await sendMetrics('ReviewsSubmitted', 1);
        await sendMetrics('ReviewProcessingTime', processingTime, 'Milliseconds');
        
        res.json(review);
    } catch (error) {
        await sendMetrics('ReviewErrors', 1);
        res.status(500).json({ error: 'Internal server error' });
    }
});
```

#### Create CloudWatch Dashboard:
```bash
# Create dashboard
aws cloudwatch put-dashboard \
    --dashboard-name MovieAnalyzer-Dashboard \
    --dashboard-body '{
        "widgets": [
            {
                "type": "metric",
                "width": 12,
                "height": 6,
                "properties": {
                    "metrics": [
                        ["AWS/ApplicationELB", "RequestCount", "LoadBalancer", "'$(echo $ALB_ARN | cut -d'/' -f2-)'"],
                        [".", "HTTPCode_Target_2XX_Count", ".", "."],
                        [".", "HTTPCode_Target_4XX_Count", ".", "."],
                        [".", "HTTPCode_Target_5XX_Count", ".", "."]
                    ],
                    "period": 300,
                    "stat": "Sum",
                    "region": "us-east-1",
                    "title": "ALB Request Metrics"
                }
            },
            {
                "type": "metric",
                "width": 12,
                "height": 6,
                "properties": {
                    "metrics": [
                        ["AWS/EC2", "CPUUtilization", "AutoScalingGroupName", "MovieAnalyzer-Frontend-ASG"],
                        ["AWS/ApplicationELB", "TargetResponseTime", "LoadBalancer", "'$(echo $ALB_ARN | cut -d'/' -f2-)'"]
                    ],
                    "period": 300,
                    "stat": "Average",
                    "region": "us-east-1",
                    "title": "Performance Metrics"
                }
            }
        ]
    }'
```

---

## Class 9: RDS & Secrets Manager

### 🎯 Learning Objectives
- Deploy RDS MySQL with Multi-AZ for high availability
- Configure automated backups and point-in-time recovery
- Implement Secrets Manager for credential management
- Connect Movie Analyzer backend to RDS database

### 📚 Theory: Managed Database Services

#### RDS Architecture
```mermaid
graph TB
    subgraph "Multi-AZ RDS"
        A[Primary RDS Instance]
        B[Standby RDS Instance]
        C[Automated Backups]
        D[Read Replicas]
    end
    
    subgraph "Applications"
        E[Backend Services]
        F[Read-Only Applications]
    end
    
    E --> A
    F --> D
    A --> B
    A --> C
    B --> C
```

#### RDS Instance Classes & Pricing
| Instance Class | vCPU | Memory | Network | Monthly Cost (t3.micro) |
|---------------|------|---------|---------|-------------------------|
| db.t3.micro | 2 | 1 GB | Low | $12.41 |
| db.t3.small | 2 | 2 GB | Low | $24.82 |
| db.t3.medium | 2 | 4 GB | Moderate | $49.64 |
| db.r5.large | 2 | 16 GB | High | $122.40 |

### 🧪 Lab 1: Basic - RDS MySQL Setup

#### Create DB Subnet Group:
```bash
# Create DB subnet group
aws rds create-db-subnet-group \
    --db-subnet-group-name movie-analyzer-db-subnet-group \
    --db-subnet-group-description "Subnet group for Movie Analyzer RDS" \
    --subnet-ids $PRIVATE_SUBNET_A $PRIVATE_SUBNET_B \
    --tags Key=Name,Value=MovieAnalyzer-DB-SubnetGroup
```

#### Create DB Security Group:
```bash
# Create RDS security group
DB_SG_ID=$(aws ec2 create-security-group \
    --group-name MovieAnalyzer-DB-SG \
    --description "Security group for RDS database" \
    --vpc-id $VPC_ID \
    --query 'GroupId' --output text)

# Allow MySQL access from app servers
aws ec2 authorize-security-group-ingress \
    --group-id $DB_SG_ID \
    --protocol tcp \
    --port 3306 \
    --source-group $APP_SG_ID
```

#### Create RDS Instance:
```bash
# Create RDS instance
aws rds create-db-instance \
    --db-instance-identifier movie-analyzer-db \
    --db-instance-class db.t3.micro \
    --engine mysql \
    --engine-version 8.0.35 \
    --master-username admin \
    --master-user-password TempPassword123! \
    --allocated-storage 20 \
    --storage-type gp3 \
    --storage-encrypted \
    --vpc-security-group-ids $DB_SG_ID \
    --db-subnet-group-name movie-analyzer-db-subnet-group \
    --multi-az \
    --backup-retention-period 7 \
    --backup-window "03:00-04:00" \
    --maintenance-window "sun:04:00-sun:05:00" \
    --auto-minor-version-upgrade \
    --deletion-protection \
    --tags Key=Name,Value=MovieAnalyzer-Database

# Wait for RDS instance to be available
aws rds wait db-instance-available \
    --db-instance-identifier movie-analyzer-db
```

### 🧪 Lab 2: Intermediate - Secrets Manager Integration

#### Create Secret in Secrets Manager:
```bash
# Create secret
SECRET_ARN=$(aws secretsmanager create-secret \
    --name MovieAnalyzer/Database/Credentials \
    --description "Database credentials for Movie Analyzer" \
    --secret-string '{
        "username": "admin",
        "password": "TempPassword123!",
        "engine": "mysql",
        "host": "movie-analyzer-db.cluster-xyz.us-east-1.rds.amazonaws.com",
        "port": 3306,
        "dbname": "movieanalyzer"
    }' \
    --query 'ARN' --output text)

# Generate random password
aws secretsmanager update-secret \
    --secret-id $SECRET_ARN \
    --generate-random-password \
    --password-length 32 \
    --exclude-characters '"@/\'
```

#### Update RDS Password:
```bash
# Get new password from Secrets Manager
NEW_PASSWORD=$(aws secretsmanager get-secret-value \
    --secret-id $SECRET_ARN \
    --query 'SecretString' --output text | jq -r '.password')

# Update RDS master password
aws rds modify-db-instance \
    --db-instance-identifier movie-analyzer-db \
    --master-user-password $NEW_PASSWORD \
    --apply-immediately
```

### 🧪 Lab 3: Advanced - Database Initialization & Backend Connection

#### Database Schema Setup:
```sql
-- Create database and tables
CREATE DATABASE IF NOT EXISTS movieanalyzer;
USE movieanalyzer;

-- Movies table
CREATE TABLE movies (
    id VARCHAR(50) PRIMARY KEY,
    title VARCHAR(255) NOT NULL,
    year INT NOT NULL,
    genre VARCHAR(100),
    director VARCHAR(255),
    description TEXT,
    poster_url VARCHAR(500),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
);

-- Reviews table
CREATE TABLE reviews (
    id BIGINT AUTO_INCREMENT PRIMARY KEY,
    movie_id VARCHAR(50) NOT NULL,
    review_text TEXT NOT NULL,
    sentiment VARCHAR(20),
    sentiment_score DECIMAL(3,2),
    confidence VARCHAR(10),
    rating DECIMAL(2,1),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    FOREIGN KEY (movie_id) REFERENCES movies(id) ON DELETE CASCADE,
    INDEX idx_movie_id (movie_id),
    INDEX idx_created_at (created_at),
    INDEX idx_sentiment (sentiment)
);

-- Insert sample movies
INSERT INTO movies (id, title, year, genre, director, description) VALUES
('shawshank', 'The Shawshank Redemption', 1994, 'Drama', 'Frank Darabont', 'Two imprisoned men bond over years, finding solace and redemption.'),
('inception', 'Inception', 2010, 'Sci-Fi', 'Christopher Nolan', 'A thief enters peoples dreams to steal secrets.'),
('interstellar', 'Interstellar', 2014, 'Sci-Fi', 'Christopher Nolan', 'A team of explorers travel through a wormhole in space.'),
('fight-club', 'Fight Club', 1999, 'Drama', 'David Fincher', 'An insomniac office worker forms an underground fight club.'),
('gladiator', 'Gladiator', 2000, 'Action', 'Ridley Scott', 'A Roman general seeks revenge against the emperor.'),
('dark-knight', 'The Dark Knight', 2008, 'Action', 'Christopher Nolan', 'Batman faces the Joker in Gotham City.');

-- Insert sample reviews
INSERT INTO reviews (movie_id, review_text, sentiment, sentiment_score, confidence, rating) VALUES
('shawshank', 'Absolutely brilliant movie with outstanding performances!', 'positive', 0.95, 'high', 5.0),
('inception', 'Mind-bending and visually stunning masterpiece.', 'positive', 0.88, 'high', 4.5),
('interstellar', 'Beautiful cinematography but confusing plot.', 'neutral', 0.12, 'medium', 3.5);
```

#### Backend Database Connection:
```java
// Backend Spring Boot Configuration
@Configuration
@ConfigurationProperties(prefix = "spring.datasource")
public class DatabaseConfig {
    
    @Autowired
    private SecretsManagerService secretsManagerService;
    
    @Bean
    @Primary
    public DataSource dataSource() {
        HikariConfig config = new HikariConfig();
        
        // Get database credentials from Secrets Manager
        DatabaseCredentials credentials = secretsManagerService.getDatabaseCredentials();
        
        config.setJdbcUrl("jdbc:mysql://" + credentials.getHost() + ":" + credentials.getPort() + "/" + credentials.getDbname());
        config.setUsername(credentials.getUsername());
        config.setPassword(credentials.getPassword());
        config.setMaximumPoolSize(10);
        config.setConnectionTimeout(30000);
        config.setIdleTimeout(600000);
        config.setMaxLifetime(1800000);
        
        return new HikariDataSource(config);
    }
}

// Secrets Manager Service
@Service
public class SecretsManagerService {
    
    private final SecretsManagerClient secretsManagerClient;
    
    public SecretsManagerService() {
        this.secretsManagerClient = SecretsManagerClient.builder()
            .region(Region.US_EAST_1)
            .build();
    }
    
    public DatabaseCredentials getDatabaseCredentials() {
        try {
            GetSecretValueRequest request = GetSecretValueRequest.builder()
                .secretId("MovieAnalyzer/Database/Credentials")
                .build();
            
            GetSecretValueResponse response = secretsManagerClient.getSecretValue(request);
            String secretString = response.secretString();
            
            ObjectMapper objectMapper = new ObjectMapper();
            return objectMapper.readValue(secretString, DatabaseCredentials.class);
        } catch (Exception e) {
            throw new RuntimeException("Error retrieving database credentials", e);
        }
    }
}
```

#### RDS Monitoring & Backup:
```bash
# Create manual snapshot
aws rds create-db-snapshot \
    --db-instance-identifier movie-analyzer-db \
    --db-snapshot-identifier movie-analyzer-manual-snapshot-$(date +%Y%m%d-%H%M%S)

# List automated backups
aws rds describe-db-instances \
    --db-instance-identifier movie-analyzer-db \
    --query 'DBInstances[0].BackupRetentionPeriod'

# Create read replica
aws rds create-db-instance-read-replica \
    --db-instance-identifier movie-analyzer-db-read-replica \
    --source-db-instance-identifier movie-analyzer-db \
    --db-instance-class db.t3.micro
```

---

## Class 10: Backend & Model Tiers

### 🎯 Learning Objectives
- Deploy backend services in private subnets
- Configure internal load balancing
- Implement security group chaining
- Complete Movie Analyzer's full-stack architecture

### 📚 Theory: Multi-Tier Architecture

#### Three-Tier Architecture
```mermaid
graph TB
    subgraph "Web Tier (Public Subnets)"
        A[Frontend Servers]
        B[Application Load Balancer]
    end
    
    subgraph "Application Tier (Private Subnets)"
        C[Backend API Servers]
        D[Internal Load Balancer]
        E[Model Servers]
    end
    
    subgraph "Data Tier (Private Subnets)"
        F[RDS Database]
        G[ElastiCache]
        H[EFS Storage]
    end
    
    B --> A
    D --> C
    D --> E
    C --> F
    C --> G
    E --> H
```

### 🧪 Lab 1: Basic - Backend Service Deployment

#### Create Backend Launch Template:
```bash
# Create backend security group
BACKEND_SG_ID=$(aws ec2 create-security-group \
    --group-name MovieAnalyzer-Backend-SG \
    --description "Security group for backend services" \
    --vpc-id $VPC_ID \
    --query 'GroupId' --output text)

# Allow backend port from web tier
aws ec2 authorize-security-group-ingress \
    --group-id $BACKEND_SG_ID \
    --protocol tcp \
    --port 8080 \
    --source-group $WEB_SG_ID

# Create backend launch template
BACKEND_LT_ID=$(aws ec2 create-launch-template \
    --launch-template-name MovieAnalyzer-Backend-LT \
    --launch-template-data '{
        "ImageId": "ami-0abcdef1234567890",
        "InstanceType": "t2.micro",
        "KeyName": "MovieAnalyzer-KeyPair",
        "SecurityGroupIds": ["'$BACKEND_SG_ID'"],
        "IamInstanceProfile": {
            "Name": "MovieAnalyzer-EC2-Profile"
        },
        "UserData": "'$(base64 -w 0 backend-userdata.sh)'"
    }' \
    --query 'LaunchTemplate.LaunchTemplateId' --output text)
```

#### Backend User Data Script:
```bash
#!/bin/bash
# backend-userdata.sh

# Update system
yum update -y

# Install Java 21
yum install -y java-21-amazon-corretto

# Install Maven
cd /opt
wget https://archive.apache.org/dist/maven/maven-3/3.9.5/binaries/apache-maven-3.9.5-bin.tar.gz
tar -xzf apache-maven-3.9.5-bin.tar.gz
ln -s apache-maven-3.9.5 maven

# Set environment variables
echo 'export M2_HOME=/opt/maven' >> /etc/profile
echo 'export PATH=$M2_HOME/bin:$PATH' >> /etc/profile
source /etc/profile

# Clone and build backend
cd /home/ec2-user
git clone https://github.com/your-repo/movie-analyzer.git
cd movie-analyzer/backend

# Build application
mvn clean package -DskipTests

# Create service user
useradd -r -s /bin/false movieanalyzer

# Create application directory
mkdir -p /opt/movie-analyzer/backend
cp target/*.jar /opt/movie-analyzer/backend/app.jar
chown -R movieanalyzer:movieanalyzer /opt/movie-analyzer

# Create systemd service
cat > /etc/systemd/system/movie-analyzer-backend.service << EOF
[Unit]
Description=Movie Analyzer Backend Service
After=network.target

[Service]
Type=simple
User=movieanalyzer
Group=movieanalyzer
WorkingDirectory=/opt/movie-analyzer/backend
ExecStart=/usr/bin/java -jar /opt/movie-analyzer/backend/app.jar
Restart=always
RestartSec=10
Environment=SPRING_PROFILES_ACTIVE=production

[Install]
WantedBy=multi-user.target
EOF

# Enable and start service
systemctl daemon-reload
systemctl enable movie-analyzer-backend
systemctl start movie-analyzer-backend
```

### 🧪 Lab 2: Intermediate - Internal Load Balancer

#### Create Internal ALB:
```bash
# Create internal ALB
INTERNAL_ALB_ARN=$(aws elbv2 create-load-balancer \
    --name MovieAnalyzer-Internal-ALB \
    --subnets $PRIVATE_SUBNET_A $PRIVATE_SUBNET_B \
    --security-groups $BACKEND_SG_ID \
    --scheme internal \
    --query 'LoadBalancers[0].LoadBalancerArn' --output text)

# Create target group for backend
BACKEND_TG_ARN=$(aws elbv2 create-target-group \
    --name MovieAnalyzer-Backend-TG \
    --protocol HTTP \
    --port 8080 \
    --vpc-id $VPC_ID \
    --health-check-path /actuator/health \
    --health-check-interval-seconds 30 \
    --health-check-timeout-seconds 5 \
    --healthy-threshold-count 2 \
    --unhealthy-threshold-count 3 \
    --query 'TargetGroups[0].TargetGroupArn' --output text)

# Create listener for internal ALB
aws elbv2 create-listener \
    --load-balancer-arn $INTERNAL_ALB_ARN \
    --protocol HTTP \
    --port 80 \
    --default-actions Type=forward,TargetGroupArn=$BACKEND_TG_ARN

# Create backend Auto Scaling Group
aws autoscaling create-auto-scaling-group \
    --auto-scaling-group-name MovieAnalyzer-Backend-ASG \
    --launch-template LaunchTemplateId=$BACKEND_LT_ID,Version=1 \
    --min-size 1 \
    --max-size 3 \
    --desired-capacity 2 \
    --target-group-arns $BACKEND_TG_ARN \
    --health-check-type ELB \
    --health-check-grace-period 300 \
    --vpc-zone-identifier "$PRIVATE_SUBNET_A,$PRIVATE_SUBNET_B"
```

### 🧪 Lab 3: Advanced - Model Tier & Security Group Chaining

#### Deploy Model Service:
```bash
# Create model service security group
MODEL_SG_ID=$(aws ec2 create-security-group \
    --group-name MovieAnalyzer-Model-SG \
    --description "Security group for ML model service" \
    --vpc-id $VPC_ID \
    --query 'GroupId' --output text)

# Allow model port from backend only
aws ec2 authorize-security-group-ingress \
    --group-id $MODEL_SG_ID \
    --protocol tcp \
    --port 5000 \
    --source-group $BACKEND_SG_ID

# Create model service launch template
MODEL_LT_ID=$(aws ec2 create-launch-template \
    --launch-template-name MovieAnalyzer-Model-LT \
    --launch-template-data '{
        "ImageId": "ami-0abcdef1234567890",
        "InstanceType": "t2.micro",
        "KeyName": "MovieAnalyzer-KeyPair",
        "SecurityGroupIds": ["'$MODEL_SG_ID'"],
        "IamInstanceProfile": {
            "Name": "MovieAnalyzer-EC2-Profile"
        },
        "UserData": "'$(base64 -w 0 model-userdata.sh)'"
    }' \
    --query 'LaunchTemplate.LaunchTemplateId' --output text)
```

#### Model Service User Data:
```bash
#!/bin/bash
# model-userdata.sh

# Update system
yum update -y

# Install Python 3.9
yum install -y python39 python39-pip

# Install git
yum install -y git

# Clone repository
cd /home/ec2-user
git clone https://github.com/your-repo/movie-analyzer.git
cd movie-analyzer/model

# Install dependencies
pip3.9 install -r requirements.txt

# Create virtual environment
python3.9 -m venv /opt/movie-analyzer/model-env
source /opt/movie-analyzer/model-env/bin/activate
pip install -r requirements.txt

# Create service user
useradd -r -s /bin/false movieanalyzer

# Create application directory
mkdir -p /opt/movie-analyzer/model
cp -r . /opt/movie-analyzer/model/
chown -R movieanalyzer:movieanalyzer /opt/movie-analyzer

# Create systemd service
cat > /etc/systemd/system/movie-analyzer-model.service << EOF
[Unit]
Description=Movie Analyzer Model Service
After=network.target

[Service]
Type=simple
User=movieanalyzer
Group=movieanalyzer
WorkingDirectory=/opt/movie-analyzer/model
Environment=PATH=/opt/movie-analyzer/model-env/bin
ExecStart=/opt/movie-analyzer/model-env/bin/python app.py
Restart=always
RestartSec=10
Environment=FLASK_ENV=production

[Install]
WantedBy=multi-user.target
EOF

# Enable and start service
systemctl daemon-reload
systemctl enable movie-analyzer-model
systemctl start movie-analyzer-model
```

#### Complete Architecture Test:
```bash
# Test end-to-end connectivity
# Get internal ALB DNS
INTERNAL_ALB_DNS=$(aws elbv2 describe-load-balancers \
    --load-balancer-arns $INTERNAL_ALB_ARN \
    --query 'LoadBalancers[0].DNSName' --output text)

# Update frontend to use internal ALB
echo "BACKEND_API_URL=http://$INTERNAL_ALB_DNS" >> /etc/environment

# Test API endpoint
curl -X POST http://$INTERNAL_ALB_DNS/api/reviews \
    -H "Content-Type: application/json" \
    -d '{
        "movieId": "inception",
        "reviewText": "This movie is absolutely mind-blowing!"
    }'

# Verify sentiment analysis
curl http://$INTERNAL_ALB_DNS/api/reviews/inception
```

---

## Class 11: Serverless & Queueing

### 🎯 Learning Objectives
- Implement Lambda functions for serverless processing
- Configure SQS for message queuing
- Build event-driven architecture
- Process image thumbnails with Lambda

### 📚 Theory: Serverless Architecture
```mermaid
graph LR
    A[S3 Upload] --> B[Lambda Trigger]
    B --> C[Process Image]
    C --> D[Save Thumbnail]
    E[API Request] --> F[SQS Queue]
    F --> G[Lambda Consumer]
```

### 🧪 Lab 1: Basic - Lambda Function Creation
```bash
# Create Lambda function
aws lambda create-function \
    --function-name MovieAnalyzer-Thumbnail \
    --runtime python3.9 \
    --role arn:aws:iam::123456789012:role/lambda-execution-role \
    --handler lambda_function.lambda_handler \
    --zip-file fileb://function.zip
```

### 🧪 Lab 2: Intermediate - SQS Integration
```bash
# Create SQS queue
aws sqs create-queue \
    --queue-name MovieAnalyzer-ProcessingQueue \
    --attributes VisibilityTimeoutSeconds=300
```

### 🧪 Lab 3: Advanced - Event-Driven Processing
```python
# Lambda function for image processing
import json
import boto3
from PIL import Image

def lambda_handler(event, context):
    s3 = boto3.client('s3')
    
    # Process S3 event
    for record in event['Records']:
        bucket = record['s3']['bucket']['name']
        key = record['s3']['object']['key']
        
        # Download image
        response = s3.get_object(Bucket=bucket, Key=key)
        image = Image.open(response['Body'])
        
        # Create thumbnail
        thumbnail = image.resize((150, 150))
        
        # Upload thumbnail
        thumbnail_key = f"thumbnails/{key}"
        # Save logic here
        
    return {'statusCode': 200}
```

---

## Class 12: S3 + KMS & Static Hosting

### 🎯 Learning Objectives
- Configure S3 buckets with versioning and lifecycle policies
- Implement encryption with KMS
- Host static websites on S3
- Migrate Movie Analyzer frontend to S3

### 📚 Theory: Object Storage & Encryption
```mermaid
graph TB
    A[S3 Bucket] --> B[Versioning]
    A --> C[Lifecycle Policies]
    A --> D[Encryption]
    D --> E[SSE-S3]
    D --> F[SSE-KMS]
    D --> G[SSE-C]
```

### 🧪 Lab 1: Basic - S3 Bucket Setup
```bash
# Create S3 bucket
aws s3 mb s3://movie-analyzer-frontend-$(date +%s)

# Enable versioning
aws s3api put-bucket-versioning \
    --bucket movie-analyzer-frontend-$(date +%s) \
    --versioning-configuration Status=Enabled

# Configure static website hosting
aws s3 website s3://movie-analyzer-frontend-$(date +%s) \
    --index-document index.html \
    --error-document error.html
```

### 🧪 Lab 2: Intermediate - KMS Encryption
```bash
# Create KMS key
aws kms create-key \
    --description "Movie Analyzer S3 encryption key" \
    --key-usage ENCRYPT_DECRYPT

# Enable bucket encryption
aws s3api put-bucket-encryption \
    --bucket movie-analyzer-frontend-$(date +%s) \
    --server-side-encryption-configuration '{
        "Rules": [{
            "ApplyServerSideEncryptionByDefault": {
                "SSEAlgorithm": "aws:kms",
                "KMSMasterKeyID": "arn:aws:kms:us-east-1:123456789012:key/12345678-1234-1234-1234-123456789012"
            }
        }]
    }'
```

### 🧪 Lab 3: Advanced - CDN Integration
```bash
# Create CloudFront distribution
aws cloudfront create-distribution \
    --distribution-config file://distribution-config.json
```

---

## Class 13: CDN, WAF & Route 53

### 🎯 Learning Objectives
- Configure CloudFront CDN for global content delivery
- Implement WAF for security protection
- Set up Route 53 for DNS management
- Create SSL certificates with ACM

### 📚 Theory: Global Content Delivery
```mermaid
graph TB
    A[Users] --> B[CloudFront Edge]
    B --> C[S3 Origin]
    D[WAF] --> B
    E[Route 53] --> B
    F[ACM Certificate] --> B
```

### 🧪 Lab 1: Basic - CloudFront Setup
```bash
# Create CloudFront distribution
aws cloudfront create-distribution \
    --distribution-config '{
        "CallerReference": "movie-analyzer-'$(date +%s)'",
        "Origins": {
            "Quantity": 1,
            "Items": [
                {
                    "Id": "S3-movie-analyzer",
                    "DomainName": "movie-analyzer-frontend.s3.amazonaws.com",
                    "S3OriginConfig": {
                        "OriginAccessIdentity": ""
                    }
                }
            ]
        },
        "DefaultCacheBehavior": {
            "TargetOriginId": "S3-movie-analyzer",
            "ViewerProtocolPolicy": "redirect-to-https",
            "MinTTL": 0,
            "ForwardedValues": {
                "QueryString": false,
                "Cookies": {"Forward": "none"}
            }
        },
        "Comment": "Movie Analyzer CDN",
        "Enabled": true
    }'
```

### 🧪 Lab 2: Intermediate - WAF Configuration
```bash
# Create WAF Web ACL
aws wafv2 create-web-acl \
    --name MovieAnalyzer-WebACL \
    --scope CLOUDFRONT \
    --default-action Allow={} \
    --rules '[{
        "Name": "RateLimitRule",
        "Priority": 1,
        "Statement": {
            "RateBasedStatement": {
                "Limit": 2000,
                "AggregateKeyType": "IP"
            }
        },
        "Action": {"Block": {}},
        "VisibilityConfig": {
            "SampledRequestsEnabled": true,
            "CloudWatchMetricsEnabled": true,
            "MetricName": "RateLimitRule"
        }
    }]'
```

### 🧪 Lab 3: Advanced - Route 53 & SSL
```bash
# Create hosted zone
aws route53 create-hosted-zone \
    --name movieanalyzer.com \
    --caller-reference $(date +%s)

# Request SSL certificate
aws acm request-certificate \
    --domain-name movieanalyzer.com \
    --subject-alternative-names www.movieanalyzer.com \
    --validation-method DNS

# Create Route 53 records
aws route53 change-resource-record-sets \
    --hosted-zone-id Z123456789 \
    --change-batch file://dns-records.json
```

---

## Class 14: Governance (Optional)

### 🎯 Learning Objectives
- Implement CloudTrail for audit logging
- Configure AWS Config for compliance
- Set up automated remediation
- Monitor security and compliance

### 📚 Theory: Governance & Compliance
```mermaid
graph TB
    A[CloudTrail] --> B[API Logging]
    C[Config] --> D[Compliance Rules]
    E[Systems Manager] --> F[Remediation]
    G[GuardDuty] --> H[Threat Detection]
```

### 🧪 Lab 1: Basic - CloudTrail Setup
```bash
# Create CloudTrail
aws cloudtrail create-trail \
    --name MovieAnalyzer-Trail \
    --s3-bucket-name movie-analyzer-cloudtrail-logs \
    --include-global-service-events \
    --is-multi-region-trail \
    --enable-log-file-validation
```

### 🧪 Lab 2: Intermediate - Config Rules
```bash
# Create Config rule for S3 bucket security
aws configservice put-config-rule \
    --config-rule '{
        "ConfigRuleName": "s3-bucket-ssl-requests-only",
        "Source": {
            "Owner": "AWS",
            "SourceIdentifier": "S3_BUCKET_SSL_REQUESTS_ONLY"
        }
    }'
```

### 🧪 Lab 3: Advanced - Automated Remediation
```bash
# Create remediation configuration
aws configservice put-remediation-configurations \
    --remediation-configurations '[{
        "ConfigRuleName": "s3-bucket-ssl-requests-only",
        "TargetType": "SSM_DOCUMENT",
        "TargetId": "AWS-PublishSNSNotification",
        "Parameters": {
            "Message": {"StaticValue": "S3 bucket policy violation detected"}
        }
    }]'
```

---

## Class 15: CloudFormation 101 (Optional)

### 🎯 Learning Objectives
- Create infrastructure as code templates
- Understand CloudFormation stacks and resources
- Implement parameters and outputs
- Deploy Movie Analyzer with CloudFormation

### 📚 Theory: Infrastructure as Code
```mermaid
graph TB
    A[CloudFormation Template] --> B[Stack]
    B --> C[Resources]
    C --> D[EC2 Instances]
    C --> E[Load Balancers]
    C --> F[RDS Databases]
    G[Parameters] --> A
    B --> H[Outputs]
```

### 🧪 Lab 1: Basic - Single EC2 Stack
```yaml
# ec2-stack.yaml
AWSTemplateFormatVersion: '2010-09-09'
Description: 'Movie Analyzer EC2 Instance'

Parameters:
  InstanceType:
    Type: String
    Default: t2.micro
    Description: EC2 instance type

Resources:
  MovieAnalyzerInstance:
    Type: AWS::EC2::Instance
    Properties:
      ImageId: ami-0abcdef1234567890
      InstanceType: !Ref InstanceType
      KeyName: MovieAnalyzer-KeyPair
      SecurityGroupIds:
        - !Ref InstanceSecurityGroup
      
  InstanceSecurityGroup:
    Type: AWS::EC2::SecurityGroup
    Properties:
      GroupDescription: Security group for Movie Analyzer
      SecurityGroupIngress:
        - IpProtocol: tcp
          FromPort: 80
          ToPort: 80
          CidrIp: 0.0.0.0/0

Outputs:
  InstanceId:
    Description: EC2 Instance ID
    Value: !Ref MovieAnalyzerInstance
  PublicDNS:
    Description: Public DNS name
    Value: !GetAtt MovieAnalyzerInstance.PublicDnsName
```

### 🧪 Lab 2: Intermediate - Multi-Resource Stack
```bash
# Deploy stack
aws cloudformation create-stack \
    --stack-name MovieAnalyzer-Infrastructure \
    --template-body file://infrastructure.yaml \
    --parameters ParameterKey=InstanceType,ParameterValue=t2.micro \
    --capabilities CAPABILITY_IAM

# Monitor stack creation
aws cloudformation describe-stack-events \
    --stack-name MovieAnalyzer-Infrastructure
```

### 🧪 Lab 3: Advanced - Nested Stacks
```yaml
# master-stack.yaml
AWSTemplateFormatVersion: '2010-09-09'
Description: 'Movie Analyzer Master Stack'

Resources:
  NetworkStack:
    Type: AWS::CloudFormation::Stack
    Properties:
      TemplateURL: https://s3.amazonaws.com/templates/network.yaml
      
  ComputeStack:
    Type: AWS::CloudFormation::Stack
    Properties:
      TemplateURL: https://s3.amazonaws.com/templates/compute.yaml
      Parameters:
        VPCId: !GetAtt NetworkStack.Outputs.VPCId
        SubnetIds: !GetAtt NetworkStack.Outputs.SubnetIds
```

---

## Class 16: Capstone IaC

### 🎯 Learning Objectives
- Deploy complete Movie Analyzer architecture
- Implement change sets for safe updates
- Configure deletion policies
- Perform end-to-end testing

### 📚 Theory: Complete Architecture Deployment
```mermaid
graph TB
    subgraph "Master Stack"
        A[Network Stack] --> B[Security Stack]
        B --> C[Compute Stack]
        C --> D[Database Stack]
        D --> E[Monitoring Stack]
    end
    
    F[Change Sets] --> A
    G[Deletion Policies] --> D
    H[Stack Outputs] --> I[Integration Testing]
```

### 🧪 Lab 1: Basic - Complete Architecture Template
```yaml
# Complete template structure
AWSTemplateFormatVersion: '2010-09-09'
Description: 'Complete Movie Analyzer Architecture'

Parameters:
  Environment:
    Type: String
    Default: Production
    AllowedValues: [Development, Staging, Production]
    
  DatabasePassword:
    Type: String
    NoEcho: true
    Description: Database password

Resources:
  # VPC and Networking
  VPC:
    Type: AWS::EC2::VPC
    Properties:
      CidrBlock: 10.0.0.0/16
      EnableDnsHostnames: true
      EnableDnsSupport: true
      Tags:
        - Key: Name
          Value: !Sub '${Environment}-MovieAnalyzer-VPC'
  
  # Database with deletion protection
  Database:
    Type: AWS::RDS::DBInstance
    Properties:
      DBInstanceIdentifier: !Sub '${Environment}-movie-analyzer-db'
      DBInstanceClass: db.t3.micro
      Engine: mysql
      MasterUsername: admin
      MasterUserPassword: !Ref DatabasePassword
      AllocatedStorage: 20
      DeletionProtection: true
    DeletionPolicy: Snapshot
```

### 🧪 Lab 2: Intermediate - Change Sets
```bash
# Create change set
aws cloudformation create-change-set \
    --stack-name MovieAnalyzer-Complete \
    --template-body file://complete-architecture.yaml \
    --change-set-name UpdateDatabase \
    --parameters ParameterKey=DatabasePassword,ParameterValue=NewPassword123

# Review change set
aws cloudformation describe-change-set \
    --change-set-name UpdateDatabase \
    --stack-name MovieAnalyzer-Complete

# Execute change set
aws cloudformation execute-change-set \
    --change-set-name UpdateDatabase \
    --stack-name MovieAnalyzer-Complete
```

### 🧪 Lab 3: Advanced - End-to-End Testing & Cleanup
```bash
# Smoke test script
#!/bin/bash
echo "Starting Movie Analyzer smoke test..."

# Get stack outputs
ALB_DNS=$(aws cloudformation describe-stacks \
    --stack-name MovieAnalyzer-Complete \
    --query 'Stacks[0].Outputs[?OutputKey==`LoadBalancerDNS`].OutputValue' \
    --output text)

# Test frontend
echo "Testing frontend..."
curl -f http://$ALB_DNS/ || exit 1

# Test API
echo "Testing API..."
curl -f -X GET http://$ALB_DNS/api/health || exit 1

# Test database connectivity
echo "Testing database..."
curl -f -X GET http://$ALB_DNS/api/movies || exit 1

# Test sentiment analysis
echo "Testing sentiment analysis..."
curl -f -X POST http://$ALB_DNS/api/reviews \
    -H "Content-Type: application/json" \
    -d '{"movieId":"inception","reviewText":"Great movie!"}' || exit 1

echo "All tests passed!"

# Cleanup script
echo "Cleaning up resources..."
aws cloudformation delete-stack --stack-name MovieAnalyzer-Complete
aws cloudformation wait stack-delete-complete --stack-name MovieAnalyzer-Complete
echo "Cleanup complete!"
```

---

## 🎓 Course Summary & Next Steps

### Architecture Achieved
```mermaid
graph TB
    subgraph "Production Architecture"
        A[Route 53 DNS] --> B[CloudFront CDN]
        B --> C[S3 Static Website]
        B --> D[Application Load Balancer]
        D --> E[Auto Scaling Group]
        E --> F[EC2 Frontend Instances]
        F --> G[Internal ALB]
        G --> H[Backend Auto Scaling]
        H --> I[RDS MySQL Multi-AZ]
        G --> J[Model Service ASG]
        J --> K[EFS Shared Storage]
    end
    
    subgraph "Monitoring & Security"
        L[CloudWatch Monitoring]
        M[SNS Notifications]
        N[WAF Protection]
        O[VPC Flow Logs]
        P[Secrets Manager]
        Q[IAM Roles]
    end
```

### Cost Optimization Summary
| Service | Monthly Cost (Free Tier) | Production Cost |
|---------|--------------------------|-----------------|
| EC2 (t2.micro) | $0 | $8.50 |
| RDS (db.t3.micro) | $0 | $12.41 |
| ALB | $16.20 | $16.20 |
| NAT Gateway | $32.40 | $32.40 |
| S3 | $0 | $2.30 |
| CloudFront | $0 | $8.50 |
| **Total** | **$48.60** | **$80.31** |

### Key Achievements
✅ Multi-tier architecture with high availability  
✅ Auto-scaling and self-healing infrastructure  
✅ Secure networking with defense in depth  
✅ Comprehensive monitoring and alerting  
✅ Infrastructure as Code with CloudFormation  
✅ Cost-optimized for production workloads  

### Next Steps
1. **Advanced Security**: Implement AWS Security Hub and GuardDuty
2. **Container Orchestration**: Migrate to EKS/ECS
3. **Microservices**: Break down monolithic services
4. **Multi-Region**: Implement disaster recovery
5. **CI/CD**: Automate deployment pipelines
6. **Advanced Monitoring**: Implement distributed tracing

---

## 📚 Additional Resources

### AWS Documentation
- [AWS Architecture Center](https://aws.amazon.com/architecture/)
- [AWS Well-Architected Framework](https://aws.amazon.com/architecture/well-architected/)
- [AWS Cost Optimization](https://aws.amazon.com/aws-cost-management/)

### Best Practices
- [AWS Security Best Practices](https://aws.amazon.com/security/security-learning/)
- [AWS Operational Excellence](https://aws.amazon.com/builders-library/)
- [AWS Reliability Pillar](https://docs.aws.amazon.com/wellarchitected/latest/reliability-pillar/)

### Community & Support
- [AWS Community Builders](https://aws.amazon.com/developer/community/community-builders/)
- [AWS re:Invent Sessions](https://reinvent.awsevents.com/)
- [AWS GitHub Samples](https://github.com/aws-samples)

---

**🚀 Congratulations on completing the AWS Comprehensive Training Guide! You've built a production-ready, scalable, and secure Movie Analyzer application using AWS best practices.**

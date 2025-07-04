# AWS Professional Course: Cloud Architecture & Engineering

## Course Introduction

**Welcome to the AWS Professional Course!** This intensive 16-week program is designed for experienced IT professionals, developers, and system administrators who want to master advanced cloud architecture and engineering on AWS. This course goes beyond the fundamentals, focusing on building secure, scalable, resilient, and cost-optimized applications. Students will architect and deploy a sophisticated, multi-tier sentiment analysis application, leveraging Infrastructure as Code (IaC) and modern DevOps practices.

### Prerequisites

This is an advanced course. Participants are expected to have:
-   **Solid AWS Foundational Knowledge:** Equivalent to AWS Certified Solutions Architect - Associate.
-   **Strong Networking Skills:** Deep understanding of IP addressing, subnetting, routing, DNS, and network security principles.
-   **Command-Line Proficiency:** Comfortable working in Linux environments (Bash/Zsh) and using tools like SSH, cURL, and text editors (Vim/Nano).
-   **Programming/Scripting Experience:** Familiarity with at least one high-level language (e.g., Python, Node.js, Go) for labs involving scripting and SDKs.
-   **Conceptual Understanding of Virtualization and Containers.**

### Course Goals

Upon successful completion of this course, students will be able to:
-   Architect complex, multi-tier applications that are secure, scalable, and highly available.
-   Implement robust security controls and governance policies across an AWS environment.
-   Automate infrastructure provisioning and management using CloudFormation and the AWS CDK.
-   Design and implement advanced networking solutions, including hybrid connectivity.
-   Build and manage CI/CD pipelines for automated application delivery.
-   Master observability practices for proactive monitoring, logging, and tracing.
-   Optimize AWS environments for cost, performance, and operational excellence.

---

## **Part 1: Foundational Security & Governance**

---

## **Week 1: Advanced Guard-rails & AWS Organizations**

**Duration:** 3 Hours (1 Hour Lecture, 2 Hours Lab)

**Core Topics:** AWS Organizations, Service Control Policies (SCPs), Advanced IAM, AWS Cost Management Suite.

---

### **1.1 Learning Objectives**

By the end of this session, students will be able to:
-   Structure a multi-account environment using AWS Organizations.
-   Enforce preventative security controls at the organizational level with SCPs.
-   Implement sophisticated cost allocation and monitoring strategies.
-   Configure programmatic budget actions to automate cost control.
-   Articulate the nuances of the Shared Responsibility Model for various service types (IaaS, PaaS, SaaS).

---

### **1.2 Detailed Content**

#### **1.2.1 Deep Dive into the Shared Responsibility Model**

-   **IaaS (e.g., EC2):**
    -   **AWS:** Manages physical hardware, global infrastructure, and virtualization hypervisor.
    -   **Customer:** Responsible for Guest OS (patching, security), application code, firewall configuration (Security Groups, NACLs), IAM permissions, and all data.
-   **PaaS (e.g., RDS, Lambda):**
    -   **AWS:** Manages the underlying infrastructure, OS, and service runtime (e.g., database engine, Lambda runtime environment).
    -   **Customer:** Responsible for application code, IAM policies granting access *to* the service, network controls (e.g., RDS Security Groups), and the data itself (including encryption choices).
-   **SaaS (e.g., AWS Shield, Trusted Advisor):**
    -   **AWS:** Manages the entire stack, from hardware to application.
    -   **Customer:** Responsible for managing users and entitlements *within* the service.

#### **1.2.2 AWS Organizations: The Foundation of Governance**

-   **Structure:**
    -   **Management Account:** The single payer account, should be used for billing and organization management *only*. No production workloads.
    -   **Organizational Units (OUs):** Hierarchical containers for grouping accounts (e.g., by environment, business unit, or security requirement).
    -   **Member Accounts:** Standard AWS accounts for running workloads, logically separated for security and billing.
-   **Best Practices for OU Structure:**
    -   **Security OU:** Contains accounts for logging, auditing, and security tooling (e.g., GuardDuty master).
    -   **Infrastructure OU:** Holds shared services like networking (Transit Gateway hub), CI/CD tooling.
    -   **Workloads/Applications OU:** Contains the actual application environments (Dev, Staging, Prod OUs).

#### **1.2.3 Service Control Policies (SCPs): Preventative Guardrails**
-   **Function:** Act as a filter on IAM permissions. SCPs do not *grant* permissions; they define the maximum allowable permissions for an account or OU.
-   **Inheritance:** Policies are inherited down the OU tree.
-   **Syntax:** Similar to IAM policies but simpler (no principal).
-   **Common Use Cases:**
    -   **Region Restriction:** Deny actions outside of approved AWS Regions.
    -   **Service Restriction:** Prevent use of unapproved or legacy services (e.g., deny `ec2:RunInstances` for non-Graviton instance types).
    -   **Root User Protection:** Deny critical actions (e.g., `iam:Delete*`) if not performed with MFA.
-   **Strategy:** Start with the `FullAWSAccess` default and apply targeted `Deny` policies. Avoid overly restrictive `Allow` lists which can be hard to maintain.

---

### **1.3 Lab: Building Your Organizational Foundation**

**Objective:** Establish a secure, multi-account structure with cost and security guardrails.

#### **Part A: AWS Organizations Setup (45 mins)**

1.  **Initialize Organization:**
    -   From your management account, navigate to AWS Organizations and create a new organization.
2.  **Create Organizational Units (OUs):**
    -   Create a `Security` OU.
    -   Create a `Workloads` OU.
    -   Inside `Workloads`, create `Dev` and `Prod` OUs.
3.  **Create Member Accounts:**
    -   Create two new member accounts using the Organizations console:
        -   `LogArchive` (provide a unique email).
        -   `AppDev` (provide another unique email).
    -   Move the `LogArchive` account into the `Security` OU.
    -   Move the `AppDev` account into the `Workloads/Dev` OU.

#### **Part B: Implement Service Control Policies (45 mins)**

1.  **Enable SCPs:** In the Organizations console, enable Service Control Policies.
2.  **Region Restriction Policy:**
    -   Create a new SCP named `Deny-Non-US-Regions`.
    -   Add a policy statement that denies all actions (`"Action": "*"`) if the requested region is not `us-east-1` or `us-west-2`.
    -   Attach this SCP to the root of your organization to enforce it everywhere.
3.  **Prevent Root User Misuse:**
    -   Create an SCP named `Protect-Critical-Config`.
    -   Add a statement to deny actions like `s3:DeleteBucket` and `iam:DeleteRole` if the principal is the root user (`aws:PrincipalIsRoot`: `true`).
    -   Attach this to the `Workloads` OU.

#### **Part C: Advanced Cost Management (30 mins)**

1.  **Activate Cost Allocation Tags:**
    -   In the Billing Dashboard under "Cost Allocation Tags", activate user-defined tags like `Project`, `Owner`, and `Environment`.
2.  **Create a Programmatic Budget:**
    -   Navigate to AWS Budgets. Create a new budget with a threshold of $50.
    -   **Configure a Budget Action:**
        -   Set the action to trigger at 80% of the budgeted amount.
        -   Define the action to apply a custom IAM policy (which you will create) to a specific role or user, effectively putting them in "read-only" mode.
        -   Define a second action to notify an SNS topic when the threshold is breached.

---

### **1.4 Teaching Notes**

-   **Account Creation:** Emphasize that creating accounts via AWS Organizations is the modern best practice, as it automates the linkage and provides centralized control. Discuss the pain of inviting and managing existing accounts.
-   **SCP Nuances:**
    -   Stress that SCPs **do not grant permissions**. An explicit `Allow` in an IAM policy is still required.
    -   Explain the implicit `Deny` and the evaluation logic: An action is allowed only if there is no explicit `Deny` at any level (IAM, SCP) AND there is an explicit `Allow` in IAM.
    -   Use the IAM Policy Simulator to demonstrate the effect of SCPs on user permissions.
-   **Budget Actions vs. Alarms:** A billing alarm just *notifies*. A budget *action* can *do something*, which is a powerful, proactive cost control mechanism. Discuss the importance of testing these actions in a non-production environment first.
-   **Multi-Account Strategy:** Use this first session to frame the "why" behind a multi-account strategy: blast radius reduction, security isolation, simplified billing, and scalability. This sets the stage for the entire course.
-   **Real-world Scenario:** Frame the lab as "You are the Lead Cloud Engineer at a startup. Your first task is to set up the AWS foundation correctly so the company can scale without chaos." This makes the exercises more engaging.

---

## **Week 2: Advanced EC2 & Systems Manager**

**Duration:** 3 Hours (1 Hour Lecture, 2 Hours Lab)

**Core Topics:** EC2 Instance Lifecycle, Pricing Models (Spot, Savings Plans), AMI Hardening, AWS Systems Manager (SSM) Fleet Manager, SSM Patch Manager, SSM Run Command.

---

### **2.1 Learning Objectives**

By the end of this session, students will be able to:
-   Analyze and select the optimal EC2 pricing model for various workloads.
-   Implement a strategy for creating and distributing hardened, custom Amazon Machine Images (AMIs).
-   Manage a fleet of EC2 instances at scale without using SSH.
-   Automate patching and command execution across entire instance fleets.
-   Leverage EC2 Image Builder to create a secure and automated AMI pipeline.

---

### **2.2 Detailed Content**

#### **2.2.1 Mastering EC2 Pricing Models**

-   **On-Demand:** Highest cost, highest flexibility. Ideal for unpredictable, short-term workloads or initial development.
-   **Reserved Instances (RIs):**
    -   **Standard RIs:** Highest discount (~72%), but locks you into a specific instance family, region, and term.
    -   **Convertible RIs:** Lower discount (~54%), but allows changing instance family, OS, and tenancy.
    -   Best for predictable, steady-state workloads (e.g., a production database server).
-   **Savings Plans:**
    -   **Compute Savings Plans:** Highest flexibility. Provides a discount (up to 66%) on EC2, Fargate, and Lambda usage in exchange for a commitment to a consistent amount of compute usage (in $/hour). The discount applies automatically to any instance family, size, or region.
    -   **EC2 Instance Savings Plans:** Less flexible. Locks you into a specific instance *family* in a specific *region* (e.g., m5 in us-east-1) for the highest discount (up to 72%).
    -   **Recommendation:** Savings Plans have largely superseded RIs for most use cases due to their flexibility.
-   **Spot Instances:**
    -   Leverages spare AWS capacity for up to 90% discount.
    -   **Mechanism:** AWS can reclaim Spot instances with a two-minute warning.
    -   **Best for:** Fault-tolerant, stateless, and non-time-critical workloads like batch processing, CI/CD runners, or containerized applications managed by an orchestrator (like EKS or ECS) that can handle node termination.
    -   **Spot Fleet:** A collection of Spot Instances (and optionally On-Demand) that attempts to meet a target capacity by launching instances from multiple specified pools.

#### **2.2.2 The "Golden AMI" Pipeline**

-   **Concept:** A "Golden" or "Hardened" AMI is a custom AMI that is pre-configured with necessary software, security patches, monitoring agents, and compliance controls.
-   **Benefits:**
    -   **Faster Launch Times:** Software is pre-installed, reducing bootstrap time.
    -   **Improved Security Posture:** AMIs are scanned and hardened *before* deployment.
    -   **Consistency:** Ensures every instance launched is identical, reducing configuration drift.
-   **EC2 Image Builder:**
    -   An AWS managed service that automates the creation, testing, and distribution of AMIs.
    -   **Components:**
        -   **Image Pipeline:** The master resource defining the workflow.
        -   **Source Image:** The base AMI to build upon (e.g., latest Amazon Linux 2).
        -   **Components:** Building blocks that define the customization and testing steps (e.g., install Nginx, run security scan). AWS provides managed components, and you can create your own.
        -   **Distribution Settings:** Defines where to copy the final AMI (which regions, accounts).

#### **2.2.3 AWS Systems Manager (SSM) for Fleet Management**

-   **SSM Agent:** A pre-installed agent on most AWS-provided AMIs that enables management capabilities.
-   **Fleet Manager:** A UI in the SSM console for viewing the state of your entire instance fleet. You can see file systems, logs, performance counters, and connect via Session Manager without needing SSH.
-   **Patch Manager:**
    -   Automates the process of patching managed instances for both security and other updates.
    -   **Patch Baselines:** Define which patches are approved for installation. You can have baselines for different environments (e.g., `Dev-Baseline` auto-approves all patches, `Prod-Baseline` auto-approves critical patches after 7 days).
    -   **Patch Groups:** Organize instances (using tags) to be patched with specific baselines.
-   **Run Command:**
    -   Allows you to remotely and securely execute commands on your managed instances at scale.
    -   Uses pre-defined `SSM Documents` (e.g., `AWS-RunShellScript`, `AWS-RunPowerShellScript`) or custom documents.
    -   **Use Cases:** Installing software, restarting services, gathering inventory, running diagnostic scripts.

---

### **2.3 Lab: Building an Automated AMI & Patching Workflow**

**Objective:** Create a pipeline to build a hardened web server AMI and then use SSM to patch a fleet of instances launched from it.

#### **Part A: Create a Golden AMI with EC2 Image Builder (60 mins)**

1.  **Define a Component:**
    -   Navigate to EC2 Image Builder -> Components.
    -   Create a new "Build" component named `Install-Nginx-And-Stress`.
    -   Use YAML to define the build steps:
        -   `yum update -y`
        -   `yum install -y nginx stress`
        -   `systemctl enable nginx`
2.  **Create an Image Pipeline:**
    -   Go to Image Pipelines and create a new pipeline.
    -   **Source Image:** Select the latest Amazon Linux 2 AMI.
    -   **Build Components:** Select the `Install-Nginx-And-Stress` component you just created.
    -   **Distribution:** Configure it to distribute the final AMI to your current region.
    -   Run the pipeline and wait for it to produce a new, custom AMI. Note its ID.

#### **Part B: Launch and Manage a Fleet with SSM (30 mins)**

1.  **Launch Fleet:**
    -   Navigate to the EC2 console.
    -   Launch two `t2.micro` instances using the custom AMI ID from Part A.
    -   **Crucially, assign them an IAM role that includes the `AmazonSSMManagedInstanceCore` policy.**
    -   Tag both instances with `Environment:Dev`.
2.  **Explore Fleet Manager:**
    -   Navigate to Systems Manager -> Fleet Manager.
    -   Observe your two new instances.
    -   Select one instance and explore its file system and logs directly from the console.

#### **Part C: Automate Patching with Patch Manager (30 mins)**

1.  **Create a Patch Group:**
    -   Go to Patch Manager -> Patch Groups. Create a patch group for `Dev` by specifying the `Environment:Dev` tag.
2.  **Configure Patching:**
    -   Select "Configure Patching".
    -   Choose the instances to patch by specifying the `Dev` tag.
    -   Select the default patch baseline for Amazon Linux 2.
    -   Schedule the patching to run on a maintenance window (e.g., every Sunday at 2 AM). For the lab, you can choose to run it on demand.
3.  **Run Patch Scan:**
    -   Use "Patch Now" to initiate a *scan* (not install) to see the current compliance state.
    -   View the patch compliance report to see which instances are missing updates.

---

### **2.4 Teaching Notes**

-   **Pricing Model Strategy:** Frame the pricing discussion around business needs. "Is this a spiky, unpredictable workload? On-Demand. Is this a 24/7 production database? Savings Plan or RI. Can this job fail and restart without issue? Spot."
-   **Golden AMI vs. User Data:** This is a critical distinction. **User Data** is for *boot-time* configuration on a generic AMI. A **Golden AMI** has the configuration *pre-baked*. For complex applications, Golden AMIs are far superior for speed, security, and immutability. EC2 Image Builder is the modern way to create them.
-   **SSM is the new SSH:** Hammer this point home. In a modern, automated environment, you should rarely, if ever, need to SSH into an individual server. SSM provides a secure, auditable, and scalable alternative for almost all administrative tasks.
-   **IAM Role is Key:** Remind students that SSM only works if the EC2 instance has the correct IAM instance profile attached. This is a common point of failure.
-   **Lab Realism:** The lab mirrors a real-world workflow:
    1.  **Build:** Create a standardized, secure base image (Image Builder).
    2.  **Deploy:** Launch instances from that trusted image (EC2 Launch).
    3.  **Operate & Maintain:** Manage and patch the running fleet at scale (SSM).

---

## **Part 2: Advanced Identity & Networking**

---

## **Week 3: Advanced IAM & Identity Federation**

**Duration:** 3 Hours (1.5 Hour Lecture, 1.5 Hours Lab)

**Core Topics:** Advanced IAM Policies, Attribute-Based Access Control (ABAC), Permissions Boundaries, IAM Access Analyzer, Identity Federation with SAML 2.0.

---

### **3.1 Learning Objectives**

By the end of this session, students will be able to:
-   Author complex IAM policies using conditions and variables.
-   Implement an Attribute-Based Access Control (ABAC) strategy using tags.
-   Use Permissions Boundaries to safely delegate permissions to developers.
-   Proactively identify and remediate overly permissive access with IAM Access Analyzer.
-   Configure Identity Federation to allow users from an external directory to access AWS.

---

### **3.2 Detailed Content**

#### **3.2.1 Advanced IAM Policy Authoring**

-   **Policy Variables:** Use placeholders like `${aws:username}` or `${ec2:ResourceTag/Project}` to create generic policies that apply contextually to users and resources.
-   **Condition Keys:** Go beyond simple `StringEquals`.
    -   `ForAllValues:StringLike`: Use when the request can have multiple values for a key (e.g., multiple tags) and ALL of them must match the pattern.
    -   `ForAnyValue:StringLike`: Use when ANY of the multiple values must match.
    -   `Null`: Check for the presence or absence of a key (e.g., `aws:RequestTag/Project` is `false` to force tagging on create).
-   **`iam:PassRole`:** A critical, often misunderstood permission. It allows a user to pass an IAM Role to an AWS service (e.g., launching an EC2 instance *with* an instance profile role). This permission must be tightly controlled to prevent privilege escalation.

#### **3.2.2 Attribute-Based Access Control (ABAC)**

-   **Concept:** An authorization strategy that defines permissions based on attributes (tags) attached to users (via the IdP), IAM roles, and AWS resources. This is in contrast to Role-Based Access Control (RBAC), which uses a predefined set of roles.
-   **Why ABAC?** It's highly scalable. In a large organization, creating and managing hundreds of specific roles (RBAC) is cumbersome. With ABAC, you can create a few generic roles and policies that use tags for fine-grained control.
-   **Example Policy:** An IAM policy that allows a user to manage EC2 instances only if the user's `Project` tag exactly matches the EC2 instance's `Project` tag.
    ```json
    {
        "Effect": "Allow",
        "Action": "ec2:StopInstances",
        "Resource": "*",
        "Condition": {
            "StringEquals": {
                "ec2:ResourceTag/Project": "${aws:PrincipalTag/Project}"
            }
        }
    }
    ```

#### **3.2.3 Permissions Boundaries**

-   **Purpose:** A safety mechanism for delegating permissions. It's an advanced IAM feature where you use a managed policy to set the *maximum* permissions that an identity-based policy can grant to a user or role.
-   **Use Case:** You are a senior cloud administrator. You want to allow developers to create their own IAM roles for their applications. However, you don't want them to accidentally create a role with `AdministratorAccess`.
-   **Mechanism:**
    1.  You create a Permissions Boundary policy (e.g., `DeveloperBoundary`) that allows a limited set of actions (e.g., only S3, DynamoDB, and Lambda actions).
    2.  You grant the developers permission to create roles, but *only* if they attach the `DeveloperBoundary` policy to the new roles.
    3.  The effective permissions of the new role are the **intersection** of its own IAM policy and the Permissions Boundary. The developer cannot escalate their privileges beyond the boundary.

#### **3.2.4 IAM Access Analyzer**

-   **Function:** A service that continuously monitors resource-based policies (like S3 bucket policies or KMS key policies) to identify those that grant access to an external principal (e.g., another AWS account or public access).
-   **Findings:** It generates findings for resources that are accessible from outside your "zone of trust" (your AWS Organization or account). This helps you proactively identify and remediate unintended access. It answers the question, "Who can access this from the outside?"

#### **3.2.5 Identity Federation with SAML 2.0**

-   **Concept:** Allows users in your corporate directory (e.g., Active Directory, Okta, Azure AD) to sign in to the AWS console or get programmatic credentials without needing to be individual IAM users.
-   **Flow:**
    1.  User browses to the Identity Provider (IdP) portal.
    2.  User authenticates with the IdP.
    3.  IdP sends a SAML assertion (a signed XML document) to the browser.
    4.  Browser posts the SAML assertion to the AWS sign-in endpoint.
    5.  AWS verifies the assertion, and uses the `sts:AssumeRoleWithSAML` action to allow the user to assume a pre-defined IAM Role.
    6.  User is granted temporary credentials for that role and redirected to the AWS console.
-   **Key Components:**
    -   **IdP:** Your source of truth for users (e.g., Okta).
    -   **IAM Identity Provider:** An entity in IAM that establishes trust with your external IdP.
    -   **IAM Role:** The role that federated users will assume. Its trust policy must trust the IAM Identity Provider.

---

### **3.3 Lab: Implementing ABAC and Permissions Boundaries**

**Objective:** Use advanced IAM features to implement a scalable and secure permissions model.

#### **Part A: Configure IAM for Attribute-Based Access Control (45 mins)**

1.  **Tag Your Principal (User/Role):**
    -   For this lab, we'll simulate a federated user. Create an IAM Role named `AppDeveloper`.
    -   In the role's tagging section, add a tag: `Key: Project, Value: Phoenix`.
2.  **Create an ABAC-driven Policy:**
    -   Create a new IAM policy named `EC2-Project-Access-Policy`.
    -   Use the JSON editor to insert the example policy from section 3.2.2, which allows `ec2:StopInstances` and `ec2:StartInstances` only when the principal's `Project` tag matches the resource's `Project` tag.
    -   Attach this policy to the `AppDeveloper` role.
3.  **Test the ABAC Policy:**
    -   Launch two `t2.micro` instances.
    -   Tag the first instance with `Project: Phoenix`.
    -   Tag the second instance with `Project: Dragon`.
    -   Assume the `AppDeveloper` role. From the console, verify that you can stop/start the "Phoenix" instance but are denied from stopping/starting the "Dragon" instance.

#### **Part B: Delegate Permissions with a Boundary (45 mins)**

1.  **Create the Permissions Boundary:**
    -   Create a new IAM managed policy named `LambdaDeveloperBoundary`.
    -   This policy should only allow essential Lambda and S3 actions (e.g., `lambda:CreateFunction`, `lambda:UpdateFunctionCode`, `s3:GetObject`, `s3:PutObject`). **Crucially, it must also allow `iam:PassRole` for a specific Lambda execution role.**
2.  **Create a Delegated Admin User:**
    -   Create a new IAM user named `CdkDeployUser`.
    -   Attach an inline policy to this user that allows them to create IAM roles (`iam:CreateRole`), but add a `Condition` that the new role *must* have the `LambdaDeveloperBoundary` attached.
    ```json
    "Condition": {
        "StringEquals": { "iam:PermissionsBoundary": "arn:aws:iam::ACCOUNT_ID:policy/LambdaDeveloperBoundary" }
    }
    ```
3.  **Test the Delegation:**
    -   Sign in as the `CdkDeployUser`.
    -   Attempt to create a new IAM role.
    -   First, try to create it *without* attaching the boundary. It should fail.
    -   Second, try to create it *with* the boundary, but give it `AdministratorAccess` permissions. The role will be created, but when you try to use it, you will find it can only perform the actions allowed by the boundary.

---

### **3.4 Teaching Notes**

-   **ABAC vs RBAC:** Use a clear analogy. RBAC is like giving someone a specific key to a specific room (`Admin` key, `Developer` key). ABAC is like giving them a master key that only works on doors with a matching colored sticker. ABAC is more flexible as you add more doors (resources) and people.
-   **Permissions Boundary is a Ceiling:** This is the best analogy. The boundary policy is the *ceiling*. The user's own policy is how high they can *reach*. Their effective permissions are how high they can reach, but no higher than the ceiling.
-   **`iam:PassRole` is Dangerous:** Spend extra time on this. A user with `iam:PassRole` and `ec2:RunInstances` can launch an EC2 instance with *any role they are allowed to pass*. If they can pass an admin role, they have effectively escalated their privileges to admin. This permission should always be scoped to specific, non-admin roles.
-   **Federation is the Goal:** In any enterprise environment, federation is the standard. IAM users should be reserved for service accounts and break-glass emergency access only. Frame the SAML discussion as the end-goal for enterprise identity management.
-   **Lab Complexity:** This lab is conceptually more difficult than the previous ones. Walk through the logic slowly, especially the "intersection" of permissions for the boundary lab. Use the IAM Policy Simulator live to show why actions are being denied or allowed.

---

## **Week 4: Advanced Storage & Data Protection**

**Duration:** 3 Hours (1 Hour Lecture, 2 Hours Lab)

**Core Topics:** EBS Deep Dive (Performance, RAID), EFS Deep Dive (Performance Modes, Lifecycle Management), AWS Backup, AWS DataSync, Introduction to Disaster Recovery.

---

### **4.1 Learning Objectives**

By the end of this session, students will be able to:
-   Provision and benchmark high-performance EBS volumes using RAID configurations.
-   Optimize EFS cost and performance using lifecycle management and provisioned throughput.
-   Design and implement a centralized, policy-based backup strategy using AWS Backup.
-   Automate data migration from on-premises or other clouds using AWS DataSync.
-   Compare and contrast different disaster recovery strategies on AWS.

---

### **4.2 Detailed Content**

#### **4.2.1 EBS Deep Dive: Maximizing Performance**

-   **Volume Types Review:**
    -   `gp3`: The new default. Provides baseline performance of 3,000 IOPS and 125 MiB/s throughput, which can be provisioned independently of storage size. More cost-effective than `gp2` for most workloads.
    -   `io2 Block Express`: The highest performance block storage, offering up to 256,000 IOPS and sub-millisecond latency. Designed for mission-critical databases like SAP HANA, Oracle, and SQL Server.
-   **EBS-Optimized Instances:** A feature that provides dedicated network capacity for I/O between EC2 instances and their EBS volumes. Essential for any I/O-intensive workload. Not all instance types support it or have it enabled by default.
-   **RAID Configurations on EBS:**
    -   **RAID 0 (Striping):** Spreads data across multiple EBS volumes.
        -   **Benefit:** Aggregates IOPS and throughput. Two 10,000 IOPS volumes become one 20,000 IOPS logical volume.
        -   **Drawback:** No redundancy. If one volume fails, the entire array is lost. Best for scratch disks or high-performance workloads where data can be easily regenerated.
    -   **RAID 1 (Mirroring):** Duplicates data across two or more EBS volumes.
        -   **Benefit:** High fault tolerance. The array can withstand the failure of one volume.
        -   **Drawback:** Write performance is limited to the speed of a single volume. Cost is doubled since storage is duplicated.

#### **4.2.2 EFS Deep Dive: Cost and Performance Optimization**

-   **Performance Modes:**
    -   **General Purpose (Default):** Ideal for latency-sensitive workloads like web serving, content management, and home directories.
    -   **Max I/O:** For large-scale, parallel workloads like big data analytics, media processing, and scientific computing. It sacrifices some latency for higher aggregate throughput.
-   **Throughput Modes:**
    -   **Bursting (Default):** Throughput scales with file system size. Good for spiky workloads.
    -   **Provisioned Throughput:** Set a specific level of throughput regardless of file system size. Use when your application needs more throughput than the bursting model provides for your data size.
-   **EFS Lifecycle Management:** Automatically and transparently moves files that haven't been accessed for a specified period (e.g., 30 days) from the standard storage class to the cost-optimized EFS Infrequent Access (IA) storage class. This is a key cost-saving feature.
-   **EFS Access Points:** Application-specific entry points into an EFS file system that enforce a user identity and a root directory for every file system request made through the access point. This allows multiple applications to securely share the same file system.

#### **4.2.3 Centralized Data Protection with AWS Backup**

-   **Concept:** A fully managed service that centralizes and automates data protection across AWS services. It replaces the need to write custom backup scripts and manage individual service backup features.
-   **Key Features:**
    -   **Central Console:** Manage backups for EBS, EFS, RDS, DynamoDB, Storage Gateway, and more from a single place.
    -   **Backup Plans:** Policy-based configuration. Define backup frequency, retention period, lifecycle rules (e.g., move to cold storage after 90 days), and target backup vault.
    -   **Backup Vaults:** Encrypted storage locations for your backups. `Backup Vault Lock` provides WORM (Write-Once-Read-Many) protection, making backups immutable, which is a key defense against ransomware.
    -   **Cross-Region & Cross-Account Backup:** A critical feature for disaster recovery and meeting compliance requirements.

#### **4.2.4 Large-Scale Data Migration with AWS DataSync**

-   **Purpose:** An online data transfer service that simplifies, automates, and accelerates moving large amounts of data between on-premises storage (NFS, SMB), other clouds, and AWS storage services (S3, EFS, FSx).
-   **How it Works:**
    -   Deploy a DataSync agent (a VM) in your on-premises environment.
    -   Create a transfer task in the AWS console, specifying the source and destination.
    -   DataSync automatically handles scripting transfers, scheduling, data integrity validation, and optimizing network utilization.

#### **4.2.5 Introduction to Disaster Recovery (DR) on AWS**

-   **Key Metrics:**
    -   **Recovery Time Objective (RTO):** The maximum acceptable delay between the interruption of service and restoration of service. "How long can we afford to be down?"
    -   **Recovery Point Objective (RPO):** The maximum acceptable amount of time since the last data recovery point. "How much data can we afford to lose?"
-   **Common Strategies (in order of increasing cost/complexity and decreasing RTO/RPO):**
    1.  **Backup and Restore:** Lowest cost, highest RTO/RPO. Back up data to S3, use CloudFormation to recreate infrastructure in a DR region after a disaster.
    2.  **Pilot Light:** A minimal version of the environment always runs in the DR region (e.g., the core database is replicated). After a disaster, scale up the application servers.
    3.  **Warm Standby:** A scaled-down but fully functional copy of the production environment always runs in the DR region. Traffic is not actively sent to it.
    4.  **Multi-Site Active/Active:** The application runs in multiple regions simultaneously, with traffic distributed between them. The most complex and expensive, but provides near-zero RTO/RPO.

---

### **4.3 Lab: Implementing a Resilient Storage and Backup Strategy**

**Objective:** Provision high-performance storage, configure a centralized backup plan, and test a basic disaster recovery scenario.

#### **Part A: High-Performance EBS with RAID 0 (45 mins)**

1.  **Provision Volumes:**
    -   Launch an EC2 instance.
    -   Create two 20 GiB `gp3` EBS volumes in the same Availability Zone.
2.  **Create RAID 0 Array:**
    -   Connect to the instance via SSM Session Manager.
    -   Use the `lsblk` command to identify the device names (e.g., `/dev/nvme1n1`, `/dev/nvme2n1`).
    -   Use `sudo mdadm --create --verbose /dev/md0 --level=0 --raid-devices=2 /dev/nvme1n1 /dev/nvme2n1` to create the RAID 0 array.
3.  **Format and Mount:**
    -   Format the new array: `sudo mkfs.ext4 -L RAID0 /dev/md0`.
    -   Create a mount point: `sudo mkdir -p /mnt/raid0`.
    -   Mount the array: `sudo mount LABEL=RAID0 /mnt/raid0`.
    -   Update `/etc/fstab` to make the mount persistent across reboots.
4.  **Benchmark (Optional):**
    -   Install a benchmarking tool like `fio` and run tests to observe the aggregated IOPS and throughput compared to a single volume.

#### **Part B: Centralize Backups with AWS Backup (45 mins)**

1.  **Create a Backup Vault:**
    -   Navigate to AWS Backup. Create a new backup vault named `ProductionVault`.
    -   **Crucially, apply a `Backup Vault Lock` in "Governance" mode.** This prevents deletion of backups for a specified period.
2.  **Create a Backup Plan:**
    -   Create a new backup plan named `Daily-Backups-Plan`.
    -   Define a backup rule: Run daily, start at 2 AM, and have a retention of 35 days.
    -   Set the lifecycle to move backups to cold storage after 7 days.
    -   Specify the destination as `ProductionVault`.
    -   Enable "Copy to region" and select a different region for disaster recovery.
3.  **Assign Resources:**
    -   Assign resources to this plan by tag. Create a new resource assignment that includes any resource with the tag `Backup-Policy: Daily`.
    -   Go to your EC2 instance from Part A and add this tag. Verify it appears in the protected resources list in AWS Backup.
4.  **Run an On-Demand Backup:**
    -   Initiate an on-demand backup job for the tagged EC2 instance. Monitor its progress and confirm it completes successfully in both the primary and DR regions.

---

### **4.4 Teaching Notes**

-   **`gp3` is the new `gp2`:** For any new workloads, students should default to `gp3`. It's cheaper and more performant at baseline. The main reason to know `gp2` is for legacy systems.
-   **RAID is a Trade-off:** RAID 0 gives performance but doubles the risk. RAID 1 gives redundancy but doubles the cost. It's a classic engineering trade-off. Emphasize that for databases, using a managed service like RDS is almost always better than building your own on EC2 with RAID.
-   **AWS Backup is a "Single Pane of Glass":** This is its main selling point. Instead of managing EBS snapshots, RDS snapshots, and EFS backups in three different places, you manage everything with one set of policies. The WORM capability of Vault Lock is a huge win for security and compliance.
-   **DR is a Business Decision:** RTO and RPO are not technical terms; they are business terms. The business decides how much downtime and data loss is acceptable, and the engineers design a solution to meet those requirements. The cost increases exponentially as RTO/RPO approach zero.
-   **Lab Realism:** The lab shows how to handle different types of requirements. Part A is for a niche, high-performance computing need. Part B is for the day-to-day, bread-and-butter operational task of ensuring all critical resources are backed up according to a consistent, centrally managed policy.

---

## **Week 5: VPC Design for Production**

**Duration:** 3 Hours (1.5 Hour Lecture, 1.5 Hours Lab)

**Core Topics:** Multi-AZ VPC Architecture, Subnet Design and IP Address Management (IPAM), Routing, NAT Gateways vs. NAT Instances, Security Layers (NACLs vs. SGs), VPC Flow Logs.

---

### **5.1 Learning Objectives**

By the end of this session, students will be able to:
-   Design a resilient, multi-AZ VPC architecture from scratch.
-   Implement a logical IP addressing scheme and subnetting strategy.
-   Configure complex routing to support public and private tiers.
-   Articulate the stateful vs. stateless nature of Security Groups and NACLs and apply them correctly.
-   Enable and analyze VPC Flow Logs to debug connectivity issues and monitor network traffic.

---

### **5.2 Detailed Content**

#### **5.2.1 Core Principle: Design for High Availability**

-   A production VPC should **always** span at least two Availability Zones (AZs).
-   Each AZ should contain a mirror set of subnets for each application tier. This ensures that the failure of a single AZ does not take down your entire application.
-   **Standard Architecture:**
    -   **VPC:** A large CIDR block (e.g., `10.0.0.0/16`).
    -   **AZ 1:**
        -   `Public Subnet 1`: For internet-facing resources like load balancers.
        -   `Private Subnet 1`: For application servers.
        -   `Data Subnet 1`: For databases.
    -   **AZ 2:**
        -   `Public Subnet 2`
        -   `Private Subnet 2`
        -   `Data Subnet 2`

#### **5.2.2 IP Address Management (IPAM) and Subnetting**

-   **VPC CIDR:** Choose a non-overlapping CIDR from the RFC 1918 private address ranges (`10.0.0.0/8`, `172.16.0.0/12`, `192.168.0.0/16`). A `/16` is a common, flexible choice.
-   **Subnetting:**
    -   Carve the VPC `/16` into smaller `/24` blocks for each subnet. A `/24` provides 251 usable IP addresses (AWS reserves 5 IPs per subnet).
    -   **Logical Naming Convention:**
        -   `10.0.1.0/24`: Public Subnet, AZ1
        -   `10.0.2.0/24`: Public Subnet, AZ2
        -   `10.0.101.0/24`: Private Subnet, AZ1
        -   `10.0.102.0/24`: Private Subnet, AZ2
        -   This makes routing and security rules much easier to read and manage.

#### **5.2.3 VPC Routing and Internet Access**

-   **Internet Gateway (IGW):** A horizontally-scaled, redundant, and highly available VPC component that allows communication between your VPC and the internet. One IGW per VPC.
-   **Route Tables:**
    -   **Public Route Table:** Associated with public subnets. Contains a route `0.0.0.0/0 -> igw-xxxxxxxx`. This directs all internet-bound traffic to the IGW.
    -   **Private Route Table(s):** Associated with private subnets. **Does not** have a route to the IGW.
-   **Egress-Only Internet Gateway:** For IPv6 traffic. Allows outbound communication from instances to the internet over IPv6, but prevents the internet from initiating a connection with your instances.
-   **NAT (Network Address Translation):**
    -   **NAT Gateway:** An AWS-managed NAT service. Highly available and scalable. Deployed into a public subnet and requires an Elastic IP. The preferred, modern choice.
    -   **NAT Instance:** A regular EC2 instance configured to perform NAT. You are responsible for its management, patching, and availability. Less common today, but useful for niche cases like traffic filtering or as a bastion host.
    -   **Private Routing to NAT:** Your private route tables will have a route `0.0.0.0/0 -> nat-xxxxxxxx` pointing to the NAT Gateway to allow private instances to access the internet for things like software updates.

#### **5.2.4 Layered Security: NACLs and Security Groups**

-   **Analogy:**
    -   **Network ACL (NACL):** Like the main security guard at the entrance to a building (subnet). They check everyone coming in and going out against a list.
    -   **Security Group (SG):** Like the keycard scanner on an individual office door (instance). It only cares about who is allowed in.
-   **Network ACLs:**
    -   **Stateless:** Return traffic must be explicitly allowed.
    -   **Operate at the Subnet Level:** A broad security control.
    -   **Rules:** Processed in numerical order. The first matching rule is applied.
    -   **Use Case:** A coarse-grained defense layer. Block known bad IP addresses at the subnet boundary. The default NACL allows all traffic. It's a best practice to keep it that way unless you have a specific need to block traffic at the subnet level.
-   **Security Groups:**
    -   **Stateful:** If you allow inbound traffic, the return traffic is automatically allowed.
    -   **Operate at the ENI/Instance Level:** A fine-grained security control.
    -   **Rules:** All rules are evaluated.
    -   **Use Case:** The primary mechanism for controlling traffic to your instances. Rules should be as specific as possible (e.g., allow port 443 from the Application Load Balancer's SG, not from `0.0.0.0/0`).

#### **5.2.5 VPC Flow Logs**

-   A feature that enables you to capture information about the IP traffic going to and from network interfaces in your VPC.
-   **Data Captured:** Source/Dest IP, Source/Dest Port, Protocol, Packet/Byte count, Action (ACCEPT/REJECT).
-   **Use Cases:**
    -   **Troubleshooting:** "Why can't my web server talk to my database?" Flow Logs will show if the traffic is being REJECTED by a Security Group or NACL.
    -   **Security Analysis:** Identifying unexpected traffic patterns or connections from suspicious IP addresses.
-   **Destination:** Can be published to CloudWatch Logs or Amazon S3.

---

### **5.3 Lab: Architecting and Building a Production VPC**

**Objective:** To design and implement a multi-AZ, three-tier VPC that will serve as the foundation for the course project.

#### **Part A: Design and VPC Creation (30 mins)**

1.  **IPAM Plan:**
    -   On paper or in a text file, plan your IP space.
    -   VPC CIDR: `10.10.0.0/16`
    -   Subnets (all `/24`):
        -   `us-east-1a`: Public (`10.10.1.0/24`), Private (`10.10.101.0/24`), Data (`10.10.201.0/24`)
        -   `us-east-1b`: Public (`10.10.2.0/24`), Private (`10.10.102.0/24`), Data (`10.10.202.0/24`)
2.  **Create the VPC and Subnets:**
    -   Use the "VPC and more" wizard in the AWS console to simplify creation.
    -   Specify your VPC CIDR and select 2 Availability Zones.
    -   Configure 2 public and 4 private subnets (we'll treat two of the private subnets as our "Data" tier).
    -   Allow AWS to create the route tables and IGW. Name everything according to your plan.

#### **Part B: Routing and NAT Configuration (30 mins)**

1.  **Create a NAT Gateway:**
    -   Navigate to NAT Gateways and create one.
    -   Place it in your `Public Subnet` in AZ `us-east-1a`.
    -   Allocate a new Elastic IP for it.
2.  **Configure Route Tables:**
    -   Examine the **Public Route Table**. Verify it has a `0.0.0.0/0` route to the IGW.
    -   Create a new **Private Route Table**.
    -   Add a `0.0.0.0/0` route pointing to the NAT Gateway you created.
    -   Associate this new Private Route Table with your two **Private Subnets** (`10.10.101.0/24` and `10.10.102.0/24`).

#### **Part C: Security Layering and Verification (30 mins)**

1.  **Create Tiered Security Groups:**
    -   `Web-SG`: Allows inbound HTTP (80) and HTTPS (443) from `0.0.0.0/0`.
    -   `App-SG`: Allows inbound TCP on port 8080 *only* from the `Web-SG`. (Use the SG ID as the source).
    -   `DB-SG`: Allows inbound TCP on port 3306 (MySQL) *only* from the `App-SG`.
2.  **Launch Test Instances:**
    -   Launch an EC2 instance in a **Public Subnet** with the `Web-SG`.
    -   Launch an EC2 instance in a **Private Subnet** with the `App-SG`.
3.  **Verify Connectivity:**
    -   From the public instance, confirm you can `curl` the private instance's IP on port 8080.
    -   From the private instance, confirm you can `curl http://www.google.com` (testing the NAT Gateway).
    -   From the public instance, try to `curl` the private IP on port 22 (SSH). It should fail, proving the SG works.

---

### **5.4 Teaching Notes**

-   **VPC Wizard is Your Friend (for learning):** While production VPCs should be built with IaC (which we'll do later), the "VPC and more" wizard is an excellent teaching tool. It visualizes the architecture and handles a lot of the boilerplate routing setup, allowing students to focus on the design.
-   **SG Chaining is Key:** The concept of sourcing a security group rule from another security group ID (not a CIDR range) is fundamental to multi-tier security. It creates a dynamic, logical relationship between tiers.
-   **Stateful vs. Stateless Analogy:** Use the building/office door analogy. It's very effective. Draw it on the whiteboard. For NACLs, draw arrows for traffic both *in* and *out* of the building, each needing a rule. For SGs, just draw an arrow *in* to the office, and explain the exit is implicitly allowed.
-   **Flow Logs for Debugging:** Frame Flow Logs as your ultimate debugging tool for L3/L4 connectivity. When a student says "I can't connect!", the first question should be "What do the Flow Logs say?". This builds good operational habits.
-   **This Lab is Foundational:** This VPC will be used for the rest of the course. Emphasize the importance of getting it right. A well-designed network makes everything that follows easier. A poorly designed one creates constant problems.

---

## **Week 6: Advanced VPC Connectivity & DNS**

**Duration:** 3 Hours (1.5 Hour Lecture, 1.5 Hours Lab)

**Core Topics:** VPC Endpoints (Gateway and Interface), VPC Peering, Transit Gateway, Route 53 Private Hosted Zones, DNS Resolution in Hybrid Scenarios.

---

### **6.1 Learning Objectives**

By the end of this session, students will be able to:
-   Securely connect a VPC to AWS services without traversing the internet using VPC Endpoints.
-   Differentiate between Gateway and Interface endpoints and their use cases.
-   Analyze the trade-offs between VPC Peering and Transit Gateway for inter-VPC communication.
-   Architect a hub-and-spoke network model with Transit Gateway.
-   Implement private DNS resolution using Route 53 Private Hosted Zones.

---

### **6.2 Detailed Content**

#### **6.2.1 Securing Service Access with VPC Endpoints**

-   **Problem:** By default, when an EC2 instance in a private subnet calls an AWS API (e.g., S3), the traffic goes out through the NAT Gateway to the public internet endpoint for that service. This is inefficient and less secure.
-   **Solution:** VPC Endpoints create a private connection between your VPC and supported AWS services, powered by AWS PrivateLink. Traffic never leaves the AWS network.
-   **Gateway Endpoints:**
    -   **Services:** S3 and DynamoDB only.
    -   **Mechanism:** A gateway that you specify as a target in your route table. It's a regional service.
    -   **Cost:** No additional charge.
    -   **Security:** Controlled via IAM policies and Endpoint Policies (a type of resource policy attached to the endpoint).
-   **Interface Endpoints:**
    -   **Services:** Most other AWS services (SSM, Kinesis, CloudWatch, etc.).
    -   **Mechanism:** An Elastic Network Interface (ENI) with a private IP address from your subnet's IP range. It acts as the entry point for traffic destined for the service.
    -   **Cost:** Charged per hour per AZ, plus a per-GB data processing charge.
    -   **Security:** Controlled by Security Groups attached to the ENI, as well as Endpoint Policies.

#### **6.2.2 Inter-VPC Communication: Peering vs. Transit Gateway**

-   **VPC Peering:**
    -   **Concept:** A one-to-one network connection between two VPCs.
    -   **Characteristics:**
        -   Traffic uses private IP addresses and stays on the AWS global backbone.
        -   **Non-Transitive:** If VPC A is peered with B, and B is peered with C, A cannot talk to C through B.
    -   **Use Case:** Simple scenarios involving a small number of VPCs that need to communicate directly. Becomes unmanageable (a "mesh" of connections) with many VPCs.
-   **AWS Transit Gateway (TGW):**
    -   **Concept:** A highly scalable, regional network transit hub that can connect thousands of VPCs and on-premises networks.
    -   **Characteristics:**
        -   Acts as a cloud router. Each attachment (VPC or VPN) connects to the TGW.
        -   **Transitive Routing:** Any VPC connected to the TGW can potentially communicate with any other connected VPC (controlled by TGW route tables).
        -   Simplifies network management by creating a "hub and spoke" model instead of a complex mesh.
    -   **Use Case:** The standard for any organization with more than a handful of VPCs, or for connecting on-premises networks to multiple VPCs.

#### **6.2.3 Advanced DNS with Route 53**

-   **Public Hosted Zones:** Manage public DNS records for your domains (e.g., `www.mycoolapp.com`).
-   **Private Hosted Zones:**
    -   **Concept:** Manage internal DNS for your VPCs. You can use custom domain names (e.g., `api.prod.internal`) that are only resolvable from within the associated VPCs.
    -   **Benefit:** Decouples your application from hard-coded private IP addresses. Your web server can call `rds.prod.internal` instead of `10.10.201.54`.
-   **Route 53 Resolver:**
    -   **Inbound Endpoints:** Allow DNS queries from your on-premises network to be resolved by Route 53 (for both public records and Private Hosted Zones).
    -   **Outbound Endpoints:** Allow instances in your VPC to resolve DNS names in your on-premises network by forwarding queries to your on-premises DNS servers.
    -   This is the key service for creating a seamless, hybrid DNS architecture.

---

### **6.3 Lab: Building a Hub-and-Spoke Network with Private Connectivity**

**Objective:** To connect multiple VPCs via a Transit Gateway and secure service access using VPC Endpoints.

#### **Part A: Create a "Spoke" VPC (30 mins)**

1.  **Create a Second VPC:**
    -   Using the VPC wizard, create a new VPC named `Spoke-VPC-1`.
    -   Give it a non-overlapping CIDR: `10.20.0.0/16`.
    -   Configure it with one public and one private subnet in a single AZ to keep it simple.
2.  **Launch a Test Instance:**
    -   Launch a `t2.micro` instance in the **private subnet** of `Spoke-VPC-1`. This instance should have no internet access initially.
    -   Give it the `AmazonSSMManagedInstanceCore` role so we can connect later.

#### **Part B: Implement a Transit Gateway Hub (45 mins)**

1.  **Create the Transit Gateway:**
    -   Navigate to the VPC console -> Transit Gateways. Create a new TGW. Accept the defaults.
2.  **Create TGW Attachments:**
    -   Create an attachment for your main project VPC from last week (`10.10.0.0/16`).
    -   Create a second attachment for `Spoke-VPC-1` (`10.20.0.0/16`).
    -   Wait for both attachments to become available.
3.  **Configure Routing:**
    -   In the **Private Route Table** of your main VPC, add a route: `Destination: 10.20.0.0/16`, `Target: <your-tgw-id>`.
    -   In the **Private Route Table** of `Spoke-VPC-1`, add a route: `Destination: 10.10.0.0/16`, `Target: <your-tgw-id>`.
4.  **Verify Connectivity:**
    -   Launch a test instance in the private subnet of your main VPC.
    -   From this instance, ping the private IP of the instance in `Spoke-VPC-1`. The ping should succeed, proving the TGW is routing traffic between the VPCs.

#### **Part C: Secure S3 Access with a Gateway Endpoint (45 mins)**

1.  **Create an S3 Bucket:** Create a new S3 bucket with a unique name. Block all public access. Upload a sample file (`test.txt`).
2.  **Create the Gateway Endpoint:**
    -   In the VPC console -> Endpoints, create a new endpoint.
    -   Select "AWS services" and search for the S3 service with the "Gateway" type.
    -   Associate the endpoint with your main project VPC.
    -   Select your **Private Route Table** to be modified by the endpoint. This will automatically add specific routes for S3's public IP ranges to the endpoint.
3.  **Attach an Endpoint Policy:**
    -   In the policy section, choose "Custom" and write a policy that only allows `s3:GetObject` actions on your specific bucket. This prevents exfiltration of data to other S3 buckets.
4.  **Verify Private Access:**
    -   Connect to the private instance in your main VPC (which has no internet access via NAT).
    -   Using the AWS CLI, try to download the file: `aws s3 cp s3://<your-bucket-name>/test.txt .`
    -   The command should succeed, proving you accessed S3 without going over the internet.
    -   Try to list all buckets (`aws s3 ls`). This should fail due to the restrictive endpoint policy.

---

### **6.4 Teaching Notes**

-   **Endpoint Cost-Benefit:** Gateway endpoints are a "no-brainer" - they are free and more secure. Always use them for S3 and DynamoDB. Interface endpoints have a cost, so the decision is a trade-off. For frequent, high-volume traffic to AWS services (like SSM or Kinesis), the cost is often justified by the improved security and potentially lower NAT Gateway data processing fees.
-   **TGW is the Scalable Choice:** Use a diagram to show a 4-VPC mesh (6 peering connections) vs. a 4-VPC hub-and-spoke (4 TGW attachments). The visual makes the scalability benefit of TGW immediately obvious.
-   **Route Tables are the Control Plane:** Emphasize that creating a TGW attachment or a peering connection does nothing by itself. You *must* update the VPC subnet route tables to direct traffic to the T-G-W or the peering connection for it to work. This is a common point of confusion.
-   **Private DNS is for Decoupling:** Just like public DNS, Private Hosted Zones are about creating stable, human-readable names that can be pointed to different resources without changing application code. It's a fundamental best practice for building resilient systems.
-   **Lab Progression:** This lab builds directly on the last one. Students now have a "production" VPC and a "spoke" VPC (which could represent a different business unit, a dev environment, or a partner VPC). They connect them with a TGW, which is a very realistic enterprise pattern. Securing S3 access is a foundational security task.

---

## **Part 3: Application Architecture & Deployment**

---

## **Week 7: Building a Resilient & Scalable Web Tier**

**Duration:** 3 Hours (1 Hour Lecture, 2 Hours Lab)

**Core Topics:** Application Load Balancer (ALB) Deep Dive, Target Groups & Health Checks, Auto Scaling Groups (ASGs), Launch Templates, Dynamic Scaling Policies.

---

### **7.1 Learning Objectives**

By the end of this session, students will be able to:
-   Architect a highly available web front-end using an Application Load Balancer across multiple AZs.
-   Configure advanced ALB features like host-based routing and listener rules.
-   Implement robust health checks to automatically detect and replace unhealthy instances.
-   Create a scalable application tier using Auto Scaling Groups and Launch Templates.
-   Configure dynamic scaling policies to automatically adjust capacity based on real-time load.

---

### **7.2 Detailed Content**

#### **7.2.1 Application Load Balancer (ALB) Architecture**

-   **Layer 7 Awareness:** Unlike a Network Load Balancer (Layer 4), an ALB understands HTTP/HTTPS traffic. This allows for more intelligent routing decisions.
-   **Core Components:**
    -   **Listeners:** Check for connection requests on a specific protocol and port (e.g., HTTPS on 443). Each listener has a set of rules.
    -   **Rules:** Evaluated in order of priority. A rule consists of a *condition* (e.g., path is `/api/*` or host header is `api.example.com`) and an *action* (e.g., forward to a specific target group).
    -   **Target Groups:** A group of registered targets (e.g., EC2 instances) that receive traffic from the ALB. Each target group has its own health check configuration.
-   **Advanced Routing:**
    -   **Path-Based Routing:** Send requests for `/images/*` to an EC2 fleet, and `/api/*` to a Fargate container fleet.
    -   **Host-Based Routing:** Send requests for `orders.example.com` to one target group, and `catalog.example.com` to another, all using the same ALB.
-   **Security Features:**
    -   **SSL/TLS Termination:** Offloads the work of encrypting/decrypting traffic from your backend instances. Integrates with AWS Certificate Manager (ACM) for free, auto-renewing certificates.
    -   **WAF Integration:** Can be directly integrated with AWS WAF to protect against common web exploits.

#### **7.2.2 Health Checks: The Key to Self-Healing**

-   **Mechanism:** The ALB periodically sends requests to each registered target on a specified port and path to determine its health.
-   **Configuration:**
    -   **Health Check Protocol/Port/Path:** Where to send the health check ping (e.g., HTTP on port 80 to `/health-check`).
    -   **Healthy/Unhealthy Threshold:** The number of consecutive successes/failures required to change the target's status.
    -   **Timeout & Interval:** How long to wait for a response and how often to perform the check.
-   **Auto Scaling Integration:** If a target fails its health checks, the ALB stops sending it traffic. If that target is part of an ASG, the ASG will see the instance is unhealthy and terminate it, launching a new, healthy replacement. This is the core of AWS's self-healing capability.

#### **7.2.3 Auto Scaling Groups (ASGs) and Launch Templates**

-   **Launch Templates vs. Launch Configurations (Legacy):**
    -   **Launch Configurations:** Older, immutable. To change anything, you must create a new one.
    -   **Launch Templates:** Newer, versioned. You can create new versions of a template (e.g., to use a new AMI) and update the ASG to use the latest version. **Launch Templates are the recommended best practice.**
-   **ASG Configuration:**
    -   **Min/Max/Desired Capacity:** Defines the boundaries of the group. The ASG will always work to maintain the `Desired` number of healthy instances.
    -   **Health Check Type:** Can be `EC2` (instance status checks) or `ELB`. For web applications, **always use `ELB`**. This ensures the instance is not just running, but is actually passing the application-level health check.
    -   **Termination Policies:** Determines which instance to terminate during a scale-in event (e.g., `OldestInstance`, `ClosestToNextInstanceHour`).

#### **7.2.4 Dynamic Scaling Policies**

-   **Simple Scaling:** "If CPU > 70%, add 1 instance." (Legacy, generally not recommended due to potential for rapid fluctuations).
-   **Step Scaling:** "If CPU is between 70-90%, add 1 instance. If CPU > 90%, add 3 instances." More nuanced than simple scaling.
-   **Target Tracking Scaling (Recommended):** The simplest and most effective method. "Keep the average CPU utilization for the group at 50%." The ASG automatically calculates how many instances to add or remove to stay at the target value. You can also track memory, network I/O, or even a custom metric from a load balancer (like `RequestCountPerTarget`).

---

### **7.3 Lab: Deploying a Highly Available and Scalable Web Application**

**Objective:** To place a fleet of web servers behind an Application Load Balancer and configure an Auto Scaling Group to automatically manage capacity.

#### **Part A: Create a Launch Template for Web Servers (30 mins)**

1.  **Use the Golden AMI:** Find the custom "Golden AMI" you created with EC2 Image Builder in Week 2.
2.  **Create Launch Template:**
    -   Navigate to EC2 -> Launch Templates. Create a new one named `WebServer-LT`.
    -   Select your Golden AMI as the source.
    -   Choose `t2.micro` as the instance type.
    -   Assign the `Web-SG` security group you created in Week 5.
    -   **Crucially, assign the IAM instance profile with `AmazonSSMManagedInstanceCore` permissions.**

#### **Part B: Configure the Application Load Balancer (45 mins)**

1.  **Create a Target Group:**
    -   Go to EC2 -> Target Groups. Create a new target group for EC2 instances.
    -   Name it `Web-App-TG`. Protocol should be HTTP on port 80.
    -   Select your production VPC.
    -   **Configure Health Checks:** Set the health check path to `/index.html`. Lower the interval to 10 seconds and the healthy threshold to 2 for faster detection during the lab.
2.  **Create the ALB:**
    -   Go to EC2 -> Load Balancers. Create a new Application Load Balancer.
    -   Name it `WebApp-ALB`. Scheme should be `internet-facing`.
    -   Select your production VPC.
    -   Map it to your two **public subnets**.
    -   Assign the `Web-SG` security group.
    -   **Create a Listener:** For HTTP on port 80, set the default action to "Forward to" your `Web-App-TG`.

#### **Part C: Create and Test the Auto Scaling Group (45 mins)**

1.  **Create the ASG:**
    -   Go to EC2 -> Auto Scaling Groups. Create a new ASG.
    -   Name it `WebServer-ASG`.
    -   Select the `WebServer-LT` Launch Template you created.
    -   Select your two **private subnets**.
    -   **Attach the Load Balancer:** Choose to attach to an existing load balancer and select your `Web-App-TG`.
    -   **Enable ELB Health Checks.** This is a critical step.
    -   Set group size: Min 2, Desired 2, Max 4.
2.  **Configure Target Tracking Scaling:**
    -   Add a scaling policy. Choose "Target tracking".
    -   Set the metric to `Average CPU Utilization` with a target value of `50`.
3.  **Test High Availability:**
    -   Wait for the ASG to launch two instances and for the ALB to report them as healthy.
    -   Get the DNS name of the ALB and access it in a browser. You should see your Nginx page.
    -   Manually terminate one of the instances. Observe the ASG automatically launch a new replacement to maintain the desired capacity of 2.
4.  **Test Scaling (Stress Test):**
    -   Connect to one of the instances via Session Manager.
    -   Run the `stress` command you installed in the AMI: `sudo stress --cpu 8 --timeout 300s`.
    -   Watch the CloudWatch metrics for the ASG. As the average CPU climbs above 50%, the ASG will launch one or two new instances to bring the average back down. Once the stress test stops, the ASG will terminate the extra instances.

---

### **7.4 Teaching Notes**

-   **ALB in Public, EC2 in Private:** This is a core architectural pattern. The ALB is the public-facing endpoint. The EC2 instances that run the application code should be in private subnets, protected from direct internet access. The only traffic they should receive is from the load balancer.
-   **Launch Templates are Key to Immutability:** Frame Launch Templates as the definition of your application server. When you need to update (e.g., new AMI, new instance type), you create a *new version* of the template and roll it out to the ASG. You don't SSH in and change running instances (the "pets vs. cattle" analogy).
-   **Health Checks are Everything:** A system is not "highly available" without robust health checks. An instance that is running but has a frozen application is useless. The ALB health check validates that the application itself is responsive, which is why the `ELB` health check type in the ASG is so important.
-   **Target Tracking is "Set and Forget":** For most common scaling needs, Target Tracking is the best choice. It's declarative ("I want 50% CPU") rather than imperative ("add 2 instances"). It's less work to set up and less prone to misconfiguration.
-   **Visualizing the Test:** The stress test is a powerful demonstration. Have the ASG history, EC2 instances list, and ALB target group health status all visible on screen at once. Students can see the CPU alarm trigger, the scaling activity start, the new instances launch, get registered with the ALB, become healthy, and start taking traffic. It's a "wow" moment that brings all the concepts together.

---

## **Week 8: Advanced Observability & Operations**

**Duration:** 3 Hours (1.5 Hour Lecture, 1.5 Hours Lab)

**Core Topics:** CloudWatch Unified Agent, Centralized Logging with CloudWatch Logs, Custom Metrics, CloudWatch Alarms & Composite Alarms, Introduction to Distributed Tracing with AWS X-Ray.

---

### **8.1 Learning Objectives**

By the end of this session, students will be able to:
-   Deploy and configure the CloudWatch Unified Agent to collect detailed system-level metrics and application logs.
-   Analyze structured logs and create metrics from log data using Metric Filters.
-   Design sophisticated alarming strategies using composite alarms to reduce alert fatigue.
-   Instrument a simple application to publish custom business-level metrics to CloudWatch.
-   Integrate AWS X-Ray to trace requests as they flow through multiple services in the application.

---

### **8.2 Detailed Content**

#### **8.2.1 Deep Monitoring with the CloudWatch Unified Agent**

-   **Beyond Basic Metrics:** Standard EC2 metrics provide CPU, basic Network, and Disk I/O. They do **not** provide Memory Usage, Disk Space Used, or other OS-level metrics.
-   **Unified Agent:** A single agent that can collect both system-level metrics and logs. It is highly configurable.
    -   **Metrics Collection:** Can collect dozens of pre-defined metrics (e.g., `mem_used_percent`, `disk_used_percent`).
    -   **Log Collection:** Can tail multiple log files and send them to CloudWatch Logs.
-   **Configuration:** The agent's behavior is defined by a JSON configuration file, which can be stored in SSM Parameter Store for easy, centralized management and deployment to a fleet of instances.

#### **8.2.2 Centralized & Structured Logging**

-   **CloudWatch Logs Concepts:**
    -   **Log Group:** A container for log streams (e.g., `/var/log/nginx/access.log`).
    -   **Log Stream:** A sequence of log events from a single source (e.g., from a specific instance).
-   **Structured Logging:** The practice of writing logs in a machine-readable format like JSON.
    -   **Why?** Unstructured logs (`INFO: User 123 logged in.`) are hard to search and analyze programmatically.
    -   Structured logs (`{"level": "INFO", "message": "User login", "userId": "123"}`) can be easily queried.
-   **CloudWatch Logs Insights:** A powerful query engine for searching and analyzing log data. Uses a purpose-built query language. You can run queries like:
    `fields @timestamp, @message | filter level == 'ERROR' | sort @timestamp desc | limit 20`
-   **Metric Filters:** Scan incoming log events for patterns (e.g., the word "Error" or a 5xx status code) and increment a CloudWatch custom metric. This allows you to alarm on events happening inside your application logs.

#### **8.2.3 Advanced Alarming Strategies**

-   **Standard Alarms:** Watch a single metric (e.g., CPUUtilization) and trigger when it crosses a threshold.
-   **Problem:** A single issue can cause many metrics to go into an alarm state, resulting in an "alert storm" that overwhelms operators.
-   **Composite Alarms:**
    -   **Concept:** Combine multiple other alarms into a single, higher-level alarm with a boolean (`AND`, `OR`, `NOT`) rule.
    -   **Use Case:** You only want to be paged if the Application Load Balancer is reporting 5xx errors **AND** the average CPU of the fleet is high **AND** the database latency is elevated.
    -   **Benefit:** Greatly reduces alert fatigue by only notifying on correlated failures that represent a real, user-impacting outage, rather than minor, transient issues.

#### **8.2.4 Custom Metrics: Measuring What Matters**

-   **Why?** Standard infrastructure metrics (CPU, memory) tell you if the *system* is healthy. They don't tell you if the *business* is healthy.
-   **What to Measure:** Business-level KPIs (Key Performance Indicators) like `NewUserSignups`, `ShoppingCartValue`, or `SentimentAnalysisJobsProcessed`.
-   **How:** Use the `PutMetricData` API call from the AWS SDK within your application code.
-   **Dimensions:** Add key-value pairs to a metric to allow for filtering (e.g., publish `SentimentAnalysisJobsProcessed` with a dimension `Language: English`). This allows you to graph and alarm on English jobs separately from Spanish jobs.

#### **8.2.5 Introduction to Distributed Tracing with AWS X-Ray**

-   **The Problem with Microservices:** In a distributed system, a single user request might flow through an ALB, a front-end service, a backend API, and a database. If the request is slow, where is the bottleneck?
-   **Solution:** X-Ray provides end-to-end tracing. It follows a request as it travels through your application components.
-   **How it Works:**
    -   The X-Ray SDK (integrated into your application) adds a unique **Trace ID** to the request headers.
    -   Each service that processes the request adds its own **Segment** of data (e.g., start time, end time, any errors) to the trace.
    -   X-Ray collects all segments for a given Trace ID and visualizes them as a timeline, creating a **Service Map** that shows the flow of requests and highlights bottlenecks and errors.

---

### **8.3 Lab: Implementing a Full Observability Stack**

**Objective:** To configure deep monitoring, centralized logging, and automated alerting for the web tier.

#### **Part A: Deploy and Configure the CloudWatch Agent (45 mins)**

1.  **Create Agent Configuration:**
    -   Create a JSON configuration file for the agent. It should:
        -   Collect memory and disk space metrics.
        -   Collect the Nginx access log from `/var/log/nginx/access.log`.
    -   Store this JSON file in SSM Parameter Store with the name `CloudWatch-Agent-Config-Web`.
2.  **Attach IAM Permissions:**
    -   Create a new IAM policy that allows `ssm:GetParameter` and `cloudwatch:PutMetricData`.
    -   Attach this policy to the IAM role used by your `WebServer-ASG` instances.
3.  **Install and Run the Agent:**
    -   Use SSM Run Command (`AWS-ConfigureAWSPackage`) to install the `AmazonCloudWatchAgent` on all instances in your `WebServer-ASG` (target them by tag).
    -   Use another SSM Run Command to start the agent, telling it to fetch its configuration from the SSM Parameter Store parameter you created.
4.  **Verify:**
    -   Navigate to CloudWatch -> Metrics. You should see a new custom namespace (`CWAgent`) with memory and disk metrics for your instances.
    -   Navigate to CloudWatch -> Log Groups. You should see a new log group for the Nginx access logs.

#### **Part B: Create Alarms and Dashboards (45 mins)**

1.  **Create a Metric Filter:**
    -   Go to your Nginx access log group.
    -   Create a metric filter that searches for `500` status codes.
    -   Have this filter publish to a new custom metric named `Nginx-5xx-Errors`.
2.  **Create a Composite Alarm:**
    -   Create a standard alarm that triggers if `Nginx-5xx-Errors` > 0.
    -   Create another standard alarm that triggers if the `Average CPUUtilization` for the ASG > 80%.
    -   Create a **Composite Alarm** named `High-Priority-WebApp-Failure`.
    -   Set the rule to `ALARM(Nginx-5xx-Errors) AND ALARM(High-CPU)`.
    -   Configure this composite alarm to send a notification to an SNS topic.
3.  **Create a Dashboard:**
    -   Create a new CloudWatch Dashboard named `WebApp-Overview`.
    -   Add widgets for:
        -   Average CPU Utilization of the ASG.
        -   ALB Request Count and 5xx Errors.
        -   The `Nginx-5xx-Errors` custom metric.
        -   Memory and Disk Space percentage from the CWAgent.

---

### **8.4 Teaching Notes**

-   **Observability vs. Monitoring:** Monitoring is about observing pre-defined metrics (CPU, Memory). Observability is about being able to ask arbitrary questions about your system's state from the outside, which requires rich data from logs, metrics, and traces. The CloudWatch Agent is the first step towards true observability.
-   **Structured Logs are a Superpower:** Emphasize this. Once logs are in JSON, you can use Logs Insights for powerful, SQL-like querying that is impossible with plain text. This transforms logs from a forensic tool into a proactive analytics tool.
-   **Composite Alarms Reduce Noise:** Tell a story about being on-call at 3 AM and getting 50 separate alerts for one database failure. Composite alarms are the cure for this. They allow you to define what a *real* outage looks like for your application and only alert on that.
-   **X-Ray is for Finding the "Why":** CloudWatch tells you *what* is broken (high latency). X-Ray tells you *why* it's broken (the backend call to the DynamoDB table is slow). It provides the context that metrics alone lack.
-   **The Lab is an Operational Foundation:** The components built in this lab (agent config in SSM, metric filters, composite alarms, dashboards) are reusable patterns that apply to almost any application running on AWS. This gives students a real-world operational toolkit.

---

## **Week 9: Building a Production Data Tier**

**Duration:** 3 Hours (1 Hour Lecture, 2 Hours Lab)

**Core Topics:** Amazon RDS Deep Dive (Multi-AZ, Read Replicas), Performance Insights, AWS Secrets Manager, Database Proxy.

---

### **9.1 Learning Objectives**

By the end of this session, students will be able to:
-   Architect a highly available and fault-tolerant database tier using RDS Multi-AZ.
-   Scale database read performance using RDS Read Replicas.
-   Analyze and troubleshoot database performance issues with Performance Insights.
-   Securely store, retrieve, and rotate database credentials using AWS Secrets Manager.
-   Improve database efficiency and resilience by implementing RDS Proxy.

---

### **9.2 Detailed Content**

#### **9.2.1 Amazon RDS for High Availability**

-   **RDS as a Managed Service:** RDS handles the undifferentiated heavy lifting of running a relational database: provisioning, OS patching, database software patching, backups, and high availability.
-   **Multi-AZ Deployment:**
    -   **Mechanism:** RDS provisions and maintains a synchronous standby replica of your primary database in a *different Availability Zone*.
    -   **Failover:** In case of primary database failure or an AZ outage, RDS automatically fails over to the standby replica. The DNS endpoint for your database remains the same; the CNAME record is simply updated to point to the new primary.
    -   **Benefit:** This provides high availability and data durability. It is **not** a scaling solution. You cannot connect to the standby for read traffic.
-   **Backup and Recovery:**
    -   **Automated Backups:** Enabled by default. Provides daily snapshots and transaction logs, allowing for point-in-time recovery (PITR) down to the second (within your retention window).
    -   **Manual Snapshots:** User-initiated backups that are kept until you explicitly delete them. Useful for archival or creating copies of a database for development or testing.

#### **9.2.2 Scaling Database Reads with Read Replicas**

-   **Concept:** A Read Replica is an *asynchronously* replicated, read-only copy of your primary database.
-   **Purpose:** To scale read-heavy workloads. You can direct application read traffic (e.g., `SELECT` queries) to one or more read replicas, reducing the load on the primary instance, which is then freed up to handle write traffic.
-   **Key Features:**
    -   Can be in the same region, a different region (for cross-region disaster recovery or lower latency for global users), or even in a different AWS account.
    -   Can be promoted to a standalone, writable primary database if needed (a manual process).
    -   Each read replica has its own unique DNS endpoint.

#### **9.2.3 Diagnosing Issues with Performance Insights**

-   **Concept:** A database performance tuning and monitoring feature that provides a simple dashboard to visualize database load.
-   **Key Metric: `DB Load`**
    -   The y-axis of the main graph. It represents the number of active sessions for your database. A session is active if it's running on CPU or waiting for a resource (e.g., waiting for a lock, waiting for I/O).
    -   By breaking down the DB Load by `waits` (what are sessions waiting for?), `SQL` (which queries are causing the load?), and `hosts` (where is the load coming from?), you can quickly pinpoint performance bottlenecks.

#### **9.2.4 Secure Credential Management with AWS Secrets Manager**

-   **The Problem:** Storing database passwords in application configuration files or environment variables is a major security risk.
-   **Solution: Secrets Manager**
    -   Provides a centralized, secure location to store secrets.
    -   Integrates with IAM for fine-grained access control to secrets.
    -   **Automatic Rotation:** Can automatically rotate credentials for supported services (like RDS) on a schedule without requiring application restarts or manual intervention.
    -   **How it Works:** Your application is granted IAM permission to call the `GetSecretValue` API. It retrieves the credentials at runtime, in memory, and uses them to connect to the database.

#### **9.2.5 Improving Connection Management with RDS Proxy**

-   **The Problem:** Applications, especially serverless ones (Lambda), can open and close database connections very frequently, which can exhaust database memory and CPU.
-   **Solution: RDS Proxy**
    -   A fully managed, highly available database proxy that sits between your application and your RDS database.
    -   **Connection Pooling:** It maintains a "pool" of established connections to the database and shares them across application requests, dramatically reducing the overhead of opening new connections.
    -   **Improved Failover:** During a database failover, the proxy can hold incoming application requests and automatically connect to the new primary database once it's available, making failovers faster and more transparent to the application.

---

### **9.3 Lab: Deploying a Secure and Resilient Database Tier**

**Objective:** To provision a Multi-AZ RDS database, secure its credentials with Secrets Manager, and connect to it from a private application server.

#### **Part A: Provision a Multi-AZ RDS Instance (45 mins)**

1.  **Create a DB Subnet Group:**
    -   Navigate to the RDS console -> Subnet Groups.
    -   Create a new subnet group named `project-db-sng`.
    -   Add the two **Data Subnets** (`10.10.201.0/24`, `10.10.202.0/24`) from your production VPC to this group.
2.  **Launch RDS Instance:**
    -   Go to Databases -> Create database.
    -   Choose "Standard Create" and select MySQL.
    -   Select a template for "Production".
    -   Give it an identifier like `project-db`.
    -   Set a master password.
    -   Choose a `db.t3.micro` instance class.
    -   **Crucially, under "Availability & durability", select "Create a standby instance" to enable Multi-AZ.**
    -   Select your production VPC and the `project-db-sng` subnet group.
    -   For "VPC security group", choose the `DB-SG` you created in Week 5.
    -   Enable Performance Insights.

#### **Part B: Secure Credentials with Secrets Manager (30 mins)**

1.  **Store the Secret:**
    -   Navigate to AWS Secrets Manager.
    -   Choose "Store a new secret".
    -   Select "Credentials for RDS database".
    -   Enter the username and password you configured for the RDS instance.
    -   Select the RDS database instance you just created.
    -   Name the secret `prod/database/creds`.
2.  **Enable Rotation:**
    -   In the secret's configuration, enable automatic rotation.
    -   Choose a rotation interval (e.g., 30 days).
    -   Secrets Manager will automatically create a Lambda function to handle the rotation process.

#### **Part C: Connect from a Private Instance (45 mins)**

1.  **Update IAM Role:**
    -   Go to the IAM role used by your `WebServer-ASG` instances.
    -   Attach a new inline policy that grants permission to `secretsmanager:GetSecretValue` for the specific ARN of your new secret.
2.  **Launch a Test Client:**
    -   Use your `WebServer-LT` to launch a new instance in a **private subnet**. Make sure it has the `App-SG` attached.
3.  **Install Client and Test:**
    -   Connect to the instance using Session Manager.
    -   Install the MySQL client: `sudo yum install mysql -y`.
    -   Use the AWS CLI to retrieve the secret:
        `aws secretsmanager get-secret-value --secret-id prod/database/creds --query SecretString --output text`
    -   This will output a JSON string. Parse the host, username, and password from it.
    -   Use the retrieved credentials to connect to the database:
        `mysql -h <rds_endpoint> -u <username> -p<password>`
    -   You should be able to connect successfully, proving the security group and secret retrieval work.

---

### **9.4 Teaching Notes**

-   **Multi-AZ is for HA, Read Replicas are for Scale:** This is a fundamental concept that is often confused. Draw a diagram showing the synchronous replication within a region for Multi-AZ vs. the asynchronous replication (potentially across regions) for a Read Replica.
-   **Performance Insights is a Game Changer:** Before Performance Insights, DBAs had to rely on complex queries against internal database tables to diagnose issues. Show a "before and after" scenario. Emphasize that the first place to look for any database slowdown is the Performance Insights dashboard.
-   **Never Hard-code Secrets:** This is a non-negotiable security best practice. Secrets Manager (or similar tools like HashiCorp Vault) is the standard way to handle credentials. The automatic rotation feature is a huge operational win.
-   **RDS Proxy is for Efficiency:** The benefits of connection pooling are most obvious with function-based compute like Lambda, where thousands of concurrent functions could try to connect to the database at once. For traditional EC2 applications, it's still beneficial but less critical.
-   **The Lab Puts it All Together:** This lab connects networking (Subnet Groups, Security Groups), compute (a bastion/test EC2 instance), and the new data tier (RDS). The final test—connecting from a private instance using credentials retrieved from Secrets Manager—is a comprehensive validation of the multi-tier architecture built so far.

---

## **Week 10: Architecting the Backend & Model Tiers**

**Duration:** 3 Hours (1 Hour Lecture, 2 Hours Lab)

**Core Topics:** Multi-Tier Architecture Patterns, Internal Load Balancers, Security Group Chaining, Decoupling with SQS, Model Serving Patterns.

---

### **10.1 Learning Objectives**

By the end of this session, students will be able to:
-   Design and deploy a secure, private backend application tier using an internal Application Load Balancer.
-   Implement a separate, scalable "model" tier for specialized processing.
-   Connect all application tiers securely using security group chaining.
-   Introduce loose coupling into the architecture by integrating a Simple Queue Service (SQS) queue.
-   Complete and test the end-to-end request flow for the sentiment analysis application.

---

### **10.2 Detailed Content**

#### **10.2.1 Multi-Tier Architecture Review**

-   **Web Tier (Public-Facing):**
    -   **Components:** Internet-facing Application Load Balancer, Auto Scaling Group of web servers (e.g., Nginx, Apache).
    -   **Responsibility:** Handles incoming user requests, terminates SSL, serves static content, and acts as a reverse proxy to the backend. It is the only tier exposed to the internet.
-   **Application Tier (Private):**
    -   **Components:** Internal Application Load Balancer, Auto Scaling Group of application servers (e.g., running Python/Flask, Node.js/Express).
    -   **Responsibility:** Executes the core business logic. Receives requests *only* from the Web Tier. Connects to the Data Tier to read/write data.
-   **Data Tier (Private):**
    -   **Components:** RDS Multi-AZ database, ElastiCache cluster, etc.
    -   **Responsibility:** Manages data persistence and retrieval. Only accepts connections from the Application Tier.

#### **10.2.2 Internal Load Balancers**

-   **Concept:** An Application Load Balancer with an `internal` scheme.
-   **Function:** It is assigned a private IP address from each subnet it's deployed in and is only accessible from within the VPC (or peered networks). It does not have a public IP address.
-   **Use Case:** To load balance traffic *between* internal tiers of an application. This is the standard way to expose the Application Tier to the Web Tier without exposing the application servers directly. It also enables the Application Tier to scale independently of the Web Tier.

#### **10.2.3 Security Group Chaining: The Core of Network Security**

-   This is the practical implementation of the tiered security model we designed in Week 5.
-   **Flow:**
    1.  The `Web-SG` (on the front-end ALB) allows traffic from the internet (`0.0.0.0/0`) on ports 80/443.
    2.  The `App-SG` (on the backend instances) has an inbound rule that allows traffic on the application port (e.g., 8080) **only from the `Web-SG`**. You use the Security Group ID as the source, not a CIDR range.
    3.  The `DB-SG` (on the RDS instance) has an inbound rule that allows traffic on the database port (e.g., 3306) **only from the `App-SG`**.
-   **Benefit:** This creates a strict, one-way flow of traffic. An attacker who compromises a web server cannot directly access the database, as the `DB-SG` does not trust the `Web-SG`. The request *must* go through the application tier.

#### **10.2.4 Decoupling with Amazon SQS (Simple Queue Service)**

-   **Problem:** Some tasks, like running a sentiment analysis model, might be slow. If the front-end application calls the model service directly and waits for a response, it can lead to long load times and poor user experience. This is "tight coupling."
-   **Solution:** Introduce a queue to decouple the services.
-   **Asynchronous Flow:**
    1.  The Application Tier receives a request to analyze some text.
    2.  Instead of calling the model directly, it publishes a message containing the text to an SQS queue. This is a very fast operation.
    3.  The Application Tier immediately returns a "Request received" response to the user.
    4.  The Model Tier instances independently poll the SQS queue, receive messages, process them, and store the results in the database.
-   **Benefits:**
    -   **Improved User Experience:** The front-end is fast and responsive.
    -   **Resilience:** If the Model Tier is down, messages simply accumulate in the queue. The system automatically recovers when the Model Tier comes back online.
    -   **Scalability:** The Application Tier and Model Tier can be scaled independently based on the queue depth.

---

### **10.3 Lab: Assembling the Full Application**

**Objective:** To deploy the backend and model tiers and integrate them to create a fully functional, end-to-end sentiment analysis application.

#### **Part A: Deploy the Backend Application Tier (45 mins)**

1.  **Create a Launch Template for App Servers:**
    -   Create a new Launch Template `AppServer-LT`.
    -   Use a standard Amazon Linux 2 AMI.
    -   In the "User Data" section, add a script to install a simple Python Flask application that listens on port 8080. This app will have two endpoints:
        -   `/` which returns "Hello from the App Tier!"
        -   `/analyze` which accepts a POST request, puts the message body into an SQS queue, and returns "Analysis request queued."
    -   Assign the `App-SG` security group and the IAM role with permissions for SSM and Secrets Manager.
2.  **Create an Internal ALB and Backend ASG:**
    -   Create a new Target Group `App-TG` for port 8080.
    -   Create a new **internal** Application Load Balancer `App-Internal-ALB`, associated with the **private subnets** and the `App-SG`.
    -   Create a new Auto Scaling Group `AppServer-ASG` using the `AppServer-LT`, associated with the **private subnets** and the `App-TG`. Set min/desired/max to 2/2/4.

#### **Part B: Deploy the Model Tier & SQS Queue (45 mins)**

1.  **Create an SQS Queue:**
    -   Navigate to SQS and create a new standard queue named `Sentiment-Analysis-Queue`.
2.  **Create a Launch Template for Model Servers:**
    -   Create a new Launch Template `ModelServer-LT`.
    -   Use a standard Amazon Linux 2 AMI.
    -   The user data script should install a Python application that continuously polls the SQS queue, "processes" the message (e.g., `time.sleep(5)` to simulate work), and logs the result.
    -   Create and assign a new `Model-SG` security group.
    -   Create and assign a new IAM role that has permissions for SQS (`ReceiveMessage`, `DeleteMessage`) and CloudWatch Logs.
3.  **Create the Model Tier ASG:**
    -   Create a final Auto Scaling Group `ModelServer-ASG` using the `ModelServer-LT` in the **private subnets**. This tier doesn't need a load balancer as it sources its work from the queue. Set min/desired/max to 2/2/4.

#### **Part C: End-to-End Integration and Testing (30 mins)**

1.  **Update the Web Tier:**
    -   Go to the Internet-facing `WebApp-ALB`.
    -   Add a new listener rule with priority 10. For the condition, use "Path is" `/api/*`.
    -   For the action, "Forward to" the `App-TG` (your backend target group). This directs all API traffic to the backend.
2.  **End-to-End Test:**
    -   From your local machine, send a POST request to the public ALB's DNS name:
        `curl -X POST http://<Public-ALB-DNS>/api/analyze -d "I love AWS"`
    -   You should receive the "Analysis request queued" response from the App Tier.
3.  **Verify Processing:**
    -   Check the CloudWatch Logs for the Model Tier instances. You should see log entries indicating that they received and "processed" the message "I love AWS" from the queue.
    -   Check the SQS console. The number of messages in the queue should briefly go up to 1 and then back down to 0.

---

### **10.4 Teaching Notes**

-   **Diagrams are Essential:** This week's architecture is the most complex so far. Start the lecture with a clear diagram showing the three tiers, the two ALBs, the security groups, and the SQS queue. Refer back to it constantly.
-   **Security Group Chaining is the "Aha!" Moment:** When students see that the `App-SG` literally contains the ID of the `Web-SG` in its rules, the concept of tiered security clicks. It's a powerful visual.
-   **Why an Internal ALB?** Students might ask, "Why not just have the web servers talk directly to the app server IPs?" Explain that the internal ALB provides a stable endpoint, health checking, and scalability for the app tier, just like the public ALB does for the web tier. It decouples the tiers.
-   **SQS for Asynchronous Operations:** The sentiment analysis model is a perfect example of a slow, asynchronous task. Use this to explain the broader pattern: any time an action doesn't need to happen in real-time for the user to get a response, a queue is likely the right tool. This improves performance and resilience.
-   **Project Milestone:** This is a huge milestone for the course project. The core, scalable, multi-tier application is now functional. The remaining weeks will focus on hardening it, optimizing it, and deploying it with IaC. Acknowledge this accomplishment with the class.

---

## **Part 4: Serverless & Modern Architectures**

---

## **Week 11: Introduction to Serverless Architectures**

**Duration:** 3 Hours (1.5 Hour Lecture, 1.5 Hours Lab)

**Core Topics:** Serverless Philosophy, AWS Lambda Deep Dive (Execution Model, VPC Integration, Concurrency), Event-Driven Architecture, Amazon SQS vs. SNS vs. EventBridge.

---

### **11.1 Learning Objectives**

By the end of this session, students will be able to:
-   Articulate the trade-offs between serverless and traditional server-based architectures.
-   Design and deploy a sophisticated AWS Lambda function, including layers and VPC networking.
-   Explain the difference between provisioned and on-demand concurrency and its impact on performance and cost.
-   Choose the appropriate serverless messaging service (SQS, SNS, EventBridge) for a given use case.
-   Build a complete, event-driven workflow triggered by an S3 event.

---

### **11.2 Detailed Content**

#### **11.2.1 The Serverless Philosophy**

-   **Definition:** Serverless computing allows you to build and run applications and services without thinking about servers. It eliminates infrastructure management tasks such as server or cluster provisioning, patching, operating system maintenance, and capacity provisioning.
-   **Core Principles:**
    -   **No Server Management:** You don't provision or maintain any servers.
    -   **Pay for Value:** Pay only for the compute time you consume, down to the millisecond. Never pay for idle.
    -   **Automatic Scaling:** The service scales automatically and precisely based on the size of the workload.
    -   **Built-in High Availability:** Services have inherent fault tolerance because they are run by AWS across multiple AZs.
-   **Trade-offs:**
    -   **Pros:** Lower operational overhead, reduced cost (for spiky workloads), faster time-to-market.
    -   **Cons:** Potential for vendor lock-in, limitations on execution duration (15 mins for Lambda), and the "cold start" phenomenon.

#### **11.2.2 AWS Lambda Deep Dive**

-   **Execution Model:**
    1.  An **event source** (e.g., S3 object upload, API Gateway request) triggers the function.
    2.  Lambda provisions a secure, isolated **execution environment** (a Firecracker micro-VM).
    3.  It downloads your code (and any layers), starts the language runtime, and executes your function's **handler method**, passing in the event data.
    4.  The environment is frozen or destroyed after execution.
-   **Cold Starts:** The first time a function is invoked (or after a long period of inactivity), Lambda has to perform all the steps in the execution model, which adds latency. Subsequent calls to a "warm" function are much faster as the environment is reused.
-   **Concurrency & Throttling:**
    -   **Concurrency:** The number of requests that your function is serving at any given time.
    -   **On-Demand Concurrency:** By default, your account has a pool of 1,000 concurrent executions per region, shared by all functions. If you exceed this, functions will be throttled (requests will be rejected).
    -   **Provisioned Concurrency:** You can pay to keep a specified number of execution environments warm and ready to execute with no cold start latency. This is for critical, latency-sensitive applications.
-   **VPC Integration:** If a Lambda function needs to access resources in your VPC (like an RDS database), it can be configured to do so. Lambda will attach an ENI from your VPC to the execution environment. This can increase cold start times and requires careful IP management.

#### **11.2.3 The "Big Three" of Serverless Messaging**

-   **Amazon SQS (Simple Queue Service):**
    -   **Pattern:** Decouples a *producer* from a *consumer*. One message is processed by **one** consumer.
    -   **Use Case:** Ideal for decoupling services, distributing work, and creating resilient, asynchronous workflows (like our sentiment analysis app).
-   **Amazon SNS (Simple Notification Service):**
    -   **Pattern:** A publish/subscribe ("pub/sub") model. A *publisher* sends a message to a *topic*, and that message is delivered to **all** subscribers to that topic.
    -   **Subscribers:** Can be Lambda functions, SQS queues, email addresses, mobile push notifications, etc.
    -   **Use Case:** Fanning out notifications. For example, a single "OrderPlaced" event could be sent to a Lambda function to process the order, an SQS queue for the shipping department, and an analytics endpoint simultaneously.
-   **Amazon EventBridge:**
    -   **Pattern:** An event bus that enables event-driven architectures. It's like SNS on steroids.
    -   **Key Features:** Receives events from AWS services, your own applications, and SaaS partners. It can then **filter** events based on their content (the event payload) and **route** them to different targets.
    -   **Use Case:** The central nervous system for a complex event-driven application. "If the event is from `aws.s3` AND the `object.size` is greater than 1GB, then route it to this specific Lambda function."

---

### **11.3 Lab: Building an Event-Driven Image Processing Pipeline**

**Objective:** To use S3 event notifications, Lambda, and SNS to create a serverless pipeline that automatically generates thumbnails for uploaded images.

#### **Part A: Create the Lambda Function and Layers (45 mins)**

1.  **Create a Lambda Layer:**
    -   On your local machine, create a directory structure: `python/lib/python3.8/site-packages`.
    -   Inside it, install the `Pillow` library: `pip install Pillow -t .`
    -   Zip the `python` directory.
    -   In the Lambda console, go to Layers and upload the zip file to create a new layer named `Pillow-Layer`. This makes the library available to our function without bloating the deployment package.
2.  **Create the Thumbnail-Generator Lambda Function:**
    -   Create a new Lambda function from scratch, using the Python 3.8 runtime.
    -   Name it `Thumbnail-Generator`.
    -   **Execution Role:** Let Lambda create a new role. We will modify it.
    -   In the function code, write a Python script that:
        -   Parses the S3 bucket name and object key from the incoming event.
        -   Uses the Boto3 S3 client to download the image in-memory.
        -   Uses the `Pillow` library to resize the image to 128x128.
        -   Saves the thumbnail to a different S3 bucket (or a different prefix in the same bucket).
3.  **Configure the Function:**
    -   Attach the `Pillow-Layer` you created.
    -   Increase the memory to 512 MB and the timeout to 30 seconds to handle larger images.

#### **Part B: Set up S3 Buckets and Event Triggers (45 mins)**

1.  **Create S3 Buckets:**
    -   Create a bucket for source images: `your-name-source-images`.
    -   Create a second bucket for the thumbnails: `your-name-thumbnails`.
2.  **Configure S3 Event Notification:**
    -   In the properties of the `source-images` bucket, create a new event notification.
    -   Configure it to trigger on all `s3:ObjectCreated:*` events.
    -   Set the destination to be your `Thumbnail-Generator` Lambda function.
    -   AWS will automatically add the necessary permissions to the Lambda function's execution role to allow S3 to invoke it.
3.  **Update IAM Role:**
    -   Go to the IAM role created for the Lambda function.
    -   Attach a policy that allows `s3:GetObject` from the source bucket and `s3:PutObject` to the thumbnail bucket.
4.  **Test the Pipeline:**
    -   Upload a JPG or PNG image to the `source-images` bucket.
    -   After a few moments, check the `thumbnails` bucket. A resized version of your image should appear.
    -   Check the CloudWatch Logs for the Lambda function to see its output and debug any issues.

#### **Part C: Add SNS Notifications for Failure (30 mins)**

1.  **Create an SNS Topic:**
    -   Go to SNS and create a new standard topic named `Image-Processing-Failures`.
    -   Subscribe your email address to this topic and confirm the subscription.
2.  **Configure Lambda Dead-Letter Queue (DLQ):**
    -   Go back to your `Thumbnail-Generator` Lambda function's configuration.
    -   In the "Asynchronous invocation" settings, specify a "Dead-Letter Queue (DLQ)".
    -   Set the destination to be the `Image-Processing-Failures` SNS topic.
    -   The Lambda execution role will need `sns:Publish` permissions on this topic.
3.  **Test the Failure Path:**
    -   Upload a non-image file (e.g., a `.txt` file) to the `source-images` bucket.
    -   The Lambda function will be invoked, but the `Pillow` library will fail to process the file, causing an error.
    -   Because it's an asynchronous invocation from S3, Lambda will retry twice. After the final failure, it will send the failed event payload to the SNS topic.
    -   You should receive an email containing the details of the failed event.

---

### **11.4 Teaching Notes**

-   **Lambda is FaaS:** Lambda is the quintessential "Function as a Service". Emphasize that you are only providing the code (the function), and AWS handles everything else about the execution environment.
-   **Layers for Dependencies:** Layers are the standard way to manage dependencies. It keeps the core function code small and allows layers to be shared across multiple functions (e.g., a common data access layer).
-   **Event-Driven Thinking:** This is a different mindset. Instead of a linear, synchronous request-response, students need to think in terms of "When *this event* happens, trigger *that action*." The lab is a perfect, tangible example of this.
-   **SQS vs SNS vs EventBridge:** Use a simple analogy.
    -   **SQS:** A letter in a mailbox. One person (consumer) picks it up and reads it.
    -   **SNS:** A megaphone. One person (publisher) shouts a message, and everyone in the area (subscribers) hears it.
    -   **EventBridge:** A smart postal sorting facility. It reads the address and content of every letter and routes it to the correct recipient based on complex rules.
-   **DLQs are Critical for Resilience:** For any asynchronous workflow, a Dead-Letter Queue is essential. It ensures that failed events are not lost forever. They can be inspected, re-processed, or archived for auditing, making the entire system more robust.

---

## **Week 12: Advanced Data Management with S3 & KMS**

**Duration:** 3 Hours (1 Hour Lecture, 2 Hours Lab)

**Core Topics:** Advanced S3 Features (Lifecycle, Replication, Intelligent-Tiering), S3 Storage Lens, Data Encryption, AWS Key Management Service (KMS).

---

### **12.1 Learning Objectives**

By the end of this session, students will be able to:
-   Architect a cost-effective data lifecycle using S3 Lifecycle Policies and Intelligent-Tiering.
-   Configure S3 Replication for disaster recovery or low-latency global data access.
-   Analyze storage usage patterns across an entire organization with S3 Storage Lens.
-   Differentiate between server-side encryption methods (SSE-S3, SSE-KMS, SSE-C).
-   Create and manage customer-managed keys (CMKs) in AWS KMS and use them to encrypt S3 objects.

---

### **12.2 Detailed Content**

#### **12.2.1 Advanced S3 Features**

-   **Static Website Hosting Review:** S3 can serve static content (HTML, CSS, JS) directly, with a bucket policy allowing public read access. This is highly scalable and cost-effective for front-end hosting.
-   **Lifecycle Policies:** Automate the movement of objects between storage classes to save costs.
    -   **Transition Actions:** Move objects to a cheaper tier after a certain number of days (e.g., move to S3-IA after 30 days, then to S3 Glacier Flexible Retrieval after 90 days).
    -   **Expiration Actions:** Automatically delete objects after a specified period (e.g., delete old logs after 1 year).
-   **S3 Intelligent-Tiering:** An automated storage class that monitors access patterns and moves objects between a frequent access tier and an infrequent access tier to optimize costs without performance impact or operational overhead. It's a great "set and forget" option for data with unknown or changing access patterns.
-   **S3 Replication:**
    -   **Cross-Region Replication (CRR):** Replicates objects to a bucket in a *different* AWS Region. Used for disaster recovery and compliance.
    -   **Same-Region Replication (SRR):** Replicates objects to a bucket in the *same* region. Used for aggregating logs from multiple sources into one bucket or creating separate dev/test environments.

#### **12.2.2 Organizational Storage Analysis with S3 Storage Lens**

-   **Concept:** An organization-wide analytics feature that provides a single view of object storage usage and activity across all accounts in your AWS Organization.
-   **Key Insights:** Provides dashboards and metrics to answer questions like:
    -   "What are my fastest-growing buckets?"
    -   "How much of my data is encrypted?"
    -   "Which buckets have incomplete multipart uploads that are costing me money?"
    -   "How much of my data could be saved by implementing a lifecycle policy?"
-   **Benefit:** Moves you from managing storage reactively to proactively optimizing cost and security posture.

#### **12.2.3 Data Encryption at Rest**

-   **Server-Side Encryption (SSE):** Encryption is handled by AWS on the server side before the data is written to disk.
    -   **SSE-S3:** AWS manages both the data key and the master key. This is the baseline, transparent encryption option.
    -   **SSE-KMS:** AWS manages the data key, but **you** manage the master key in AWS Key Management Service (KMS). This gives you more control and auditability.
    -   **SSE-C:** You provide and manage your own encryption keys. AWS performs the encryption, but does not store your key. You must send the key with every request. This is for customers with extreme security requirements.
-   **Client-Side Encryption:** You encrypt the data *before* you send it to S3. You are fully responsible for key management and the encryption/decryption process.

#### **12.2.4 AWS Key Management Service (KMS) Deep Dive**

-   **Concept:** A managed service that makes it easy to create and control the encryption keys used to encrypt your data. It uses FIPS 140-2 validated hardware security modules (HSMs) to protect your keys.
-   **Key Hierarchy (Envelope Encryption):**
    1.  Your data is encrypted with a unique **Data Key**.
    2.  The Data Key itself is then encrypted by a **Customer Master Key (CMK)** that you manage in KMS.
    3.  The encrypted data and the *encrypted* data key are stored together.
    4.  To decrypt, your application must have IAM permission to ask KMS to decrypt the data key using the CMK.
-   **Customer Master Keys (CMKs):**
    -   **AWS Managed CMK:** Created and managed by AWS on your behalf for use with specific AWS services.
    -   **Customer Managed CMK:** You create, manage, and define the access policies for this key. This provides full control and an audit trail in CloudTrail of every time the key is used. This is the recommended practice for sensitive data.
-   **Key Policies:** The primary resource-based policy that controls access to a CMK. You define who (which users or roles) can manage the key and who can use it for encrypt/decrypt operations.

---

### **12.3 Lab: Securing and Optimizing the S3 Data Layer**

**Objective:** To migrate the application's static assets to a secure, encrypted S3 bucket and implement lifecycle policies for cost optimization.

#### **Part A: Migrate Static Assets to S3 (30 mins)**

1.  **Create and Configure the S3 Bucket:**
    -   Create a new S3 bucket named `your-name-webapp-assets`.
    -   In the "Properties" tab, enable "Static website hosting".
    -   Upload a sample `index.html` and `error.html` file.
2.  **Apply a Public Read Bucket Policy:**
    -   In the "Permissions" tab, edit the bucket policy to allow public `s3:GetObject` for all objects in the bucket.
    ```json
    {
        "Version": "2012-10-17",
        "Statement": [
            {
                "Sid": "PublicReadGetObject",
                "Effect": "Allow",
                "Principal": "*",
                "Action": "s3:GetObject",
                "Resource": "arn:aws:s3:::your-name-webapp-assets/*"
            }
        ]
    }
    ```
3.  **Test Access:** Verify you can access the `index.html` file via the public static website endpoint URL.

#### **Part B: Implement Encryption with a Customer Managed CMK (45 mins)**

1.  **Create a Customer Managed Key (CMK):**
    -   Navigate to KMS. Create a new symmetric key.
    -   Give it an alias like `alias/S3-WebApp-Key`.
    -   Define key administrators (your current role).
    -   In the key usage permissions, add the IAM role used by your `WebServer-ASG` instances to allow it to use the key for decryption.
2.  **Enforce Bucket Encryption:**
    -   Go back to your `webapp-assets` bucket properties.
    -   Under "Default encryption", enable server-side encryption and choose `AWS-KMS`.
    -   Select your `S3-WebApp-Key` from the dropdown list.
3.  **Update Bucket Policy:**
    -   Add a `Deny` statement to your bucket policy that denies any `s3:PutObject` request if it does not include the `x-amz-server-side-encryption` header specifying `aws:kms`. This enforces that all new objects *must* be encrypted.
4.  **Verify Encryption:**
    -   Upload a new file to the bucket.
    -   Select the file and view its properties. It should show that it is encrypted with AWS-KMS using your specific key.

#### **Part C: Configure Data Lifecycle and Replication (45 mins)**

1.  **Create a Replication Destination Bucket:**
    -   Create a new bucket in a *different region* named `your-name-webapp-assets-dr`.
    -   Enable versioning on both the source and destination buckets (replication requires versioning).
2.  **Configure Replication:**
    -   On your source bucket, go to "Management" -> "Replication rules".
    -   Create a new rule to replicate all objects to the destination bucket.
    -   Have S3 create a new IAM role to handle the replication permissions.
3.  **Configure Lifecycle Policy:**
    -   On your source bucket, create a new lifecycle rule.
    -   **Rule 1:** For current versions, transition objects to `S3 Intelligent-Tiering` after 30 days.
    -   **Rule 2:** For non-current (previous) versions, expire them after 7 days to save costs.
4.  **Test Replication:**
    -   Upload a new object to your source bucket. After a few moments, verify that it appears in the destination bucket in the other region.

---

### **12.4 Teaching Notes**

-   **S3 Tiers are a Cost-Saving Pillar:** A huge amount of AWS spend is on S3. Understanding and using lifecycle policies and Intelligent-Tiering is a critical skill for any cloud professional. Intelligent-Tiering is often the easiest and safest bet for data with unknown access patterns.
-   **KMS is All About Control and Audit:** SSE-S3 is secure, but SSE-KMS is what enterprises use. The ability to control the key policy and see every use of the key in CloudTrail is a fundamental requirement for compliance and security investigations.
-   **Envelope Encryption:** Use a physical analogy. Your data is in a box (data key). You lock that box with a key. You then put that key inside a high-security safe (the CMK in KMS). To open the box, you must first have permission to open the safe to get the key. This makes the concept much easier to grasp.
-   **Replication for DR:** S3 Cross-Region Replication is one of the simplest and most effective DR strategies for data. In the event of a regional outage, your data is already safe in another region, ready to be accessed by recovered application instances.
-   **The Lab is a Security Checklist:** The steps in this lab mirror a real-world security and cost-optimization checklist for S3: host static assets, enable public access correctly, enforce encryption with a customer key, and set up lifecycle and DR policies.

---

## **Part 5: Global Architecture & Security**

---

## **Week 13: Edge Networking & Security**

**Duration:** 3 Hours (1 Hour Lecture, 2 Hours Lab)

**Core Topics:** Amazon CloudFront (CDN), Caching, AWS WAF, Route 53 DNS, AWS Certificate Manager (ACM).

---

### **13.1 Learning Objectives**

By the end of this session, students will be able to:
-   Deploy a CloudFront distribution to accelerate the delivery of a static website.
-   Configure caching behaviors and invalidations to optimize content delivery.
-   Protect an application from common web exploits using AWS WAF.
-   Register a domain name and manage DNS records using Route 53.
-   Provision and associate a free, public SSL/TLS certificate using AWS Certificate Manager.

---

### **13.2 Detailed Content**

#### **13.2.1 Global Content Delivery with Amazon CloudFront**

-   **Concept:** CloudFront is a global Content Delivery Network (CDN). It caches copies of your content (images, videos, static files) in hundreds of Edge Locations around the world.
-   **How it Works:**
    1.  A user requests your content.
    2.  DNS (Route 53) directs them to the nearest CloudFront Edge Location.
    3.  If the content is already cached at the edge (**Cache Hit**), it's served immediately with very low latency.
    4.  If it's not cached (**Cache Miss**), CloudFront forwards the request to your origin (e.g., your S3 bucket or ALB), retrieves the content, caches it at the edge, and then serves it to the user.
-   **Key Components:**
    -   **Distribution:** The main CloudFront configuration.
    -   **Origin:** Where CloudFront fetches the original content (e.g., an S3 bucket, an ALB, a MediaStore container).
    -   **Behavior:** A set of rules that defines how CloudFront handles requests for a given URL path pattern (e.g., `/images/*`). You can define caching policies, allowed HTTP methods, and which origin to use.
-   **Origin Access Control (OAC):** A feature that securely locks down your S3 bucket so that it can *only* be accessed through CloudFront, not via its public S3 URL. This is a critical security best practice.

#### **13.2.2 Caching Strategies**

-   **Cache Keys:** The set of values that uniquely identify an object in the cache (e.g., URL path, headers, query strings). By default, CloudFront uses the URL. If your application serves different content based on a header (e.g., language), you must add that header to the cache key.
-   **Time-to-Live (TTL):** How long an object stays in the cache before CloudFront re-validates it with the origin. Can be controlled by `Cache-Control` headers from the origin or overridden in the CloudFront behavior settings.
-   **Invalidations:** A way to forcibly remove an object from the cache before its TTL expires (e.g., after you upload a new version of `style.css`). Invalidations cost money and should be used sparingly. A better strategy is to use versioned filenames (e.g., `style-v2.css`).

#### **13.2.3 Application Layer Protection with AWS WAF**

-   **Concept:** A web application firewall that helps protect your web applications from common web exploits that could affect availability, compromise security, or consume excessive resources.
-   **How it Works:** WAF inspects incoming HTTP/HTTPS requests *before* they reach your application (ALB or CloudFront). It checks them against a set of rules.
-   **Rule Types:**
    -   **IP Sets:** Block traffic from known malicious IP addresses.
    -   **Rate-Based Rules:** Block IPs that send an excessive number of requests in a 5-minute period (helps mitigate DDoS attacks).
    -   **Managed Rule Groups:** Pre-configured sets of rules managed by AWS or Marketplace vendors to protect against common threats like the OWASP Top 10 (e.g., SQL injection, Cross-Site Scripting).
    -   **Custom Rules:** Write your own rules to inspect request headers, body, or query strings for specific patterns.

#### **13.2.4 DNS and Certificates: Route 53 & ACM**

-   **Amazon Route 53:** A highly available and scalable Domain Name System (DNS) web service.
    -   **Alias Records:** A Route 53-specific record type that lets you map your custom domain name (e.g., `www.myapp.com`) to an AWS resource (like a CloudFront distribution or an ALB). It's like a CNAME but works for root domains and is generally preferred for AWS resources.
-   **AWS Certificate Manager (ACM):**
    -   A service that lets you easily provision, manage, and deploy public and private SSL/TLS certificates for use with AWS services.
    -   **Public Certificates:** Free of charge and automatically renewed by ACM. These are essential for enabling HTTPS on your CloudFront distribution or ALB.
    -   **Regionality:** A certificate for an ALB must be requested in the same region as the ALB. A certificate for CloudFront **must be requested in us-east-1 (N. Virginia)**, regardless of where your origin is.

---

### **13.3 Lab: Deploying a Global, Secure Web Application**

**Objective:** To put a CloudFront distribution and WAF in front of the S3 static website, and point a custom domain name to it using Route 53.

#### **Part A: Create the CloudFront Distribution (45 mins)**

1.  **Request a Public Certificate:**
    -   Navigate to ACM **in the `us-east-1` region**.
    -   Request a public certificate for a domain you own (e.g., `*.yourdomain.com`).
    -   Complete the DNS validation step by adding the CNAME record provided by ACM to your domain's DNS settings.
2.  **Create the Distribution:**
    -   Navigate to CloudFront. Create a new distribution.
    -   **Origin Domain:** Select your `webapp-assets` S3 bucket.
    -   **S3 bucket access:** Select "Yes use OAC (Origin Access Control)" and let CloudFront create the necessary settings and update the bucket policy. This secures your bucket.
    -   **Viewer Protocol Policy:** Redirect HTTP to HTTPS.
    -   **Alternate Domain Names (CNAMEs):** Add the domain name you want to use (e.g., `www.yourdomain.com`).
    -   **Custom SSL Certificate:** Select the ACM certificate you just created.
    -   Deploy the distribution (this can take 5-10 minutes).

#### **Part B: Configure AWS WAF (30 mins)**

1.  **Create a Web ACL:**
    -   Navigate to WAF & Shield. Create a new Web ACL.
    -   Name it `WebApp-ACL`. Resource type should be "CloudFront distributions".
    -   Associate it with the CloudFront distribution you just created.
2.  **Add Managed Rules:**
    -   Add rules -> "Add managed rule groups".
    -   Add the **"AWSManagedRulesCommonRuleSet"**. This provides protection against a wide range of common attacks.
    -   Add the **"Amazon IP reputation list"** to block known malicious IP sources.
3.  **Create a Custom Rate-Based Rule:**
    -   Add a new custom rule. Set the type to "Rate-based rule".
    -   Set the rate limit to 500 requests per 5 minutes.
    -   Set the action to "Block". This helps prevent simple HTTP flood attacks.

#### **Part C: Configure DNS with Route 53 (45 mins)**

1.  **Create a Hosted Zone:**
    -   Navigate to Route 53. If you don't have one, create a new Public Hosted Zone for your domain.
    -   If your domain is registered elsewhere, you will need to update the Name Server (NS) records at your registrar to point to the AWS name servers provided by Route 53.
2.  **Create an Alias Record:**
    -   Inside your hosted zone, create a new record.
    -   For the record name, enter `www` (or leave it blank for the root domain).
    -   Select record type "A".
    -   Enable the "Alias" toggle.
    -   For the route traffic to, choose "Alias to CloudFront distribution" and select your distribution from the dropdown.
3.  **Test Everything:**
    -   Wait for DNS to propagate.
    -   In your browser, navigate to `https://www.yourdomain.com`. You should see your static website, served securely over HTTPS via CloudFront.
    -   Test the WAF by trying to access the site with a malicious query string, e.g., `https://www.yourdomain.com/?q=<script>alert(1)</script>`. The request should be blocked by the WAF common rule set.

---

### **13.4 Teaching Notes**

-   **CloudFront for Performance AND Security:** While CloudFront is known as a CDN for performance, it's also a critical security layer. It absorbs DDoS attacks at the edge and provides a place to integrate WAF, shielding your origin from malicious traffic.
-   **OAC is the new OAI:** Origin Access Identity (OAI) is the older way to restrict S3 access to CloudFront. Origin Access Control (OAC) is the new, more secure, and more flexible best practice. All new distributions should use OAC.
-   **ACM Certificate in `us-east-1`:** This is a common "gotcha". For a CloudFront distribution, the public certificate **must** be created in the N. Virginia (`us-east-1`) region, no matter where the origin or users are. This is a hard requirement.
-   **Alias Records are Superior:** When pointing to AWS resources, Alias records are almost always better than CNAMEs. They are free, they can be used at the zone apex (root domain), and they resolve faster.
-   **The Final User-Facing Architecture:** This lab completes the user-facing portion of the project. The combination of Route 53 for DNS, ACM for SSL/TLS, CloudFront for the CDN, and WAF for security is the standard, best-practice architecture for serving any modern web application on AWS.

---

## **Week 14: Governance & Compliance Automation**

**Duration:** 3 Hours (1.5 Hour Lecture, 1.5 Hours Lab)

**Core Topics:** AWS CloudTrail, AWS Config, AWS Security Hub, AWS GuardDuty.

---

### **14.1 Learning Objectives**

By the end of this session, students will be able to:
-   Configure a CloudTrail organization trail to create an immutable audit log of all API activity.
-   Use AWS Config rules to continuously monitor for and automatically remediate non-compliant resources.
-   Centralize security findings from multiple AWS services into AWS Security Hub.
-   Enable intelligent threat detection with Amazon GuardDuty.
-   Understand how these services work together to form a comprehensive governance and compliance framework.

---

### **14.2 Detailed Content**

#### **14.2.1 Auditing with AWS CloudTrail**

-   **Concept:** CloudTrail is the audit log for your AWS account. It records nearly every API call made, whether from the console, CLI, or an SDK. It answers the question: "**Who** did **what**, **where**, and **when**?"
-   **Organization Trail:** The absolute best practice. A trail configured in the management account of an AWS Organization that automatically logs all events for all member accounts to a central, secure S3 bucket. This ensures member accounts cannot tamper with their own audit logs.
-   **CloudTrail Lake:** An advanced feature that provides a managed data store for CloudTrail events. It allows you to run complex, SQL-based queries on your audit data without having to set up a separate analytics pipeline (e.g., with Athena).

#### **14.2.2 Compliance as Code with AWS Config**

-   **Concept:** AWS Config is a service that continuously assesses, audits, and evaluates the configurations of your AWS resources. It answers the question: "Is my environment **configured** according to my policies?"
-   **How it Works:**
    1.  **Configuration Recorder:** Discovers supported resources in your account and records their configuration as "Configuration Items" (CIs). It also records any changes to those CIs.
    2.  **Config Rules:** These are the policies you want to enforce. A rule represents your desired configuration state.
        -   **AWS Managed Rules:** A large library of pre-built rules for common best practices (e.g., `s3-bucket-public-read-prohibited`, `rds-instance-public-access-check`).
        -   **Custom Rules:** You can write your own rules using Lambda or Guard (a policy-as-code language).
    3.  **Conformance Packs:** A collection of Config rules and remediation actions that can be deployed as a single entity across an organization. Often mapped to compliance frameworks like PCI-DSS or HIPAA.
-   **Automated Remediation:** For many rules, you can configure an automatic remediation action using an SSM Automation document. For example, if the `s3-bucket-public-read-prohibited` rule finds a non-compliant bucket, it can automatically trigger an action to disable public access.

#### **14.2.3 Centralized Security Posture Management with AWS Security Hub**

-   **The Problem:** Security findings can come from many different services: GuardDuty, Config, IAM Access Analyzer, Inspector, etc. Looking at each service's console individually is inefficient.
-   **Solution:** Security Hub provides a "single pane of glass" for security. It automatically aggregates, normalizes, and prioritizes security findings from various AWS services and third-party tools.
-   **Key Features:**
    -   **Consolidated Findings:** A unified view of all security alerts.
    -   **Security Standards Checks:** Automatically runs continuous checks against your environment based on standards like the AWS Foundational Security Best Practices and CIS AWS Foundations Benchmark.
    -   **Automated Response:** Integrates with EventBridge, allowing you to trigger automated remediation workflows (e.g., a Lambda function or Step Function) when specific findings are generated.

#### **14.2.4 Intelligent Threat Detection with Amazon GuardDuty**

-   **Concept:** A managed threat detection service that continuously monitors for malicious activity and unauthorized behavior to protect your AWS accounts and workloads.
-   **How it Works:** It analyzes multiple data sources without any agents required:
    -   **VPC Flow Logs:** To detect unusual network traffic patterns (e.g., communication with known crypto-mining servers).
    -   **DNS Logs:** To identify communication with malicious domains.
    -   **CloudTrail Logs:** To find suspicious API activity (e.g., an IAM user from an unusual location disabling CloudTrail, or a sudden spike in EC2 launches).
-   **Findings:** GuardDuty generates actionable security findings with severity levels. It is not a prevention tool; it is a detection and alerting tool. Its findings are a key input into Security Hub.

---

### **14.3 Lab: Building a Governance and Compliance Framework**

**Objective:** To enable and integrate the core AWS governance services to provide automated detection and alerting for common security misconfigurations.

#### **Part A: Configure a CloudTrail Organization Trail (30 mins)**

1.  **Create a Central S3 Bucket:** In your AWS Organization's **management account**, create a new S3 bucket named `your-org-cloudtrail-logs` with default security settings (block all public access).
2.  **Create the Trail:**
    -   In the CloudTrail console (still in the management account), create a new trail.
    -   Name it `Organization-Trail`.
    -   **Crucially, check the box to "Enable for all accounts in my organization".**
    -   Point the trail to log to your central S3 bucket.
    -   Enable "Log file validation" to ensure integrity.
3.  **Verify:** After a few minutes, you should see log files from your member accounts starting to appear in the central S3 bucket.

#### **Part B: Deploy AWS Config Conformance Pack (45 mins)**

1.  **Enable AWS Config:** In your **management account**, go to the AWS Config console. In settings, enable recording for all resources in all accounts (delegated administrator setup).
2.  **Deploy a Conformance Pack:**
    -   Navigate to Conformance Packs and deploy a new one.
    -   Select the "Operational Best Practices for S3" sample template.
    -   This pack contains rules like checking for public access, requiring encryption, and enabling versioning.
    -   Deploy this pack to your entire organization (or a specific OU).
3.  **Review Compliance:**
    -   After 10-15 minutes, the rules will run. Go to the Config dashboard and review the compliance status.
    -   Identify a non-compliant resource (e.g., an S3 bucket without versioning enabled).
    -   Manually remediate the issue (enable versioning on the bucket).
    -   Tell Config to re-evaluate the rule and watch it change from "Noncompliant" to "Compliant".

#### **Part C: Centralize Findings with Security Hub and GuardDuty (45 mins)**

1.  **Designate a Delegated Administrator:**
    -   In your **management account**, go to Security Hub -> Settings.
    -   Designate one of your member accounts (e.g., a dedicated `Security-Tooling` account if you have one, or your `LogArchive` account for this lab) as the delegated administrator for Security Hub and GuardDuty.
2.  **Enable Services in the Delegated Admin Account:**
    -   **Log in to the delegated admin account.**
    -   Go to the GuardDuty console. Enable it. It will automatically be enabled for your entire organization.
    -   Go to the Security Hub console. Enable it. It will also auto-enable for the organization and begin pulling in findings from Config and GuardDuty.
3.  **Generate a Finding:**
    -   Go to the GuardDuty console -> Settings -> Sample findings. Generate some sample findings.
4.  **Review in Security Hub:**
    -   Go back to the Security Hub console.
    -   In the "Findings" section, you should see the sample GuardDuty findings, as well as any real findings from the AWS Config rules you deployed.
    -   Explore the dashboard to see your overall security score based on the AWS Foundational Security Best Practices standard.

---

### **14.4 Teaching Notes**

-   **The Governance Stack:** Explain how these services work together:
    -   **CloudTrail:** Records everything that happens (the audit log).
    -   **GuardDuty:** Analyzes the logs to find *threats*.
    -   **Config:** Records the state of everything and compares it to your desired state to find *misconfigurations*.
    -   **Security Hub:** Gathers all the findings from GuardDuty and Config into one place to be managed.
-   **Prevention vs. Detection:** SCPs (from Week 1) are for *preventing* bad things from happening. Config, GuardDuty, and Security Hub are for *detecting* bad things that have happened or misconfigurations that exist. A complete governance strategy needs both.
-   **Delegated Administrator is Key:** For services like GuardDuty and Security Hub, the delegated administrator pattern is essential. It allows you to manage the service for the entire organization from a single, non-management account, which aligns with the principle of least privilege.
-   **Conformance Packs are Powerful:** Manually setting up dozens of Config rules is tedious. Conformance packs allow you to treat compliance as code, deploying a full set of rules for a standard like PCI or SOC 2 with a single action.
-   **Automated Remediation is the Goal:** While the lab has students do manual remediation, stress that the end goal is to automate it. When Config finds a non-compliant S3 bucket, it should trigger an SSM document that fixes it immediately, without human intervention. This is how you scale security.

---

## **Part 6: Infrastructure as Code & Capstone**

---

## **Week 15: Infrastructure as Code (IaC) with CloudFormation**

**Duration:** 3 Hours (1.5 Hour Lecture, 1.5 Hours Lab)

**Core Topics:** Infrastructure as Code (IaC) Principles, CloudFormation Fundamentals (Templates, Stacks, Resources, Parameters, Outputs), Change Sets, Drift Detection.

---

### **15.1 Learning Objectives**

By the end of this session, students will be able to:
-   Explain the business and technical benefits of managing infrastructure as code.
-   Author a CloudFormation template in YAML to define AWS resources.
-   Use parameters to create reusable, customizable templates.
-   Use outputs to export important values from a stack for use by other stacks.
-   Safely preview and apply infrastructure changes using Change Sets.
-   Detect and reconcile manual changes to a stack using Drift Detection.

---

### **15.2 Detailed Content**

#### **15.2.1 Why Infrastructure as Code?**

-   **The Problem with the Console:** Clicking in the console is great for learning and one-off tasks, but it's not repeatable, it's error-prone, and it doesn't scale. How do you deploy an identical copy of your production environment for staging? How do you roll back a change that caused an outage?
-   **Solution: IaC:** The process of managing and provisioning computer data centers through machine-readable definition files, rather than physical hardware configuration or interactive configuration tools.
-   **Benefits:**
    -   **Automation:** Eliminates manual, error-prone tasks.
    -   **Repeatability:** Deploy the same environment dozens of times, perfectly.
    -   **Version Control:** Store your infrastructure definition in Git. You can see who changed what and when, and you can roll back to previous versions.
    -   **Peer Review:** Infrastructure changes can be reviewed and approved via pull requests, just like application code.
    -   **Documentation:** The template itself serves as documentation for what is deployed.

#### **15.2.2 CloudFormation Fundamentals**

-   **Template:** A text file (JSON or YAML) that defines a collection of AWS resources. This is the blueprint for your infrastructure.
-   **Stack:** A collection of AWS resources that you manage as a single unit. When you deploy a CloudFormation template, AWS provisions the resources as a *stack*.
-   **Core Template Sections:**
    -   **`AWSTemplateFormatVersion`:** The version of the template format (optional but recommended).
    -   **`Description`:** A text description of the template's purpose (optional but recommended).
    -   **`Parameters`:** Defines inputs that you can pass to the template when you create or update a stack. This allows for customization (e.g., passing in an instance type or VPC ID).
    -   **`Resources`:** The heart of the template. This section defines the AWS resources to be created (e.g., `AWS::EC2::Instance`, `AWS::S3::Bucket`). Each resource has a logical ID and a set of properties.
    -   **`Outputs`:** Declares output values that you can view in the console or import into other stacks. This is how you "connect" stacks (e.g., a VPC stack outputs its subnet IDs, which are then used by an application stack).
-   **Intrinsic Functions:** Functions built into CloudFormation that help you manage your stacks.
    -   **`!Ref`:** References the value of a parameter or the ID of a resource.
    -   **`!GetAtt`:** Retrieves an attribute of a resource (e.g., the DNS name of an ALB).
    -   **`!Sub`:** Substitutes variables in a string.
    -   **`!ImportValue`:** Imports a value from an output of another stack.

#### **15.2.3 Safe Deployments with Change Sets**

-   **The Problem:** When you update a stack, how do you know exactly what CloudFormation is going to change *before* it happens? A small template change could accidentally cause a resource to be replaced, leading to downtime.
-   **Solution: Change Sets:** A change set is a preview of the changes that CloudFormation will make to your stack. It shows you which resources will be added, modified, or deleted.
-   **Workflow:**
    1.  You create a change set with your new template or parameters.
    2.  CloudFormation generates a list of proposed changes.
    3.  You review the change set. If it's what you expect, you "execute" it to apply the changes. If not, you can delete it and nothing will have been changed in your environment.
-   **Best Practice:** **Always** use change sets for updating existing stacks, especially in production.

#### **15.2.4 Detecting Manual Changes with Drift Detection**

-   **The Problem:** Someone logs into the console and manually changes a resource that is managed by CloudFormation (e.g., changes a security group rule). This creates a "drift" where the actual configuration no longer matches the template's definition. This undermines the benefits of IaC.
-   **Solution: Drift Detection:** A CloudFormation feature that allows you to detect whether a stack's actual configuration has drifted from its expected configuration.
-   **How it Works:** You initiate a drift detection operation on a stack. CloudFormation compares the current resource configurations with the template and reports any differences.
-   **Remediation:** If drift is detected, the best practice is to use CloudFormation to bring the resource back into compliance by performing a stack update that re-applies the original configuration.

---

### **15.3 Lab: Creating Your First CloudFormation Stack**

**Objective:** To write a basic CloudFormation template from scratch to deploy a single EC2 instance, and then update it safely using a change set.

#### **Part A: Author and Deploy a Basic Template (1 hour)**

1.  **Create a YAML Template:**
    -   On your local machine, create a new file named `single-ec2-stack.yaml`.
2.  **Define Parameters:**
    -   Add a `Parameters` section.
    -   Create a parameter called `LatestAmiId` of type `AWS::SSM::Parameter::Value<AWS::EC2::Image::Id>` with a default value of `/aws/service/ami-amazon-linux-latest/amzn2-ami-hvm-x86_64-gp2`. This special type will automatically fetch the latest Amazon Linux 2 AMI ID from SSM Parameter Store.
    -   Create another parameter called `InstanceType` of type `String` with a default value of `t2.micro`.
3.  **Define the EC2 Resource:**
    -   Add a `Resources` section.
    -   Define a resource with the logical ID `MyEC2Instance`.
    -   Set its type to `AWS::EC2::Instance`.
    -   For its properties:
        -   `ImageId`: Use `!Ref LatestAmiId` to reference the parameter.
        -   `InstanceType`: Use `!Ref InstanceType` to reference the parameter.
        -   `Tags`: Add a tag with a Key of `Name` and a Value of `CFN-Created-EC2`.
4.  **Define an Output:**
    -   Add an `Outputs` section.
    -   Define an output with the logical ID `InstanceId`.
    -   Set its value to `!Ref MyEC2Instance`.
    -   Add a description.
5.  **Deploy the Stack:**
    -   Navigate to the CloudFormation console. Create a new stack "with new resources".
    -   Upload your template file.
    -   Give the stack a name like `Lab-EC2-Stack`.
    -   Leave the parameters as their defaults and create the stack.
    -   Watch the "Events" tab as CloudFormation provisions the instance. Once complete, check the "Outputs" tab and verify the instance ID.

#### **Part B: Update the Stack with a Change Set (30 mins)**

1.  **Modify the Template:**
    -   Add a new `SecurityGroups` property to your `MyEC2Instance` resource. The value should be a list containing a security group ID from your account (e.g., the default security group).
2.  **Create a Change Set:**
    -   In the CloudFormation console, select your stack and choose "Update".
    -   Upload your modified template.
    -   On the next screen, CloudFormation will show the changes to your template. Proceed.
    -   **Crucially, at the bottom of the "Review" page, expand "Change set preview".** You should see a modification planned for your EC2 instance.
    -   Instead of clicking "Update stack", click "Create change set".
3.  **Review and Execute:**
    -   The console will take you to the change set page. Review the proposed changes. It should show a "Modify" action for your EC2 instance.
    -   If it looks correct, click "Execute change set".
    -   CloudFormation will now apply the change to your stack.

---

### **15.4 Teaching Notes**

-   **YAML over JSON:** While CloudFormation supports both, YAML is generally preferred as it's more human-readable and allows for comments. Encourage students to use YAML from the start.
-   **IaC is a Mindset Shift:** Emphasize that once you adopt IaC, the template becomes the "source of truth". All changes should be made in the template and deployed via CloudFormation. Manual changes (making changes in the console) should be strictly forbidden as they cause drift.
-   **Use the Documentation:** The AWS CloudFormation User Guide is essential. No one memorizes all the resource types and properties. Teach students how to find the documentation for a resource (e.g., `AWS::EC2::VPC`) to see all its required properties and available attributes.
-   **Change Sets are a Safety Net:** Compare creating a change set to a `git diff` or a pull request review. It's the step where you pause and verify your intended changes before applying them to a live system. It is the single most important best practice for updating production stacks.
-   **Start Simple:** The lab starts with a single EC2 instance for a reason. IaC can be daunting. By starting with one simple resource, students can grasp the core concepts (parameters, resources, outputs) without being overwhelmed by a complex, multi-resource template.

---

## **Week 16: Capstone Project: Full Stack Deployment with IaC**

**Duration:** 3 Hours (30 mins Lecture, 2.5 Hours Lab)

**Core Topics:** Advanced CloudFormation (Nested Stacks, Cross-Stack References, Deletion Policies), Capstone Project Deployment, Course Review.

---

### **16.1 Learning Objectives**

By the end of this session, students will be able to:
-   Structure a complex application into multiple, logical CloudFormation templates using nested stacks.
-   Share resources and values between stacks using cross-stack references (`Export`/`ImportValue`).
-   Protect critical resources from accidental deletion using Deletion Policies.
-   Deploy the entire, multi-tier sentiment analysis application from scratch using a single root CloudFormation template.
-   Synthesize the knowledge gained throughout the course to build a production-grade AWS solution.

---

### **16.2 Detailed Content**

#### **16.2.1 Structuring a Large Application with Nested Stacks**

-   **The Problem with Monolithic Templates:** A single template for our entire application would be thousands of lines long. It would be difficult to read, manage, and reuse. A small change to one part would require updating the entire, massive stack.
-   **Solution: Nested Stacks:** CloudFormation allows you to treat a template as a resource (`AWS::CloudFormation::Stack`) within another template. This allows you to create a hierarchy of stacks.
-   **Common Structure:**
    -   **Root Stack:** The main stack you deploy. It contains no resources itself, only other nested stacks and the parameters needed to link them.
    -   **Network Stack:** Defines the foundational components: VPC, Subnets, Route Tables, NAT Gateway. It `Outputs` the IDs of these resources.
    -   **Security Stack:** Defines all the Security Groups. It `Imports` the VPC ID from the Network Stack and `Outputs` the IDs of the security groups.
    -   **Application Stack(s):** Defines the application tiers (e.g., Web Tier ALB, ASG, Launch Template). It `Imports` values from the Network and Security stacks.

#### **16.2.2 Sharing Resources with Cross-Stack References**

-   **Concept:** The mechanism for passing `Outputs` from one stack to be used as inputs in another stack.
-   **How it Works:**
    1.  **`Export`:** In the `Outputs` section of a stack (e.g., the Network stack), you give an output a unique `Export` name.
        ```yaml
        Outputs:
          VPCID:
            Description: The ID of the VPC
            Value: !Ref VPC
            Export:
              Name: !Sub "${AWS::StackName}-VPCID" 
        ```
    2.  **`Fn::ImportValue`:** In another stack (e.g., the Security stack), you use the `!ImportValue` intrinsic function to reference the exported value.
        ```yaml
        Resources:
          WebAppSG:
            Type: AWS::EC2::SecurityGroup
            Properties:
              VpcId: !ImportValue My-Network-Stack-VPCID
        ```
-   **Important:** A stack cannot be deleted if another stack is importing one of its outputs. This creates a safety dependency.

#### **16.2.3 Protecting State with Deletion Policies**

-   **Problem:** By default, when you delete a CloudFormation stack, it deletes all the resources it created. This is catastrophic for stateful resources like an RDS database or an S3 bucket full of user data.
-   **Solution: `DeletionPolicy`**
    -   An attribute you can set on any resource in a template.
    -   **`DeletionPolicy: Retain`:** When the stack is deleted, CloudFormation will abandon the resource but will not delete it. You are then responsible for managing it manually.
    -   **`DeletionPolicy: Snapshot`:** For resources that support it (like RDS, ElastiCache, Redshift), CloudFormation will take a final snapshot before deleting the resource.
-   **Best Practice:** **Always** set `DeletionPolicy: Retain` (or `Snapshot`) on your stateful resources (databases, S3 buckets with important data).

---

### **16.3 Lab: Capstone Project Deployment**

**Objective:** To deploy the entire multi-tier sentiment analysis application using a set of nested CloudFormation templates.

**(Note: This is a challenging, capstone lab. Provide students with the completed templates, but have them assemble and deploy the stacks themselves.)**

#### **Provided Templates:**

-   `00-root.yaml`: The main template that nests all the others.
-   `01-network.yaml`: Creates the VPC, Subnets, IGW, NAT Gateway, and Route Tables. Exports resource IDs.
-   `02-security.yaml`: Creates all Security Groups. Imports the VPC ID. Exports SG IDs.
-   `03-datastore.yaml`: Creates the S3 bucket and the RDS instance. Imports network and security info. Sets `DeletionPolicy: Retain`. Exports the bucket name and DB endpoint.
-   `04-app.yaml`: Creates the Launch Templates, IAM Roles, ALBs, and ASGs for all three application tiers. Imports values from all previous stacks.

#### **Part A: Deploy the Foundation (1 hour)**

1.  **Deploy the Network Stack:**
    -   First, deploy the `01-network.yaml` template as a standalone stack named `Prod-Network`.
    -   Examine its outputs and exported values.
2.  **Deploy the Security Stack:**
    -   Deploy the `02-security.yaml` template as a standalone stack named `Prod-Security`.
    -   This stack will use `!ImportValue` to get the VPC ID from the `Prod-Network` stack.
    -   Examine its outputs.
3.  **Deploy the Data Store Stack:**
    -   Deploy the `03-datastore.yaml` template as `Prod-Data`.
    -   Review the template to see how it uses `DeletionPolicy`.

#### **Part B: Deploy the Full Application with the Root Stack (1.5 hours)**

1.  **Upload Templates to S3:**
    -   CloudFormation needs a location to fetch the nested stack templates from. Create a new S3 bucket (e.g., `cfn-templates-yourname`).
    -   Upload the `01-network.yaml`, `02-security.yaml`, `03-datastore.yaml`, and `04-app.yaml` files to this bucket.
2.  **Review the Root Template:**
    -   Open `00-root.yaml`.
    -   Observe how it defines resources of type `AWS::CloudFormation::Stack`.
    -   Notice the `TemplateURL` property points to the S3 location of the nested template.
    -   See how `Parameters` are used to pass values (like exported VPC IDs) between the nested stacks.
3.  **Deploy the Capstone Stack:**
    -   In the CloudFormation console, create a new stack using the `00-root.yaml` template.
    -   Give it a name like `Sentiment-App-Capstone`.
    -   You will need to provide the S3 bucket URL where you stored the templates as a parameter.
    -   Deploy the stack.
4.  **Observe the Magic:**
    -   Watch the "Nested Stacks" tab. You will see CloudFormation provision the network stack first, then the security stack, then the data store, and finally the application tier, all in the correct order based on their dependencies.
5.  **Final Test and Teardown:**
    -   Once the root stack is `CREATE_COMPLETE`, the entire application is running.
    -   Find the public ALB DNS name in the outputs of the App stack and test the application end-to-end as you did in Week 10.
    -   To clean up, simply **delete the `Sentiment-App-Capstone` root stack**. CloudFormation will correctly tear down all the nested stacks and their resources in the reverse order of creation.
    -   Verify that the S3 bucket and RDS instance (which had `DeletionPolicy: Retain`) were *not* deleted and clean them up manually.

---

### **16.4 Teaching Notes**

-   **This is the Summit:** Congratulate the students. Reaching this point and successfully deploying the capstone project means they have demonstrated a professional-level understanding of AWS architecture. This is a significant achievement.
-   **Nested Stacks are the Pro Standard:** Simple projects can use single templates, but any real-world application of significant size will use nested stacks (or a higher-level tool like Terraform or the CDK which use similar concepts). It is the standard for managing complex environments with CloudFormation.
-   **Show the Power of IaC:** The teardown is just as important as the deployment. The ability to destroy an entire, complex application environment with a single command is a powerful demonstration of the benefits of IaC.
-   **The "Retain" Deletion Policy is a Lifesaver:** Tell the story of someone accidentally deleting a production stack and wiping out the main database because they forgot to set the Deletion Policy. It's a lesson students will remember.
-   **Course Review:** After the lab, spend the last 20-30 minutes reviewing the entire course journey. Put up the initial architecture diagram from Week 1 and then the final architecture diagram. Show how each week added a new piece to the puzzle, from basic EC2 to a fully automated, secure, global, and resilient application deployed with Infrastructure as Code. This provides a strong sense of closure and accomplishment.

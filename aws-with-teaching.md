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

-   **Understanding the Free Tier & `t2.micro`:**
    -   **Be Specific:** The Free Tier includes 750 hours of `t2.micro` (or `t3.micro` in regions where `t2.micro` is unavailable) usage *per month* for 12 months from account creation. Emphasize that 750 hours is enough to run one instance 24/7 for a month. If they run two instances, they will consume their free hours twice as fast.
    -   **Lifecycle:** What happens after 12 months? The card on file starts getting charged standard On-Demand rates. What happens if they exceed 750 hours in a month? They get charged for the overage.
    -   **Actionable Advice:** Show students how to use the "Top Free Tier Services by Usage" widget in the Billing console. Advise them to set a specific billing alarm with a threshold of $1 to get an early warning before significant charges are incurred.

-   **Explaining Burstable Performance (T-family instances):**
    -   **The "Leaky Bucket" Analogy:** This is the most effective way to explain CPU credits.
        -   Imagine every T-instance has a bucket for "CPU Credits".
        -   When the instance is idle (using less CPU than its baseline), it's slowly filling the bucket with credits.
        -   When the application needs to "burst" (e.g., handle a sudden spike in traffic), it can drain credits from the bucket to perform at 100% CPU.
        -   If the bucket runs empty, the instance's performance is "throttled" down to its low baseline level until it can earn more credits.
    -   **Show, Don't Just Tell:** In the CloudWatch console, graph the `CPUCreditBalance` and `CPUCreditUsage` metrics for an instance. Run a `stress` command on the instance and have the class watch the credit balance plummet and the usage spike. This makes the concept tangible.
    -   **When to Use It:** Burstable instances are perfect for workloads that are usually idle but have occasional peaks: development servers, low-traffic websites, microservices. They are a poor choice for sustained, high-CPU tasks like video encoding or scientific computing.

-   **Introducing Monitoring (The "Why, What, How"):**
    -   **Why:** Start with the core principle: "You can't manage, secure, or optimize what you can't measure." Monitoring isn't optional; it's fundamental.
    -   **What (Level 1 - The Basics):** In the EC2 console's "Monitoring" tab, show the default metrics AWS provides: `CPUUtilization`, `NetworkIn/Out`, `DiskRead/WriteOps`. Explain what each one tells you about the instance's health.
    -   **What (Level 2 - The Missing Pieces):** Ask the class, "What critical metrics are missing here?" Guide them to realize that **Memory Usage** and **Disk Space Used** are not provided by default. This is because AWS cannot see "inside" the guest operating system. This provides the perfect motivation for needing the CloudWatch Agent.
    -   **How (The Tools):** Introduce CloudWatch as the central monitoring service. Explain that it's a repository for Metrics, Logs, and Events. The lab in Week 8 will cover the "how" of deploying the agent, but this week plants the seed for why it's necessary.

-   **Pricing Models are a Core Concept:** Spend time on this. Choosing the right pricing model is one of the most impactful cost-optimization levers. Use the AWS Pricing Calculator to model different scenarios for the class.

-   **Golden AMIs are an Operational Best Practice:** Contrast the "Golden AMI" (immutable infrastructure) pattern with logging into a running server to apply updates (mutable infrastructure). The Golden AMI approach is more reliable, repeatable, and scalable.

-   **EC2 Image Builder Automates the Toil:** Manually creating Golden AMIs is tedious and error-prone. Image Builder codifies the process, making it a repeatable, automated pipeline. Frame it as "CI/CD for your machine images."

-   **SSM is the Modern Way to Manage Fleets:** SSH should not be used for fleet-wide management. It doesn't scale, it's hard to audit, and it requires open ports. SSM is the secure, scalable, and auditable solution for tasks like patching and running commands.

-   **The Power of Patch Manager:** Demonstrate the "compliance" view in Patch Manager. Show students how they can see, at a glance, which instances out of a fleet of hundreds are missing critical security patches. This is a powerful visual for demonstrating its value.

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
-   **Session Manager vs. SSH: A Security Showdown:** Frame this as a direct comparison.
    -   **Attack Surface:** SSH requires opening port 22 to the internet (or a bastion), which is a constant target for scans and brute-force attacks. Session Manager requires **zero** inbound ports, completely eliminating that attack vector.
    -   **Credentials:** SSH relies on fragile, long-lived key pairs that can be lost, stolen, or compromised. Session Manager uses temporary credentials derived from an IAM role, which automatically expire.
    -   **Auditing:** With SSH, you only know that someone connected. With Session Manager, every command run during a session can be logged to CloudWatch Logs and CloudTrail, providing a complete, immutable audit trail.
    -   **Authorization:** SSH authorization is binary (`sshd_config`). Session Manager authorization is granular IAM. You can grant a user permission to start a session, but not to terminate instances, for example.

-   **IAM Role Attachment - The Two-Part Handshake:** Use a "valet key" analogy.
    1.  **The Trust Policy (`sts:AssumeRole`):** This is like giving the valet company (the EC2 Service, `ec2.amazonaws.com`) permission to use your car keys. It defines *who* can assume the role. The principal is the service.
    2.  **The Permissions Policy (e.g., `AmazonSSMManagedInstanceCore`):** This is like telling the valet what they are allowed to do (park the car, retrieve the car). It defines *what* the service can do *after* assuming the role.
    -   Show both JSON policy documents side-by-side to make this distinction clear. It's a fundamental IAM concept that is often confusing.

-   **SSM Port Forwarding is a Developer's Best Friend:**
    -   **The Scenario:** You have an RDS database in a private subnet. The security group only allows access from the application tier's security group. How can a developer on their laptop connect their local SQL client to it for debugging?
    -   **The Wrong Way:** Temporarily adding their home IP address to the database security group. This is a huge security risk and hard to manage.
    -   **The Right Way:** Use SSM Port Forwarding. A developer with the correct IAM permissions can run a single CLI command (`aws ssm start-session...`) to create a secure tunnel *through* a managed EC2 instance to the database. It forwards a local port (e.g., `localhost:3306`) to the private RDS endpoint. No security group changes are needed.

-   **Session Preferences as Enforceable Policy:**
    -   Show the JSON configuration for session preferences in the SSM console. You can define things like:
        -   **Idle session timeout:** Automatically terminate sessions after 15 minutes of inactivity.
        -   **Shell profiles:** Pre-load the shell with specific aliases or environment variables.
        -   **KMS Key Encryption:** Encrypt the entire session stream with a customer-managed KMS key for maximum security.
    -   Explain that these are not just suggestions; they are enforced by the service, providing a strong governance control.

-   **ABAC is the Future of Permissions:** While RBAC (Role-Based Access Control) is common, it can lead to "role explosion" (hundreds of slightly different roles). ABAC (Attribute-Based Access Control) is more scalable. You create a few generic roles with policies that say, "Allow access to resources if the user's `project` tag matches the resource's `project` tag." This is a more advanced but powerful model for large enterprises.

-   **Permissions Boundaries are a Safety Net:** Use the analogy of a "leash". You might give a junior developer a role that has `AdministratorAccess` (the permissions), but you put a Permissions Boundary on that role that says they cannot perform any actions on IAM or Organizations resources. Even though their permissions *allow* it, the boundary *prevents* it. It's a powerful tool for safely delegating permissions without losing control.

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

-   **EBS vs. EFS: The Core Distinction:**
    -   **Analogy:** EBS is like an internal hard drive in your desktop computer. It's directly attached, it's very fast (low latency), and only that one computer can use it. EFS is like a network-attached storage (NAS) drive in your office. It's accessible over the network, multiple people (EC2 instances) can connect to it and share files simultaneously, but there's slightly more latency because of the network hop.
    -   **Scope:** EBS is zonal. An EBS volume lives in a single AZ and can only be attached to an instance in that same AZ. EFS is regional. It has mount targets in multiple AZs, providing built-in resilience.
    -   **Use Case:** Use EBS for things that need high speed and are tied to a single instance, like a database's data files or a boot volume. Use EFS for shared content that needs to be accessed by multiple instances, like a web server farm's shared assets or a content management system's uploads directory.

-   **NFS Protocol Basics:**
    -   You don't need to be an NFS expert, but understand the basics. It's a client-server protocol. The EC2 instances are the "clients," and the EFS service is the "server."
    -   The `mount` command establishes the connection. The EFS mount helper (which is installed in the lab) simplifies this process by handling the recommended NFS options for you.
    -   Explain that because it's a network protocol, traffic between the EC2 instance and EFS must be allowed by security groups. The EFS security group needs an inbound rule allowing NFS traffic (port 2049) from the security group of the EC2 instances that need to connect.

-   **EFS Performance & Cost Tuning:**
    -   **Performance Modes:** For 99% of workloads, "General Purpose" is the correct choice. "Max I/O" is a highly specialized mode for massively parallel workloads (like big data analytics) and actually has *higher* latency for common operations.
    -   **Throughput Modes:** "Bursting" is the default and is fine for most use cases. It provides a baseline throughput and allows you to burst higher based on the size of your file system. If you have a consistently high-throughput workload, "Provisioned Throughput" allows you to pay for a guaranteed level of performance.
    -   **Lifecycle Management:** This is a key cost-optimization feature. Show them how to set up a policy to automatically move files that haven't been accessed in 30 days to the EFS Infrequent Access (IA) storage class, which is significantly cheaper.

-   **RAID is for Performance, Not Resilience (on EBS):**
    -   It's critical to stress that using RAID 0 with EBS volumes *increases* the risk of data loss. If any single EBS volume in the RAID array fails, the entire array is lost.
    -   EBS volumes are already replicated within their AZ, so you don't use RAID for resilience. You use it purely to stripe I/O across multiple volumes to achieve higher IOPS or throughput than a single volume can provide. This is a pattern for extreme performance requirements.
    -   **Resilience comes from Snapshots and Multi-AZ architectures**, not from RAID on EBS.

-   **AWS Backup is a "Single Pane of Glass" for Data Protection:** Before AWS Backup, you had to manage backups separately for each service (EBS snapshots, RDS snapshots, EFS backups, etc.). AWS Backup centralizes this. You create a single "Backup Plan" that can define backup frequency, lifecycle (when to move to cold storage), and retention for resources across multiple services, all tagged with `backup-plan: daily-prod`, for example. It greatly simplifies data protection policy management.

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

-   **IPAM is the Foundation:** Before you lay a single brick (subnet), you need the architectural blueprint (the IP Address Management scheme). A well-planned CIDR range for the VPC and a logical scheme for subnets (e.g., `10.10.1.x` for public-az-1, `10.10.101.x` for private-az-1) makes the network understandable and scalable. The worst mistake is to pick random, overlapping CIDR ranges.

-   **Subnet Routing Logic Explained:**
    -   **Analogy:** A Route Table is like a set of road signs for a specific neighborhood (subnet). Each sign (route) tells network traffic where to go based on its final destination address.
    -   **The "Most Specific Match" Rule:** This is the single most important rule of routing. The router will always choose the most specific route available. A route to `10.10.0.0/16` (the local VPC route) is less specific than a route to `0.0.0.0/0` (the Internet Gateway). A request to `10.10.1.5` will match the `/16` route and stay local. A request to Google's `8.8.8.8` will not match the local route, so it will fall through to the `0.0.0.0/0` route and be sent to the IGW.
    -   **One Route Table, Many Subnets:** You can (and should) associate subnets with the same function (e.g., all public subnets) with the same route table. This simplifies management.

-   **NAT Gateway Costs & Alternatives:**
    -   **The Cost Model:** Be very clear about this, as it's a common source of surprise bills. You pay an hourly rate for each NAT Gateway to be running, **PLUS** a per-gigabyte data processing charge for all traffic that flows through it.
    -   **High Availability:** A NAT Gateway is a zonal resource. For true HA, you must provision one NAT Gateway in **each** Availability Zone that has private subnets needing outbound internet access, and create specific route tables for each AZ's private subnets to point to their local NAT Gateway.
    -   **The Alternative (NAT Instance):** You can run your own NAT service on an EC2 instance. It's more complex to manage (you are responsible for patching, HA, and scaling it), but it can be significantly cheaper for high-throughput workloads because you don't pay the data processing fee, only the EC2 instance cost. It's a classic cost vs. management trade-off.

-   **VPC Flow Logs for Monitoring & Security:**
    -   **What it is:** A log of all the IP traffic flowing to and from network interfaces in your VPC. It records the source/destination IPs and ports, the protocol, and whether the traffic was `ACCEPT` or `REJECT` by a Security Group or Network ACL.
    -   **What it is NOT:** It is **not** a packet capture. It does not contain the payload of the traffic.
    -   **Demonstration:** Enable Flow Logs to publish to a CloudWatch Log Group. Go to Logs Insights and run a query to find rejected traffic: `fields @timestamp, @message | filter @message like /REJECT/`. This is a powerful way to troubleshoot security group rules or detect malicious scanning activity.

-   **SG vs. NACL - The "Bouncer vs. Border Fence" Analogy:**
    -   A **Security Group** is like a bouncer at the door of a club (the EC2 instance). It's stateful; if it lets you in, it automatically knows to let you out again. It can check your specific ID (reference another SG).
    -   A **Network ACL** is like the border fence for the entire country (the subnet). It's stateless; it checks your passport on the way in and on the way out, requiring explicit rules for both directions. It's a blunt instrument, good for blocking known bad IPs at the subnet level.
    -   **Best Practice:** Rely primarily on Security Groups for fine-grained control. Leave the default Network ACL (which allows all traffic) in place unless you have a specific need to block traffic at the subnet level.

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

-   **Endpoint vs. Internet Routing: The "Private Road" Analogy:**
    -   **Internet Routing:** By default, when an EC2 instance calls an AWS service API like S3, the traffic goes out through a NAT Gateway to the public S3 endpoint. This is like taking public highways to get to a building across town. It works, but it's exposed to public traffic and you pay tolls (data transfer costs).
    -   **Endpoint Routing:** A VPC Endpoint creates a private, dedicated road directly from your VPC to the S3 service within the AWS network. The traffic never touches the public internet. It's more secure, lower latency, and there are no tolls (no data transfer costs for Gateway Endpoints).

-   **Gateway vs. Interface Endpoints:**
    -   **Gateway Endpoints (S3/DynamoDB):** Think of this as a special, high-performance "exit ramp" from your VPC directly to the S3 or DynamoDB service. You configure it by adding a `prefix list` for the service to your route table. It's a routing instruction. They are free.
    -   **Interface Endpoints (Most other services):** This is like placing a "branch office" of the target service *inside* your VPC. It's an Elastic Network Interface (ENI) with a private IP address in your subnet. You communicate with it via its private DNS name. Because it consumes an IP and provides broader service access, it has an hourly cost.
    -   **When to use which?** Always use a Gateway Endpoint for S3 and DynamoDB. For any other service (SQS, Kinesis, SSM, etc.), you must use an Interface Endpoint.

-   **VPC Endpoint Policies are an Extra Layer of Security:**
    -   An endpoint policy is a resource-based policy that you attach directly to the endpoint itself. It acts as an additional guardrail.
    -   **Scenario:** You have an Interface Endpoint for SQS. The IAM role on your EC2 instance allows it to access any SQS queue (`sqs:*`). You can apply an endpoint policy that says, "Only allow `sqs:SendMessage` actions through this endpoint, and only to the `prod-orders-queue`."
    -   Now, even if the EC2 instance is compromised, the attacker cannot use the endpoint to delete messages or access other queues. It's a powerful tool for enforcing least privilege at the network level. Show the JSON policy structure to make this concrete.

-   **Demonstrating Cost Savings:**
    -   The most significant saving is on **NAT Gateway data processing charges**. If you have instances in a private subnet that are uploading or downloading terabytes of data to S3, that traffic would normally flow through a NAT Gateway, incurring a fee for every gigabyte processed.
    -   By implementing an S3 Gateway Endpoint, that same traffic now flows over the "private road" and bypasses the NAT Gateway entirely, reducing the data processing bill to zero. For data-intensive applications, this can lead to thousands of dollars in savings.

-   **TGW is the Hub, Peering is Point-to-Point:**
    -   **VPC Peering:** It's like a direct extension cord between two VPCs. Simple for connecting a few VPCs, but it quickly becomes a complex mesh that is hard to manage (`n*(n-1)/2` connections).
    -   **Transit Gateway (TGW):** It's like a power strip or a network router. Every VPC plugs into the central TGW hub. Any VPC can talk to any other VPC through the hub (if the route tables allow it). This "hub-and-spoke" model is vastly more scalable and manageable for any organization with more than a handful of VPCs.

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

-   **ALB in Public, EC2 in Private: A Core Security Pattern:** This is a non-negotiable architecture for web applications. The ALB is the hardened, managed, public-facing entry point. The EC2 instances, which contain your sensitive application code and have access to the database tier, should **never** be in a public subnet. Their security groups should only allow traffic from the ALB's security group, nothing else.

-   **Health Checks are the Heartbeat of HA:** A system isn't highly available if it can't detect failures.
    -   **EC2 vs. ELB Health Checks:** The default ASG health check (`EC2`) only verifies that the instance is running. This is insufficient. An instance can be running, but the Nginx process could be frozen. The `ELB` health check type is critical because the ASG will only consider an instance "healthy" if it is *also* passing the application-level health checks from the load balancer.
    -   **Configuring Health Checks:** A good health check endpoint should be lightweight and test the application's true health. It shouldn't be the homepage, which might be complex. A dedicated `/health` endpoint that returns a simple `200 OK` is a best practice. The thresholds (e.g., "fail after 2 consecutive failures over 20 seconds") are a trade-off between reacting quickly and avoiding false positives from transient network blips.

-   **Configuring Scaling Policies:**
    -   **Target Tracking is the "Easy Button":** For most web apps, Target Tracking is the best choice. It's declarative. You tell the ASG *what* you want (e.g., "keep average CPU at 50%"), and it handles the *how*.
    -   **Step vs. Simple Scaling:** These are more imperative and give you finer control, but are more complex to manage. Step scaling is useful if you want to add capacity more aggressively during extreme traffic spikes (e.g., "if CPU > 90%, add 4 instances at once"). Simple scaling is largely legacy.
    -   **Choosing a Metric:** CPU is a common starting point, but `RequestCountPerTarget` on the ALB is often a better metric for web applications, as it's more directly tied to user traffic.

-   **Instance Refresh for Graceful Deployments:**
    -   This is how you perform a zero-downtime deployment for an EC2-based application. The ASG will perform a "rolling update".
    -   It will launch a new instance with the updated Launch Template, wait for it to pass the ELB health checks, and then terminate one of the old instances. It repeats this process until the entire fleet has been replaced.
    -   The "minimum healthy percentage" setting is key. Setting it to `90%` ensures that the ASG will not take more than 10% of your fleet offline at any time during the update, preserving your application's capacity.

-   **Cross-Zone Load Balancing:**
    -   This should almost always be **enabled** for Application Load Balancers.
    -   **Without it:** If you have 10 instances in AZ-A and 2 instances in AZ-B, the ALB will send 50% of traffic to the AZ-A group and 50% to the AZ-B group. This means the 2 instances in AZ-B will receive far more traffic per instance and become overloaded.
    -   **With it:** The ALB treats all instances in the target group as a single pool, regardless of AZ. It will distribute traffic evenly to all 12 instances, ensuring no single instance gets overloaded.

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

-   **Observability vs. Monitoring:**
    -   **Monitoring:** "Tells you whether the system is working." It's about observing pre-defined metrics that you have decided are important (e.g., CPU, Memory).
    -   **Observability:** "Tells you why the system isn't working." It's about being able to ask arbitrary questions about your system's state from the outside without having to ship new code to answer them. This requires rich, high-cardinality data from logs, metrics, and traces. The CloudWatch Agent is the first step towards true observability because it provides the raw data needed.

-   **Metric Aggregation and Statistics:**
    -   **Analogy:** Think of a cash register. `Sum` is the total revenue for the day. `Average` is the average transaction value. `Max` is the largest single sale. `SampleCount` is the number of transactions.
    -   **Why it Matters:** The statistic you choose dramatically changes the meaning of a metric. For `CPUUtilization`, you almost always want the `Average` across the ASG. For `NetworkIn`, you might want the `Sum` to calculate total data transfer for billing. For latency, `Average` can be misleading; using percentiles like `p95` or `p99` (which CloudWatch supports) gives a much better picture of user experience.

-   **Log Filtering and Searching (Logs Insights):**
    -   **Structured Logs are a Superpower:** Emphasize this. Once logs are in JSON, you can use Logs Insights for powerful, SQL-like querying that is impossible with plain text. This transforms logs from a forensic tool into a proactive analytics tool.
    -   **Live Demo:** Perform a live demo of Logs Insights. Start with a simple filter (`filter @message like /ERROR/`), then add parsing (`parse @message "[*] *" as level, msg`), and finally group and aggregate (`| stats count(*) by level`). This shows the progression from simple search to powerful analytics.

-   **Alarm State Management:**
    -   **The Three States:** `OK`, `ALARM`, `INSUFFICIENT_DATA`. The last one is critical. It means CloudWatch does not have enough data points to evaluate the alarm's condition. This often happens if an instance stops reporting metrics entirely. You should always configure how this state is treated; often, treating `INSUFFICIENT_DATA` as `ALARM` is the safest option.
    -   **Composite Alarms Reduce Noise:** Tell a story about being on-call at 3 AM and getting 50 separate alerts for one database failure (high CPU, high latency, high connections, etc.). Composite alarms are the cure for this. They allow you to define what a *real* outage looks like for your application (e.g., "high 5xx errors from the ALB **AND** high CPU on the app tier") and only alert on that.

-   **Dashboard Creation as Storytelling:**
    -   A good dashboard tells a story about the health of an application. It should be organized from the top down.
    -   **Top Layer (User Experience):** Start with metrics that represent the user experience: ALB latency, 5xx error rate, `RequestCount`.
    -   **Middle Layer (Application Tier):** Show the ASG metrics: Average CPU/Memory, number of running instances.
    -   **Bottom Layer (Data Tier):** Show the key database metrics: RDS CPU, connection count.
    -   This structure allows an operator to quickly determine if a problem is impacting users and then drill down to find the responsible tier.

-   **X-Ray is for Finding the "Why":** CloudWatch tells you *what* is broken (high latency). X-Ray tells you *why* it's broken (the backend call to the DynamoDB table is slow). It provides the context that metrics alone lack.

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

-   **Multi-AZ is for HA, Read Replicas are for Scale:** This is a fundamental concept that is often confused.
    -   **Analogy:** Multi-AZ is like having a "hot spare" engine on a plane, running in perfect sync. If the primary engine fails, the spare takes over instantly and the plane keeps flying. You don't get more speed, just more reliability. A Read Replica is like adding more engines to the plane to carry more cargo (read traffic). It increases capacity but doesn't help if the main engine fails.
    -   **Replication Type:** Multi-AZ uses **synchronous** replication. A write is not considered complete until it has been committed on both the primary and the standby. This guarantees data durability but has a small latency impact. Read Replicas use **asynchronous** replication. There can be a small "replica lag," so it's not suitable for read-after-write consistency needs.

-   **Database Parameter Groups: The "Settings Menu":**
    -   Every database engine has hundreds of configuration settings (`innodb_buffer_pool_size`, `max_connections`, etc.). A Parameter Group is simply a container for these settings.
    -   **Best Practice:** You can **never** modify the `default` parameter group. Always create a new, custom parameter group for your database. This allows you to tune settings for your specific workload and control when changes are applied. `Static` parameters require a database reboot to take effect, while `Dynamic` parameters can be applied immediately.

-   **Connection Pooling and RDS Proxy:**
    -   **The Problem:** Establishing a database connection is an expensive operation (TCP handshake, SSL negotiation, authentication). If every Lambda invocation or application request opens a new connection, the database can be quickly overwhelmed and run out of memory.
    -   **The Solution:** RDS Proxy acts as a "connection broker." It maintains a warm pool of established connections to the database. Your application connects to the proxy (which is very fast), borrows a connection from the pool to run its query, and then returns it.
    -   **Key Benefit:** This dramatically improves performance and resilience, especially for serverless workloads. It also makes database failovers almost transparent to the application, as the proxy will hold and route new connections to the promoted standby.

-   **Backup, Restore, and Snapshot Procedures:**
    -   **Automated vs. Manual:** Automated backups are for point-in-time recovery (e.g., "restore the database to how it looked last Tuesday at 2:37 PM"). They are automatically deleted after their retention period. Manual snapshots are for long-term archival or creating clones for dev/test environments. They persist until you delete them.
    -   **Restore Drill:** The only way to know if your backups work is to test them. A crucial operational task is a periodic "restore drill," where you restore a snapshot to a new temporary instance and verify the data's integrity.

-   **Secret Rotation with Secrets Manager:**
    -   **How it Works:** Show a diagram of the rotation process.
        1.  Secrets Manager invokes a dedicated Lambda function on a schedule.
        2.  The Lambda function generates a new, strong password.
        3.  It connects to the RDS database using the *old* password and issues a command to change the password to the new one (`ALTER USER...`).
        4.  It then updates the secret value stored in Secrets Manager with the *new* password.
    -   Your application, which reads the secret at runtime, will automatically pick up the new password on its next restart or connection attempt without any code changes. This is a massive security and operational win.

-   **Performance Insights is a Game Changer:** Before Performance Insights, DBAs had to rely on complex queries against internal database tables to diagnose issues. Show a "before and after" scenario. Emphasize that the first place to look for any database slowdown is the Performance Insights dashboard to see what SQL queries or wait events are causing the load.

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

-   **Diagrams are Essential:** This week's architecture is the most complex so far. Start the lecture with a clear diagram showing the three tiers, the two ALBs (one internet-facing, one internal), the security groups, and the SQS queue. Show the flow of a user request from the internet, through the web tier, to the app tier, and finally the message being placed on the queue for the model tier. Refer back to it constantly.

-   **Security Group Chaining is the "Aha!" Moment:** When students see that the `App-SG` literally contains the ID of the `Web-SG` in its inbound rule source, the concept of tiered security clicks. It's powerful because it's dynamic; if the web tier scales up and adds new instances, you don't need to update the `App-SG` rules, as the new instances automatically belong to the `Web-SG`. It's far superior to static IP-based rules.

-   **Microservices vs. Monolith:**
    -   **Analogy:** A monolith is like a Swiss Army knife. It has many tools, but they are all physically coupled together. You can't upgrade the screwdriver without getting a whole new knife. A microservices architecture is like a toolbox. Each tool (service) is independent. You can upgrade your screwdriver without touching your hammer.
    -   **Trade-offs:** Microservices allow for independent scaling, independent technology choices (e.g., Python for the ML tier, Go for the API tier), and smaller, more focused teams. The cost is increased operational complexity: you now have a distributed system, and you need to worry about things like network latency, service discovery, and distributed tracing.

-   **SQS for Asynchronous Operations:** The sentiment analysis model is a perfect example of a slow, asynchronous task.
    -   **Why it's better:** What if the model takes 10 seconds to run? In a synchronous model, the user's browser would spin for 10 seconds, which is a terrible experience. By using SQS, the App Tier can respond to the user in milliseconds with "Request received." The decoupling also adds resilience; if the Model Tier is down for an hour, messages simply queue up, and no data is lost. The system heals itself when the Model Tier is healthy again.

-   **Service Discovery & Distributed Tracing (Theory):**
    -   **Service Discovery (e.g., Cloud Map):** In a dynamic, auto-scaling environment, how does Service A find the IP address of Service B? It can't be hard-coded. Service Discovery acts like a private phonebook for your services. When a new instance of Service B launches, it registers its IP address in the phonebook. Service A can then query the phonebook to get a list of healthy, available IPs for Service B.
    -   **Distributed Tracing (e.g., X-Ray):** Once you have many services, it becomes hard to trace a single request as it hops between them. X-Ray solves this by injecting a unique "Trace ID" into the request header. Each service adds its own timing data to the trace. X-Ray then stitches this together into a "service map" that visually shows the entire request path, highlighting errors and performance bottlenecks. These are advanced topics, but it's important for students to know what they are and why they're needed in a microservices world.

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

-   **Lambda is FaaS:** Lambda is the quintessential "Function as a Service". Emphasize that you are only providing the code (the function), and AWS handles everything else about the execution environment. This is a powerful paradigm shift that lets developers focus on business logic instead of infrastructure management.

-   **Cold Start Considerations:**
    -   **What it is:** A cold start is the latency incurred on the first invocation of a Lambda function. The service has to provision the execution environment, download your code, and start the runtime. Subsequent calls to this "warm" environment are much faster.
    -   **What influences it?** The biggest factors are the size of your deployment package, the chosen runtime (interpreted languages like Python and Node.js are generally faster to start than compiled languages like Java or .NET), and whether the function needs to connect to a VPC (which adds ENI attachment time).
    -   **How to mitigate it:**
        -   Keep your deployment package small. Use Lambda Layers for large dependencies.
        -   For latency-critical applications, use **Provisioned Concurrency**, which pays to keep a specified number of environments warm and ready to go, eliminating cold starts entirely for those invocations.

-   **Lambda Configuration Options:**
    -   **Memory is also CPU (and Network):** This is a key concept. In Lambda, you only configure the memory allocation. However, increasing the memory also proportionally increases the vCPU allocated to your function, as well as the available network bandwidth. If your function is CPU-bound, increasing its memory can make it run faster, which can actually *reduce* your costs since you are billed on a combination of memory and duration (GB-seconds).
    -   **Timeout:** Set the timeout just high enough for your function's expected execution time, with a small buffer. A timeout that is too short will cause functions to fail, while a timeout that is too long can lead to runaway costs if there's a bug causing an infinite loop.

-   **Error Handling and Retries:**
    -   **Asynchronous vs. Synchronous:** How Lambda handles errors depends on the invocation type.
        -   **Synchronous (e.g., API Gateway):** If the function fails, the error is returned directly to the caller. There are no automatic retries. The *caller* is responsible for retrying.
        -   **Asynchronous (e.g., S3, SNS):** If the function fails, Lambda will automatically retry the invocation twice, with delays between retries. If all three attempts fail, the event is sent to a Dead-Letter Queue (DLQ) if one is configured.
    -   **DLQs are Critical for Resilience:** For any asynchronous workflow, a Dead-Letter Queue is essential. It ensures that failed events are not lost forever. They can be inspected, re-processed, or archived for auditing, making the entire system more robust.

-   **Monitoring and Debugging:**
    -   **CloudWatch Logs are your best friend:** Every `print` or `console.log` statement in your function goes directly to a CloudWatch Log Stream. This is the primary way to debug. Each log stream also includes the `START`, `END`, and `REPORT` lines, which show the exact duration, memory used, and billable time for each invocation.
    -   **X-Ray for Distributed Applications:** When your Lambda function calls other AWS services (like S3, DynamoDB, or another Lambda function), use the X-Ray SDK to trace those calls. It will show you a visual breakdown of where your function is spending its time, which is invaluable for pinpointing performance bottlenecks.

-   **Event-Driven Thinking:** This is a different mindset. Instead of a linear, synchronous request-response, students need to think in terms of "When *this event* happens, trigger *that action*." The lab is a perfect, tangible example of this.

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

-   **S3 Bucket Naming:**
    -   **Globally Unique:** Bucket names share a global namespace across all AWS accounts. You cannot create a bucket with the same name as any other bucket in any other account. This is why you often see bucket names with organization and region prefixes (e.g., `mycompany-us-east-1-logs`).
    -   **DNS Compliance:** Bucket names must be DNS-compliant (no uppercase letters, no underscores, etc.). This is because they can be accessed via a DNS name (`bucket-name.s3.amazonaws.com`).

-   **CORS Configuration for API Calls:**
    -   **The "Same-Origin" Problem:** For security, web browsers enforce a "same-origin policy," which prevents a script on `domain-a.com` from making an API request to `domain-b.com`.
    -   **The Solution:** Cross-Origin Resource Sharing (CORS) is the mechanism to relax this policy. If your front-end is hosted on CloudFront (`www.myapp.com`) and it needs to make API calls to your ALB (`api.myapp.com`), you need to configure a CORS policy on the *server* (the ALB or the application behind it) that explicitly allows requests from the `www.myapp.com` origin. The S3 static website lab doesn't require CORS, but it's a critical concept for connecting a front-end to a backend API.

-   **S3 Cost Optimization Strategies:**
    -   **Storage Tiers are a Cost-Saving Pillar:** A huge amount of AWS spend is on S3. Understanding and using the different storage tiers is critical.
    -   **S3 Intelligent-Tiering is the "Easy Button":** For data with unknown or changing access patterns, this is the best choice. It automatically moves objects between frequent and infrequent access tiers to save you money without you having to do any analysis. There is a small monitoring and automation fee, so for data you *know* will be accessed infrequently, S3 Standard-IA is cheaper.
    -   **Lifecycle Policies for Predictable Patterns:** If you have a predictable access pattern (e.g., "logs are accessed frequently for 30 days, then rarely for a year, then can be archived"), a lifecycle policy is perfect. You can create a rule to automatically transition the data: S3 Standard -> S3-IA -> S3 Glacier -> Expire.

-   **Versioning and Lifecycle Policies - A Powerful Combo:**
    -   **Versioning is Your "Undo" Button:** Enabling versioning on a bucket is a critical data protection best practice. If you accidentally delete or overwrite an object, you can simply restore the previous version. It protects against human error.
    -   **Managing the Cost of Versioning:** The downside is that you now store (and pay for) multiple copies of your objects. This is where lifecycle policies come in. You can configure a lifecycle rule specifically for **noncurrent versions** to automatically delete them after a certain period (e.g., 30 days), keeping your storage costs in check while still providing a recovery window.

-   **KMS is All About Control and Audit:** SSE-S3 is secure, but SSE-KMS is what enterprises use. The ability to control the key policy (who can use the key) and see a full audit trail in CloudTrail of every single time the key is used for an encrypt or decrypt operation is a fundamental requirement for most compliance and security frameworks.

-   **Envelope Encryption:** Use a physical analogy. Your data is in a box. You lock that box with a key (the **Data Key**). You then put that key inside a high-security safe that requires multiple permissions to open (the **CMK in KMS**). To open the data box, you must first have permission to open the KMS safe to get the key. This is much more secure and scalable than managing millions of individual data keys.

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

-   **CloudFront for Performance AND Security:** While CloudFront is known as a CDN for performance, it's also a critical security layer. It absorbs DDoS attacks at the edge and provides a place to integrate WAF, shielding your origin from malicious traffic. The Origin Access Control (OAC) feature is a non-negotiable best practice that ensures users cannot bypass CloudFront and access your S3 bucket directly.

-   **CDN Caching Concepts:**
    -   **Cache Hit vs. Cache Miss:** A "cache hit" is when CloudFront can serve the content directly from the edge location, resulting in very low latency. A "cache miss" is when it has to go back to the origin (e.g., your S3 bucket) to fetch the content first. The goal is to maximize the "cache hit ratio."
    -   **Cache Key & TTL:** The `Cache-Control` and `Expires` HTTP headers sent by your origin are the primary way to control the Time-to-Live (TTL) for an object in the cache. The **Cache Key** determines what makes a request unique. By default, it's just the URL path. If your origin serves different content based on a query string or a cookie, you *must* add that element to the cache key, or all users will receive the same cached version.

-   **Invalidation Procedures (and why to avoid them):**
    -   An invalidation is a command to forcibly remove an object from every CloudFront edge cache around the world.
    -   **The Problem:** Invalidations are slow to propagate and you pay for every path you invalidate. They should be considered a last resort for emergency fixes.
    -   **The Better Way (Versioning):** A much better strategy for deploying new static assets (like CSS or JS files) is to embed a version identifier in the filename (e.g., `styles.d2a8e6.css` instead of `styles.css`). When you deploy a new version, the HTML references a new filename. This is a completely new object to the CDN, so it results in an immediate cache miss and fetch from the origin. This "cache busting" technique provides instant updates without needing slow and expensive invalidations.

-   **WAF Rule Configuration:**
    -   **Managed Rules are your First Line of Defense:** Always start by enabling the `AWSManagedRulesCommonRuleSet` and the `AmazonIpReputationList`. These are maintained and updated by the AWS threat intelligence team and protect you from the vast majority of common, automated attacks like SQL injection and botnets.
    -   **Rate-Based Rules:** These are crucial for mitigating application-level DDoS attacks or credential stuffing attempts. The key is to find the right threshold. Set it too low, and you might block legitimate power users; set it too high, and it won't be effective. Start with a high threshold and tune it down as you analyze your traffic patterns.
    -   **Action: Count vs. Block:** When deploying a new WAF rule, it's a best practice to first set its action to `Count`. This will log all the requests that *would have been blocked* without actually blocking them. You can then analyze these logs in CloudWatch to ensure the rule isn't generating false positives before you switch it to `Block` mode.

-   **DNS Propagation Explained:**
    -   **Analogy:** When you update a DNS record, it's like sending out a change of address notice. It takes time for that notice to reach every post office (DNS resolver) in the world.
    -   **TTL:** The Time-to-Live (TTL) on a DNS record tells resolvers how long they should cache the answer. A lower TTL (e.g., 60 seconds) means changes will propagate faster, but it also means more queries to your hosted zone (which has a cost). A higher TTL (e.g., 24 hours) is more efficient but makes changes slower.
    -   **Alias Records are Superior:** When pointing to AWS resources, Alias records are almost always better than CNAMEs. They are free, they can be used at the zone apex (root domain), and they resolve faster because they are integrated with the AWS backbone.

-   **ACM Certificate in `us-east-1` for CloudFront:** This is a common "gotcha". For a CloudFront distribution, the public certificate **must** be created in the N. Virginia (`us-east-1`) region, no matter where the origin or users are. This is a hard requirement.

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

-   **Governance is about Guardrails, not Gates:** Frame governance in a positive light. It's not about stopping developers from doing their work (a gate). It's about providing a safe, paved road for them to move quickly (a guardrail). Good governance enables speed by preventing costly mistakes.

-   **Understanding Compliance Requirements:**
    -   **Shared Responsibility Model:** Reiterate this concept. AWS is responsible for the compliance *of* the cloud (their physical data centers, hardware). The customer is responsible for compliance *in* the cloud (how they configure their VPCs, S3 buckets, etc. to meet a specific standard).
    -   **What are they?** Use an analogy. A compliance framework like PCI-DSS (for credit cards) or HIPAA (for healthcare) is like a building code. It's a detailed set of rules you must follow if you want to operate in that industry (e.g., "all data at rest must be encrypted," "all API calls must be logged"). AWS provides the tools (KMS, CloudTrail) to meet the requirements, but the customer must implement them correctly.

-   **CloudTrail Event Analysis:**
    -   **Who, What, When, Where:** Every CloudTrail event answers these four questions about an API call. Who made the call? What did they call? When did they call it? From what IP address?
    -   **CloudTrail Insights is for Anomaly Detection:** Standard CloudTrail tells you *what* happened. CloudTrail Insights tells you when something *unusual* happened. It automatically analyzes your management event history to establish a baseline of normal activity and then creates events when it detects deviations, such as a sudden spike in EC2 instance launches in a region you never use. It's a powerful, automated threat detection tool.
    -   **CloudTrail Lake for Deep Analysis:** This is a game-changer for auditing. Instead of being stored as JSON files in S3, events are stored in a queryable, SQL-based data store. This makes it much easier for security teams to run complex, multi-year analyses for forensic investigations or compliance reporting without building a complex data pipeline.

-   **AWS Config - The "Time Machine" for your Resources:**
    -   Config continuously records the configuration state of your AWS resources. It's like having a version history for every resource. You can see exactly what a security group's rules were last Tuesday and who changed them.
    -   **Rule Customization:** The real power of Config is in its rules engine. Managed rules cover common best practices, but Custom Rules (written as Lambda functions) allow you to enforce any organizational policy you can imagine (e.g., "all EC2 instances must have a 'cost-center' tag," "all EBS volumes must be of type `gp3`").

-   **Remediation Workflows:**
    -   **Detect and Remediate:** This is the core loop of automated governance.
        1.  **Detect:** An AWS Config rule detects a non-compliant resource (e.g., an S3 bucket is made public).
        2.  **Alert:** The rule change triggers an EventBridge event.
        3.  **Remediate:** The event invokes an SSM Automation document (or a Lambda function) that takes corrective action (e.g., calls the `PutPublicAccessBlock` API to make the bucket private again).
    -   This closes the loop, automatically fixing misconfigurations in near real-time, often before a human even notices the alert. This is a huge step towards a self-healing infrastructure.

-   **Security Hub and GuardDuty:**
    -   **GuardDuty:** This is your intelligent threat detection service. It analyzes VPC Flow Logs, DNS logs, and CloudTrail events to find evidence of malicious activity (e.g., cryptocurrency mining, communication with known command-and-control servers).
    -   **Security Hub:** This is the "single pane of glass" for security. It aggregates findings from GuardDuty, Config, Inspector, and other services into one place. It scores your environment against security standards like the CIS Benchmarks and provides a prioritized list of security issues to address.

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

-   **IaC is a Mindset Shift:** Just like source code, infrastructure is now defined in text files. This means it can be version controlled (Git), code reviewed (Pull Requests), and deployed through automated pipelines (CI/CD). This is a fundamental shift from clicking in the console. The console is for learning and exploring; production infrastructure should always be managed as code.

-   **Anatomy of a Template:** Walk through the main sections of a CloudFormation template.
    -   `Parameters`: The "inputs" to your template. They make your templates reusable.
    -   `Mappings`: A static "lookup table." Great for selecting an AMI ID based on the chosen region, for example.
    -   `Resources`: The "meat" of the template. This is where you declare the AWS resources you want to create.
    -   `Outputs`: The "return values" of your stack. They allow you to expose important information (like a load balancer's DNS name or a bucket's name) to other stacks or for easy lookup.

-   **Template Validation & Linting:**
    -   **`aws cloudformation validate-template`:** This command is your first line of defense. It checks for syntax errors in your YAML/JSON before you ever try to deploy.
    -   **`cfn-lint`:** This is a more advanced, open-source linter that goes beyond syntax. It checks for best practices, logical errors, and deprecated resource types. Integrating `cfn-lint` into a CI/CD pipeline is a professional best practice.

-   **Parameter Constraints:**
    -   These are your "guardrails" for template inputs. Using them prevents simple user error.
    -   Show examples: A `MinLength` on a password parameter, an `AllowedPattern` (RegEx) for a bucket name, or an `AllowedValues` list to restrict users to specific EC2 instance types. This makes your templates more robust and user-friendly.

-   **Stack Dependencies: `Ref` vs. `Fn::GetAtt`:**
    -   **Implicit Dependencies:** Most of the time, CloudFormation is smart enough to figure out the correct creation order. If you reference a security group's ID in the properties of an EC2 instance, CloudFormation knows it must create the security group first. This is an *implicit dependency*.
    -   **`Ref`:** This intrinsic function generally returns the *default identifier* of a resource. For an `AWS::EC2::Instance`, `Ref` returns the Instance ID. For an `AWS::S3::Bucket`, it returns the bucket name.
    -   **`Fn::GetAtt`:** This intrinsic function lets you access *other attributes* of a resource. For an `AWS::EC2::Instance`, you could use `Fn::GetAtt` to get its `PrivateIp` or `PublicDnsName`.
    -   **Explicit Dependencies (`DependsOn`):** Sometimes, there is no direct reference between two resources, but a dependency still exists (e.g., you need an S3 bucket policy to be in place before an application starts). The `DependsOn` attribute lets you explicitly tell CloudFormation to create one resource before another. Use it sparingly, as it can slow down deployments.

-   **Change Sets are a "Dry Run" for Your Deployment:**
    -   **Never update a production stack directly.** This is the most important lesson of this week.
    -   A Change Set is a preview of exactly what CloudFormation will do if you execute the update. It will show you which resources will be **Added**, which will be **Modified**, and, most importantly, which will be **Deleted** or **Replaced**.
    -   Reviewing a change set with a colleague is the CloudFormation equivalent of a code review. It's a critical safety step that can prevent catastrophic mistakes, like accidentally replacing a production database.

-   **Deletion Policies: Protecting Your State:**
    -   By default, when you delete a CloudFormation stack, it deletes all the resources it created. This is great for temporary environments but dangerous for production.
    -   The `DeletionPolicy: Retain` attribute tells CloudFormation to "orphan" the resource when the stack is deleted, leaving it behind in your account. This is essential for stateful resources like S3 buckets and RDS databases. You can also use `DeletionPolicy: Snapshot` on services that support it (like EBS and RDS) to get a final backup before the resource is deleted.

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

-   **Nested Stacks are about Modularity and Reusability:**
    -   **Analogy:** A large, monolithic CloudFormation template is like a single giant script with no functions. A nested stack architecture is like a well-structured program with modular functions.
    -   **Benefits:** You can create a "standard" VPC stack and reuse it for multiple different applications. Teams can own their own stacks (e.g., the networking team owns the VPC stack, the app team owns the application stack). It's easier to manage and understand smaller, focused templates.

-   **Stack Dependencies and Cross-Stack References:**
    -   **`DependsOn` for Orchestration:** A nested stack doesn't automatically know that it depends on another. In the Capstone project, the `ApplicationStack` `DependsOn` the `DatabaseStack`. This tells the parent stack to wait until the database stack is completely finished before it even starts creating the application stack. This is crucial for ordering.
    -   **Passing Outputs as Parameters:** This is the primary way to share information between stacks.
        1.  The `VPCStack` defines an `Output` with the ID of the public subnets.
        2.  The `ApplicationStack` defines a `Parameter` to accept the subnet IDs.
        3.  The root stack acts as the "glue," using `Fn::GetAtt` to get the output from the VPC stack and passing it as a parameter into the application stack.
    -   **`Fn::ImportValue` for Decoupling:** An alternative to passing parameters is to have one stack `Export` a value (like the VPC ID) and have another stack use `Fn::ImportValue` to reference it. This is simpler for a few values but can create tight coupling; you cannot update the stack that is exporting a value if it's being used by another stack. Parameter passing is generally more flexible.

-   **Deployment Strategies:**
    -   **Blue/Green Deployments with CloudFormation:** This is an advanced but powerful pattern. You use CloudFormation to stand up an entire new "green" environment (ALB, ASG, etc.) alongside the existing "blue" one. Once the green environment is tested and healthy, you use Route 53 to shift traffic from blue to green. The old blue environment can then be decommissioned. This provides zero-downtime deployments with an instant rollback capability (just point DNS back to blue).
    -   **CI/CD Pipeline:** The capstone is deployed manually, but in a real-world scenario, this would be automated. A `git push` would trigger a pipeline (like AWS CodePipeline) that would lint the template, run security checks, create a change set, wait for a manual approval step, and then execute the change set.

-   **Troubleshooting Techniques:**
    -   **Stack Events are Your Friend:** The "Events" tab for a stack is the first place to look when a deployment fails. It provides a chronological log of what CloudFormation tried to do and why it failed. Look for the `CREATE_FAILED` or `UPDATE_FAILED` status and read the `Status reason` column. It will often contain a specific error message from the underlying service (e.g., "The instance type 't3.superlarge' does not exist").
    -   **Rollback Failures:** Sometimes, even the rollback can fail, leaving the stack in an `UPDATE_ROLLBACK_FAILED` state. This usually happens if the original failure left a resource in a state that CloudFormation can't recover from. The only solution is to manually fix the underlying resource that is causing the problem and then tell CloudFormation to retry the rollback.
    -   **Drift Detection:** After a stack is created, a well-meaning colleague might manually change a security group in the console. **Drift Detection** allows you to see the difference between the "expected" state (defined in the template) and the "actual" state of the resources. Regularly running drift detection helps you find and fix unauthorized or accidental manual changes.

-   **The Capstone is the Culmination:** This final lab brings together every single concept from the entire course—networking, security, compute, storage, databases, and governance—and codifies it all into a single, automated, repeatable deployment. It's the ultimate demonstration of cloud proficiency. Congratulate the students on completing a project that mirrors a real-world, professional cloud deployment.

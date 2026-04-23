---
title: "The AWS Services Every DevOps Engineer Uses — A Practical Guide with Real-World Scenarios"
seoTitle: "AWS Basics: 15 Core Services Explained Simply"
seoDescription: "Learn 15 core AWS services every DevOps engineer uses with real-world use cases, simple explanations, and a clear cloud system mental model."
datePublished: 2026-04-23T16:11:45.697Z
cuid: cmoboivme003m1qiad3z43p63
slug: the-aws-services-every-devops-engineer-uses-a-practical-guide-with-real-world-scenarios
cover: https://cdn.hashnode.com/uploads/covers/69b5b16d1f4d4527ed9008ad/d057cbae-8ed8-400c-9b68-48b5120f76ea.png
tags: cloud, linux, aws, backend, cloud-computing, devops, beginners, software-engineering, system-design, ci-cd

---

> There are over 200 AWS services. But on any given day, a DevOps engineer is working with roughly 15 of them. This article breaks down every one of those core services — what they do, why they exist, and when you'd actually reach for them in a real production environment.

---

## Table of Contents

1. [EC2 — Your Virtual Server in the Cloud](#1-ec2--your-virtual-server-in-the-cloud)
2. [VPC — Your Private Network on AWS](#2-vpc--your-private-network-on-aws)
3. [EBS — Persistent Storage for Your Instances](#3-ebs--persistent-storage-for-your-instances)
4. [S3 — Object Storage for Everything](#4-s3--object-storage-for-everything)
5. [IAM — Who Can Do What on AWS](#5-iam--who-can-do-what-on-aws)
6. [CloudWatch — Observability and Monitoring](#6-cloudwatch--observability-and-monitoring)
7. [Lambda — Run Code Without Managing Servers](#7-lambda--run-code-without-managing-servers)
8. [AWS CI/CD Suite — CodePipeline, CodeBuild, CodeDeploy](#8-aws-cicd-suite--codepipeline-codebuild-codedeploy)
9. [AWS Config — Compliance and Configuration Tracking](#9-aws-config--compliance-and-configuration-tracking)
10. [Billing and Cost Management — Know What You're Spending](#10-billing-and-cost-management--know-what-youre-spending)
11. [AWS KMS — Encryption and Certificate Management](#11-aws-kms--encryption-and-certificate-management)
12. [CloudTrail — The Audit Log of Your AWS Account](#12-cloudtrail--the-audit-log-of-your-aws-account)
13. [EKS — Managed Kubernetes on AWS](#13-eks--managed-kubernetes-on-aws)
14. [ECS and Fargate — Container Orchestration the AWS Way](#14-ecs-and-fargate--container-orchestration-the-aws-way)
15. [ELK Stack (Elasticsearch) — Searching and Analyzing Logs](#15-elk-stack-elasticsearch--searching-and-analyzing-logs)
16. [Putting It All Together — How These Services Work as a System](#16-putting-it-all-together--how-these-services-work-as-a-system)
17. [Summary — Which Service for What](#17-summary--which-service-for-what)

---

## 1. EC2 — Your Virtual Server in the Cloud

### What it is

**EC2 (Elastic Compute Cloud)** is AWS's virtual machine service. When you need a server — to run an application, host a database, run a script, or process data — EC2 is where you go. You pick the operating system, the hardware specs (CPU, RAM), and you're billed per second of usage.

### Key concepts

- **Instance Types**: Define your compute power. `t3.micro` for small apps, `c6i.large` for CPU-heavy workloads, `r6g.xlarge` for memory-intensive processes.
- **AMI (Amazon Machine Image)**: The OS template your instance boots from. AWS provides official Ubuntu, Amazon Linux, and Windows AMIs. You can also create custom ones.
- **Key Pairs**: RSA keys for SSH access. AWS stores the public key; you keep the private `.pem` file.
- **Elastic IP**: A static public IP that stays the same across instance restarts.
- **Auto Scaling Groups**: Automatically add or remove EC2 instances based on traffic load — critical for production reliability.

### Real-world scenario

> Your team builds a Node.js REST API. You launch an EC2 instance (`t3.medium`, Ubuntu 22.04), SSH in, install Node.js, deploy the app, and put NGINX in front of it as a reverse proxy. Traffic spikes on weekends — so you configure an Auto Scaling Group to spin up extra instances automatically and terminate them when traffic drops.

### When to use EC2

- Running long-lived backend services or APIs
- Hosting databases (though RDS is usually better)
- Running batch processing jobs on a schedule
- Any workload that needs full OS-level control

---

## 2. VPC — Your Private Network on AWS

### What it is

**VPC (Virtual Private Cloud)** is your own isolated, private network within AWS. Every resource you create — EC2 instances, RDS databases, Lambda functions — lives inside a VPC. You control the networking: what IP ranges exist, which resources are public vs private, and what traffic is allowed in and out.

### Key concepts

- **CIDR Block**: The IP address range of your VPC. Example: `10.0.0.0/16` gives you 65,536 IP addresses to work with.
- **Subnets**: Subdivisions of your VPC's IP range.
  - **Public Subnet**: Resources here have a route to the internet (via an Internet Gateway). Your web servers live here.
  - **Private Subnet**: No direct internet access. Your databases, internal services, and backend logic live here — safely isolated from the public internet.
- **Internet Gateway (IGW)**: The door between your VPC and the internet. Attach one to your VPC to allow public traffic.
- **NAT Gateway**: Allows resources in a private subnet to *initiate* outbound internet connections (e.g., downloading packages) without being reachable from the internet.
- **Security Groups**: Virtual firewalls at the instance level. They are **stateful** — if you allow inbound traffic on port 80, the response is automatically allowed outbound.
- **Network ACLs (NACLs)**: Firewalls at the subnet level. They are **stateless** — you must explicitly allow both inbound and outbound traffic. Think of them as a second layer of defense beyond Security Groups.
- **Route Tables**: Define where network traffic is directed. Each subnet is associated with a route table that tells traffic how to reach its destination.

### Real-world scenario

> You're deploying a production web application. Your architecture uses a **public subnet** for the NGINX load balancer and web servers (accessible from the internet), and a **private subnet** for your PostgreSQL database (not accessible from the internet at all). The database can still download OS updates because of the **NAT Gateway**. Security Groups ensure only your web servers can talk to the database on port 5432 — nothing else can reach it.

### When to use VPC

- Always — every AWS resource lives in a VPC. The question is how you configure it.
- Separating environments: one VPC for production, one for staging, one for development.
- Multi-tier architectures: keeping databases and internal services in private subnets, web-facing resources in public subnets.

---

## 3. EBS — Persistent Storage for Your Instances

### What it is

**EBS (Elastic Block Store)** is a network-attached hard drive for your EC2 instance. When you launch an EC2 instance, it gets a root EBS volume (the OS disk). But you can attach additional EBS volumes for storing application data, logs, or databases.

The key word is **persistent** — unlike the instance's local storage (which is wiped when the instance stops), EBS volumes survive instance stops and reboots. You can also detach a volume from one instance and reattach it to another.

### Key concepts

- **Volume Types**:
  - `gp3` — General Purpose SSD. The default for most workloads. Fast, cost-effective.
  - `io2` — Provisioned IOPS SSD. For databases requiring consistently high throughput (e.g., MySQL, PostgreSQL in high-traffic production).
  - `st1` — Throughput Optimized HDD. For large sequential reads — data warehouses, log processing.
  - `sc1` — Cold HDD. For infrequent access. Cheapest option.
- **Snapshots**: Point-in-time backups of an EBS volume stored in S3. You can create a new volume from a snapshot — useful for disaster recovery or cloning environments.
- **Encryption**: EBS volumes can be encrypted using AWS KMS. Data at rest and in transit between the instance and the volume is encrypted.

### Real-world scenario

> You run a MongoDB database on an EC2 instance. The root volume holds the OS, but you attach a separate 500GB `gp3` EBS volume for MongoDB's data directory. Every night, an automated job creates an EBS snapshot. If the instance ever goes down or data gets corrupted, you restore the snapshot and attach the volume to a new instance — minimal data loss.

### When to use EBS vs S3

| Use EBS when... | Use S3 when... |
|---|---|
| You need block storage attached to a running EC2 instance | You need to store files, objects, backups, or static assets |
| Running a database on EC2 | Storing application logs long-term |
| Need low-latency disk I/O | Serving images or videos to users |
| OS disk for an instance | Archiving old data cheaply |

---

## 4. S3 — Object Storage for Everything

### What it is

**S3 (Simple Storage Service)** is AWS's object storage service — and one of its oldest and most used. Unlike EBS (which is a disk attached to an instance), S3 is a flat key-value store where you upload files (called **objects**) into containers (called **buckets**). S3 is globally durable, highly available, and infinitely scalable.

AWS guarantees **99.999999999% (11 nines) durability** — meaning if you store 10 million objects, you can expect to lose one object every 10,000 years.

### Key concepts

- **Buckets**: Containers for your objects. Bucket names must be globally unique across all AWS accounts.
- **Objects**: Files stored in S3. Each object has a key (its path/name), the data itself, and metadata.
- **Storage Classes**: Different tiers of storage for different access patterns:
  - `S3 Standard` — Frequent access. Default for most use cases.
  - `S3 Intelligent-Tiering` — Automatically moves data between access tiers based on usage.
  - `S3 Standard-IA` — Infrequent Access. Cheaper storage but higher retrieval cost.
  - `S3 Glacier` — Archival. Very cheap, but retrieval takes minutes to hours.
- **Bucket Policies and ACLs**: Control who can access your bucket and its contents.
- **Versioning**: Keep multiple versions of an object. Protects against accidental deletion or overwrite.
- **Lifecycle Rules**: Automatically transition objects to cheaper storage classes or delete them after a set time.
- **Static Website Hosting**: Host a static HTML/CSS/JS website directly from an S3 bucket.
- **Pre-signed URLs**: Generate a time-limited URL that gives temporary access to a private S3 object — great for secure file downloads.

### Real-world scenarios

> **Scenario 1 — CI/CD Artifacts**: Your CodeBuild pipeline compiles your application and uploads the build artifact (a `.zip` file) to an S3 bucket. CodeDeploy then pulls that artifact and deploys it to your EC2 fleet.

> **Scenario 2 — Static Frontend**: Your React app is built with `npm run build` and the output is uploaded to an S3 bucket with static website hosting enabled. CloudFront sits in front of it as a CDN — users globally get the files from the nearest edge location.

> **Scenario 3 — Backup Storage**: Every night, your database dumps are uploaded to S3 with a lifecycle rule that moves them to Glacier after 30 days and deletes them after 1 year.

---

## 5. IAM — Who Can Do What on AWS

### What it is

**IAM (Identity and Access Management)** is how you control access to AWS. It's the answer to the question: *"Who is allowed to do what, on which resources?"*

Every action taken in AWS — whether by a human, an application, or another AWS service — goes through IAM. It's one of the most critical services to understand correctly because misconfigured IAM is one of the top causes of cloud security incidents.

### Key concepts

- **Users**: Individual identities for humans (developers, admins). Each user has their own credentials (password + access keys).
- **Groups**: Collections of users. Attach a policy to a group and every user in it inherits those permissions.
- **Roles**: Identities for **services and applications** — not humans. When an EC2 instance needs to access S3, you attach an IAM Role to it rather than embedding access keys in your code.
- **Policies**: JSON documents defining what actions are allowed or denied on which resources.
- **Principle of Least Privilege**: Every user, role, or service should have only the minimum permissions it needs to do its job — nothing more.

### Example IAM Policy

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "s3:GetObject",
        "s3:PutObject"
      ],
      "Resource": "arn:aws:s3:::my-app-bucket/*"
    }
  ]
}
```

This policy allows reading and writing objects in a specific S3 bucket — and nothing else.

### Real-world scenario

> Your Node.js app running on EC2 needs to upload user profile pictures to S3. Instead of hardcoding AWS access keys in your `.env` file (a serious security risk), you create an **IAM Role** with an S3 write policy and attach it to the EC2 instance. Your application code uses the AWS SDK, which automatically picks up the role's temporary credentials from the instance metadata. No keys stored anywhere.

### When to use what

| Need | IAM Concept to Use |
|---|---|
| A developer needs AWS Console access | IAM User |
| Multiple devs need the same permissions | IAM Group |
| An EC2 instance needs to call another AWS service | IAM Role |
| Define what actions are allowed | IAM Policy |
| A Lambda function needs to write to DynamoDB | IAM Role attached to Lambda |

---

## 6. CloudWatch — Observability and Monitoring

### What it is

**CloudWatch** is AWS's native monitoring and observability service. If something is happening in your AWS infrastructure — CPU spikes, memory pressure, error rates, application logs — CloudWatch is where you see it, alert on it, and respond to it.

### Key concepts

- **Metrics**: Numerical data points over time. AWS services automatically publish metrics to CloudWatch — EC2 CPU usage, RDS connections, Lambda invocation counts, S3 request counts, etc.
- **Logs**: CloudWatch Logs stores log output from your applications, Lambda functions, ECS containers, and more. You can search, filter, and query logs using **CloudWatch Logs Insights**.
- **Alarms**: Trigger notifications or automated actions when a metric crosses a threshold. Example: alert when CPU > 80% for 5 minutes.
- **Dashboards**: Custom visual dashboards combining multiple metrics into one view. Ideal for an at-a-glance status of your entire system.
- **CloudWatch Events / EventBridge**: React to events in your AWS environment — e.g., trigger a Lambda function whenever a new file is uploaded to S3.
- **Container Insights**: Enhanced monitoring for ECS and EKS — tracks container-level CPU, memory, network, and disk usage.

### Real-world scenario

> Your production API suddenly starts responding slowly. CloudWatch shows that EC2 CPU is at 95% and memory is spiking. A CloudWatch Alarm you set up fires and sends an alert to your team's Slack channel via SNS. The Auto Scaling Group kicks in and spins up two more instances. You open CloudWatch Logs Insights, query the last 15 minutes of logs, and find a specific API endpoint causing a database N+1 query problem. Fix deployed within the hour.

### When to use CloudWatch

- Monitor EC2, RDS, Lambda, ECS — any AWS service
- Stream application logs from anywhere in your infrastructure
- Set up alarms for critical thresholds (CPU, memory, error rate, latency)
- Build operations dashboards for your team
- Trigger automated responses to events

---

## 7. Lambda — Run Code Without Managing Servers

### What it is

**AWS Lambda** is a **serverless compute service** — you upload a function, define what triggers it, and AWS handles everything else: provisioning servers, scaling, patching, availability. You pay only for the milliseconds your code actually runs.

Lambda changed how developers think about backend architecture. Instead of running a server 24/7 to handle occasional tasks, you write a function that runs only when needed.

### Key concepts

- **Triggers**: What causes your Lambda to run. Common triggers include:
  - API Gateway (HTTP request)
  - S3 event (file uploaded)
  - CloudWatch Events (scheduled cron)
  - SQS (message in a queue)
  - DynamoDB Streams (data change)
- **Execution Environment**: Lambda supports Node.js, Python, Java, Go, Ruby, and more. You can also bring a custom runtime.
- **Timeout**: Maximum execution time is 15 minutes. Lambda is not designed for long-running processes.
- **Cold Starts**: When a Lambda hasn't been invoked recently, the first request experiences a slight delay while AWS initializes the execution environment. Subsequent requests are fast.
- **Concurrency**: Lambda scales automatically — if 1,000 requests arrive simultaneously, AWS runs 1,000 instances of your function in parallel.

### Real-world scenarios

> **Scenario 1 — Image Processing**: A user uploads a profile picture to S3. An S3 event triggers a Lambda function that resizes the image to multiple dimensions (thumbnail, medium, full) and stores them back in S3. No server needed.

> **Scenario 2 — Scheduled Job**: Every night at midnight, a CloudWatch Events rule triggers a Lambda that queries the database for inactive users and sends them a re-engagement email via SES.

> **Scenario 3 — Webhook Handler**: A payment provider sends a webhook to your API Gateway endpoint. Lambda receives it, validates the signature, updates the order status in DynamoDB, and returns a 200 response — all in under 100ms.

### Lambda vs EC2 — When to use what

| Use Lambda when... | Use EC2 when... |
|---|---|
| Short, event-driven tasks | Long-running processes |
| Unpredictable or spiky traffic | Consistent, steady traffic |
| You want zero server management | You need full OS-level control |
| Pay-per-execution model fits | Predictable compute costs |
| Tasks under 15 minutes | Tasks exceeding 15 minutes |

---

## 8. AWS CI/CD Suite — CodePipeline, CodeBuild, CodeDeploy

### What it is

AWS offers a fully managed CI/CD suite that covers the entire journey from a code commit to a live deployment — without relying on third-party tools.

The three services work together in a pipeline:

```
Code Commit → CodePipeline → CodeBuild → CodeDeploy → Production
```

### CodePipeline — The Orchestrator

**CodePipeline** is the glue. It defines the stages of your delivery pipeline and orchestrates the flow — source → build → test → deploy. Think of it as the conductor of the CI/CD orchestra.

- Monitors your source (GitHub, CodeCommit, S3) for changes
- Triggers the next stage automatically when the previous one succeeds
- Supports manual approval stages (useful before production deployments)
- Integrates with CodeBuild, CodeDeploy, Lambda, and third-party tools

### CodeBuild — The Builder

**CodeBuild** is the CI engine. When CodePipeline triggers it, CodeBuild spins up a temporary build environment, runs your build commands (install → test → compile → package), and produces a build artifact.

- Defined by a `buildspec.yml` file in your repository
- Supports Node.js, Python, Java, Go, Docker, and more
- Scales automatically — no build servers to manage
- Build logs stream to CloudWatch Logs in real-time

**Example `buildspec.yml` for a Node.js app:**

```yaml
version: 0.2

phases:
  install:
    runtime-versions:
      nodejs: 20
    commands:
      - npm install

  pre_build:
    commands:
      - npm test

  build:
    commands:
      - npm run build
      - zip -r app.zip . -x "node_modules/*"

artifacts:
  files:
    - app.zip
```

### CodeDeploy — The Deployer

**CodeDeploy** takes the build artifact and deploys it to your target infrastructure — EC2 instances, Lambda functions, or ECS services.

- Supports rolling deployments, blue/green deployments, and all-at-once deployments
- Defined by an `appspec.yml` file that specifies deployment hooks (before install, after install, start server, etc.)
- Automatically rolls back if a deployment fails health checks

### AWS CI/CD vs Jenkins vs GitHub Actions

| | AWS CodePipeline + CodeBuild + CodeDeploy | Jenkins | GitHub Actions |
|---|---|---|---|
| **Hosting** | Fully managed by AWS | Self-hosted (you manage the server) | Managed by GitHub |
| **Integration** | Native AWS service integration | Plugin-based, very flexible | Tight GitHub integration |
| **Scaling** | Automatic | Manual (add Jenkins agents) | Automatic |
| **Cost** | Pay per pipeline run | EC2/server cost + maintenance | Free tier + pay per minute |
| **Setup** | Moderate | Complex | Simple |
| **Best for** | Teams already deep in AWS | Complex, custom enterprise pipelines | GitHub-based open source or small teams |

### Real-world scenario

> Every time a developer pushes to the `main` branch on GitHub, CodePipeline detects the change. It triggers CodeBuild, which runs tests and packages the app into a `.zip` artifact uploaded to S3. CodeDeploy then pulls that artifact and performs a **blue/green deployment** on the EC2 fleet — spinning up new instances with the new code, routing traffic to them, and terminating the old ones only after health checks pass. Zero downtime deployment, fully automated.

---

## 9. AWS Config — Compliance and Configuration Tracking

### What it is

**AWS Config** is a service that continuously records the configuration state of your AWS resources and evaluates them against desired rules. It answers the question: *"What does my infrastructure look like right now, and has it always been compliant?"*

### Key concepts

- **Configuration Recorder**: Tracks changes to resource configurations over time. If someone changes a Security Group rule, Config records what it was before and after.
- **Config Rules**: Define the desired state of your resources. AWS provides managed rules (e.g., "all S3 buckets must have encryption enabled") or you can write custom rules using Lambda.
- **Compliance Dashboard**: A view of which resources are compliant or non-compliant with your rules.
- **Configuration History**: A complete timeline of configuration changes for any resource — useful for audits and debugging.

### Real-world scenario

> Your organization has a rule: *"No EC2 instance should have port 22 (SSH) open to 0.0.0.0/0."* You create an AWS Config rule enforcing this. When a developer accidentally opens SSH to the world in a Security Group, Config immediately flags it as non-compliant and sends an alert via SNS. You can even trigger an automated Lambda remediation that revokes the offending rule automatically.

### When to use AWS Config

- Compliance and governance — prove your infrastructure meets security standards
- Audit trails for security investigations
- Drift detection — know when manual changes deviate from your Infrastructure as Code
- Regulated industries (finance, healthcare) where configuration evidence is required

---

## 10. Billing and Cost Management — Know What You're Spending

### What it is

AWS provides a suite of tools to understand, monitor, and optimize your cloud spending. Running unexpected bills is one of the most common painful experiences for new AWS users — these tools exist to prevent that.

### Key services

- **AWS Cost Explorer**: Visualize your spending over time, broken down by service, region, or tag. See trends and identify cost spikes.
- **AWS Budgets**: Set spending thresholds and get alerts when you approach or exceed them. Example: alert when monthly spend exceeds $50.
- **AWS Cost and Usage Report (CUR)**: Detailed, raw billing data exportable to S3 — used for deep cost analysis and integration with BI tools.
- **Savings Plans and Reserved Instances**: Commit to a usage level for 1–3 years in exchange for significant discounts (up to 72% off On-Demand pricing).
- **AWS Trusted Advisor**: Analyzes your account and gives recommendations for cost optimization, security, performance, and fault tolerance.
- **Resource Tagging**: Tag every resource (EC2, S3, RDS) with metadata like `Project`, `Environment`, `Team`. Then filter costs by tag to see exactly what each team or project is spending.

### Real-world scenario

> You're a DevOps engineer at a startup. You set up an **AWS Budget** alerting at 80% of the monthly limit. Mid-month, the alert fires. You open Cost Explorer and see that a developer left a large EC2 instance (`m5.4xlarge`) running all week for a test that finished days ago. You terminate it, saving hundreds of dollars. You then implement a Lambda function triggered by CloudWatch Events that automatically stops non-production instances every night at 10 PM.

---

## 11. AWS KMS — Encryption and Certificate Management

### What it is

**AWS KMS (Key Management Service)** is a managed service for creating and controlling the encryption keys used to protect your data. If you encrypt an EBS volume, an S3 bucket, an RDS database, or a secret in Secrets Manager — KMS is the engine doing the key management underneath.

### Key concepts

- **Customer Master Keys (CMKs)**: The encryption keys you create and manage in KMS. AWS never gives you the raw key material — all encrypt/decrypt operations happen inside KMS's hardware security modules (HSMs).
- **AWS Managed Keys**: Keys that AWS creates and manages on your behalf for services like S3, EBS, and RDS. You don't control rotation, but AWS handles it automatically.
- **Key Rotation**: KMS can automatically rotate your keys annually. Old key versions are retained to decrypt data encrypted with them.
- **Key Policies**: IAM-style policies that control who can use or manage a KMS key.
- **Envelope Encryption**: KMS doesn't directly encrypt your data — it encrypts a **data key**, which then encrypts your data. This is efficient for large datasets.

### ACM — AWS Certificate Manager

While KMS handles encryption keys, **ACM (AWS Certificate Manager)** handles **SSL/TLS certificates** for your web applications — the certificates that enable HTTPS.

- Provision free SSL certificates for your domains
- Automatically renews certificates before expiry
- Integrates directly with ALB (Application Load Balancer), CloudFront, and API Gateway
- No server-side certificate installation needed

### Real-world scenario

> Your application stores user health records. Compliance requires all data at rest to be encrypted. You create a KMS CMK, use it to encrypt your RDS database and the S3 bucket storing documents. When a developer needs to query the database, their IAM role grants them `kms:Decrypt` permission — but only for that specific key. If they're removed from the IAM group, they immediately lose decryption access even if they still have database credentials.

---

## 12. CloudTrail — The Audit Log of Your AWS Account

### What it is

**AWS CloudTrail** records every API call made in your AWS account — who did what, when, from where, and on which resource. Every AWS Console click, every CLI command, every SDK call — CloudTrail captures it.

It's the difference between waking up to a breached account and saying *"we have no idea what happened"* versus having a complete forensic record of every action taken in the 24 hours before the breach.

### Key concepts

- **Events**: Each recorded API call. Contains: who made the call (user/role/service), what action was taken, which resource was affected, the timestamp, and the source IP address.
- **Trails**: Configuration that delivers CloudTrail events to an S3 bucket for long-term storage and analysis.
- **Management Events**: Control plane actions — creating/deleting EC2 instances, modifying Security Groups, changing IAM policies. Enabled by default.
- **Data Events**: Data plane actions — S3 object-level operations (GetObject, PutObject), Lambda invocations. Must be explicitly enabled (higher cost).
- **CloudTrail Insights**: Detects unusual API activity patterns that may indicate a security incident.
- **Integration with CloudWatch Logs**: Stream CloudTrail events to CloudWatch for real-time alerting on specific actions (e.g., alert whenever someone calls `DeleteSecurityGroup`).

### Real-world scenario

> A critical S3 bucket containing customer data is found to have its encryption suddenly disabled. Using CloudTrail, you query the events for that bucket and find: at 2:14 AM, an IAM user named `ci-deploy-user` called `PutBucketEncryption` with encryption set to none — from an IP address in a foreign country. The user's access keys had been leaked in a public GitHub commit three weeks prior. CloudTrail gives you the exact evidence needed to contain the breach, rotate credentials, and file an incident report.

### CloudTrail vs CloudWatch — What's the difference?

| | CloudTrail | CloudWatch |
|---|---|---|
| **Tracks** | WHO did WHAT (API calls) | WHAT is happening (metrics, logs) |
| **Use case** | Security auditing, compliance | Performance monitoring, alerting |
| **Data type** | API event records | Metrics, logs, traces |
| **Question it answers** | "Who deleted that security group?" | "Why is my CPU at 100%?" |

---

## 13. EKS — Managed Kubernetes on AWS

### What it is

**EKS (Elastic Kubernetes Service)** is AWS's managed Kubernetes service. Kubernetes (K8s) is the industry-standard system for orchestrating containerized applications — automating deployment, scaling, and management of containers across a cluster of machines.

EKS removes the hardest part of Kubernetes: setting up and maintaining the **control plane** (the brain of the Kubernetes cluster). AWS manages it for you — high availability, automatic upgrades, integrated with IAM, VPC, and CloudWatch.

### Key concepts

- **Control Plane**: The Kubernetes master — manages the cluster state, schedules workloads, handles API requests. AWS manages this in EKS.
- **Worker Nodes**: The EC2 instances (or Fargate pods) that actually run your containers. You manage these (or let AWS manage them with managed node groups).
- **Pods**: The smallest deployable unit in Kubernetes — one or more containers running together.
- **Deployments**: Define how many replicas of a pod to run and how to update them.
- **Services**: Expose your pods to network traffic — internal (ClusterIP), external (LoadBalancer), or via a domain name (Ingress).
- **Helm**: The package manager for Kubernetes — install and manage complex applications on K8s with a single command.

### Real-world scenario

> A company runs a microservices architecture — 12 separate services (auth, users, payments, notifications, etc.). Running and scaling 12 separate EC2 instances manually is a nightmare. They migrate to EKS. Now, each service is a Docker container deployed as a Kubernetes Deployment. EKS automatically distributes the containers across worker nodes, restarts failed pods, scales specific services up when traffic increases, and performs rolling updates with zero downtime. The team manages the entire cluster with `kubectl` commands and Helm charts.

### When to use EKS

- You're running microservices at scale
- Your team already knows Kubernetes or is willing to learn it
- You need fine-grained control over container orchestration
- You want portability — Kubernetes runs on any cloud, so EKS workloads can be migrated

---

## 14. ECS and Fargate — Container Orchestration the AWS Way

### What it is

**ECS (Elastic Container Service)** is AWS's own proprietary container orchestration service — like Kubernetes, but simpler and deeply AWS-native. If EKS is Kubernetes on AWS, ECS is AWS's own answer to container orchestration.

**Fargate** is the serverless compute engine for ECS (and EKS). Instead of managing EC2 worker nodes, with Fargate you define your container's CPU and memory requirements and AWS runs it — no servers to provision, patch, or scale.

### ECS vs EKS — Which to choose?

| | ECS | EKS |
|---|---|---|
| **Complexity** | Simpler, AWS-native concepts | Full Kubernetes — steeper learning curve |
| **Portability** | AWS-only | Kubernetes runs anywhere |
| **Control** | Less control, more managed | More control, more responsibility |
| **Ecosystem** | AWS tools only | Huge Kubernetes ecosystem |
| **Best for** | Teams new to containers, AWS-first orgs | Teams familiar with K8s, multi-cloud needs |

### ECS + Fargate scenario

> Your team builds a Docker image of a Node.js app and pushes it to ECR (Elastic Container Registry). You define an ECS Task Definition (CPU: 512, Memory: 1024, container image: your ECR image). You create an ECS Service with Fargate as the launch type — specifying 3 desired tasks. AWS spins up 3 instances of your container with no EC2 management on your part. An ALB routes traffic across them. When you push a new image, ECS performs a rolling update — replacing containers one at a time with the new version.

---

## 15. ELK Stack (Elasticsearch) — Searching and Analyzing Logs

### What it is

**ELK** stands for **Elasticsearch, Logstash, and Kibana** — a powerful open-source stack for ingesting, storing, searching, and visualizing log data at scale. AWS offers a managed version called **Amazon OpenSearch Service** (formerly Amazon Elasticsearch Service).

- **Elasticsearch / OpenSearch**: The search and analytics engine. Stores logs as JSON documents and makes them instantly searchable.
- **Logstash**: The data ingestion pipeline — collects logs from various sources, transforms them, and sends them to Elasticsearch.
- **Kibana / OpenSearch Dashboards**: The visualization layer — build dashboards, run queries, create visual analytics on top of your log data.

### Why logs need a dedicated system

CloudWatch Logs works for basic log viewing. But when you're running dozens of services generating millions of log lines per hour, you need something purpose-built for:

- **Full-text search across all logs** — find every occurrence of a specific error message across all services instantly
- **Aggregations** — count errors per service, per hour, per endpoint
- **Visual dashboards** — graphs of error rates, latency distributions, traffic patterns
- **Correlation** — trace a single user request across multiple microservices

### Real-world scenario

> An e-commerce platform runs 15 microservices. When a user reports a failed checkout, the engineer needs to trace the request across the payment service, inventory service, and notification service — all of which generate separate logs. In Kibana, they filter by the user's `requestId`, and instantly see the entire journey: the payment service succeeded, the inventory service threw a `StockException`, which caused the notification service to receive a null order and crash. Root cause identified in 30 seconds, not 30 minutes.

### When to use ELK vs CloudWatch

| | CloudWatch Logs | ELK / OpenSearch |
|---|---|---|
| **Best for** | Basic AWS service log monitoring | Complex multi-service log analysis |
| **Search** | Basic filter patterns | Powerful full-text search |
| **Visualization** | Basic dashboards | Rich Kibana dashboards |
| **Cost** | Pay per GB ingested | Higher setup cost, more powerful |
| **Setup** | Zero — built into AWS | Requires setup and maintenance |
| **Use when** | Small apps, single-service logs | Microservices, high-volume logs |

---

## 16. Putting It All Together — How These Services Work as a System

Here's what a complete, production-grade AWS architecture looks like using these services together:

```
Developer pushes code to GitHub
        ↓
CodePipeline detects the change
        ↓
CodeBuild runs tests + builds Docker image → pushes to ECR
        ↓
CodeDeploy / ECS performs rolling deployment
        ↓
                    ┌─────────────────────────────────────────────┐
                    │              VPC                             │
                    │  ┌──────────────┐    ┌──────────────────┐  │
Internet ──────────────→ ALB (Public  │    │ ECS/EKS Cluster  │  │
                    │  │ Subnet)      │───→│ (Private Subnet) │  │
                    │  └──────────────┘    └──────────┬───────┘  │
                    │                                 │           │
                    │                      ┌──────────▼───────┐  │
                    │                      │ RDS Database     │  │
                    │                      │ (Private Subnet) │  │
                    │                      └──────────────────┘  │
                    └─────────────────────────────────────────────┘
                    
Monitoring layer:
- CloudWatch → metrics, alarms, dashboards
- CloudTrail → who did what
- ELK/OpenSearch → log aggregation and search
- AWS Config → compliance and drift detection

Security layer:
- IAM → access control
- KMS → encryption at rest
- Security Groups + NACLs → network firewall
- ACM → SSL/TLS certificates

Cost layer:
- AWS Budgets → spending alerts
- Cost Explorer → spending visibility
- Resource Tags → cost attribution
```

---

## 17. Summary — Which Service for What

| Service | One-Line Purpose | Reach for it when... |
|---|---|---|
| **EC2** | Virtual machines | You need a server you control |
| **VPC** | Private network | Always — every resource lives in one |
| **EBS** | Disk storage for EC2 | Your instance needs persistent storage |
| **S3** | Object/file storage | Storing files, backups, static assets |
| **IAM** | Access control | Controlling who/what can access AWS |
| **CloudWatch** | Monitoring & alerting | Tracking metrics, logs, setting alarms |
| **Lambda** | Serverless functions | Short, event-driven tasks |
| **CodePipeline** | CI/CD orchestration | Automating your delivery pipeline |
| **CodeBuild** | Build and test automation | Compiling, testing, packaging code |
| **CodeDeploy** | Automated deployments | Deploying to EC2, Lambda, or ECS |
| **AWS Config** | Compliance tracking | Audit trails, governance, drift detection |
| **Billing Tools** | Cost visibility | Monitoring and controlling AWS spend |
| **KMS** | Encryption key management | Encrypting data at rest |
| **ACM** | SSL/TLS certificates | Enabling HTTPS on your apps |
| **CloudTrail** | API audit logging | Security investigations, compliance |
| **EKS** | Managed Kubernetes | Container orchestration at scale |
| **ECS** | AWS-native containers | Simpler container orchestration |
| **Fargate** | Serverless containers | Running containers without managing EC2 |
| **ELK/OpenSearch** | Log search & analytics | Searching logs across many services |

---

## Conclusion

These 15 services aren't just a list to memorize — they're a system. Each one solves a specific problem in the lifecycle of building, deploying, securing, and operating software in the cloud.

The mental model to carry forward is this:

- **Compute**: EC2, Lambda, ECS, EKS, Fargate
- **Storage**: S3, EBS
- **Networking**: VPC, Security Groups
- **CI/CD**: CodePipeline, CodeBuild, CodeDeploy
- **Security**: IAM, KMS, ACM, CloudTrail, AWS Config
- **Observability**: CloudWatch, ELK/OpenSearch
- **Cost**: Budgets, Cost Explorer

You don't need to master all of them on day one. But understanding what each one does — and more importantly, *why it exists* — is what separates someone who follows tutorials from someone who can architect real systems.

This is just the beginning. Each of these services has enough depth to fill its own article. Keep building, keep deploying, and keep asking *why* — not just *how*.

---

*Found this helpful? This is part of my ongoing series documenting my DevOps and cloud learning journey. Follow along at [From Code to Cloud](https://ashmitcodes.hashnode.dev) .* 🚀
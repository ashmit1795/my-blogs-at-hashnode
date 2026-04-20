---
title: "From Zero to Production: Deploying Node.js on AWS EC2 with PM2 and NGINX"
seoTitle: "How to Run Node.js on AWS EC2 with PM2 & NGINX"
seoDescription: "Learn how to deploy a Node.js app on AWS EC2 from scratch — covering CLI setup, PM2 process management, NGINX reverse proxy, and Security Groups."
datePublished: 2026-04-20T08:35:01.551Z
cuid: cmo6xvyg6004i2dof6qb90f1r
slug: from-zero-to-production-deploying-node-js-on-aws-ec2-with-pm2-and-nginx
cover: https://cdn.hashnode.com/uploads/covers/69b5b16d1f4d4527ed9008ad/aee9b8ba-1844-4c8a-94fb-4e8bc3bba409.jpg
ogImage: https://cdn.hashnode.com/uploads/og-images/69b5b16d1f4d4527ed9008ad/123aa805-2049-4fa3-92ff-a13be518f6a7.png
tags: aws, nginx, nodejs, cloud-computing, devops

---

> A step-by-step guide covering everything from setting up your AWS account to running your Node.js app behind NGINX with PM2 process management.

---

## Table of Contents

1. [Setting Up Your AWS Account](#1-setting-up-your-aws-account)
2. [Installing AWS CLI](#2-installing-aws-cli)
3. [Configuring the AWS CLI](#3-configuring-the-aws-cli)
4. [What is an EC2 Instance?](#4-what-is-an-ec2-instance)
5. [Creating an EC2 Instance Using the AWS Console (GUI)](#5-creating-an-ec2-instance-using-the-aws-console-gui)
6. [Creating an EC2 Instance Using the AWS CLI](#6-creating-an-ec2-instance-using-the-aws-cli)
7. [SSH Into Your EC2 Instance](#7-ssh-into-your-ec2-instance)
8. [Updating the Instance and Installing Node.js & Git](#8-updating-the-instance-and-installing-nodejs--git)
9. [Cloning Your Node.js App from GitHub](#9-cloning-your-nodejs-app-from-github)
10. [Setting Up the App](#10-setting-up-the-app)
11. [Starting the App and Configuring Security Group Ports](#11-starting-the-app-and-configuring-security-group-ports)
12. [Accessing Your API via Public IP and Port](#12-accessing-your-api-via-public-ip-and-port)
13. [Introducing PM2 — Process Management Done Right](#13-introducing-pm2--process-management-done-right)
14. [NGINX and Reverse Proxy — The Gateway to Your App](#14-nginx-and-reverse-proxy--the-gateway-to-your-app)
15. [Setting Up NGINX Step by Step](#15-setting-up-nginx-step-by-step)
16. [Summary and Conclusion](#16-summary-and-conclusion)

---

## 1. Setting Up Your AWS Account

Before anything else, you need an AWS account. AWS (Amazon Web Services) is the world's most widely used cloud platform, offering hundreds of services — from virtual machines to databases to AI tools.

### Steps to Create an AWS Account

1. Go to [https://aws.amazon.com](https://aws.amazon.com) and click **"Create an AWS Account"**.
2. Enter your **email address** and choose an **account name**.
3. Set a **root user password**. This is the most privileged account — treat it like the keys to your entire cloud kingdom.
4. Enter your **contact information** (personal or business).
5. Add a **payment method** (credit/debit card). AWS has a generous Free Tier, so you won't be charged for basic usage within limits.
6. Complete **phone verification**.
7. Choose a **support plan** — select "Basic Support (Free)" to start.

### Important: Secure Your Root User

The **root user** has unrestricted access to every AWS service and resource. Best practices:

- Enable **Multi-Factor Authentication (MFA)** on the root account immediately. Go to: IAM → Dashboard → Add MFA.
- Do **not** use the root user for everyday tasks. Instead, create an **IAM user** with appropriate permissions for day-to-day work.
- Never share or commit your root credentials anywhere.

> 💡 **Think of the root user like a superuser (`sudo`) on steroids — it can delete your entire account, so guard it carefully.**

---

## 2. Installing AWS CLI

The **AWS CLI (Command Line Interface)** lets you interact with AWS services directly from your terminal. Instead of clicking through the console UI, you can run commands to create instances, manage storage, configure security, and much more — all from your shell.

### Why Use the CLI?

- Faster than the GUI for repetitive tasks
- Scriptable and automatable
- Essential for DevOps workflows and CI/CD pipelines

### Installation

#### macOS

```bash
curl "https://awscli.amazonaws.com/AWSCLIV2.pkg" -o "AWSCLIV2.pkg"
sudo installer -pkg AWSCLIV2.pkg -target /
```

#### Linux (Ubuntu/Debian)

```bash
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install
```

#### Windows

Download and run the MSI installer from:
`https://awscli.amazonaws.com/AWSCLIV2.msi`

### Verify Installation

```bash
aws --version
# Output: aws-cli/2.x.x Python/3.x.x ...
```

---

## 3. Configuring the AWS CLI

After installation, you need to connect the CLI to your AWS account by providing credentials.

### Step 1: Create an IAM Access Key

1. Go to the AWS Console → **IAM** → **Users**.
2. Create a new user (or select your existing IAM user).
3. Attach the policy **"AmazonEC2FullAccess"** (or "AdministratorAccess" for full access).
4. Go to **Security Credentials** → **Create Access Key**.
5. Download or copy the **Access Key ID** and **Secret Access Key**. You won't be able to see the secret key again after this screen.

### Step 2: Run AWS Configure

```bash
aws configure
```

You'll be prompted for:

```
AWS Access Key ID [None]: AKIAIOSFODNN7EXAMPLE
AWS Secret Access Key [None]: wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY
Default region name [None]: ap-south-1
Default output format [None]: json
```

- **Region**: Choose the region closest to your users (e.g., `ap-south-1` for Mumbai, `us-east-1` for N. Virginia).
- **Output format**: `json` is the most common and machine-readable.

### Verify Configuration

```bash
aws sts get-caller-identity
```

Output:
```json
{
    "UserId": "AIDAIOSFODNN7EXAMPLE",
    "Account": "123456789012",
    "Arn": "arn:aws:iam::123456789012:user/your-username"
}
```

If you see this, your CLI is properly authenticated.

---

## 4. What is an EC2 Instance?

**EC2 (Elastic Compute Cloud)** is AWS's virtual machine service. An EC2 instance is essentially a computer running in AWS's data center that you can rent by the hour or second.

### Key Concepts

| Term | Meaning |
|---|---|
| **Instance** | A virtual machine running in the cloud |
| **Instance Type** | The hardware spec (CPU, RAM) — e.g., `t2.micro`, `t3.medium` |
| **AMI** | Amazon Machine Image — the OS template used to launch the instance |
| **Region** | Geographic location of the data center |
| **Availability Zone** | Isolated data centers within a region |
| **Key Pair** | SSH keys for secure login to your instance |
| **Security Group** | A virtual firewall controlling inbound/outbound traffic |
| **Elastic IP** | A static public IP that doesn't change on restart |

### Instance Types — Choosing the Right Size

AWS instance types follow a naming convention: `[family][generation].[size]`

Examples:
- `t2.micro` — 1 vCPU, 1 GB RAM (Free Tier eligible)
- `t3.small` — 2 vCPU, 2 GB RAM
- `t3.medium` — 2 vCPU, 4 GB RAM
- `m5.large` — 2 vCPU, 8 GB RAM (for production workloads)

For learning and small projects, `t2.micro` or `t3.micro` is perfectly fine and free under the AWS Free Tier.

### How Billing Works

EC2 instances are billed per second (with a minimum of 60 seconds). When an instance is **stopped**, you are not billed for compute — but you are still billed for the attached **EBS storage volume**. When **terminated**, everything is deleted.

---

## 5. Creating an EC2 Instance Using the AWS Console (GUI)

The AWS Console is the web-based dashboard. This is the easiest way to get started.

### Steps

1. Log in to [https://console.aws.amazon.com](https://console.aws.amazon.com).
2. Search for **EC2** in the search bar and click it.

3. Click **"Launch Instance"**.

### Configure the Instance

**Name:** Give your instance a meaningful name, like `my-node-app-server`.

**Amazon Machine Image (AMI):** Choose **Ubuntu Server 22.04 LTS (HVM), SSD Volume Type**. This is free-tier eligible and a great OS for running Node.js apps.

**Instance Type:** Select `t2.micro` (Free Tier eligible).

**Key Pair (Login):**
- Click **"Create new key pair"**.
- Name it something like `my-ec2-key`.
- Choose **RSA** and **.pem** format.
- Click **"Create key pair"** — the `.pem` file downloads automatically. **Keep this file safe. You cannot re-download it.**

**Network Settings:**
- Leave the default VPC and subnet.
- Check **"Allow SSH traffic from"** → select **My IP** (recommended) or Anywhere (for learning).
- Check **"Allow HTTP traffic from the internet"**.
- Check **"Allow HTTPS traffic from the internet"**.

**Storage:** Leave the default 8 GB gp2 volume (free tier eligible).

**Launch:**
- Click **"Launch Instance"**.
- Wait 1–2 minutes for the instance to reach the **"Running"** state.

---

## 6. Creating an EC2 Instance Using the AWS CLI

This is the power-user approach. The CLI gives you fine-grained control and is scriptable.

### Concepts You Need First

#### What is an AMI (Amazon Machine Image)?

An AMI is a **pre-configured template** that contains the operating system (and optionally software) needed to launch an EC2 instance. Think of it as a **disk snapshot** that gets loaded onto your instance.

- Every instance is launched from an AMI.
- AMIs are region-specific — an AMI ID in `us-east-1` won't work in `ap-south-1`.
- AWS provides official AMIs for Ubuntu, Amazon Linux, Windows Server, etc.
- You can create your own custom AMIs from a running instance (useful for **golden images** in production).

**Finding the Ubuntu 22.04 AMI ID for your region** (I used `ami-05d2d839d4f73aafb` for `ap-south-1`):

```bash
aws ec2 describe-images \
  --owners 099720109477 \
  --filters "Name=name,Values=ubuntu/images/hvm-ssd/ubuntu-jammy-22.04-amd64-server-*" \
  --output table
```

The `099720109477` is **Canonical's** (Ubuntu's official publisher) AWS account ID. Filtering by this owner ensures you're only getting official, trusted Ubuntu images — not community-uploaded ones.

> 💡 **Can't find the AMI ID via CLI?** No worries. You can also find it directly through the AWS Console:
> 1. Go to the **EC2 Dashboard** in the AWS Console.
> 2. In the left sidebar, under **Images**, click **AMI Catalog**.
> 3. Search for `Ubuntu 22.04` and make sure you're in the correct region (top-right corner).
> 4. Click **Select** on the AMI you want and copy the **AMI ID** (e.g., `ami-05d2d839d4f73aafb`).
>
> Always double-check that the AMI is from a verified provider (look for the **"AWS"** or **"Canonical"** badge) before using it.

#### What is a Security Group?

A **Security Group** is a virtual firewall that controls what traffic can reach your EC2 instance. It operates at the instance level and evaluates rules for every connection attempt.

Key characteristics:
- **Stateful**: If you allow inbound traffic on port 80, the response traffic is automatically allowed outbound. You don't need a separate outbound rule.
- **Whitelist-only**: Security groups only allow traffic — they cannot explicitly deny. Everything not allowed is implicitly denied.
- **Multiple groups**: An instance can be associated with multiple security groups.

**Inbound Rules** — control traffic coming INTO your instance:
- Protocol (TCP, UDP, ICMP)
- Port range (e.g., 22 for SSH, 80 for HTTP, 443 for HTTPS, 3000 for Node.js)
- Source (IP address, CIDR range, or another security group)

**Outbound Rules** — control traffic going OUT from your instance (default: allow all).

---

### Step-by-Step: Create EC2 Instance via CLI

#### Step 1: Create a Key Pair

```bash
aws ec2 create-key-pair \
  --key-name my-ec2-key \
  --query 'KeyMaterial' \
  --output text > my-ec2-key.pem

chmod 400 my-ec2-key.pem
```

`chmod 400` makes the key file read-only for the owner — SSH requires this.

#### Step 2: Create a Security Group

```bash
# Create the security group
aws ec2 create-security-group \
  --group-name my-node-app-sg \
  --description "Security group for Node.js app server"
```

You'll receive a `GroupId` in the output like `sg-0abc123def456789`. Save it.

```bash
# Allow SSH (port 22)
aws ec2 authorize-security-group-ingress \
  --group-name my-node-app-sg \
  --protocol tcp \
  --port 22 \
  --cidr 0.0.0.0/0

# Allow HTTP (port 80)
aws ec2 authorize-security-group-ingress \
  --group-name my-node-app-sg \
  --protocol tcp \
  --port 80 \
  --cidr 0.0.0.0/0

# Allow HTTPS (port 443)
aws ec2 authorize-security-group-ingress \
  --group-name my-node-app-sg \
  --protocol tcp \
  --port 443 \
  --cidr 0.0.0.0/0
```

> ⚠️ `0.0.0.0/0` means "allow from anywhere". For SSH in production, restrict this to your own IP for better security: `--cidr YOUR_IP/32`.

#### Step 3: Get Your AMI ID

```bash
AMI_ID=$(aws ec2 describe-images \
  --owners 099720109477 \
  --filters "Name=name,Values=ubuntu/images/hvm-ssd/ubuntu-jammy-22.04-amd64-server-*" \
  --query 'sort_by(Images, &CreationDate)[-1].ImageId' \
  --output text)

echo $AMI_ID
```

#### Step 4: Launch the Instance

```bash
aws ec2 run-instances \
  --image-id $AMI_ID \
  --count 1 \
  --instance-type t2.micro \
  --key-name my-ec2-key \
  --security-groups my-node-app-sg \
  --tag-specifications 'ResourceType=instance,Tags=[{Key=Name,Value=my-node-app-server}]'
```

#### Step 5: Get Your Instance's Public IP

```bash
aws ec2 describe-instances \
  --filters "Name=tag:Name,Values=my-node-app-server" \
  --query 'Reservations[0].Instances[0].PublicIpAddress' \
  --output text
```

You'll get something like `13.235.67.45`. Save this — it's how you'll SSH in.

---

## 7. SSH Into Your EC2 Instance

**SSH (Secure Shell)** is a cryptographic network protocol used to securely connect to a remote machine over the internet. Your `.pem` key file acts as your identity proof.

### SSH Command

```bash
ssh -i my-ec2-key.pem ubuntu@<YOUR_PUBLIC_IP>
```

- `-i my-ec2-key.pem` — specifies the private key file
- `ubuntu` — the default username for Ubuntu AMIs (it's `ec2-user` for Amazon Linux)
- `<YOUR_PUBLIC_IP>` — the public IP from the previous step

### First Connection

On first connection, you'll see:

```
The authenticity of host '13.235.67.45 (13.235.67.45)' can't be established.
ECDSA key fingerprint is SHA256:xxxxxxxxxxxx.
Are you sure you want to continue connecting (yes/no)?
```

Type `yes` and press Enter. You're now inside your EC2 instance!

```
ubuntu@ip-172-31-xx-xx:~$
```

### Common SSH Issues

| Error | Fix |
|---|---|
| `Permission denied (publickey)` | Wrong username or key file mismatch |
| `Unprotected private key file` | Run `chmod 400 my-ec2-key.pem` |
| `Connection timed out` | SSH (port 22) not open in Security Group |
| `Connection refused` | Instance not fully started yet — wait a minute |

---

## 8. Updating the Instance and Installing Node.js & Git

Once inside your EC2 instance, the first thing you should always do is update the system packages.

### Update System Packages

```bash
sudo apt update && sudo apt upgrade -y
```

- `apt update` — refreshes the package list from repositories
- `apt upgrade -y` — installs all available upgrades, `-y` auto-confirms

### Install Git

```bash
sudo apt install git -y
git --version
# git version 2.x.x
```

### Install Node.js (via NodeSource — Recommended)

The version of Node.js in Ubuntu's default package manager is often outdated. Use NodeSource to install a specific LTS version:

```bash
# Install Node.js 20 LTS
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -
sudo apt install -y nodejs
```

Verify:

```bash
node --version   # v20.x.x
npm --version    # 10.x.x
```

> **Why Node 20?** Node 20 is the current LTS (Long Term Support) version — production-ready, stable, and supported until April 2026.

---

## 9. Cloning Your Node.js App from GitHub

Now let's get your application code onto the server.

```bash
# Navigate to your home directory (you're likely already here)
cd ~

# Clone your repository
git clone https://github.com/YOUR_USERNAME/YOUR_REPO_NAME.git

# Navigate into the project
cd YOUR_REPO_NAME
```

### If Your Repository is Private

You'll need to authenticate. The recommended way is to use a **GitHub Personal Access Token (PAT)**:

1. Go to GitHub → Settings → Developer Settings → Personal Access Tokens → Tokens (classic).
2. Generate a new token with `repo` scope.
3. Use it during clone:

```bash
git clone https://YOUR_TOKEN@github.com/YOUR_USERNAME/YOUR_REPO_NAME.git
```

Or configure it globally:

```bash
git config --global credential.helper store
git clone https://github.com/YOUR_USERNAME/YOUR_REPO_NAME.git
# Enter your username and token when prompted
```

---

## 10. Setting Up the App

### Install Dependencies

```bash
npm install
```

This installs all packages listed in `package.json` into the `node_modules` folder.

### Create the `.env` File

Most Node.js apps use environment variables for configuration (API keys, database URLs, secrets). These are never committed to Git (and shouldn't be). You need to create the `.env` file manually on the server.

```bash
nano .env
```

Add your environment variables:

```env
PORT=3000
NODE_ENV=production
DATABASE_URL=mongodb+srv://username:password@cluster.mongodb.net/mydb
JWT_SECRET=your_super_secret_key
API_KEY=your_third_party_api_key
```

Save and exit: `Ctrl + X`, then `Y`, then `Enter`.

### Verify the Setup

```bash
# Check if the file was created correctly
cat .env
```

> 🔒 **Security Note**: Never commit your `.env` file. Ensure `.env` is in your `.gitignore`. Sensitive data in version control is a severe security vulnerability.

---

## 11. Starting the App and Configuring Security Group Ports

### Start Your App (for testing)

```bash
node index.js
# or
node server.js
# or whatever your entry point is
```

If your app starts successfully, you'll see something like:

```
Server running on port 3000
Connected to database
```

Your app is now running inside the EC2 instance on port 3000. But can you access it from your browser? **Not yet** — the Security Group is blocking that port.

### Understanding Inbound Rules in Depth

When you launch an EC2 instance, it has a public IP. The internet can technically route traffic to that IP, but whether traffic actually reaches your application depends on the **Security Group's inbound rules**.

Here's the flow:

```
Internet → EC2 Public IP → Security Group Check → EC2 Instance → Application (port 3000)
```

If your Security Group has no rule for port 3000, the traffic is **silently dropped** before it ever reaches your app. The Security Group acts as a gate.

**Inbound Rule components:**
- **Type**: Protocol type (Custom TCP, HTTP, HTTPS, SSH, etc.)
- **Protocol**: TCP, UDP, or ICMP
- **Port Range**: The specific port or range (e.g., `3000` or `8000-9000`)
- **Source**: Who can send traffic — `0.0.0.0/0` (everyone), your IP, another security group, etc.

### Opening Port 3000 via CLI

```bash
aws ec2 authorize-security-group-ingress \
  --group-name my-node-app-sg \
  --protocol tcp \
  --port 3000 \
  --cidr 0.0.0.0/0
```

### Opening Port 3000 via Console (GUI)

1. Go to EC2 → **Security Groups** in the left sidebar.
2. Select your security group (`my-node-app-sg`).
3. Click the **"Inbound rules"** tab → **"Edit inbound rules"**.
4. Click **"Add rule"**:
   - Type: `Custom TCP`
   - Port range: `3000`
   - Source: `Anywhere-IPv4` (`0.0.0.0/0`)
5. Click **"Save rules"**.

> 💡 Changes to security group rules take effect **immediately** — no restart needed.

---

## 12. Accessing Your API via Public IP and Port

Now that your app is running and port 3000 is open in the Security Group, you can access your API from anywhere.

### Get Your Instance's Public IP

```bash
# From your LOCAL terminal (not the EC2 SSH session)
aws ec2 describe-instances \
  --filters "Name=tag:Name,Values=my-node-app-server" \
  --query 'Reservations[0].Instances[0].PublicIpAddress' \
  --output text
```

Or find it in the EC2 Console under **Instance → Public IPv4 address**.

### Access the API

Open your browser or use curl:

```bash
# Browser
http://<YOUR_PUBLIC_IP>:3000

# With curl
curl http://<YOUR_PUBLIC_IP>:3000/api/hello

# Example with an actual IP
curl http://13.235.67.45:3000/api/users
```

### Testing with Postman

If you use Postman, just set the base URL to:
```
http://13.235.67.45:3000
```
And hit your endpoints as usual.

> ⚠️ **Important**: EC2 Public IPs are **ephemeral** — they change every time you stop and start the instance. For a permanent IP, you can allocate and associate an **Elastic IP** address (free as long as it's attached to a running instance).

---

## 13. Introducing PM2 — Process Management Done Right

So your app is running with `node index.js`. Now ask yourself:

**What happens if your Node.js app crashes?**

It just... dies. The server is still running but your app is gone. Users get connection refused. You have to SSH back in and restart it manually.

**What happens when you close your SSH terminal?**

Your app is killed. Because `node index.js` runs as a foreground process attached to your terminal session. When the session ends, the process ends.

This is where **PM2** comes in.

### What is PM2?

**PM2 (Process Manager 2)** is a production-grade process manager for Node.js applications. It keeps your applications alive forever — automatically restarting them if they crash or if the server reboots.

### Why PM2? — Feature Deep Dive

#### 1. Automatic Restart on Crash
PM2 monitors your process. If it crashes (unhandled exception, out of memory, etc.), PM2 **automatically restarts it** within milliseconds. Your users experience minimal downtime.

#### 2. Startup Script Generation
PM2 can register itself as a system service (via `systemd` on Linux). This means your app **automatically starts on server reboot** — no manual intervention needed.

#### 3. Process Monitoring
PM2 gives you a real-time dashboard showing:
- CPU usage
- Memory usage
- Uptime
- Restart count
- Process status

#### 4. Log Management
PM2 captures `stdout` and `stderr` from your Node.js app and saves them to log files. You can stream logs in real-time, rotate them, and inspect historical logs.

#### 5. Cluster Mode
PM2 can run your Node.js app in **cluster mode**, spawning one process per CPU core. This takes advantage of multi-core servers and significantly improves throughput for CPU-bound operations.

#### 6. Zero-Downtime Reloads
PM2 supports `pm2 reload`, which performs a rolling restart — new processes come up before old ones go down — meaning zero downtime during app updates.

### Installing PM2

```bash
sudo npm install -g pm2
```

Verify:
```bash
pm2 --version
```

### Starting Your App with PM2

```bash
# Navigate to your project directory
cd ~/YOUR_REPO_NAME

# Start the app
pm2 start index.js --name "my-node-app"
```

Output:
```
[PM2] Starting /home/ubuntu/YOUR_REPO_NAME/index.js in fork_mode (1 instance)
[PM2] Done.
┌─────┬──────────────────┬─────────────┬─────────┬─────────┬──────────┬────────┬──────┬───────────┐
│ id  │ name             │ namespace   │ version │ mode    │ pid      │ uptime │ ↺    │ status    │
├─────┼──────────────────┼─────────────┼─────────┼─────────┼──────────┼────────┼──────┼───────────┤
│ 0   │ my-node-app      │ default     │ 1.0.0   │ fork    │ 12345    │ 0s     │ 0    │ online    │
└─────┴──────────────────┴─────────────┴─────────┴─────────┴──────────┴────────┴──────┴───────────┘
```

### Essential PM2 Commands

```bash
# List all running processes
pm2 list

# Monitor in real-time (CPU, memory, logs)
pm2 monit

# View logs
pm2 logs my-node-app

# View last 100 lines of logs
pm2 logs my-node-app --lines 100

# Restart the app
pm2 restart my-node-app

# Stop the app (keeps it in PM2's list)
pm2 stop my-node-app

# Delete the app from PM2
pm2 delete my-node-app

# Reload with zero downtime
pm2 reload my-node-app
```

### Configure PM2 to Start on Boot

This is the most important PM2 step for production:

```bash
# Generate startup script
pm2 startup
```

PM2 will output a command that looks like:

```bash
sudo env PATH=$PATH:/usr/bin /usr/lib/node_modules/pm2/bin/pm2 startup systemd -u ubuntu --hp /home/ubuntu
```

**Copy and run that exact command** (it's different for each system). Then:

```bash
# Save the current process list so PM2 knows what to start on boot
pm2 save
```

Now if your EC2 instance reboots, PM2 will automatically start your app. You can test this:

```bash
sudo reboot
# Wait 1 minute, then SSH back in
ssh -i my-ec2-key.pem ubuntu@<YOUR_PUBLIC_IP>
pm2 list
# Your app should be running!
```

### Using an Ecosystem File (Advanced)

For more complex configurations, PM2 supports an ecosystem file:

```bash
pm2 ecosystem
```

This creates a `ecosystem.config.js`:

```javascript
module.exports = {
  apps: [{
    name: 'my-node-app',
    script: './index.js',
    instances: 'max',      // Use all CPU cores (cluster mode)
    exec_mode: 'cluster',
    watch: false,
    env: {
      NODE_ENV: 'development',
      PORT: 3000
    },
    env_production: {
      NODE_ENV: 'production',
      PORT: 3000
    },
    error_file: './logs/err.log',
    out_file: './logs/out.log',
    log_date_format: 'YYYY-MM-DD HH:mm:ss'
  }]
}
```

Start with the ecosystem file:
```bash
pm2 start ecosystem.config.js --env production
```

---

## 14. NGINX and Reverse Proxy — The Gateway to Your App

Your Node.js app is running on port 3000. Users currently have to access it via `http://13.235.67.45:3000`. This is ugly, insecure, and impractical. You want:

- `http://yourdomain.com` → your app
- `https://yourdomain.com` → your app (with SSL)
- No port numbers in the URL

This is where **NGINX** enters the picture.

### What is NGINX?

**NGINX** (pronounced "engine-x") is a high-performance web server, reverse proxy, load balancer, and HTTP cache. It was designed to handle massive amounts of concurrent connections efficiently.

Originally created to solve the C10k problem (handling 10,000+ concurrent connections), NGINX uses an **event-driven, non-blocking architecture** — meaning it can handle thousands of simultaneous connections using minimal memory, unlike traditional thread-based servers.

### What is a Reverse Proxy?

A **reverse proxy** is a server that sits in front of your application servers and forwards client requests to them.

Here's the traffic flow without a reverse proxy:

```
Client → EC2 Public IP:3000 → Node.js App
```

Here's the flow with NGINX as a reverse proxy:

```
Client → EC2 Public IP:80 (or 443) → NGINX → localhost:3000 → Node.js App
```

### Why Use NGINX as a Reverse Proxy?

#### 1. Clean URLs (No Port Numbers)
Users hit `http://yourdomain.com` (port 80) or `https://yourdomain.com` (port 443). NGINX receives the request on port 80/443 and forwards it internally to port 3000. The user never sees `:3000`.

#### 2. SSL/TLS Termination
NGINX handles HTTPS encryption/decryption. Your Node.js app only deals with plain HTTP internally. This simplifies your app and is more performant — SSL operations are handled by NGINX's optimized C code.

#### 3. Load Balancing
If you run multiple instances of your Node.js app (e.g., on different ports or different servers), NGINX can distribute traffic across them — improving reliability and throughput.

#### 4. Static File Serving
NGINX is extremely efficient at serving static files (HTML, CSS, JS, images). You can configure it to serve your frontend directly without hitting Node.js at all — freeing Node.js to handle only API requests.

#### 5. Security Layer
NGINX acts as a first line of defense:
- Rate limiting (prevent DDoS/brute force attacks)
- Request size limits (prevent large payload attacks)
- IP whitelisting/blacklisting
- Hiding your Node.js version and internal architecture from clients

#### 6. Compression
NGINX can gzip compress responses before sending them to clients, reducing bandwidth usage and speeding up page loads.

#### 7. Buffering
NGINX can buffer responses from your Node.js app, protecting it from slow clients. Without buffering, Node.js holds connections open for slow clients, consuming memory. With NGINX, Node.js returns the response to NGINX and is immediately free.

---

## 15. Setting Up NGINX Step by Step

### Step 1: Install NGINX

```bash
sudo apt update
sudo apt install nginx -y
```

### Step 2: Verify NGINX is Running

```bash
sudo systemctl status nginx
```

You should see `Active: active (running)`. You can also open `http://<YOUR_PUBLIC_IP>` in a browser — you'll see the NGINX default welcome page.

```bash
# Start NGINX (if not already running)
sudo systemctl start nginx

# Enable NGINX to start on boot
sudo systemctl enable nginx
```

### Step 3: Understanding NGINX Configuration Structure

NGINX's configuration lives in `/etc/nginx/`:

```
/etc/nginx/
├── nginx.conf              # Main config file
├── sites-available/        # Available site configs (inactive)
│   └── default             # Default site
├── sites-enabled/          # Active site configs (symlinked from sites-available)
│   └── default → ../sites-available/default
├── conf.d/                 # Alternative location for configs
└── snippets/               # Reusable config snippets
```

The pattern is:
1. Create your site config in `sites-available/`
2. Create a symbolic link in `sites-enabled/` to activate it
3. Test and reload NGINX

### Step 4: Create Your NGINX Configuration

Remove the default config:
```bash
sudo rm /etc/nginx/sites-enabled/default
```

Create a new config file for your app:

```bash
sudo nano /etc/nginx/sites-available/my-node-app
```

Paste the following configuration:

```nginx
server {
    listen 80;
    server_name _;   # _ means "match any hostname" — replace with your domain if you have one

    # Logging
    access_log /var/log/nginx/my-node-app.access.log;
    error_log /var/log/nginx/my-node-app.error.log;

    # Gzip compression
    gzip on;
    gzip_types text/plain application/json application/javascript text/css;

    location / {
        # Forward requests to Node.js app
        proxy_pass http://localhost:3000;

        # Pass real client IP to Node.js
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;

        # Pass original Host header
        proxy_set_header Host $http_host;

        # WebSocket support (if your app uses it)
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";

        # Timeouts
        proxy_connect_timeout 60s;
        proxy_send_timeout 60s;
        proxy_read_timeout 60s;
    }
}
```

**Config breakdown:**
- `listen 80` — NGINX listens on port 80 (HTTP)
- `server_name _` — matches all hostnames (replace with your actual domain like `api.yourdomain.com`)
- `proxy_pass http://localhost:3000` — this is the core reverse proxy directive, forwarding all traffic to your Node.js app
- `proxy_set_header` directives pass important metadata to your Node.js app (real client IP, protocol, etc.)
- `proxy_http_version 1.1` and `Connection "upgrade"` enable WebSocket proxying

### Step 5: Enable the Configuration

```bash
# Create a symlink to activate the config
sudo ln -s /etc/nginx/sites-available/my-node-app /etc/nginx/sites-enabled/

# Test for syntax errors
sudo nginx -t
```

Expected output:
```
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
```

If there's an error, it'll tell you exactly which line. Fix it before reloading.

### Step 6: Reload NGINX

```bash
sudo systemctl reload nginx
```

`reload` applies the new configuration without dropping existing connections, unlike `restart`.

### Step 7: Update Security Group

Now that NGINX is listening on port 80, you can remove port 3000 from the security group (optional but good practice — users shouldn't bypass NGINX):

```bash
# Allow port 80 (if not already allowed)
aws ec2 authorize-security-group-ingress \
  --group-name my-node-app-sg \
  --protocol tcp \
  --port 80 \
  --cidr 0.0.0.0/0

# Optionally revoke direct access to port 3000
aws ec2 revoke-security-group-ingress \
  --group-name my-node-app-sg \
  --protocol tcp \
  --port 3000 \
  --cidr 0.0.0.0/0
```

### Step 8: Test the Setup

```bash
# From your local machine
curl http://<YOUR_PUBLIC_IP>/api/hello
```

You should get a response from your Node.js app — but now on port 80, served through NGINX.

### NGINX Useful Commands

```bash
# Check NGINX status
sudo systemctl status nginx

# Test configuration
sudo nginx -t

# Reload config (no downtime)
sudo systemctl reload nginx

# Restart NGINX (brief downtime)
sudo systemctl restart nginx

# View access logs in real-time
sudo tail -f /var/log/nginx/my-node-app.access.log

# View error logs
sudo tail -f /var/log/nginx/my-node-app.error.log
```

### Bonus: NGINX Rate Limiting

To protect your API from abuse:

```nginx
# Add this in the http {} block in /etc/nginx/nginx.conf
limit_req_zone $binary_remote_addr zone=api_limit:10m rate=10r/s;

# Then in your server block
location /api/ {
    limit_req zone=api_limit burst=20 nodelay;
    proxy_pass http://localhost:3000;
}
```

This limits each IP to 10 requests per second with a burst of 20.

---

## 16. Summary and Conclusion

Congratulations! You've gone from zero to a fully production-ready Node.js deployment on AWS EC2. Let's recap the full architecture you've built:

### The Complete Stack

```
Internet
    ↓
AWS Security Group (Firewall — allows port 80, 443)
    ↓
EC2 Instance (Ubuntu Server 22.04)
    ↓
NGINX (Port 80/443 — Reverse Proxy)
    ↓
Node.js App (Port 3000 — managed by PM2)
    ↓
Your Application Logic / Database
```

### What You Learned

| Topic | Key Takeaway |
|---|---|
| **AWS Account** | Root user is powerful — protect it with MFA and use IAM users for everyday work |
| **AWS CLI** | Faster than GUI, scriptable, essential for DevOps |
| **EC2** | Virtual machines in the cloud — billed per second, scalable on demand |
| **AMI** | OS template for launching instances — region-specific |
| **Security Groups** | Stateful virtual firewalls — whitelist-only, changes apply instantly |
| **SSH** | Secure remote access using key-pair authentication |
| **PM2** | Process manager that keeps your Node app alive across crashes and reboots |
| **NGINX** | High-performance reverse proxy that handles SSL, routing, compression, and security |

### Why This Architecture is Production-Ready

1. **Reliability**: PM2 restarts your app on crash. NGINX keeps accepting connections.
2. **Security**: Users never directly access your Node.js port. NGINX is the only entry point.
3. **Performance**: NGINX handles static files efficiently, buffers responses, and can be extended with caching.
4. **Maintainability**: PM2 manages logs, and you can deploy updates with zero-downtime reloads.
5. **Scalability**: Add more EC2 instances behind an AWS Load Balancer, with NGINX handling upstream routing.

### What's Next?

Now that you have the basics down, here's what to explore next:

- **Domain + SSL**: Point a domain to your EC2 IP and use **Certbot (Let's Encrypt)** to add free HTTPS.
- **Elastic IP**: Assign a static IP so your address doesn't change on restart.
- **RDS**: Move your database off the EC2 instance and onto AWS RDS for managed, durable database hosting.
- **S3**: Store user-uploaded files in S3 instead of the EC2 filesystem.
- **CloudFront**: Add a CDN in front of NGINX for global edge caching and reduced latency.
- **Auto Scaling**: Set up Auto Scaling Groups to automatically add/remove EC2 instances based on traffic.
- **CI/CD**: Automate deployments with GitHub Actions — push to `main`, and your EC2 updates automatically.

---

> **Liked this guide?** Share it with developers who are getting started with AWS. Drop a comment if you hit any issues — the deployment world is full of gotchas, and the community grows stronger when we share what we know.

---

*Written with ❤️ — Happy deploying!*
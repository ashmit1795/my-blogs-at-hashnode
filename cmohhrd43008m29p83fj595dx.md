---
title: "Getting Started with Ansible — From Zero to Hero 🚀"
seoTitle: "Ansible Explained: From Basics to Real Automation"
seoDescription: "Learn Ansible from scratch with a hands-on EC2 lab. Understand inventory, playbooks, roles, and automate real-world server setups."
datePublished: 2026-04-27T17:49:01.353Z
cuid: cmohhrd43008m29p83fj595dx
slug: getting-started-with-ansible-from-zero-to-hero
cover: https://cdn.hashnode.com/uploads/covers/69b5b16d1f4d4527ed9008ad/bc440284-bde9-4314-be8f-9d4241ed0e0d.png
tags: ansible, cloud-computing, automation, devops, backend-development, learninginpublic

---

> You have 50 servers. You need to install nginx on all of them, update a config file, and restart the service. Do you SSH into each one manually? Or do you write it once and let a tool handle the rest? That's exactly the problem Ansible was built to solve — and by the end of this article, you'll understand not just *how* to use it, but *why* it works the way it does.

---

## Table of Contents

1. [What is Ansible and Why Does It Exist?](#1-what-is-ansible-and-why-does-it-exist)
2. [The Mental Model — How Ansible Thinks](#2-the-mental-model--how-ansible-thinks)
3. [Phase 1 — Setting Up the Lab (Two EC2 Instances)](#3-phase-1--setting-up-the-lab-two-ec2-instances)
4. [Phase 2 — The Inventory File (The Heart of Ansible)](#4-phase-2--the-inventory-file-the-heart-of-ansible)
5. [Phase 3 — Ad-Hoc Commands](#5-phase-3--ad-hoc-commands)
6. [Phase 4 — Ansible Playbooks](#6-phase-4--ansible-playbooks)
7. [Phase 5 — Ansible Roles](#7-phase-5--ansible-roles)
8. [The Full Picture — How Everything Connects](#8-the-full-picture--how-everything-connects)
9. [Summary and What's Next](#10-summary-and-whats-next)

---

## 1. What is Ansible and Why Does It Exist?

### The Problem It Solves

Imagine you're a DevOps engineer at a growing startup. You have 50 servers — some running your API, some running background workers, some running databases. A security patch just dropped and you need to:

1. Update all servers
2. Install a new package on the web servers
3. Modify a config file on each one
4. Restart the relevant service

Without a tool like Ansible, your only option is to SSH into each server one by one and run the commands manually. That's slow, error-prone, and doesn't scale. If you have 500 servers next year, it becomes impossible.

**Ansible solves this by letting you describe what you want your servers to look like — and it makes them look that way. All of them. At once.**

### What Makes Ansible Special?

There are other configuration management tools out there — **Puppet**, **Chef**, **SaltStack**. They all solve the same general problem. But Ansible has one key architectural advantage that sets it apart:

**It is completely agentless.**

Other tools require you to install a software agent on every machine you want to manage. That agent runs continuously, phones home to a central server, and applies configuration. This adds overhead, a new attack surface, and something else to maintain.

Ansible needs nothing on the target machines beyond:
- **SSH access**
- **Python** (which Ubuntu has installed by default)

That's it. Ansible runs from your laptop or a central control node, SSHs into each target machine, executes the necessary commands, and exits. Clean, simple, and minimal.

```
Your Laptop / Ansible Control Node
        │
        │─── SSH ──→ Server 1 (no agent needed)
        │─── SSH ──→ Server 2 (no agent needed)
        │─── SSH ──→ Server N (no agent needed)
```

### Where Ansible Sits in the DevOps World

Ansible is a **configuration management and automation tool**. Here's how it fits in:

| Layer | Tool Examples | What it does |
|---|---|---|
| Infrastructure provisioning | Terraform, CloudFormation | Create the servers/infrastructure |
| Configuration management | **Ansible**, Puppet, Chef | Configure what's on those servers |
| CI/CD | Jenkins, GitHub Actions, CodePipeline | Build and deploy your application |
| Container orchestration | Kubernetes, ECS | Run and scale containerized apps |

Ansible sits in the middle — after your server exists, it makes your server ready. And because it's also great at orchestration, it's often used across the CI/CD and deployment layers too.

> 📖 **Official docs starting point**: [What is Ansible?](https://docs.ansible.com/ansible/latest/getting_started/introduction.html) — bookmark `docs.ansible.com`. You'll live there.

---

## 2. The Mental Model — How Ansible Thinks

Before touching any terminal, building the right mental model saves hours of confusion later.

### Ansible's Core Philosophy: Desired State

Ansible is **declarative** in spirit. You don't tell Ansible *how* to do something step by step. You describe *what you want the end state to be*, and Ansible figures out how to get there.

For example, instead of writing:

```bash
# imperative — telling it HOW
apt-get update
apt-get install -y nginx
systemctl start nginx
systemctl enable nginx
```

You write in Ansible:

```yaml
# declarative — telling it WHAT you want
- name: Install and start nginx
  apt:
    name: nginx
    state: present    # "I want nginx to be present"
  service:
    name: nginx
    state: started    # "I want nginx to be running"
    enabled: yes      # "I want nginx to start on boot"
```

The difference matters because of **idempotency** — which is Ansible's most important concept.

### Idempotency — The Core Concept

**Idempotency** means: running the same operation multiple times produces the same result as running it once.

If nginx is already installed, Ansible won't install it again. If the config file already has the right content, Ansible won't touch it. It only makes a change when a change is actually needed.

This matters enormously in practice:
- You can run the same playbook every day as a health check
- You can safely re-run a failed playbook from the middle
- You never worry about accidentally running something twice

> 💡 **Why this is powerful**: In a traditional shell script, running it twice might install nginx twice, restart it unnecessarily, or create duplicate config entries. With Ansible, it's always safe to run again.

### The Four Core Concepts

Before you write a single line of Ansible, understand these four terms:

| Concept | What it is | Analogy |
|---|---|---|
| **Inventory** | The list of servers Ansible manages | An address book of your servers |
| **Module** | A single reusable unit of work (install a package, copy a file, etc.) | A built-in function |
| **Task** | One call to a module with specific arguments | One line of your to-do list |
| **Playbook** | A YAML file containing one or more plays (groups of tasks) | Your full instruction manual |

And one more that builds on top of all of these:

| Concept | What it is |
|---|---|
| **Role** | A structured, reusable collection of tasks, handlers, templates, and variables for a specific purpose |

You'll build understanding of each one progressively through this article.

---

## 3. Phase 1 — Setting Up the Lab (Two EC2 Instances)

Theory only takes you so far. Let's build a real lab.

### Why Two Instances?

Ansible has two types of machines in its world:

- **Control Node**: The machine where Ansible is installed and where you run commands *from*. This is your laptop or a dedicated server.
- **Target Node (Managed Node)**: The machine Ansible manages — it gets SSH'd into and configured. Ansible is **not** installed here.

We'll simulate this with two EC2 instances on AWS.

### Step 1: Launch Both EC2 Instances

Go to **AWS Console → EC2 → Launch Instance**.

**Instance 1 — Ansible Control Node**
```
Name:          ansible-control
AMI:           Ubuntu Server 22.04 LTS
Instance type: t2.micro (Free Tier eligible)
Key pair:      Create new → name it "ansible-key" → RSA → .pem → Download
Security Group: Allow SSH (port 22) from Anywhere
```

**Instance 2 — Ansible Target Node**
```
Name:          ansible-target
AMI:           Ubuntu Server 22.04 LTS
Instance type: t2.micro
Key pair:      Select the SAME "ansible-key" ← This is critical
Security Group: Allow SSH (port 22) from Anywhere
```

> ⚠️ **Why the same key pair?** Ansible will SSH *from* the control node *into* the target node. For that to work, the control node needs the private key, and the target node needs to accept it. Using the same key pair for both makes this setup straightforward.

### Step 2: SSH Into the Control Node

From your **local machine**:

```bash
chmod 400 ansible-key.pem
ssh -i ansible-key.pem ubuntu@<control-node-public-ip>
```

`chmod 400` sets the file to read-only for the owner. SSH refuses to use a key file that's too permissive — this command fixes that.

### Step 3: Install Ansible on the Control Node ONLY

Once inside the control node:

```bash
sudo apt update
sudo apt install -y software-properties-common
sudo add-apt-repository --yes --update ppa:ansible/ansible
sudo apt install -y ansible
ansible --version
```

You should see output like:
```
ansible [core 2.x.x]
  config file = /etc/ansible/ansible.cfg
  python version = 3.x.x
  ...
```

> 💡 **Why install from the PPA?** Ubuntu's default apt repository often has an outdated version of Ansible. The official `ppa:ansible/ansible` always has the latest stable version.

**The target node gets nothing installed.** That's the entire point of agentless architecture — your target machines stay clean.

> 📖 **Official installation docs**: [Installing Ansible](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html) — covers all platforms (macOS, Windows via WSL, various Linux distros).

### Step 4: Copy the PEM Key to the Control Node

From your **local machine**:

```bash
scp -i ansible-key.pem ansible-key.pem ubuntu@<control-node-public-ip>:~/.ssh/
```

`scp` (secure copy) transfers the file over SSH. We're copying the key *to* the control node so that Ansible can use it when SSHing into the target.

Then on the **control node**, fix permissions:

```bash
chmod 400 ~/.ssh/ansible-key.pem
```

### Step 5: Verify SSH Works Manually First

Before letting Ansible do anything, verify the SSH connection works with your own hands:

```bash
ssh -i ~/.ssh/ansible-key.pem ubuntu@<target-node-PRIVATE-ip>
```

> 💡 **Why the private IP?** Both instances are in the same AWS VPC (Virtual Private Cloud). Within the same VPC, machines communicate over private IP addresses — which is faster, more secure, and free. Using the public IP would route traffic out to the internet and back in unnecessarily.

If this SSH command works — types `exit` to return to the control node — Ansible will work. If SSH doesn't work, Ansible won't either. Always verify the foundation first.

---

## 4. Phase 2 — The Inventory File (The Heart of Ansible)

Ansible needs to know which servers it manages. You tell it using an **inventory file**. This is arguably the most important concept to get right because everything else depends on it.

### Create Your Project Directory

```bash
mkdir ~/ansible-lab && cd ~/ansible-lab
```

Good practice: keep all Ansible files for a project in one directory.

### Create the Inventory File

```bash
nano inventory.ini
```

```ini
# inventory.ini

[webservers]
ansible-target ansible_host=<target-private-ip> ansible_user=ubuntu ansible_ssh_private_key_file=~/.ssh/ansible-key.pem

[dbservers]
# you'd add database server IPs here later

[all:vars]
ansible_python_interpreter=/usr/bin/python3
```

Let's break down every piece of this:

### Understanding the Inventory Syntax

```ini
ansible-target                              ← Alias (human-readable name)
  ansible_host=10.0.1.45                   ← The actual IP Ansible connects to
  ansible_user=ubuntu                      ← The SSH username
  ansible_ssh_private_key_file=~/.ssh/...  ← The private key to authenticate with
```

**`[webservers]`** — This is a **group**. Groups are how you organize and target subsets of your servers. Instead of specifying individual IPs in every command, you say "run this on webservers" and Ansible knows exactly which machines that means.

**`[all:vars]`** — Variables applied to ALL hosts in the inventory. `ansible_python_interpreter=/usr/bin/python3` tells Ansible to use Python 3, avoiding warnings on modern Ubuntu systems.

### Groups — Where Ansible's Real Power Lives

```ini
[webservers]
web1 ansible_host=10.0.0.1
web2 ansible_host=10.0.0.2

[dbservers]
db1 ansible_host=10.0.0.3
db2 ansible_host=10.0.0.4

[production:children]   # A group OF groups
webservers
dbservers

[production:vars]       # Variables applied to the production group
env=production
```

`[production:children]` is a **nested group** — it contains other groups. Targeting `production` means your command hits all web and database servers. This is how real infrastructure inventories are organized — you never manually list 200 IPs in a command.

> 💡 **Why this matters**: In a real company, you might have `staging`, `production`, and `development` environments. Each is a group. You can target just `staging` to safely test a change before running it against `production`.

### Verify the Inventory

```bash
# See all hosts Ansible knows about
ansible all -i inventory.ini --list-hosts

# See only webservers
ansible webservers -i inventory.ini --list-hosts
```

> 📖 **Official inventory docs**: [How to build your inventory](https://docs.ansible.com/ansible/latest/inventory_guide/intro_inventory.html) — this page explains static inventories in full detail, plus dynamic inventories (which automatically pull server lists from AWS, GCP, etc.).

---

## 5. Phase 3 — Ad-Hoc Commands

Before writing a full playbook, **ad-hoc commands** let you run a single task directly from the terminal. No file needed. Think of them as Ansible's "quick command" mode — perfect for testing, one-off tasks, and checking connectivity.

### The Syntax

```bash
ansible <host-or-group> -i <inventory> -m <module> -a "<arguments>"
```

- `-m` = module (the action you want to perform)
- `-a` = arguments to pass to the module
- `--become` = run as sudo (equivalent to `sudo` on the target)

### The Ping Test — Always Start Here

```bash
ansible all -i inventory.ini -m ping
```

Expected output:
```json
ansible-target | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
```

> ⚠️ **This is NOT an ICMP ping (like `ping google.com`)**. Ansible's ping module tests the full connection: SSH authentication works, Python is available on the target, and Ansible can execute. It's a complete connectivity and readiness check.

If this succeeds — your entire Ansible setup is working correctly.

### Real Ad-Hoc Examples

**Check uptime on all servers:**
```bash
ansible all -i inventory.ini -m command -a "uptime"
```

The `command` module runs a raw shell command on the target. Simple and direct.

**Install nginx (without writing a playbook):**
```bash
ansible webservers -i inventory.ini -m apt -a "name=nginx state=present" --become
```

Breaking this down:
- `-m apt` — use the `apt` module (handles package management on Debian/Ubuntu)
- `name=nginx` — the package to manage
- `state=present` — "I want this package to be installed"
- `--become` — run with sudo (installing packages requires root)

> 💡 **`state=present` vs `state=absent`**: `present` installs the package if it's not there. `absent` removes it if it is. This is idempotency in action — `state=present` doesn't reinstall if nginx is already installed.

**Start a service:**
```bash
ansible webservers -i inventory.ini -m service -a "name=nginx state=started" --become
```

**Copy a file to all servers:**
```bash
ansible all -i inventory.ini -m copy -a "src=/tmp/test.txt dest=/tmp/test.txt"
```

**Get all facts about a host (everything Ansible knows about it):**
```bash
ansible all -i inventory.ini -m setup
```

This returns a massive JSON object containing: OS details, CPU info, memory, disk, network interfaces, IP addresses, and much more. These **facts** become available as variables you can use inside playbooks — for example, conditionally running a task only on Ubuntu, or using the server's hostname in a config file.

> 📖 **Module reference**: [Ansible Built-in Modules](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/index.html) — spend 10 minutes browsing this. Look at `apt`, `copy`, `service`, `file`, `user`, `template`. You'll reference this page constantly. Each module page shows all its parameters with examples.

---

## 6. Phase 4 — Ansible Playbooks

Ad-hoc commands are great for one-off tasks. But for anything repeatable, version-controlled, and production-grade — you use **Playbooks**.

A Playbook is a **YAML file** that defines everything Ansible should do. It's your infrastructure as code — checked into Git, reviewed like application code, and executed consistently every time.

### The Anatomy of a Playbook

```
Playbook (a .yml file)
  └── Play 1 — targets: webservers
        └── Task 1: Update apt cache
        └── Task 2: Install nginx
        └── Task 3: Start nginx
        └── Task 4: Copy config file
  └── Play 2 — targets: dbservers
        └── Task 1: Install MySQL
        └── Task 2: Configure MySQL
```

A **play** maps a group of hosts to a list of tasks. A **playbook** contains one or more plays. Each **task** calls one module with specific arguments.

### Your First Playbook

```bash
nano nginx-setup.yml
```

```yaml
---
- name: Setup Nginx on Web Servers        # Human-readable name for this play
  hosts: webservers                        # Which group from inventory to target
  become: true                             # Run all tasks with sudo

  tasks:

    - name: Update apt cache
      apt:
        update_cache: yes                  # Equivalent to: apt update

    - name: Install nginx
      apt:
        name: nginx
        state: present                     # Install if not present — idempotent

    - name: Start nginx and enable on boot
      service:
        name: nginx
        state: started                     # Ensure it's running
        enabled: yes                       # Auto-start on server reboot

    - name: Create a custom index page
      copy:
        content: "<h1>Deployed by Ansible 🚀</h1>"
        dest: /var/www/html/index.html
        owner: www-data
        group: www-data
        mode: "0644"                       # File permissions
```

**Run the playbook:**

```bash
ansible-playbook -i inventory.ini nginx-setup.yml
```

You'll see output like:

```
PLAY [Setup Nginx on Web Servers] ******************************************

TASK [Gathering Facts] *****************************************************
ok: [ansible-target]

TASK [Update apt cache] ****************************************************
changed: [ansible-target]

TASK [Install nginx] *******************************************************
changed: [ansible-target]

TASK [Start nginx and enable on boot] **************************************
ok: [ansible-target]

TASK [Create a custom index page] ******************************************
changed: [ansible-target]

PLAY RECAP *****************************************************************
ansible-target : ok=5   changed=3   unreachable=0   failed=0
```

**Run it again:**

```
TASK [Update apt cache] ****************************************************
ok: [ansible-target]       ← was "changed", now "ok" — nothing to do

TASK [Install nginx] *******************************************************
ok: [ansible-target]       ← already installed, no change needed
```

This is **idempotency** in action. The second run finds everything already in the desired state and makes zero changes. Safe to run a hundred times.

### Understanding YAML in Ansible

If you're new to YAML, the indentation matters — it's how YAML structures data:

```yaml
---                          # Document start marker (optional but good practice)
- name: My Play             # A list item starts with -
  hosts: webservers          # Key: value
  become: true

  tasks:                     # A key whose value is a list
    - name: Task One         # List item
      apt:                   # Module name (key)
        name: nginx          # Module argument (indented under module)
        state: present
```

> ⚠️ **Common YAML mistake**: Never use tabs — only spaces. Two spaces per indentation level is the Ansible convention.

### Variables in Playbooks

Hard-coding values directly in tasks is bad practice. If you decide to change the package name or path, you'd have to hunt through the entire file. Variables solve this:

```yaml
---
- name: Setup Web Server
  hosts: webservers
  become: true
  vars:
    package_name: nginx
    web_root: /var/www/html
    custom_message: "Hello from Ansible"

  tasks:
    - name: Install {{ package_name }}
      apt:
        name: "{{ package_name }}"
        state: present

    - name: Create index page
      copy:
        content: "<h1>{{ custom_message }} — {{ inventory_hostname }}</h1>"
        dest: "{{ web_root }}/index.html"
```

**`{{ inventory_hostname }}`** is a **magic variable** — Ansible automatically fills it with the hostname of the current target machine. You didn't define it — Ansible provides it for every play. Other useful magic variables:

| Variable | Value |
|---|---|
| `inventory_hostname` | The host's name from the inventory |
| `ansible_host` | The IP address of the host |
| `ansible_user` | The SSH user |
| `ansible_facts['os_family']` | OS type (Debian, RedHat, etc.) |
| `ansible_facts['memtotal_mb']` | Total memory in MB |

> 📖 **Variable docs**: [Using Variables](https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_variables.html) — the **variable precedence** section on this page is essential reading. When the same variable is defined in multiple places (inventory, playbook, command line), which one wins? This page explains the priority order.

### Handlers — Run Only When Something Changes

Consider this scenario: you copy a new nginx config file to the server. You want nginx to restart so it picks up the new config. But you only want to restart if the config actually changed — not on every playbook run (restarting nginx unnecessarily causes brief downtime).

**Handlers** solve this exactly:

```yaml
  tasks:
    - name: Copy nginx config
      copy:
        src: nginx.conf
        dest: /etc/nginx/nginx.conf
      notify: Restart nginx          # Tell the handler to fire (only if this task CHANGED)

    - name: Copy another config
      copy:
        src: sites.conf
        dest: /etc/nginx/sites-enabled/default
      notify: Restart nginx          # Same handler — will only run ONCE even if notified twice

  handlers:
    - name: Restart nginx            # Name matches the notify string exactly
      service:
        name: nginx
        state: restarted
```

Key behavior of handlers:
- Only runs if at least one task that notifies it actually **changed** something
- Runs **once at the very end** of the play, not immediately when notified
- If the same handler is notified by 5 different tasks, it still only runs once

> 💡 **Why this matters**: Config files change infrequently. You don't want nginx restarting on every playbook run — only when the config actually changed. Handlers give you this precision automatically.

### Loops — Apply a Task to Multiple Items

```yaml
    - name: Install required packages
      apt:
        name: "{{ item }}"
        state: present
      loop:
        - nginx
        - curl
        - git
        - htop
```

The task runs once for each item in the loop. `{{ item }}` is replaced with the current loop value. Much cleaner than writing four separate install tasks.

### Conditionals — Run Tasks Based on Facts

```yaml
    - name: Install nginx (Ubuntu/Debian only)
      apt:
        name: nginx
        state: present
      when: ansible_facts['os_family'] == "Debian"

    - name: Install nginx (RHEL/CentOS only)
      yum:
        name: nginx
        state: present
      when: ansible_facts['os_family'] == "RedHat"
```

`ansible_facts` is populated by the `setup` module that automatically runs at the beginning of every play (the "Gathering Facts" step you saw in the output). It gives you detailed information about each target machine, which you can use to make your playbooks work correctly across different OS types.

> 📖 **Playbook intro**: [Intro to Playbooks](https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_intro.html) — the official guide covers everything above in more depth, with additional examples.

---

## 7. Phase 5 — Ansible Roles

Playbooks work perfectly well — until they don't. Once a playbook grows beyond a few dozen tasks, it becomes hard to read, hard to maintain, and impossible to reuse across projects.

**Roles are the solution.** A role is a self-contained, reusable package for a specific concern — an nginx role, a mysql role, a users role, a docker role. You write it once, test it, and reuse it across any project.

Think of roles as **npm packages for infrastructure**.

### Why Roles?

Without roles, you might have a single `site.yml` that's 800 lines long, handling web server config, database setup, user management, firewall rules, and more — all tangled together.

With roles:

```yaml
# site.yml — clean and readable
- name: Configure Web Servers
  hosts: webservers
  become: true
  roles:
    - nginx
    - firewall

- name: Configure Database Servers
  hosts: dbservers
  become: true
  roles:
    - mysql
    - firewall
```

The logic lives in the roles. `site.yml` is just an orchestrator.

### The Role Directory Structure

This is the standard structure Ansible expects:

```
roles/
  nginx/
    tasks/          ← The main logic (what to do)
    handlers/       ← Handlers triggered by tasks
    templates/      ← Jinja2 template files (.j2) — dynamic config files
    files/          ← Static files to copy as-is
    vars/           ← Role variables (higher priority — harder to override)
    defaults/       ← Default variable values (lowest priority — easily overridden)
    meta/           ← Role metadata (author, dependencies, supported platforms)
```

Ansible **automatically loads `main.yml`** from each of these directories. You never need to import or include them manually — the directory structure itself is the convention Ansible follows.

### Create a Role with ansible-galaxy

Instead of creating all these directories by hand:

```bash
cd ~/ansible-lab
ansible-galaxy init roles/nginx
```

`ansible-galaxy init` scaffolds the entire structure instantly. Verify it:

```bash
tree roles/nginx
```

```
roles/nginx/
├── defaults/
│   └── main.yml
├── files/
├── handlers/
│   └── main.yml
├── meta/
│   └── main.yml
├── tasks/
│   └── main.yml
├── templates/
└── vars/
    └── main.yml
```

### Filling In the Role

**`roles/nginx/defaults/main.yml`** — Default variables (easily overridden):
```yaml
---
nginx_port: 80
nginx_worker_processes: auto
nginx_index_content: "<h1>Served by Ansible-managed nginx</h1>"
```

These are your role's "configuration knobs" — sensible defaults that anyone using the role can override when they need different values.

**`roles/nginx/tasks/main.yml`** — The main logic:
```yaml
---
- name: Install nginx
  apt:
    name: nginx
    state: present
    update_cache: yes

- name: Deploy nginx config from template
  template:
    src: nginx.conf.j2               # From roles/nginx/templates/
    dest: /etc/nginx/nginx.conf
    owner: root
    group: root
    mode: "0644"
  notify: Restart nginx              # Trigger handler only if config changed

- name: Deploy custom index page
  copy:
    content: "{{ nginx_index_content }}"
    dest: /var/www/html/index.html
    owner: www-data
    group: www-data
    mode: "0644"

- name: Ensure nginx is started and enabled on boot
  service:
    name: nginx
    state: started
    enabled: yes
```

**`roles/nginx/handlers/main.yml`** — Handlers:
```yaml
---
- name: Restart nginx
  service:
    name: nginx
    state: restarted
```

**`roles/nginx/templates/nginx.conf.j2`** — A Jinja2 template:
```nginx
# Managed by Ansible — do not edit manually
# Any manual changes will be overwritten on the next playbook run

worker_processes {{ nginx_worker_processes }};

events {
    worker_connections 1024;
}

http {
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;

    server {
        listen {{ nginx_port }};
        server_name {{ inventory_hostname }};

        location / {
            root /var/www/html;
            index index.html;
        }

        access_log /var/log/nginx/access.log;
        error_log  /var/log/nginx/error.log;
    }
}
```

The `.j2` extension stands for **Jinja2** — Python's templating engine. Ansible processes this file before copying it to the target: every `{{ variable }}` is replaced with its actual value. The file that lands on the server has no template syntax — just the final config with real values filled in.

> 💡 **Why templates instead of `copy`?** Static files are the same for every server. Templates let you generate different config files per server based on variables — the hostname, port, memory size, or any other value. One template file handles infinite server variations.

### `defaults` vs `vars` — An Important Distinction

```
defaults/main.yml  →  Lowest priority. Designed to be overridden.
vars/main.yml      →  Higher priority. Stronger, harder to override.
```

Use `defaults` for values that role users *should* customize (ports, paths, content). Use `vars` for values that are internal implementation details that shouldn't normally be changed.

### Using the Role in a Playbook

```yaml
# site.yml
---
- name: Configure Web Servers
  hosts: webservers
  become: true
  roles:
    - nginx                     # Ansible looks in ./roles/nginx/ automatically

- name: Configure DB Servers
  hosts: dbservers
  become: true
  roles:
    - mysql                     # You'd create this separately
```

### Overriding Role Variables

```yaml
  roles:
    - role: nginx
      vars:
        nginx_port: 8080                          # Override default port 80
        nginx_index_content: "<h1>Staging</h1>"  # Override default content
```

Since `nginx_port` and `nginx_index_content` are in `defaults/`, they have the lowest priority — easily overridden here. This is the role's design: sensible defaults, fully customizable.

### Run the playbook:

```bash
ansible-playbook -i inventory.ini site.yml
```

### Ansible Galaxy — Community Roles

You don't always need to write roles from scratch. [galaxy.ansible.com](https://galaxy.ansible.com) is a marketplace of community-contributed roles — like npm for Ansible.

```bash
# Install a production-grade nginx role by Jeff Geerling
ansible-galaxy role install geerlingguy.nginx

# Install from a requirements file (team practice)
ansible-galaxy install -r requirements.yml
```

**`requirements.yml`**:
```yaml
---
roles:
  - name: geerlingguy.nginx
  - name: geerlingguy.mysql
  - name: geerlingguy.docker
```

Once installed, use it exactly like your own role:

```yaml
  roles:
    - geerlingguy.nginx
```

> 💡 **Jeff Geerling's roles** (`geerlingguy.*`) are some of the best-maintained community roles available. Study their source code — they're a masterclass in how production Ansible roles are structured.

> 📖 **Roles docs**: [Roles Guide](https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_reuse_roles.html) — covers role directory structure, variable precedence, role dependencies, and role sharing in depth.

---

## 8. The Full Picture — How Everything Connects

Now that you understand each piece, here's how they all fit together:

```
site.yml  (entry point — the orchestrator)
  │
  ├── Play 1: targets webservers
  │     └── Role: nginx
  │           ├── tasks/main.yml       ← What to do (install, configure, start)
  │           ├── handlers/main.yml    ← React to changes (restart on config change)
  │           ├── templates/           ← Dynamic config files (Jinja2)
  │           ├── files/               ← Static files to copy
  │           └── defaults/main.yml    ← Configurable defaults (port, paths, etc.)
  │
  └── Play 2: targets dbservers
        └── Role: mysql
              └── ... (same structure)
```

### Essential Playbook Commands

```bash
# Run the full playbook
ansible-playbook -i inventory.ini site.yml

# Dry run — see what WOULD change without making any changes
ansible-playbook -i inventory.ini site.yml --check

# Run with verbose output (great for debugging)
ansible-playbook -i inventory.ini site.yml -v      # verbose
ansible-playbook -i inventory.ini site.yml -vvv    # very verbose (full SSH output)

# Only run tasks with a specific tag
ansible-playbook -i inventory.ini site.yml --tags "config"

# Skip tasks with a specific tag
ansible-playbook -i inventory.ini site.yml --skip-tags "install"

# Only run against one specific host (even if the play targets a group)
ansible-playbook -i inventory.ini site.yml --limit ansible-target

# List all tasks that would run (without running them)
ansible-playbook -i inventory.ini site.yml --list-tasks
```

> 💡 **`--check` is your best friend** before running a playbook in production. It simulates the run and tells you exactly what would change — without touching anything.

---

## 9. Summary and What's Next

Here's everything you covered in a single view:

| Concept | What it is | Why it matters |
|---|---|---|
| **Agentless** | No software needed on target machines | Simple to adopt — just needs SSH + Python |
| **Idempotency** | Running twice = same result as running once | Safe to re-run, safe to automate |
| **Inventory** | The list of servers Ansible manages | Organize servers into groups, target them precisely |
| **Ad-hoc commands** | One-off tasks run from the terminal | Quick testing, emergency fixes |
| **Modules** | Reusable units of work (apt, service, copy, etc.) | The vocabulary Ansible uses to talk to servers |
| **Playbooks** | YAML files defining tasks to run on hosts | Your infrastructure as code |
| **Handlers** | Tasks that run only when notified by a change | Restart services only when config actually changed |
| **Variables** | Parameterize your playbooks | Reusable, flexible, environment-aware |
| **Facts** | Auto-collected data about target hosts | Enable conditionals and dynamic behavior |
| **Roles** | Reusable, self-contained task packages | Structure and reuse across projects |
| **Jinja2 Templates** | Dynamic config files with variables | One template, infinite server variations |
| **Ansible Galaxy** | Community role marketplace | Don't reinvent the wheel |

### What's Next After This?

Now that you have the fundamentals, here's where to go deeper:

- **Ansible Vault** — Encrypt sensitive data (passwords, API keys) inside your playbooks and roles. Never store secrets in plaintext.
- **Dynamic Inventories** — Instead of a static `inventory.ini`, pull your server list dynamically from AWS EC2, GCP, or Azure. Servers come and go — your inventory should too.
- **Ansible in CI/CD** — Trigger Ansible playbooks from GitHub Actions or Jenkins on every deployment. True infrastructure automation.
- **Testing with Molecule** — Test your Ansible roles in Docker containers before deploying to real servers. Catch bugs early.
- **Ansible with Terraform** — Terraform provisions the infrastructure (creates EC2 instances). Ansible configures what's on them. The two tools complement each other perfectly.

---

### The One-Line Summary

> **Ansible lets you describe what you want your servers to look like — and it makes them look that way. All of them. Every time. Safely.**

---

*This article is part of my ongoing DevOps learning series. Follow along at [From Code to Cloud](https://ashmitcodes.hashnode.dev) — documenting everything I learn, the way I wish someone had explained it to me.* 🚀
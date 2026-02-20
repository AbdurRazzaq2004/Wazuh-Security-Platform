# 🖥️ Wazuh Agent — The Eyes & Ears on Every Endpoint

## What is the Wazuh Agent?

The **Wazuh Agent** is a lightweight piece of software you **install on every machine you want to monitor** — whether it's a laptop, server, cloud VM, or container.

> **In simple words:** Imagine hiring a security guard for each of your computers. That guard watches everything happening on the machine — who's logging in, what files are being changed, what software is installed, what's running — and reports back to headquarters (the Wazuh Server). **That guard is the Wazuh Agent.**

---

## 📦 Where Can You Install It?

The Wazuh Agent runs on virtually **any operating system**:

| OS | Supported? |
|---|---|
| **Linux** (Ubuntu, CentOS, RHEL, Debian, etc.) | ✅ |
| **Windows** (Server & Desktop) | ✅ |
| **macOS** | ✅ |
| **Solaris** | ✅ |
| **AIX** | ✅ |
| **HP-UX** | ✅ |

It works on:
- 💻 Laptops & Desktops
- 🖥️ Physical Servers
- ☁️ Cloud Instances (AWS EC2, Azure VMs, GCP Compute)
- 🐳 Containers & Virtual Machines

---

## 🏗️ Agent Architecture

The agent has **two main parts**:

### 1. Agent Modules (The Data Collectors)
These are like **specialized sensors** — each one watches a different aspect of your system.

### 2. Agent Daemon (The Manager)
This is the **"brain"** of the agent. It:
- **Encrypts** all the data before sending it
- **Manages** which modules are active
- **Handles remote configuration** from the server
- **Authenticates** with the server using unique keys

![Wazuh Agent Architecture](https://documentation.wazuh.com/current/_images/agent-architecture1.png)

---

## 🧩 Agent Modules Explained (One by One)

Each module does a specific job. Think of them as **apps on your phone** — you can enable or disable them based on what you need.

### 📋 1. Log Collector
**What it does:** Reads log files from the operating system and applications.

> **Simple Example:** When someone tries to SSH into your Linux server and types the wrong password 10 times, the Log Collector reads that from `/var/log/auth.log` and sends it to the server. The server then says "Hey, that looks like a brute-force attack!" and raises an alert.

- Reads flat log files (e.g., `/var/log/syslog`)
- Reads Windows Event Logs
- Supports XPath filters for Windows events
- Can handle multi-line log formats (like Linux Audit logs)
- Enriches JSON events with extra metadata

---

### ⚡ 2. Command Execution
**What it does:** Runs specific commands on the machine at scheduled intervals and reports the output.

> **Simple Example:** You configure it to run `df -h` every 5 minutes to check disk space. If your disk hits 95% full, the Wazuh Server detects it and alerts you before your system crashes.

- Runs **only authorized commands** (security first!)
- Reports output back to the Wazuh Server
- Use cases: disk space, logged-in users, running services, etc.

---

### 📁 3. File Integrity Monitoring (FIM)
**What it does:** Watches your files and directories for any changes.

> **Simple Example:** An attacker modifies your `/etc/passwd` file to add a backdoor user. FIM detects this immediately and alerts: "Hey! Someone just changed `/etc/passwd` — here's what changed, who did it, and when."

- Detects file **creation**, **modification**, and **deletion**
- Tracks changes in **permissions**, **ownership**, and **content**
- Captures **who**, **what**, and **when** in real-time
- Can monitor **Windows Registry** keys too

---

### ✅ 4. Security Configuration Assessment (SCA)
**What it does:** Scans your system to check if it's configured securely, following industry best practices.

> **Simple Example:** CIS Benchmark says "SSH root login should be disabled." SCA checks your `sshd_config` and finds `PermitRootLogin yes`. It flags this as **FAILED** and tells you exactly how to fix it.

- Uses **CIS Benchmarks** out of the box
- You can create **custom checks** for your own policies
- Continuous scanning — not just a one-time check
- Shows pass/fail/not-applicable results on the dashboard

---

### 📊 5. System Inventory
**What it does:** Collects an inventory of everything on the system — like a detailed census.

> **Simple Example:** You want to know which servers are still running an old, vulnerable version of OpenSSL. System Inventory tells you exactly which machines have what versions installed.

Collects:
- **OS version** and hardware info
- **Network interfaces** and IP addresses
- **Running processes**
- **Installed applications** (with versions)
- **Open ports**
- Stores data in local **SQLite databases** for remote querying

---

### 🦠 6. Malware Detection
**What it does:** Looks for signs of malware using behavior-based detection (not just signatures).

> **Simple Example:** A rootkit hides itself by making its process invisible to `ps` command. Wazuh's malware detection module checks for **hidden processes**, **hidden files**, and **hidden ports** — finding threats that traditional antivirus might miss.

- **Non-signature-based** approach (catches zero-day threats)
- Detects **rootkits** at kernel and user level
- Finds **hidden processes**, **hidden files**, and **hidden ports**
- Monitors **system calls** for anomalies

---

### 🚨 7. Active Response
**What it does:** Automatically takes action when a threat is detected.

> **Simple Example:** Someone is brute-forcing your SSH. After 5 failed attempts, Active Response automatically **blocks their IP address** using iptables — without any human intervention.

Built-in responses:
- **Block** a network connection (firewall rules)
- **Stop** a running process
- **Delete** a malicious file

Custom responses (you build them):
- Run a file in a sandbox
- Capture network traffic
- Scan a file with an external antivirus

---

### 🐳 8. Container Security Monitoring
**What it does:** Monitors Docker containers for security issues.

> **Simple Example:** Someone runs a container in **privileged mode** (which is a huge security risk). The agent detects this and alerts: "Container XYZ is running with full host privileges!"

- Integrates with the **Docker Engine API**
- Detects changes to **container images**
- Monitors **network configuration** changes
- Alerts on **data volume** modifications
- Detects **users executing commands** inside running containers

---

### ☁️ 9. Cloud Security Monitoring
**What it does:** Monitors your cloud infrastructure (AWS, Azure, GCP).

> **Simple Example:** Someone creates a new IAM user in your AWS account at 3 AM. The cloud security module detects this via the AWS API and alerts you immediately.

- Communicates **natively** with cloud provider APIs
- Detects infrastructure changes:
  - New users created
  - Security groups modified
  - Instances stopped/started
- Collects cloud service logs:
  - **AWS CloudTrail**
  - **GCP Pub/Sub**
  - **Azure Active Directory**

---

## 🔒 Communication with Wazuh Server

The agent doesn't just send data in plain text — it's **secure by design**:

| Feature | Details |
|---|---|
| **Encryption** | AES-256 encryption (Blowfish optional) |
| **Protocol** | TCP or UDP (TCP by default) |
| **Default Port** | 1514/TCP |
| **Compression** | Real-time data compression |
| **Flow Control** | Prevents flooding, queues events when needed |
| **Authentication** | Unique per-agent encryption keys |

### Enrollment Process
Before an agent can connect for the first time, it must **enroll** with the server:
1. Agent sends enrollment request to server (port 1515)
2. Server generates a **unique key** for the agent
3. Agent uses this key for all future communications
4. All data is encrypted with this key

---

## 🎮 Managing Agents

Once enrolled, agents can be managed **remotely** from the Wazuh Server:

- ✅ **Upgrade** agents remotely
- ✅ **Change configuration** remotely
- ✅ **Monitor** agent status (active, disconnected, never connected)
- ✅ **Enable/disable** specific modules

---

## 💡 Key Takeaway

> The Wazuh Agent is your **first line of defense**. It sits on every endpoint, collects security-relevant data through its modular architecture, encrypts it, and ships it to the Wazuh Server for analysis. Without agents, you have no visibility. With agents, you see everything.

---

## 🔗 References
- Official Docs: https://documentation.wazuh.com/current/getting-started/components/wazuh-agent.html
- Installation Guide: https://documentation.wazuh.com/current/installation-guide/wazuh-agent/index.html

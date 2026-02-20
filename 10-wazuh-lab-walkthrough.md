# 🧪 Wazuh Lab Walkthrough — Complete Hands-On Guide

> **Lab Environment:** Wazuh 4.14.3 All-in-One (Single Node) deployed on AWS EC2 (Ubuntu 24.04.3 LTS)
>
> **Cloud Provider:** AWS — 2 × EC2 instances in `us-east-1` (N. Virginia): 1 Wazuh Server + 1 Agent
>
> **Purpose:** Install Wazuh, enroll agents, and explore every section of the Wazuh Dashboard.

---

## 📌 What This Lab Covers

1. SSH into an AWS EC2 Ubuntu instance
2. Install Wazuh using the quickstart script
3. Access the Wazuh Dashboard for the first time
4. Explore all Dashboard modules (Endpoint Security, IT Hygiene, MITRE ATT&CK, etc.)
5. Open AWS Security Group to allow agent communication (port 1514/1515)
6. Deploy a second EC2 instance and enroll it as a Wazuh Agent
7. Explore Vulnerability Detection, Configuration Assessment (SCA), and more

---

## Step 1 — SSH into the Ubuntu EC2 Instance

![SSH into Ubuntu Server](./images/1.png)

### What's Happening Here

We are **SSH-ing into our AWS EC2 Ubuntu instance** for the first time. This is the server where we will install Wazuh (all-in-one deployment).

You can see the typical Ubuntu SSH welcome message:

- `Enable ESM Apps to receive additional future security updates`
- `Ubuntu comes with ABSOLUTELY NO WARRANTY`
- System info like uptime, packages, etc.

### Why This Step

Before installing anything, we need terminal access to the server. We used:

```bash
ssh -i "your-key.pem" ubuntu@<ec2-public-ip>
```

This is our **Wazuh Server** — it will run the Wazuh Manager, Indexer, and Dashboard (all-in-one single node).

---

## Step 2 — Running the Wazuh Installation Script

![Wazuh Installation Script Running](./images/2.png)

### What's Happening Here

We are running the **Wazuh quickstart installation script**. The terminal shows the installation progress:

- `INFO: --- Dependencies ---` — Installing required packages (`apt-transport-https`, `debhelper`)
- `INFO: Wazuh repository added` — Wazuh package repo is now configured
- `INFO: --- Configuration files ---` — Generating config files
- `INFO: Generating the root certificate` — Creating TLS/SSL certificates
- `INFO: Generating Admin certificates` — Admin certs for dashboard access
- `INFO: Generating Wazuh indexer certificates` — Certs for indexer communication
- `INFO: Generating Wazuh dashboard certificates` — Dashboard certs

### Commands Used

```bash
curl -sO https://packages.wazuh.com/4.x/wazuh-install.sh
sudo bash ./wazuh-install.sh -a
```

The `-a` flag means **all-in-one** — it installs the Wazuh Manager, Indexer, and Dashboard on the same machine.

### What Gets Installed

| Component | What It Does |
|---|---|
| **Wazuh Manager** | Receives and analyzes events from agents |
| **Wazuh Indexer** | Stores alerts as searchable JSON documents |
| **Wazuh Dashboard** | Web UI for visualization (port 443) |

> ⚠️ After installation completes, it prints the **admin password**. Save it — you need it to log in!

---

## Step 3 — Wazuh Dashboard: First Login (No Agents Yet)

![Wazuh Dashboard Overview — No Agents](./images/3.png)

### What's Happening Here

We've opened the Wazuh Dashboard in the browser at `https://<ec2-public-ip>:443` and logged in with the admin credentials.

This is the **Overview page** — but notice the message:

> **"This instance has no agents registered. Please deploy agents to begin monitoring your endpoints."**

### What We See

| Element | Status |
|---|---|
| **Agents Summary** | 0 agents — none registered yet |
| **Critical severity** | Rule level 15 or higher — no events yet |
| **Endpoint Security modules** | Listed but empty (no data without agents) |

### Modules Visible (But Empty)

- **Configuration Assessment** — Scan your assets as part of a configuration assessment audit
- **Malware Detection** — Check indicators of compromise triggered by malware infections or cyberattacks
- **File Integrity Monitoring** — Alerts related to file changes, including permissions, content, ownership, and attributes

### Why It's Empty

The Wazuh Dashboard shows data **only from enrolled agents**. Since we just installed the server and haven't deployed any agents yet, everything is at zero. This is expected.

---

## Step 4 — Dashboard Modules Overview (Sidebar)

![Dashboard Modules — Endpoint Security & Security Operations](./images/4.png)

### What's Happening Here

We're exploring the **left sidebar / modules panel** of the Wazuh Dashboard. This shows all available security modules organized by category:

**Endpoint Security:**

| Module | Description |
|---|---|
| **Configuration Assessment** | Scan your assets as part of a configuration assessment audit (CIS benchmarks) |
| **File Integrity Monitoring** | Alerts related to file changes — permissions, content, ownership, and attributes |
| **Malware Detection** | Check indicators of compromise triggered by malware infections or cyberattacks |

**Security Operations:**

| Module | Description |
|---|---|
| **IT Hygiene** | Assess system, software, processes, and network layers |
| **GDPR** | General Data Protection Regulation — processing of personal data |
| **NIST 800-53** | US federal security controls framework |
| And more... | PCI DSS, HIPAA, TSC compliance modules |

> 💡 These modules will remain empty until we deploy agents and enable the corresponding sections in `ossec.conf`.

---

## Step 5 — Discover Tab: Wazuh Alerts Index Pattern

![Discover Tab — wazuh-alerts index](./images/5.png)

### What's Happening Here

We're in the **Discover** tab (similar to OpenSearch/Kibana Discover). This is the raw data explorer.

### Key Elements Visible

| Element | What It Shows |
|---|---|
| **Index patterns** | `wazuh-alerts-*`, `wazuh-monitoring-*`, `wazuh-statistics-*` |
| **Search bar** | DQL (Dashboard Query Language) for searching alerts |
| **Time range** | `Last 24 hours` with date picker |
| **Hits** | `404 hits` — total matching documents in this time window |
| **Available fields** | Fields like `index`, `agent.id`, `data.command` that you can add as columns |

### What Are These Index Patterns?

| Index Pattern | What It Stores |
|---|---|
| `wazuh-alerts-*` | All security alerts generated by rules |
| `wazuh-monitoring-*` | Agent connection/status monitoring data |
| `wazuh-statistics-*` | Performance and statistics data |

This is where SOC analysts do their **raw investigation** — searching through alerts, filtering by fields, and exporting data.

---

## Step 6 — Threat Hunting: Authentication Failure Events

![Threat Hunting — Authentication Failures](./images/6.png)

### What's Happening Here

We're in the **Threat Hunting** module, filtered to show **Authentication failure** events.

### Key Elements Visible

| Element | What It Shows |
|---|---|
| **Module** | Threat Hunting |
| **Filter applied** | `Authentication failure` |
| **Manager name** | `ip-172-31-28-40` (our Wazuh server) |
| **Time range** | Last 24 hours |
| **Dashboard + Events tabs** | Dashboard shows charts, Events shows raw logs |
| **Hits** | 78 authentication failure events detected |
| **Columns** | Timestamp, agent.name, rule.description, rule.level, rule.id |

### Why Authentication Failures Matter

These events indicate someone (or something) tried to log in and **failed**. This could mean:

- **Brute force attacks** — someone trying many passwords
- **Misconfigured services** — wrong credentials in automation
- **Unauthorized access attempts** — someone trying to break in

> This is one of the most common alert types in Wazuh. In a real SOC, you'd investigate the **source IP**, check how many attempts there were, and potentially block the attacker using Active Response.

---

## Step 7 — Deploy New Agent: OS Selection

![Deploy New Agent — Select OS](./images/7.png)

### What's Happening Here

We're deploying our **first Wazuh Agent**. The Dashboard provides a built-in agent deployment wizard.

### Step 1: Select the Operating System

The wizard shows three options:

| OS | Package Types |
|---|---|
| **LINUX** | RPM (amd64), RPM (aarch64), DEB (amd64), DEB (aarch64) |
| **WINDOWS** | MSI (32/64 bits) |
| **macOS** | Intel, Apple Silicon |

Since our second EC2 instance runs **Ubuntu (Debian-based)**, we select **LINUX → DEB amd64**.

### Server Address Visible

The wizard asks for a **server address** — this is the Wazuh Manager IP. We can see `34.224.212.65` entered (public IP at that point).

---

## Step 8 — Deploy New Agent: Server Address & Package Selection

![Deploy New Agent — Package Selection](./images/8.png)

### What's Happening Here

Continuing the agent deployment wizard — we've selected **Linux** and now we're confirming the specific package type and server address:

- **DEB amd64** — For Ubuntu, Debian ✅ (this is what we need)
- **Server address:** `98.81.179.28` (the Wazuh Manager's public IP)

### Why the IP Changed

In Step 7, the IP was `34.224.212.65`, and now it's `98.81.179.28`. This happens when you **stop and restart** an EC2 instance — AWS assigns a new public IP (unless you use an Elastic IP).

---

## Step 9 — Deploy New Agent: Installation Command

![Deploy New Agent — Install Command](./images/9.png)

### What's Happening Here

The wizard generates the **exact command** to run on the agent machine:

```bash
wget https://packages.wazuh.com/4.x/apt/pool/main/w/wazuh-agent/wazuh-agent_4.14.3-1_amd64.deb && \
sudo WAZUH_MANAGER='98.81.179.28' \
     WAZUH_AGENT_GROUP='default' \
     WAZUH_AGENT_NAME='ubuntu' \
     dpkg -i ./wazuh-agent_4.14.3-1_amd64.deb
```

### Breaking Down the Command

| Part | What It Does |
|---|---|
| `wget ...wazuh-agent_4.14.3-1_amd64.deb` | Downloads the Wazuh Agent package |
| `WAZUH_MANAGER='98.81.179.28'` | Sets the Wazuh Server IP (our EC2 public IP) |
| `WAZUH_AGENT_GROUP='default'` | Assigns the agent to the `default` group |
| `WAZUH_AGENT_NAME='ubuntu'` | Names this agent "ubuntu" |
| `dpkg -i ./wazuh-agent...deb` | Installs the downloaded package |

### Configuration Visible

- **Group selected:** `default`
- **Requirements note:** Administrator privileges needed, Bash shell required

> ⚠️ **Important:** For the agent to connect, port **1514** and **1515** must be open on the Wazuh Server's Security Group. That's what we do next!

---

## Step 10 — AWS Security Group: Inbound Rules

![AWS EC2 Security Group — Inbound Rules](./images/10.png)

### What's Happening Here

We're in the **AWS EC2 Console → Security Groups** section. This is the firewall configuration for our Wazuh Server EC2 instance.

### Security Group Details

| Field | Value |
|---|---|
| **Security Group Name** | `launch-wizard-1` |
| **Security Group ID** | `sg-0409dfe0b07ea77dd` |
| **VPC ID** | `vpc-0f92627c7e7f23e9c` |
| **Inbound Rules Count** | 6 permission entries |
| **Outbound Rules Count** | 1 permission entry |

### Why This Is Critical

For the Wazuh Agent to connect to the Wazuh Server, the following ports **must be open** in the inbound rules:

| Port | Protocol | Purpose |
|---|---|---|
| **1514** | TCP | Agent event data transmission |
| **1515** | TCP | Agent enrollment/registration |
| **443** | TCP | Dashboard web access (HTTPS) |
| **22** | TCP | SSH access |

> If you skip this step, the agent will fail to connect with a "Connection refused" error. This is the **#1 most common issue** when deploying agents on AWS.

---

## Step 11 — AWS EC2: Two Instances Running

![AWS EC2 Instances — 2 Instances Running](./images/11.png)

### What's Happening Here

We're in the **AWS EC2 Instances** dashboard. We can see **2 instances** running:

| Instance | Role |
|---|---|
| **Instance 1 (wazuh)** | Wazuh Server (Manager + Indexer + Dashboard) — `t2.medium` |
| **Instance 2** | Wazuh Agent (the endpoint we're monitoring) |

### Columns Visible

| Column | What It Shows |
|---|---|
| **Name** | Instance name tag (`wazuh`) |
| **Instance State** | Running ✅ |
| **Instance Type** | `t2.medium` |
| **Status Check** | `2/2 checks passed` |
| **Availability Zone** | `us-east-1` (N. Virginia) |
| **Public IPv4 DNS** | Public address to connect |

> 💡 This is our lab setup: **2 EC2 instances** in the same VPC. One runs Wazuh all-in-one, the other runs the Wazuh Agent.

---

## Step 12 — Wazuh Dashboard: First Agent Enrolled

![Wazuh Agents — 1 Agent Active](./images/12.png)

### What's Happening Here

Back in the Wazuh Dashboard → **Endpoints** section. Now we can see our **first agent has enrolled successfully!**

### Agent Details

| Field | Value |
|---|---|
| **Agent ID** | `001` |
| **Name** | `ip-172-31-27-237` |
| **Status** | 🟢 Active |
| **IP Address** | `172.31.27.237` |
| **Group** | `default` |
| **Operating System** | Ubuntu 24.04.3 LTS |
| **Cluster Node** | `node01` |
| **Version** | `v4.14.3` |

### Agents by Status

| Status | Count |
|---|---|
| 🟢 Active | 1 |
| 🔴 Disconnected | 0 |
| ⚫ Never Connected | 0 |

### What Changed

Earlier in Step 3, the dashboard said "No agents registered." Now after:

1. Opening the Security Group ports (Step 10)
2. Running the install command on the second EC2 (Step 9)
3. Starting the agent service (`sudo systemctl start wazuh-agent`)

...the agent connected and appears as **Active** ✅

---

## Step 13 — Individual Agent Dashboard

![Agent Details — ip-172-31-27-237](./images/13.png)

### What's Happening Here

We clicked on agent `001` to view its **individual dashboard**. This shows everything about this specific endpoint.

### Agent Information

| Field | Value |
|---|---|
| **ID** | 001 |
| **Status** | 🟢 Active |
| **IP Address** | 172.31.27.237 |
| **Version** | Wazuh v4.14.3 |
| **Group** | default |
| **Operating System** | Ubuntu 24.04.3 LTS |
| **Cluster Node** | node01 |
| **Registration Date** | Feb 20, 2026 @ 11:10:20 |
| **Last Keep Alive** | Feb 20, 2026 @ 11:18:20 |

### System Inventory (Visible)

| Field | Value |
|---|---|
| **Cores** | 2 |
| **Memory** | 3.868 GB |
| **CPU** | Intel(R) Xeon(R) |

### Available Tabs for This Agent

| Tab | What It Shows |
|---|---|
| **Threat Hunting** | Security alerts for this agent |
| **File Integrity Monitoring** | File change events on this agent |
| **Configuration Assessment** | SCA/CIS benchmark results |
| **MITRE ATT&CK** | ATT&CK technique mappings |
| **Vulnerability Detection** | CVEs found on this agent |
| **Stats** | Agent performance statistics |
| **Configuration** | Current agent configuration |

---

## Step 14 — AWS EC2: Managing Instances

![AWS EC2 — Instances Dashboard Sidebar](./images/14.png)

### What's Happening Here

We're back in the **AWS EC2 Console**, viewing the EC2 sidebar navigation and instances list. This shows the full range of AWS EC2 management sections.

### EC2 Sidebar Sections Visible

| Section | Purpose |
|---|---|
| **Dashboard** | EC2 overview |
| **Instances** | Running instances |
| **Instance Types** | Available machine types |
| **Launch Templates** | Saved instance configurations |
| **Spot Requests** | Spot instance management |
| **Savings Plans** | Cost optimization |
| **Reserved Instances** | Long-term reservations |
| **Dedicated Hosts** | Dedicated hardware |
| **Capacity Reservations** | Reserved capacity |
| **AMIs / AMI Catalog** | Machine images |
| **Volumes / Snapshots** | EBS storage management |
| **Security Groups** | Firewall rules |
| **Elastic IPs** | Static public IPs |
| **Key Pairs** | SSH key management |

> We're preparing to launch/manage the second EC2 instance for the second Wazuh Agent.

---

## Step 15 — Wazuh Dashboard: Two Agents Enrolled

![Wazuh Agents — 2 Agents](./images/15.png)

### What's Happening Here

Now we have **2 agents** enrolled in Wazuh:

| Agent ID | Name | Status |
|---|---|---|
| **001** | `ip-172-31-27-237` | 🟢 Active |
| **002** | `test-server2` | ⚫ Never Connected |

### Agents by Status

| Status | Count |
|---|---|
| 🟢 Active | 1 |
| ⚫ Never Connected | 1 |

### Top 5 OS / Groups

- **OS:** `ubuntu` (1 agent reporting)
- **Groups:** `default` (2 agents assigned)

> Agent 002 (`test-server2`) has been enrolled but hasn't established its first connection yet (status: **Never Connected**). Once we start the agent service on that machine, it will switch to Active.

---

## Step 16 — Agents Detailed List View

![Agents Detailed List — Both Active](./images/16.png)

### What's Happening Here

The **expanded agents table** showing full details for both agents — now **both are active**:

| Field | Agent 001 | Agent 002 |
|---|---|---|
| **Name** | ip-172-31-27-237 | test-server2 |
| **IP Address** | 172.31.27.237 | 172.31.25.172 |
| **Group** | default | default |
| **OS** | Ubuntu 24.04.3 LTS | Ubuntu 24.04.3 LTS |
| **Cluster Node** | node01 | node01 |
| **Version** | v4.14.3 | v4.14.3 |
| **Status** | 🟢 Active | 🟢 Active |

### Key Takeaway

Both agents are:
- Same OS (Ubuntu 24.04.3 LTS)
- Same Wazuh version (v4.14.3)
- In the same group (`default`)
- Both reporting to the same cluster node (`node01`)
- Both have action buttons for management (restart, upgrade, etc.)

---

## Step 17 — Configuration Assessment (SCA) Dashboard

![Configuration Assessment — test-server2](./images/17.png)

### What's Happening Here

We're in the **Configuration Assessment (SCA)** module for `test-server2`. This module runs **CIS benchmark scans** to check if the system is configured securely.

### What We See

| Element | Description |
|---|---|
| **Dashboard tab** | Overview charts and metrics |
| **Inventory tab** | Detailed check-by-check results |
| **Events tab** | Raw SCA events ← We're here |
| **Filter** | `manager.name: ip-172-31-17-228` and `rule.groups: sca` and `agent.id: 002` |
| **Time range** | Last 24 hours |
| **Hits** | 280 SCA events |

### Columns Visible

- **Timestamp** — When the SCA check ran
- **data.sca.check.title** — What's being checked
- **data.sca.check.file** — Config file being evaluated

> SCA scans run automatically when the agent is enrolled. It checks the system against CIS benchmarks and reports which checks **passed**, **failed**, or are **not applicable**.

---

## Step 18 — Dashboard Overview with 2 Active Agents

![Wazuh Overview — 2 Active Agents](./images/18.png)

### What's Happening Here

Back to the **Overview** page. Now the dashboard has data because we have **2 active agents**:

### Agents Summary

| Status | Count |
|---|---|
| 🟢 Active | 2 |
| 🔴 Disconnected | 0 |

### Modules Listed

Now the Endpoint Security and Security Operations modules are visible and contain data:

- **Configuration Assessment** — SCA scan results available
- **File Integrity Monitoring** — File change events being collected
- **Malware Detection** — Rootcheck running
- **IT Hygiene** — System inventory collecting
- **GDPR** — Compliance mappings active

> Compare this to Step 3 — the dashboard was empty. Now with 2 agents sending data, all modules are populated.

---

## Step 19 — IT Hygiene: System Dashboard

![IT Hygiene — Dashboard, Platforms](./images/19.png)

### What's Happening Here

We're exploring the **IT Hygiene** module (System Inventory). This shows the state of all monitored endpoints.

### Tabs Available

| Tab | What It Shows |
|---|---|
| **Dashboard** | Overview with charts |
| **OS / Hardware** | Operating system and hardware info |
| **Platform** | Platform distribution ← We're here |
| **System** | System-level details |
| **Software** | Installed packages |
| **Processes** | Running processes |
| **Network** | Network interfaces and connections |
| **Identity** | User/group information |
| **Services** | Running services |

### What We See

| Element | Value |
|---|---|
| **Cluster** | `ip-172-31-17-228` |
| **Top 5 Platforms** | `ubuntu` (both agents run Ubuntu) |
| **Agents listed** | `ip-172-31-27-237` and `test-server2` |
| **OS Details** | Ubuntu 24.04.3 LTS (Noble Numbat), Kernel 6.14.0-1018-aws |

> This data is collected by the `<syscollector>` module in `ossec.conf`. It inventories all software, hardware, network, and processes on each agent.

---

## Step 20 — IT Hygiene: Network Tab

![IT Hygiene — Network (Ports & Destinations)](./images/20.png)

### What's Happening Here

We're in the **IT Hygiene → Network** tab. This shows network-related inventory data.

### Tabs in Network Section

`Addresses` → `Interfaces` → `Protocols` → `Listeners` → `Traffic`

### What We See

| Element | Details |
|---|---|
| **Filter** | `NOT destination_port:0` (filtering out port 0 noise) |
| **Top 5 Destination Ports** | **443** (HTTPS), **1514** (Wazuh agent comm), **80** (HTTP), **34007** |

### Why Ports 443 and 1514?

| Port | Why It Appears |
|---|---|
| **443** | Agents/browsers connecting to the Wazuh Dashboard |
| **1514** | Agents sending event data to the Wazuh Manager |

### Agent Network Data

- `test-server2` at `172.31.25.172` — multiple connections visible
- `ip-172-31-27-237` at `172.31.27.237` — connections to various ports

These are the two most critical ports in a Wazuh deployment. Seeing them here confirms the agents are communicating properly.

---

## Step 21 — IT Hygiene: Services Tab

![IT Hygiene — Services](./images/21.png)

### What's Happening Here

We're in the **IT Hygiene → Services** tab. This shows all running services discovered on the monitored agents.

### What We See

| Element | Details |
|---|---|
| **Filter** | `wazuh.cluster.name: ip-172-31-17-228` |
| **Top 5 Services** | `apport` and others |
| **Total Unique Services** | 179 services discovered |
| **Total Hits** | 356 service entries |

### Services Visible

- `ModemManager` — Network modem management
- `systemd-tpm2-setup-early` — TPM security setup
- `networking` — Network configuration
- `ldconfig` — Library cache management

### Why This Matters

- **Unexpected services** = potential security risk (backdoor, unauthorized software)
- **Missing services** = something critical might not be running
- Useful for **compliance audits** — verify only approved services are running

---

## Step 22 — MITRE ATT&CK: Dashboard

![MITRE ATT&CK — Dashboard & Tactics](./images/22.png)

### What's Happening Here

We're in the **MITRE ATT&CK** module — **Dashboard** tab. This maps Wazuh alerts to the MITRE ATT&CK framework.

### What We See

| Element | Description |
|---|---|
| **Alerts evolution over time** | Timeline chart showing ATT&CK-tagged alerts |
| **Top tactics** | Valid Accounts, Defense Evasion, Privilege Escalation, Persistence, Initial Access |
| **Filter** | `manager.name: ip-172-31-17-228` and `rule.mitre.id: exists` |
| **Time range** | Last 24 hours |

### Top Tactics Detected

| Tactic | What It Means |
|---|---|
| **Valid Accounts** | Attackers using legitimate credentials |
| **Defense Evasion** | Techniques to avoid detection |
| **Sudo and Sudo Caching** | Privilege escalation via sudo |
| **Create Account** | New accounts being created |
| **Disable or Modify Tools** | Security tools being tampered with |
| **Persistence** | Maintaining access after initial compromise |
| **Privilege Escalation** | Gaining higher-level permissions |
| **Initial Access** | First entry point into the system |

### Additional Sections

- **Attacks by technique** — detailed technique breakdown
- **Top tactics by agent** — which agents trigger which tactics
- **MITRE techniques by agent** — per-agent technique mapping

> Every Wazuh rule can be mapped to MITRE ATT&CK tactics and techniques. This turns isolated alerts into a **coherent attack narrative**.

---

## Step 23 — MITRE ATT&CK: Intelligence Tab

![MITRE ATT&CK — Intelligence (Groups, Mitigations, Software)](./images/23.png)

### What's Happening Here

We're exploring the **MITRE ATT&CK → Intelligence** tab. This is a built-in reference database of threat intelligence.

### Sections Available

| Section | What It Contains |
|---|---|
| **Groups** | 150+ known threat actor groups |
| **Mitigations** | Recommended countermeasures for each technique |
| **Software** | Known malware and tools used by threat actors |
| **Tactics** | The 14 ATT&CK tactic categories |
| **Techniques** | Specific attack techniques with descriptions |

### Groups Visible (150 threat actor groups)

| Group ID | Name | Description |
|---|---|---|
| G0018 | admin@338 | China-based cyber threat group targeting financial/economic orgs |
| G0130 | Ajax Security Team | Iranian threat group |
| G0138 | Andariel | North Korean sub-group of Lazarus |
| G1007 | Aoqin Dragon | Chinese-linked espionage group |

### Why This Is Useful

You can **search for any threat group** and see:
- What techniques they use
- What software/malware they deploy
- What mitigations defend against them

> This is built into Wazuh — no extra tools needed. It's like having a MITRE ATT&CK encyclopedia inside your SIEM.

---

## Step 24 — MITRE ATT&CK: Events Tab

![MITRE ATT&CK — Events](./images/24.png)

### What's Happening Here

We're in the **MITRE ATT&CK → Events** tab. This shows the **raw alert data** for events that matched MITRE ATT&CK rules.

### What We See

| Element | Description |
|---|---|
| **Filter** | `manager.name: ip-172-31-17-228` and `rule.mitre.id: exists` |
| **Framework tab** | Currently viewing the events under Framework |
| **Columns** | Timestamp, agent.name |
| **Export** | `Export Formatted` button to download results |
| **Timestamps** | Events from `Feb 20, 2026 @ 10:55` to `11:21` |

### Agents in Events

- `ip-172-31-27-237` (Agent 001)
- `ip-172-31-17-228` (Wazuh Manager itself)

Each row is an individual alert that has been tagged with a MITRE ATT&CK technique ID. You can click any row to see the full JSON with:
- Rule ID and description
- MITRE technique and tactic
- Agent details
- Raw log data

---

## Step 25 — Vulnerability Detection: Dashboard

![Vulnerability Detection — Dashboard Overview](./images/25.png)

### What's Happening Here

We're in the **Vulnerability Detection** module — **Dashboard** tab. This shows all known CVEs (Common Vulnerabilities and Exposures) detected on our agents.

### Vulnerability Counts

| Severity | Count |
|---|---|
| 🔴 **Critical** | 18 |
| 🟠 **High** | 344 |
| 🟡 **Medium** | 758 |
| 🔵 **Low** | 134 |
| ⚪ **Pending Evaluation** | 1,566 |

### Top Sections

| Section | What It Shows |
|---|---|
| **Top 5 Vulnerabilities** | `CVE-2022-3219` (22 occurrences), `CVE-2025-68972` (22), `CVE-2025-68973` |
| **Top OS** | `Ubuntu 24.04.3 LTS (Noble Numbat)` — 2,820 vulnerabilities |
| **Top 5 Agents** | `test-server2` — 1,420 vulnerabilities, `ip-172-31-27-237` — 1,400 |
| **Top 5 Packages** | `linux-aws` (2,352), `glib-2.0` (20) |

### How It Works

1. The `<syscollector>` module inventories all installed packages
2. The `<vulnerability-detection>` module cross-references package versions against CVE databases
3. Results appear here with severity ratings

> This is powered by `<vulnerability-detection><enabled>yes</enabled></vulnerability-detection>` in `ossec.conf`.

---

## Step 26 — Vulnerability Detection: Events (Filtered by Agent)

![Vulnerability Detection — Events for test-server2](./images/26.png)

### What's Happening Here

We're in the **Vulnerability Detection → Events** tab, filtered to show vulnerabilities for **test-server2** specifically.

### What We See

| Element | Description |
|---|---|
| **Filter** | `agent.name: test-server2` and `Evaluated / Under evaluation` |
| **Total hits** | 1,420 vulnerabilities on this agent |
| **Columns** | Agent name, Package name, Package version, Vulnerability description, Severity, Vulnerability ID |

### Sample Vulnerability Visible

- **Package:** `libgnutls30t64` version `3.8.3-1.1ubuntu3.4`
- **Description:** "A flaw was found in the GnuTLS library..."
- **Agent:** `test-server2`

### Why This View Matters

This is where you do **per-agent vulnerability assessment**:
- Which packages on this specific server are vulnerable?
- What's the severity of each vulnerability?
- What should be patched first? (Critical → High → Medium → Low)

> 💡 **Pro Tip:** Filter by `vulnerability.severity: Critical` to focus on the most dangerous ones first.

---

## Step 27 — Configuration Assessment: CIS Benchmark Results

![Configuration Assessment — CIS Ubuntu 24.04 Benchmark](./images/27.png)

### What's Happening Here

We're in the **Configuration Assessment (SCA)** module for `test-server2`, viewing the **CIS Ubuntu Linux 24.04 LTS Benchmark v1.0.0** results.

### Scan Results

| Status | Count |
|---|---|
| ✅ **Passed** | 120 checks |
| ❌ **Failed** | 117 checks |
| ⬜ **Not Applicable** | 42 checks |

### Compliance Score

With 120 passed out of 237 applicable checks, the score is approximately **50%**.

### What This Means

- **120 passed** — these security configurations are correctly set
- **117 failed** — these are **misconfigurations** that need to be fixed for hardening
- **42 not applicable** — checks that don't apply to this system

### Sample Check Visible

- **ID:** 35500
- **Title:** "Ensure mounting of cramfs filesystems is disabled"
- **Policy:** CIS Ubuntu Linux 24.04 LTS Benchmark v1.0.0

### What To Do With Failed Checks

Click on any **failed check** to see:
- **What's wrong** — description of the misconfiguration
- **Rationale** — why it matters for security
- **Remediation** — exact commands to fix it
- **Compliance mapping** — which standards it violates (PCI DSS, HIPAA, etc.)

---

## Step 28 — Configuration Assessment: Events Tab

![Configuration Assessment — Events](./images/28.png)

### What's Happening Here

We're in the **Configuration Assessment → Events** tab for `test-server2`. This shows the **raw SCA events** as a timeline.

### What We See

| Element | Description |
|---|---|
| **Filter** | `manager.name: ip-172-31-17-228` and `rule.groups: sca` and `agent.id: 002` |
| **Columns** | Timestamp, SCA event details |
| **Timeline** | Events plotted over time showing when SCA scans ran |
| **Timestamps** | Events clustered around `Feb 20, 2026 @ 11:22` |

### How SCA Events Work

Every time the SCA module runs a scan (by default every 12 hours), it generates events for each check:
- Check passed → event logged
- Check failed → event logged with alert
- Check status changed (was passing, now failing) → higher priority alert

> These events feed into the SCA Dashboard (Step 27) to calculate the pass/fail/not-applicable counts.

---

---

# 📋 Lab Summary

## What We Did (Complete Flow)

### Phase 1: Infrastructure Setup (AWS)

| Step | Screenshot | What We Did |
|---|---|---|
| 1 | `1.png` | SSH into Ubuntu EC2 instance |
| 2 | `2.png` | Ran Wazuh quickstart installation script |
| 3 | `3.png` | Accessed Dashboard — no agents yet |
| 4 | `4.png` | Explored Dashboard modules (sidebar) |

### Phase 2: Initial Dashboard Exploration

| Step | Screenshot | What We Did |
|---|---|---|
| 5 | `5.png` | Explored Discover tab — `wazuh-alerts-*` index patterns, 404 hits |
| 6 | `6.png` | Threat Hunting — 78 authentication failure events |

### Phase 3: Agent Deployment

| Step | Screenshot | What We Did |
|---|---|---|
| 7 | `7.png` | Deploy Agent wizard — selected Linux DEB amd64, server `34.224.212.65` |
| 8 | `8.png` | Confirmed package selection, updated server IP to `98.81.179.28` |
| 9 | `9.png` | Got the install command with `WAZUH_MANAGER='98.81.179.28'` |
| 10 | `10.png` | Configured AWS Security Group `sg-0409dfe0b07ea77dd` (6 inbound rules) |
| 11 | `11.png` | Verified 2 EC2 instances running (`wazuh` = t2.medium, 2/2 checks passed) |
| 12 | `12.png` | First agent enrolled — `001 ip-172-31-27-237` active |
| 13 | `13.png` | Viewed individual agent details (2 cores, 3.868 GB RAM, Intel Xeon) |

### Phase 4: Second Agent Deployment

| Step | Screenshot | What We Did |
|---|---|---|
| 14 | `14.png` | AWS EC2 console — managing instances (full sidebar visible) |
| 15 | `15.png` | Second agent enrolled — `002 test-server2` (never connected) |
| 16 | `16.png` | Verified both agents active — `172.31.27.237` and `172.31.25.172` |

### Phase 5: Security Module Exploration

| Step | Screenshot | What We Did |
|---|---|---|
| 17 | `17.png` | SCA events for test-server2 — 280 hits, CIS checks |
| 18 | `18.png` | Overview page — 2 active agents with data |
| 19 | `19.png` | IT Hygiene — system/platforms, Ubuntu 24.04.3 LTS, Kernel 6.14.0-1018-aws |
| 20 | `20.png` | IT Hygiene — network (top ports: 443, 1514, 80, 34007) |
| 21 | `21.png` | IT Hygiene — services (179 unique services, 356 total entries) |
| 22 | `22.png` | MITRE ATT&CK — dashboard, top tactics: Valid Accounts, Defense Evasion |
| 23 | `23.png` | MITRE ATT&CK — intelligence (150+ threat groups: admin@338, Ajax Security Team, Andariel) |
| 24 | `24.png` | MITRE ATT&CK — raw events with timestamps |
| 25 | `25.png` | Vulnerability Detection — 18 Critical, 344 High, 758 Medium, 134 Low, 1566 Pending |
| 26 | `26.png` | Vulnerability events for test-server2 — 1,420 hits, libgnutls30t64 flaw |
| 27 | `27.png` | SCA CIS Ubuntu 24.04 Benchmark — 120 pass / 117 fail / 42 N/A (50% score) |
| 28 | `28.png` | SCA events timeline — clustered around 11:22 AM |

---

## Key Takeaways

1. **Wazuh needs agents to show data** — an empty dashboard just means no agents are enrolled yet
2. **AWS Security Groups are critical** — port 1514/1515 must be open for agents to connect
3. **Agent deployment is simple** — the Dashboard wizard generates the exact install command
4. **IT Hygiene gives full inventory** — packages, ports, services, network interfaces
5. **MITRE ATT&CK is built-in** — alerts are automatically mapped to tactics and techniques
6. **Vulnerability Detection found 1,000+ CVEs** — even a fresh Ubuntu server has hundreds of known vulnerabilities
7. **CIS Benchmark score ~50%** — a default Ubuntu installation fails about half the security checks
8. **Everything connects back to `ossec.conf`** — modules must be enabled in config to generate data

---

## References

- Wazuh Quick Start: https://documentation.wazuh.com/current/quickstart.html
- Wazuh Agent Deployment: https://documentation.wazuh.com/current/installation-guide/wazuh-agent/index.html
- Wazuh Dashboard: https://documentation.wazuh.com/current/getting-started/components/wazuh-dashboard.html
- Capabilities: https://documentation.wazuh.com/current/user-manual/capabilities/index.html
- ossec.conf Reference: https://documentation.wazuh.com/current/user-manual/reference/ossec-conf/index.html

# 🧪 Wazuh Lab Walkthrough — Complete Hands-On Guide# 🧪 Wazuh Lab Walkthrough — Complete Hands-On Guide# 🧪 Wazuh Dashboard Lab Walkthrough — Complete Hands-On Guide



> **Lab Environment:** Wazuh 4.14.3 All-in-One (Single Node) deployed on AWS EC2 (Ubuntu 24.04.3 LTS)



> **Cloud Provider:** AWS — 2 × EC2 instances in `us-east-1` (N. Virginia): 1 Wazuh Server + 1 Agent> **Lab Environment:** Wazuh All-in-One (Single Node) deployed on AWS EC2 (Ubuntu)> **Lab Environment:** Wazuh All-in-One (Single Node) Deployment



> **Purpose:** Install Wazuh, enroll agents, and explore every section of the Wazuh Dashboard.> **Purpose:** Install Wazuh, enroll agents, and explore every section of the Wazuh Dashboard.> **Purpose:** Explore every section of the Wazuh Dashboard, understand what each module does, and learn how to navigate as a SOC Analyst / DevOps Engineer.



---> **Cloud Provider:** AWS (2 × EC2 instances — 1 Wazuh Server + Agents)



## 📌 What This Lab Covers---



1. SSH into an AWS EC2 Ubuntu instance---

2. Install Wazuh using the quickstart script

3. Access the Wazuh Dashboard for the first time## 📌 What This Lab Covers

4. Explore all Dashboard modules (Endpoint Security, IT Hygiene, MITRE ATT&CK, etc.)

5. Open AWS Security Group to allow agent communication (port 1514/1515)## 📌 What This Lab Covers

6. Deploy a second EC2 instance and enroll it as a Wazuh Agent

7. Explore Vulnerability Detection, Configuration Assessment (SCA), and moreIn this lab, we installed Wazuh on a single node, enrolled agents, configured `ossec.conf` to enable key security modules, and then explored **every major section** of the Wazuh Dashboard. Below is a walkthrough of each step with screenshots.



---1. SSH into an AWS EC2 Ubuntu instance



---2. Install Wazuh using the quickstart script---



## Step 1 — SSH into the Ubuntu EC2 Instance3. Access the Wazuh Dashboard for the first time



![SSH into Ubuntu Server](./images/1.png)4. Explore all Dashboard modules (Endpoint Security, IT Hygiene, MITRE ATT&CK, etc.)---



### What's Happening Here5. Open AWS Security Group to allow agent communication (port 1514/1515)



We are **SSH-ing into our AWS EC2 Ubuntu instance** for the first time. This is the server where we will install Wazuh (all-in-one deployment).6. Deploy a second EC2 instance and enroll it as a Wazuh Agent## 🖥️ Step 1 — Wazuh Dashboard Login & Overview



You can see the typical Ubuntu SSH welcome message:7. Explore Vulnerability Detection, Configuration Assessment (SCA), and more



- `Enable ESM Apps to receive additional future security updates`![Wazuh Dashboard Login / Overview](./images/1.png)

- `Ubuntu comes with ABSOLUTELY NO WARRANTY`

- System info like uptime, packages, etc.---



### Why This Step### What We're Seeing



Before installing anything, we need terminal access to the server. We used:---When you first access the Wazuh Dashboard (via `https://<your-ip>:443`), you land on the **Overview** page. This is your **security command center** — a bird's-eye view of your entire monitored environment.



```bash

ssh -i "your-key.pem" ubuntu@<ec2-public-ip>

```## Step 1 — SSH into the Ubuntu EC2 Instance### Key Things on the Overview Page



This is our **Wazuh Server** — it will run the Wazuh Manager, Indexer, and Dashboard (all-in-one single node).



---![SSH into Ubuntu Server](./images/1.png)| Element | What It Shows |



## Step 2 — Running the Wazuh Installation Script|---|---|



![Wazuh Installation Script Running](./images/2.png)### What's Happening Here| **Total Agents** | How many agents are enrolled (active, disconnected, never connected) |



### What's Happening Here| **Security Events** | Total number of security events over the selected time range |



We are running the **Wazuh quickstart installation script**. The terminal shows the installation progress:We are **SSH-ing into our AWS EC2 Ubuntu instance** for the first time. This is the server where we will install Wazuh (all-in-one deployment).| **Alert Level Distribution** | Breakdown of alerts by severity (low, medium, high, critical) |



- `INFO: --- Dependencies ---` — Installing required packages (`apt-transport-https`, `debhelper`)| **Top MITRE ATT&CK Tactics** | Most common attack techniques detected |

- `INFO: Wazuh repository added` — Wazuh package repo is now configured

- `INFO: --- Configuration files ---` — Generating config filesYou can see the typical Ubuntu SSH welcome message:| **Top 5 Agents** | Agents generating the most alerts |

- `INFO: Generating the root certificate` — Creating TLS/SSL certificates

- `INFO: Generating Admin certificates` — Admin certs for dashboard access| **Top 5 Rules** | Most frequently triggered detection rules |

- `INFO: Generating Wazuh indexer certificates` — Certs for indexer communication

- `INFO: Generating Wazuh dashboard certificates` — Dashboard certs- `Enable ESM Apps to receive additional future security updates`



### Commands Used- `Ubuntu comes with ABSOLUTELY NO WARRANTY`> 💡 *"The overview tab is self-explanatory, yet... Dashboards are for visibility, not for in-depth investigation."* — SOC 201 Podcast



```bash- System info like uptime, packages, etc.

curl -sO https://packages.wazuh.com/4.x/wazuh-install.sh

sudo bash ./wazuh-install.sh -a---

```

### Why This Step

The `-a` flag means **all-in-one** — it installs the Wazuh Manager, Indexer, and Dashboard on the same machine.

## 📊 Step 2 — Security Events Overview

### What Gets Installed

Before installing anything, we need terminal access to the server. We used:

| Component | What It Does |

|---|---|![Security Events Overview](./images/2.png)

| **Wazuh Manager** | Receives and analyzes events from agents |

| **Wazuh Indexer** | Stores alerts as searchable JSON documents |```bash

| **Wazuh Dashboard** | Web UI for visualization (port 443) |

ssh -i "your-key.pem" ubuntu@<ec2-public-ip>### What We're Seeing

> ⚠️ After installation completes, it prints the **admin password**. Save it — you need it to log in!

```The security events timeline and distribution. This shows the **volume and pattern** of alerts over time.

---



## Step 3 — Wazuh Dashboard: First Login (No Agents Yet)

This is our **Wazuh Server** — it will run the Wazuh Manager, Indexer, and Dashboard (all-in-one single node).### Why This Matters

![Wazuh Dashboard Overview — No Agents](./images/3.png)

- **Spikes** in the timeline = something happened (attack, misconfiguration, new agent online)

### What's Happening Here

---- **Flat lines** at zero = either nothing is happening (good) or your detection isn't working (bad!)

We've opened the Wazuh Dashboard in the browser at `https://<ec2-public-ip>:443` and logged in with the admin credentials.

- Use the **time picker** (top right) to zoom into specific time ranges

This is the **Overview page** — but notice the message:

## Step 2 — Running the Wazuh Installation Script

> **"This instance has no agents registered. Please deploy agents to begin monitoring your endpoints."**

### SOC Analyst Workflow

### What We See

![Wazuh Installation Script Running](./images/2.png)```

| Element | Status |

|---|---|1. Look at the timeline → Any unusual spikes?

| **Agents Summary** | 0 agents — none registered yet |

| **Critical severity** | Rule level 15 or higher — no events yet |### What's Happening Here2. Check alert levels → How many high/critical?

| **Endpoint Security modules** | Listed but empty (no data without agents) |

3. Check top agents → Which machines are noisiest?

### Modules Visible (But Empty)

We are running the **Wazuh quickstart installation script**. The terminal shows the installation progress:4. Check top rules → What's being detected most?

- **Configuration Assessment** — Scan your assets as part of a configuration assessment audit

- **Malware Detection** — Check indicators of compromise triggered by malware infections or cyberattacks5. Drill into specifics → Click to investigate

- **File Integrity Monitoring** — Alerts related to file changes, including permissions, content, ownership, and attributes

- `INFO: --- Dependencies ---` — Installing required packages (`apt-transport-https`, `debhelper`)```

### Why It's Empty

- `INFO: Wazuh repository added` — Wazuh package repo is now configured

The Wazuh Dashboard shows data **only from enrolled agents**. Since we just installed the server and haven't deployed any agents yet, everything is at zero. This is expected.

- `INFO: --- Configuration files ---` — Generating config files---

---

- `INFO: Generating the root certificate` — Creating TLS/SSL certificates

## Step 4 — Dashboard Modules Overview (Sidebar)

- `INFO: Generating Admin certificates` — Admin certs for dashboard access## 🔌 Step 3 — Agents Overview

![Dashboard Modules — Endpoint Security & Security Operations](./images/4.png)

- `INFO: Generating Wazuh indexer certificates` — Certs for indexer communication

### What's Happening Here

![Agents Overview](./images/3.png)

We're exploring the **left sidebar / modules panel** of the Wazuh Dashboard. This shows all available security modules organized by category:

### Commands Used

**Endpoint Security:**

### What We're Seeing

| Module | Description |

|---|---|```bashThe **Agents** section shows all enrolled Wazuh agents with their status, OS, IP address, and agent version.

| **Configuration Assessment** | Scan your assets as part of a configuration assessment audit (CIS benchmarks) |

| **File Integrity Monitoring** | Alerts related to file changes — permissions, content, ownership, and attributes |curl -sO https://packages.wazuh.com/4.x/wazuh-install.sh

| **Malware Detection** | Check indicators of compromise triggered by malware infections or cyberattacks |

sudo bash ./wazuh-install.sh -a### Agent Statuses

**Security Operations:**

```

| Module | Description |

|---|---|| Status | Color | Meaning |

| **IT Hygiene** | Assess system, software, processes, and network layers |

| **GDPR** | General Data Protection Regulation — processing of personal data |The `-a` flag means **all-in-one** — it installs the Wazuh Manager, Indexer, and Dashboard on the same machine.|---|---|---|

| **NIST 800-53** | US federal security controls framework |

| And more... | PCI DSS, HIPAA, TSC compliance modules || **Active** | 🟢 Green | Agent is connected and sending data |



> 💡 These modules will remain empty until we deploy agents and enable the corresponding sections in `ossec.conf`.### What Gets Installed| **Disconnected** | 🔴 Red | Agent was connected but lost connection |



---| **Never Connected** | ⚫ Grey | Agent was enrolled but never established a connection |



## Step 5 — Discover Tab: Wazuh Alerts Index Pattern| Component | What It Does || **Pending** | 🟡 Yellow | Agent is waiting for enrollment |



![Discover Tab — wazuh-alerts index](./images/5.png)|---|---|



### What's Happening Here| **Wazuh Manager** | Receives and analyzes events from agents |### What You Can Do Here



We're in the **Discover** tab (similar to OpenSearch/Kibana Discover). This is the raw data explorer.| **Wazuh Indexer** | Stores alerts as searchable JSON documents |- Click on any agent to see its **detailed view** (events, inventory, config)



### Key Elements Visible| **Wazuh Dashboard** | Web UI for visualization (port 443) |- Filter agents by **OS, status, group**



| Element | What It Shows |- See agent **version** (check if any agents need upgrading)

|---|---|

| **Index patterns** | `wazuh-alerts-*`, `wazuh-monitoring-*`, `wazuh-statistics-*` |> ⚠️ After installation completes, it prints the **admin password**. Save it — you need it to log in!- Check **last keep-alive** time (when did the agent last check in?)

| **Search bar** | DQL (Dashboard Query Language) for searching alerts |

| **Time range** | `Last 24 hours` with date picker |

| **Hits** | `404 hits` — total matching documents in this time window |

| **Available fields** | Fields like `index`, `agent.id`, `data.command` that you can add as columns |------



### What Are These Index Patterns?



| Index Pattern | What It Stores |## Step 3 — Wazuh Dashboard: First Login (No Agents Yet)## 🖥️ Step 4 — Individual Agent Dashboard

|---|---|

| `wazuh-alerts-*` | All security alerts generated by rules |

| `wazuh-monitoring-*` | Agent connection/status monitoring data |

| `wazuh-statistics-*` | Performance and statistics data |![Wazuh Dashboard Overview — No Agents](./images/3.png)![Individual Agent Details](./images/4.png)



This is where SOC analysts do their **raw investigation** — searching through alerts, filtering by fields, and exporting data.



---### What's Happening Here### What We're Seeing



## Step 6 — Threat Hunting: Authentication Failure EventsWhen you click on a specific agent, you get its **dedicated dashboard** — everything about that one endpoint.



![Threat Hunting — Authentication Failures](./images/6.png)We've opened the Wazuh Dashboard in the browser at `https://<ec2-public-ip>:443` and logged in with the admin credentials.



### What's Happening Here### Sections Available Per Agent



We're in the **Threat Hunting** module, filtered to show **Authentication failure** events.This is the **Overview page** — but notice the message:



### Key Elements Visible| Section | What It Shows |



| Element | What It Shows |> **"This instance has no agents registered. Please deploy agents to begin monitoring your endpoints."**|---|---|

|---|---|

| **Module** | Threat Hunting || **Overview** | Summary of events, alert levels, top rules for this agent |

| **Filter applied** | `Authentication failure` |

| **Manager name** | `ip-172-31-28-40` (our Wazuh server) |### What We See| **Configuration** | Current agent configuration (which modules are enabled) |

| **Time range** | Last 24 hours |

| **Dashboard + Events tabs** | Dashboard shows charts, Events shows raw logs || **Inventory** | OS, packages, processes, ports, network interfaces |

| **Hits** | 78 authentication failure events detected |

| **Columns** | Timestamp, agent.name, rule.description, rule.level, rule.id || Element | Status || **SCA** | Security Configuration Assessment results for this agent |



### Why Authentication Failures Matter|---|---|| **FIM** | File Integrity Monitoring events on this agent |



These events indicate someone (or something) tried to log in and **failed**. This could mean:| **Agents Summary** | 0 agents — none registered yet || **Vulnerabilities** | Known CVEs affecting this specific agent |



- **Brute force attacks** — someone trying many passwords| **Critical severity** | Rule level 15 or higher — no events yet |

- **Misconfigured services** — wrong credentials in automation

- **Unauthorized access attempts** — someone trying to break in| **Endpoint Security modules** | Listed but empty (no data without agents) |---



> This is one of the most common alert types in Wazuh. In a real SOC, you'd investigate the **source IP**, check how many attempts there were, and potentially block the attacker using Active Response.



---### Why It's Empty## ✅ Step 5 — Configuration Assessment (SCA)



## Step 7 — Deploy New Agent: OS Selection



![Deploy New Agent — Select OS](./images/7.png)The Wazuh Dashboard shows data **only from enrolled agents**. Since we just installed the server and haven't deployed any agents yet, everything is at zero. This is expected.![SCA Module Overview](./images/5.png)



### What's Happening Here



We're deploying our **first Wazuh Agent**. The Dashboard provides a built-in agent deployment wizard.---### What We're Seeing



### Step 1: Select the Operating SystemThe **Security Configuration Assessment** module showing CIS benchmark scan results.



The wizard shows three options:## Step 4 — Dashboard Modules Overview (Sidebar)



| OS | Package Types |### What You See

|---|---|

| **LINUX** | RPM (amd64), RPM (aarch64), DEB (amd64), DEB (aarch64) |![Dashboard Modules — Endpoint Security & Security Operations](./images/4.png)

| **WINDOWS** | MSI (32/64 bits) |

| **macOS** | Intel, Apple Silicon || Column | Description |



Since our second EC2 instance runs **Ubuntu (Debian-based)**, we select **LINUX → DEB amd64**.### What's Happening Here|---|---|



### Server Address Visible| **Policy** | The benchmark being checked (e.g., CIS Ubuntu 22.04) |



The wizard asks for a **server address** — this is the Wazuh Manager IP. We can see `34.224.212.65` entered (public IP at that point).We're exploring the **left sidebar / modules panel** of the Wazuh Dashboard. This shows all available security modules organized by category:| **Score** | Compliance percentage (higher is better) |



---| **Passed** | Number of checks that passed |



## Step 8 — Deploy New Agent: Server Address & Package Selection**Endpoint Security:**| **Failed** | Number of checks that failed (these need fixing!) |



![Deploy New Agent — Package Selection](./images/8.png)| **Not Applicable** | Checks that don't apply to this system |



### What's Happening Here| Module | Description |



Continuing the agent deployment wizard — we've selected **Linux** and now we're confirming the specific package type and server address:|---|---|### How to Read It



- **DEB amd64** — For Ubuntu, Debian ✅ (this is what we need)| **Configuration Assessment** | Scan your assets as part of a configuration assessment audit (CIS benchmarks) |- **Score 39%** = Only 39% of CIS checks pass. Your system needs hardening.

- **Server address:** `98.81.179.28` (the Wazuh Manager's public IP)

| **File Integrity Monitoring** | Alerts related to file changes — permissions, content, ownership, and attributes |- Click on **Failed** checks to see exactly what's wrong and how to fix it.

### Why the IP Changed

| **Malware Detection** | Check indicators of compromise triggered by malware infections or cyberattacks |- Each failed check includes **Remediation** steps.

In Step 7, the IP was `34.224.212.65`, and now it's `98.81.179.28`. This happens when you **stop and restart** an EC2 instance — AWS assigns a new public IP (unless you use an Elastic IP).



---

**Security Operations:**> This module is powered by `<sca><enabled>yes</enabled></sca>` in `ossec.conf`.

## Step 9 — Deploy New Agent: Installation Command



![Deploy New Agent — Install Command](./images/9.png)

| Module | Description |---

### What's Happening Here

|---|---|

The wizard generates the **exact command** to run on the agent machine:

| **IT Hygiene** | Assess system, software, processes, and network layers |## ✅ Step 6 — SCA Policy Details

```bash

wget https://packages.wazuh.com/4.x/apt/pool/main/w/wazuh-agent/wazuh-agent_4.14.3-1_amd64.deb && \| And more... | PCI DSS, GDPR, HIPAA, NIST, TSC compliance modules |

sudo WAZUH_MANAGER='98.81.179.28' \

     WAZUH_AGENT_GROUP='default' \![SCA Policy Details](./images/6.png)

     WAZUH_AGENT_NAME='ubuntu' \

     dpkg -i ./wazuh-agent_4.14.3-1_amd64.deb> 💡 These modules will remain empty until we deploy agents and enable the corresponding sections in `ossec.conf`.

```

### What We're Seeing

### Breaking Down the Command

---Drilling into a specific SCA policy to see **individual check results**.

| Part | What It Does |

|---|---|

| `wget ...wazuh-agent_4.14.3-1_amd64.deb` | Downloads the Wazuh Agent package |

| `WAZUH_MANAGER='98.81.179.28'` | Sets the Wazuh Server IP (our EC2 public IP) |## Step 5 — Discover Tab: Wazuh Alerts Index Pattern### Check Result Details

| `WAZUH_AGENT_GROUP='default'` | Assigns the agent to the `default` group |

| `WAZUH_AGENT_NAME='ubuntu'` | Names this agent "ubuntu" |

| `dpkg -i ./wazuh-agent...deb` | Installs the downloaded package |

![Discover Tab — wazuh-alerts index](./images/5.png)| Field | What It Shows |

### Configuration Visible

|---|---|

- **Group selected:** `default`

- **Requirements note:** Administrator privileges needed, Bash shell required### What's Happening Here| **ID** | Unique check identifier (e.g., 28500) |



> ⚠️ **Important:** For the agent to connect, port **1514** and **1515** must be open on the Wazuh Server's Security Group. That's what we do next!| **Title** | What's being checked (e.g., "Ensure /tmp is a separate partition") |



---We're in the **Discover** tab (similar to OpenSearch/Kibana Discover). This is the raw data explorer.| **Status** | Pass ✅ / Fail ❌ / Not Applicable ⬜ |



## Step 10 — AWS Security Group: Inbound Rules| **Rationale** | Why this check matters for security |



![AWS EC2 Security Group — Inbound Rules](./images/10.png)### Key Elements Visible| **Remediation** | Step-by-step instructions to fix the issue |



### What's Happening Here| **Compliance** | Which standards this maps to (CIS, PCI DSS, HIPAA, etc.) |



We're in the **AWS EC2 Console → Security Groups** section. This is the firewall configuration for our Wazuh Server EC2 instance.| Element | What It Shows |



### Security Group Details|---|---|> 💡 **Pro Tip:** Sort by **Failed** first. These are your actionable items. Fix the easy ones first to quickly improve your score.



| Field | Value || **Index patterns** | `wazuh-alerts-*`, `wazuh-monitoring-*`, `wazuh-statistics-*` |

|---|---|

| **Security Group Name** | `launch-wizard-1` || **Search bar** | DQL (Dashboard Query Language) for searching alerts |---

| **Security Group ID** | `sg-0409dfe0b07ea77dd` |

| **VPC ID** | `vpc-0f92627c7e7f23e9c` || **Time range** | `Last 24 hours` with date picker |

| **Inbound Rules Count** | 6 permission entries |

| **Outbound Rules Count** | 1 permission entry || **Hits** | `404 hits` — total matching documents in this time window |## 🦠 Step 7 — Malware Detection Module



### Why This Is Critical| **Available fields** | Fields like `index` that you can add as columns |



For the Wazuh Agent to connect to the Wazuh Server, the following ports **must be open** in the inbound rules:![Malware Detection](./images/7.png)



| Port | Protocol | Purpose |### What Are These Index Patterns?

|---|---|---|

| **1514** | TCP | Agent event data transmission |### What We're Seeing

| **1515** | TCP | Agent enrollment/registration |

| **443** | TCP | Dashboard web access (HTTPS) || Index Pattern | What It Stores |The **Malware Detection** module showing alerts related to malware activity, rootkits, and IOCs.

| **22** | TCP | SSH access |

|---|---|

> If you skip this step, the agent will fail to connect with a "Connection refused" error. This is the **#1 most common issue** when deploying agents on AWS.

| `wazuh-alerts-*` | All security alerts generated by rules |### Types of Alerts You'll See Here

---

| `wazuh-monitoring-*` | Agent connection/status monitoring data |

## Step 11 — AWS EC2: Two Instances Running

| `wazuh-statistics-*` | Performance and statistics data || Alert Type | Description |

![AWS EC2 Instances — 2 Instances Running](./images/11.png)

|---|---|

### What's Happening Here

This is where SOC analysts do their **raw investigation** — searching through alerts, filtering by fields, and exporting data.| **Rootkit detection** | Known rootkit files or behaviors found |

We're in the **AWS EC2 Instances** dashboard. We can see **2 instances** running:

| **Trojaned binary** | System binary replaced with malicious version |

| Instance | Role |

|---|---|---| **Hidden process** | Process running but invisible to `ps` |

| **Instance 1 (wazuh)** | Wazuh Server (Manager + Indexer + Dashboard) — `t2.medium` |

| **Instance 2** | Wazuh Agent (the endpoint we're monitoring) || **Hidden port** | Network port open but invisible to `netstat` |



### Columns Visible## Step 6 — Threat Hunting: Authentication Failure Events| **Suspicious file** | Executable dropped in commonly abused directories |



| Column | What It Shows || **Threat intelligence match** | File hash/IP matches known malware database |

|---|---|

| **Name** | Instance name tag (`wazuh`) |![Threat Hunting — Authentication Failures](./images/6.png)

| **Instance State** | Running ✅ |

| **Instance Type** | `t2.medium` |> This module is powered by `<rootcheck><disabled>no</disabled></rootcheck>` in `ossec.conf`.

| **Status Check** | `2/2 checks passed` |

| **Availability Zone** | `us-east-1` (N. Virginia) |### What's Happening Here

| **Public IPv4 DNS** | Public address to connect |

---

> 💡 This is our lab setup: **2 EC2 instances** in the same VPC. One runs Wazuh all-in-one, the other runs the Wazuh Agent.

We're in the **Threat Hunting** module, filtered to show **Authentication failure** events.

---

## 📁 Step 8 — File Integrity Monitoring (FIM)

## Step 12 — Wazuh Dashboard: First Agent Enrolled

### Key Elements Visible

![Wazuh Agents — 1 Agent Active](./images/12.png)

![FIM Overview](./images/8.png)

### What's Happening Here

| Element | What It Shows |

Back in the Wazuh Dashboard → **Endpoints** section. Now we can see our **first agent has enrolled successfully!**

|---|---|### What We're Seeing

### Agent Details

| **Module** | Threat Hunting |The **File Integrity Monitoring** module showing file change events across monitored endpoints.

| Field | Value |

|---|---|| **Filter applied** | `Authentication failure` |

| **Agent ID** | `001` |

| **Name** | `ip-172-31-27-237` || **Manager name** | `ip-172-31-28-40` (our Wazuh server) |### What FIM Tracks

| **Status** | 🟢 Active |

| **IP Address** | `172.31.27.237` || **Time range** | Last 24 hours |

| **Group** | `default` |

| **Operating System** | Ubuntu 24.04.3 LTS || **Dashboard + Events tabs** | Dashboard shows charts, Events shows raw logs || Event Type | Icon | Description |

| **Cluster Node** | `node01` |

| **Version** | `v4.14.3` || **Count** | Number of authentication failure events detected ||---|---|---|



### Agents by Status| **Added** | ➕ | New file was created |



| Status | Count |### Why Authentication Failures Matter| **Modified** | ✏️ | Existing file was changed |

|---|---|

| 🟢 Active | 1 || **Deleted** | 🗑️ | File was removed |

| 🔴 Disconnected | 0 |

| ⚫ Never Connected | 0 |These events indicate someone (or something) tried to log in and **failed**. This could mean:



### What Changed### Dashboard Elements



Earlier in Step 3, the dashboard said "No agents registered." Now after:- **Brute force attacks** — someone trying many passwords- **Events over time** — Timeline of file changes



1. Opening the Security Group ports (Step 10)- **Misconfigured services** — wrong credentials in automation- **Top agents** — Which endpoints have the most file changes

2. Running the install command on the second EC2 (Step 9)

3. Starting the agent service (`sudo systemctl start wazuh-agent`)- **Unauthorized access attempts** — someone trying to break in- **Top files** — Most frequently modified files



...the agent connected and appears as **Active** ✅- **Actions summary** — Count of added/modified/deleted



---> This is one of the most common alert types in Wazuh. In a real SOC, you'd investigate the **source IP**, check how many attempts there were, and potentially block the attacker using Active Response.



## Step 13 — Individual Agent Dashboard> This module is powered by `<syscheck><disabled>no</disabled></syscheck>` in `ossec.conf`.



![Agent Details — ip-172-31-27-237](./images/13.png)---



### What's Happening Here---



We clicked on agent `001` to view its **individual dashboard**. This shows everything about this specific endpoint.## Step 7 — Deploy New Agent: OS Selection



### Agent Information## 📁 Step 9 — FIM Event Details



| Field | Value |![Deploy New Agent — Select OS](./images/7.png)

|---|---|

| **ID** | 001 |![FIM Event Details](./images/9.png)

| **Status** | 🟢 Active |

| **IP Address** | 172.31.27.237 |### What's Happening Here

| **Version** | Wazuh v4.14.3 |

| **Group** | default |### What We're Seeing

| **Operating System** | Ubuntu 24.04.3 LTS |

| **Cluster Node** | node01 |We're deploying our **first Wazuh Agent**. The Dashboard provides a built-in agent deployment wizard.Drilling into a specific FIM event to see **exactly what changed**.

| **Registration Date** | Feb 20, 2026 @ 11:10:20 |

| **Last Keep Alive** | Feb 20, 2026 @ 11:18:20 |



### System Inventory (Visible)### Step 1: Select the Operating System### Event Detail Fields



| Field | Value |

|---|---|

| **Cores** | 2 |The wizard shows three options:| Field | What It Shows |

| **Memory** | 3.868 GB |

| **CPU** | Intel(R) Xeon(R) ||---|---|



### Available Tabs for This Agent| OS | Package Types || **File path** | `/etc/passwd`, `/etc/shadow`, etc. |



| Tab | What It Shows ||---|---|| **Event type** | Modified / Added / Deleted |

|---|---|

| **Threat Hunting** | Security alerts for this agent || **LINUX** | RPM (amd64), RPM (aarch64), DEB (amd64), DEB (aarch64) || **Changed attributes** | Which attributes changed (content, permissions, owner, etc.) |

| **File Integrity Monitoring** | File change events on this agent |

| **Configuration Assessment** | SCA/CIS benchmark results || **WINDOWS** | MSI (32/64 bits) || **Old value → New value** | What the attribute was before and after |

| **MITRE ATT&CK** | ATT&CK technique mappings |

| **Vulnerability Detection** | CVEs found on this agent || **macOS** | Intel, Apple Silicon || **User** | Who made the change |

| **Stats** | Agent performance statistics |

| **Configuration** | Current agent configuration || **Process** | Which process made the change |



---Since our second EC2 instance runs **Ubuntu (Debian-based)**, we select **LINUX → DEB amd64**.| **Timestamp** | Exactly when the change occurred |



## Step 14 — AWS EC2: Managing Instances| **MD5 / SHA1 / SHA256** | File hash before and after (proves content changed) |



![AWS EC2 — Instances Dashboard Sidebar](./images/14.png)---



### What's Happening Here> 💡 **This answers the 5 WH questions:** WHO changed it, WHAT was changed, WHERE (file path), WHEN (timestamp), and HOW (which process).



We're back in the **AWS EC2 Console**, viewing the EC2 sidebar navigation and instances list. This shows the full range of AWS EC2 management sections.## Step 8 — Deploy New Agent: Package Selection



### EC2 Sidebar Sections Visible---



| Section | Purpose |![Deploy New Agent — Package Selection](./images/8.png)

|---|---|

| **Dashboard** | EC2 overview |## 🔎 Step 10 — Threat Hunting Module

| **Instances** | Running instances |

| **Instance Types** | Available machine types |### What's Happening Here

| **Launch Templates** | Saved instance configurations |

| **Spot Requests** | Spot instance management |![Threat Hunting](./images/10.png)

| **Savings Plans** | Cost optimization |

| **Reserved Instances** | Long-term reservations |Continuing the agent deployment wizard — we've selected **Linux** and now we're choosing the specific package type:

| **Dedicated Hosts** | Dedicated hardware |

| **Capacity Reservations** | Reserved capacity |### What We're Seeing

| **AMIs / AMI Catalog** | Machine images |

| **Volumes / Snapshots** | EBS storage management |- **RPM amd64** — For Red Hat, CentOS, Amazon Linux, FedoraThe **Threat Hunting** module — the SOC analyst's primary investigation workspace.

| **Security Groups** | Firewall rules |

| **Elastic IPs** | Static public IPs |- **RPM aarch64** — For ARM-based Linux (Graviton)

| **Key Pairs** | SSH key management |

- **DEB amd64** — For Ubuntu, Debian ✅ (this is what we need)### How to Use Threat Hunting

> We're preparing to launch/manage the second EC2 instance for the second Wazuh Agent.

- **DEB aarch64** — For ARM-based Ubuntu/DebianThis is a **raw alert explorer** with powerful search and filtering:

---



## Step 15 — Wazuh Dashboard: Two Agents Enrolled

We select **DEB amd64** since our second EC2 instance is Ubuntu on x86_64.| Feature | How to Use It |

![Wazuh Agents — 2 Agents](./images/15.png)

|---|---|

### What's Happening Here

---| **Search bar** | Type any keyword — IP, username, rule ID, file path |

Now we have **2 agents** enrolled in Wazuh:

| **Time range** | Use the date picker to narrow down to a specific window |

| Agent ID | Name | Status |

|---|---|---|## Step 9 — Deploy New Agent: Installation Command| **Filters** | Add filters for agent, rule level, rule group, MITRE technique |

| **001** | `ip-172-31-27-237` | 🟢 Active |

| **002** | `test-server2` | ⚫ Never Connected || **Columns** | Customize which columns are shown in the results table |



### Agents by Status![Deploy New Agent — Install Command](./images/9.png)| **Alert details** | Click any row to expand the full JSON alert |



| Status | Count |

|---|---|

| 🟢 Active | 1 |### What's Happening Here### SOC Investigation Flow

| ⚫ Never Connected | 1 |

```

### Top 5 OS / Groups

The wizard generates the **exact command** to run on the agent machine. We can see:1. Start with a time range (when did the incident happen?)

- **OS:** `ubuntu` (1 agent reporting)

- **Groups:** `default` (2 agents assigned)2. Filter by alert level ≥ 10 (focus on high severity)



> Agent 002 (`test-server2`) has been enrolled but hasn't established its first connection yet (status: **Never Connected**). Once we start the agent service on that machine, it will switch to Active.```bash3. Search for the IOC (IP, hash, domain, username)



---wget https://packages.wazuh.com/4.x/apt/pool/main/w/wazuh-agent/wazuh-agent_4.14.3-1_amd64.deb && \4. Expand alerts to read full context



## Step 16 — Agents Detailed List Viewsudo WAZUH_MANAGER='98.81.179.28' \5. Correlate — are multiple agents affected?



![Agents Detailed List — Both Active](./images/16.png)     WAZUH_AGENT_GROUP='default' \6. Document findings for the incident report



### What's Happening Here     WAZUH_AGENT_NAME='ubuntu' \```



The **expanded agents table** showing full details for both agents — now **both are active**:     dpkg -i ./wazuh-agent_4.14.3-1_amd64.deb



| Field | Agent 001 | Agent 002 |```---

|---|---|---|

| **Name** | ip-172-31-27-237 | test-server2 |

| **IP Address** | 172.31.27.237 | 172.31.25.172 |

| **Group** | default | default |### Breaking Down the Command## 🔎 Step 11 — Threat Hunting: Alert Details

| **OS** | Ubuntu 24.04.3 LTS | Ubuntu 24.04.3 LTS |

| **Cluster Node** | node01 | node01 |

| **Version** | v4.14.3 | v4.14.3 |

| **Status** | 🟢 Active | 🟢 Active || Part | What It Does |![Threat Hunting Alert Details](./images/11.png)



### Key Takeaway|---|---|



Both agents are:| `wget ...wazuh-agent_4.14.3-1_amd64.deb` | Downloads the Wazuh Agent package |### What We're Seeing

- Same OS (Ubuntu 24.04.3 LTS)

- Same Wazuh version (v4.14.3)| `WAZUH_MANAGER='98.81.179.28'` | Sets the Wazuh Server IP (our EC2 public IP) |Expanding a specific alert in the Threat Hunting module to see its **full JSON payload**.

- In the same group (`default`)

- Both reporting to the same cluster node (`node01`)| `WAZUH_AGENT_GROUP='default'` | Assigns the agent to the `default` group |

- Both have action buttons for management (restart, upgrade, etc.)

| `WAZUH_AGENT_NAME='ubuntu'` | Names this agent "ubuntu" |### What the JSON Contains

---

| `dpkg -i ./wazuh-agent...deb` | Installs the downloaded package |

## Step 17 — Configuration Assessment (SCA) Dashboard

```json

![Configuration Assessment — test-server2](./images/17.png)

### Configuration Visible{

### What's Happening Here

  "agent": { "id": "001", "name": "ubuntu-server", "ip": "10.0.1.50" },

We're in the **Configuration Assessment (SCA)** module for `test-server2`. This module runs **CIS benchmark scans** to check if the system is configured securely.

- **Server address:** `98.81.179.28` (the Wazuh manager's public IP)  "rule": {

### What We See

- **Group:** `default`    "id": "5710",

| Element | Description |

|---|---|- **Agent name:** Can be customized    "level": 10,

| **Dashboard tab** | Overview charts and metrics |

| **Inventory tab** | Detailed check-by-check results |    "description": "SSH brute-force attack",

| **Events tab** | Raw SCA events ← We're here |

| **Filter** | `manager.name: ip-172-31-17-228` and `rule.groups: sca` and `agent.id: 002` |> ⚠️ **Important:** For the agent to connect, port **1514** and **1515** must be open on the Wazuh Server's Security Group. That's what we do next!    "mitre": { "id": ["T1110"], "tactic": ["Credential Access"] },

| **Time range** | Last 24 hours |

| **Hits** | 280 SCA events |    "pci_dss": ["10.2.4"],



### Columns Visible---    "gdpr": ["IV_35.7.d"]



- **Timestamp** — When the SCA check ran  },

- **data.sca.check.title** — What's being checked

- **data.sca.check.file** — Config file being evaluated## Step 10 — AWS Security Group: Inbound Rules  "data": {



> SCA scans run automatically when the agent is enrolled. It checks the system against CIS benchmarks and reports which checks **passed**, **failed**, or are **not applicable**.    "srcip": "192.168.1.100",



---![AWS EC2 Security Group — Inbound Rules](./images/10.png)    "dstuser": "root"



## Step 18 — Dashboard Overview with 2 Active Agents  },



![Wazuh Overview — 2 Active Agents](./images/18.png)### What's Happening Here  "full_log": "sshd[12345]: Failed password for root from 192.168.1.100..."



### What's Happening Here}



Back to the **Overview** page. Now the dashboard has data because we have **2 active agents**:We're in the **AWS EC2 Console → Security Groups** section. This is the firewall configuration for our Wazuh Server EC2 instance.```



### Agents Summary



| Status | Count |### Security Group Details> Every field is searchable. Every field can be used as a filter. This is the power of Wazuh + Indexer.

|---|---|

| 🟢 Active | 2 |

| 🔴 Disconnected | 0 |

| Field | Value |---

### Modules Listed

|---|---|

Now the Endpoint Security and Security Operations modules are visible and contain data:

| **Security Group Name** | `launch-wizard-1` |## 💀 Step 12 — Vulnerability Detection Overview

- **Configuration Assessment** — SCA scan results available

- **File Integrity Monitoring** — File change events being collected| **Security Group ID** | `sg-0409dfe0b07ea77dd` |

- **Malware Detection** — Rootcheck running

- **IT Hygiene** — System inventory collecting| **VPC ID** | `vpc-0f92627c7e7f23e9c` |![Vulnerability Detection Overview](./images/12.png)

- **GDPR** — Compliance mappings active

| **Inbound Rules Count** | 6 permission entries |

> Compare this to Step 3 — the dashboard was empty. Now with 2 agents sending data, all modules are populated.

| **Outbound Rules Count** | 1 permission entry |### What We're Seeing

---

The **Vulnerability Detection** module showing known CVEs across all monitored endpoints.

## Step 19 — IT Hygiene: System Dashboard

### Why This Is Critical

![IT Hygiene — Dashboard, Platforms](./images/19.png)

### Dashboard Elements

### What's Happening Here

For the Wazuh Agent to connect to the Wazuh Server, the following ports **must be open** in the inbound rules:

We're exploring the **IT Hygiene** module (System Inventory). This shows the state of all monitored endpoints.

| Element | What It Shows |

### Tabs Available

| Port | Protocol | Purpose ||---|---|

| Tab | What It Shows |

|---|---||---|---|---|| **Total vulnerabilities** | Count across all agents |

| **Dashboard** | Overview with charts |

| **OS / Hardware** | Operating system and hardware info || **1514** | TCP | Agent event data transmission || **Severity chart** | Critical / High / Medium / Low breakdown |

| **Platform** | Platform distribution ← We're here |

| **System** | System-level details || **1515** | TCP | Agent enrollment/registration || **Top vulnerable agents** | Which endpoints need patching first |

| **Software** | Installed packages |

| **Processes** | Running processes || **443** | TCP | Dashboard web access (HTTPS) || **Top CVEs** | Most common vulnerabilities |

| **Network** | Network interfaces and connections |

| **Identity** | User/group information || **22** | TCP | SSH access || **Vulnerability table** | Detailed list with CVE ID, package, severity, agent |

| **Services** | Running services |



### What We See

> If you skip this step, the agent will fail to connect with a "Connection refused" error. This is the **#1 most common issue** when deploying agents on AWS.> This module is powered by `<vulnerability-detection><enabled>yes</enabled></vulnerability-detection>` in `ossec.conf` and requires `<syscollector>` to be enabled.

| Element | Value |

|---|---|

| **Cluster** | `ip-172-31-17-228` |

| **Top 5 Platforms** | `ubuntu` (both agents run Ubuntu) |------

| **Agents listed** | `ip-172-31-27-237` and `test-server2` |

| **OS Details** | Ubuntu 24.04.3 LTS (Noble Numbat), Kernel 6.14.0-1018-aws |



> This data is collected by the `<syscollector>` module in `ossec.conf`. It inventories all software, hardware, network, and processes on each agent.## Step 11 — AWS EC2: Two Instances Running## 💀 Step 13 — Vulnerability Alert Details



---



## Step 20 — IT Hygiene: Network Tab![AWS EC2 Instances — 2 Instances Running](./images/11.png)![Vulnerability Alert Details](./images/13.png)



![IT Hygiene — Network (Ports & Destinations)](./images/20.png)



### What's Happening Here### What's Happening Here### What We're Seeing



We're in the **IT Hygiene → Network** tab. This shows network-related inventory data.Drilling into a specific vulnerability to see its **full details**.



### Tabs in Network SectionWe're in the **AWS EC2 Instances** dashboard. We can see **2 instances** running:



`Addresses` → `Interfaces` → `Protocols` → `Listeners` → `Traffic`### Vulnerability Detail Fields



### What We See| Instance | Role |



| Element | Details ||---|---|| Field | Example |

|---|---|

| **Filter** | `NOT destination_port:0` (filtering out port 0 noise) || **Instance 1** | Wazuh Server (Manager + Indexer + Dashboard) ||---|---|

| **Top 5 Destination Ports** | **443** (HTTPS), **1514** (Wazuh agent comm), **80** (HTTP), **34007** |

| **Instance 2** | Wazuh Agent (the endpoint we're monitoring) || **CVE** | CVE-2024-1234 |

### Why Ports 443 and 1514?

| **Severity** | Critical (CVSS 9.8) |

| Port | Why It Appears |

|---|---|### Columns Visible| **Package** | openssl 3.0.2-0ubuntu1 |

| **443** | Agents/browsers connecting to the Wazuh Dashboard |

| **1514** | Agents sending event data to the Wazuh Manager || **Status** | Active / Fixed |



### Agent Network Data| Column | What It Shows || **Description** | Buffer overflow allowing remote code execution... |



- `test-server2` at `172.31.25.172` — multiple connections visible|---|---|| **References** | Links to NVD, vendor advisories |

- `ip-172-31-27-237` at `172.31.27.237` — connections to various ports

| **Name** | Instance name tag || **Remediation** | Upgrade to version X.Y.Z |

These are the two most critical ports in a Wazuh deployment. Seeing them here confirms the agents are communicating properly.

| **Instance State** | Running / Stopped || **Affected agent** | web-server-01 |

---

| **Instance Type** | Machine size (t2.medium, t3.large, etc.) |

## Step 21 — IT Hygiene: Services Tab

| **Status Check** | Health check status |---

![IT Hygiene — Services](./images/21.png)

| **Availability Zone** | `us-east-1` (N. Virginia) |

### What's Happening Here

| **Public IPv4 DNS** | Public address to connect |## ⚔️ Step 14 — MITRE ATT&CK Module

We're in the **IT Hygiene → Services** tab. This shows all running services discovered on the monitored agents.



### What We See

> 💡 This is our lab setup: **2 EC2 instances** in the same VPC. One runs Wazuh all-in-one, the other runs the Wazuh Agent.![MITRE ATT&CK Overview](./images/14.png)

| Element | Details |

|---|---|

| **Filter** | `wazuh.cluster.name: ip-172-31-17-228` |

| **Top 5 Services** | `apport` and others |---### What We're Seeing

| **Total Unique Services** | 179 services discovered |

| **Total Hits** | 356 service entries |The **MITRE ATT&CK** module showing security events mapped to adversary tactics and techniques.



### Services Visible## Step 12 — Wazuh Dashboard: First Agent Enrolled



- `ModemManager` — Network modem management### What the MITRE View Shows

- `systemd-tpm2-setup-early` — TPM security setup

- `networking` — Network configuration![Wazuh Agents — 1 Agent Active](./images/12.png)

- `ldconfig` — Library cache management

| Element | Description |

### Why This Matters

### What's Happening Here|---|---|

- **Unexpected services** = potential security risk (backdoor, unauthorized software)

- **Missing services** = something critical might not be running| **Tactics bar chart** | Which of the 14 ATT&CK tactics have been observed |

- Useful for **compliance audits** — verify only approved services are running

Back in the Wazuh Dashboard → **Endpoints** section. Now we can see our **first agent has enrolled successfully!**| **Techniques table** | Specific techniques with alert counts |

---

| **Heatmap** | Visual representation of technique frequency |

## Step 22 — MITRE ATT&CK: Dashboard

### Agent Details

![MITRE ATT&CK — Dashboard & Tactics](./images/22.png)

### Reading the MITRE Dashboard

### What's Happening Here

| Field | Value |- **Most common tactics** = where your environment is being targeted most

We're in the **MITRE ATT&CK** module — **Dashboard** tab. This maps Wazuh alerts to the MITRE ATT&CK framework.

|---|---|- **Rare tactics** = either you're not being targeted OR your detection has gaps

### What We See

| **Agent ID** | `001` |- Click any technique to see the **specific alerts** that triggered it

| Element | Description |

|---|---|| **Name** | `ip-172-31-27-237` |

| **Alerts evolution over time** | Timeline chart showing ATT&CK-tagged alerts |

| **Top tactics** | Valid Accounts, Defense Evasion, Privilege Escalation, Persistence, Initial Access || **Status** | 🟢 Active |---

| **Filter** | `manager.name: ip-172-31-17-228` and `rule.mitre.id: exists` |

| **Time range** | Last 24 hours || **Count** | 1 agent total |



### Top Tactics Detected## ⚔️ Step 15 — MITRE ATT&CK Technique Details



| Tactic | What It Means |### What Changed

|---|---|

| **Valid Accounts** | Attackers using legitimate credentials |![MITRE ATT&CK Technique Details](./images/15.png)

| **Defense Evasion** | Techniques to avoid detection |

| **Sudo and Sudo Caching** | Privilege escalation via sudo |Earlier in Step 3, the dashboard said "No agents registered." Now after:

| **Create Account** | New accounts being created |

| **Disable or Modify Tools** | Security tools being tampered with |1. Opening the Security Group ports (Step 10)### What We're Seeing

| **Persistence** | Maintaining access after initial compromise |

| **Privilege Escalation** | Gaining higher-level permissions |2. Running the install command on the second EC2 (Step 9)Drilling into a specific MITRE ATT&CK technique to see which **alerts and agents** are associated.

| **Initial Access** | First entry point into the system |

3. Starting the agent service (`sudo systemctl start wazuh-agent`)

### Additional Sections

### Why This Matters

- **Attacks by technique** — detailed technique breakdown

- **Top tactics by agent** — which agents trigger which tactics...the agent connected and appears as **Active** ✅Instead of seeing isolated alerts, you now see them in the **context of an attack chain**:

- **MITRE techniques by agent** — per-agent technique mapping

```

> Every Wazuh rule can be mapped to MITRE ATT&CK tactics and techniques. This turns isolated alerts into a **coherent attack narrative**.

---T1110 (Brute Force) → Multiple failed SSH logins from same IP

---

T1078 (Valid Accounts) → Successful login after brute force

## Step 23 — MITRE ATT&CK: Intelligence Tab

## Step 13 — Individual Agent DashboardT1059 (Command Execution) → Suspicious commands run post-login

![MITRE ATT&CK — Intelligence (Groups, Mitigations, Software)](./images/23.png)

T1547 (Persistence) → New startup script created

### What's Happening Here

![Agent Details — ip-172-31-27-237](./images/13.png)```

We're exploring the **MITRE ATT&CK → Intelligence** tab. This is a built-in reference database of threat intelligence.



### Sections Available

### What's Happening Here> 💡 This turns random alerts into a **coherent attack narrative**.

| Section | What It Contains |

|---|---|

| **Groups** | 150+ known threat actor groups |

| **Mitigations** | Recommended countermeasures for each technique |We clicked on agent `001` to view its **individual dashboard**. This shows everything about this specific endpoint.---

| **Software** | Known malware and tools used by threat actors |

| **Tactics** | The 14 ATT&CK tactic categories |

| **Techniques** | Specific attack techniques with descriptions |

### Agent Information## 🔧 Step 16 — IT Hygiene / System Inventory

### Groups Visible (150 threat actor groups)



| Group ID | Name | Description |

|---|---|---|| Field | Value |![IT Hygiene](./images/16.png)

| G0018 | admin@338 | China-based cyber threat group targeting financial/economic orgs |

| G0130 | Ajax Security Team | Iranian threat group ||---|---|

| G0138 | Andariel | North Korean sub-group of Lazarus |

| G1007 | Aoqin Dragon | Chinese-linked espionage group || **ID** | 001 |### What We're Seeing



### Why This Is Useful| **Status** | 🟢 Active |The **IT Hygiene** module (also known as System Inventory) showing the state of your endpoints.



You can **search for any threat group** and see:| **IP Address** | 172.31.27.237 |

- What techniques they use

- What software/malware they deploy| **Version** | Wazuh v4.14.3 |### Inventory Categories

- What mitigations defend against them

| **Group** | default |

> This is built into Wazuh — no extra tools needed. It's like having a MITRE ATT&CK encyclopedia inside your SIEM.

| **Operating System** | Ubuntu 24.04.3 LTS || Category | What It Shows |

---

| **Cluster Node** | node01 ||---|---|

## Step 24 — MITRE ATT&CK: Events Tab

| **Registration Date** | Feb 20, 2026 @ 11:10:20 || **Packages** | All installed software with versions |

![MITRE ATT&CK — Events](./images/24.png)

| **Processes** | Currently running processes |

### What's Happening Here

### Available Tabs for This Agent| **Ports** | Open/listening ports and associated services |

We're in the **MITRE ATT&CK → Events** tab. This shows the **raw alert data** for events that matched MITRE ATT&CK rules.

| **Network** | Network interfaces, IPs, MACs |

### What We See

| Tab | What It Shows || **OS** | Operating system name, version, kernel |

| Element | Description |

|---|---||---|---|| **Hardware** | CPU, RAM, serial numbers |

| **Filter** | `manager.name: ip-172-31-17-228` and `rule.mitre.id: exists` |

| **Framework tab** | Currently viewing the events under Framework || **Threat Hunting** | Security alerts for this agent |

| **Columns** | Timestamp, agent.name |

| **Export** | `Export Formatted` button to download results || **File Integrity Monitoring** | File change events on this agent |> This data is collected by `<wodle name="syscollector">` in `ossec.conf`.

| **Timestamps** | Events from `Feb 20, 2026 @ 10:55` to `11:21` |

| **Configuration Assessment** | SCA/CIS benchmark results |

### Agents in Events

| **MITRE ATT&CK** | ATT&CK technique mappings |---

- `ip-172-31-27-237` (Agent 001)

- `ip-172-31-17-228` (Wazuh Manager itself)| **Vulnerability Detection** | CVEs found on this agent |



Each row is an individual alert that has been tagged with a MITRE ATT&CK technique ID. You can click any row to see the full JSON with:| **Stats** | Agent performance statistics |## 📦 Step 17 — System Inventory: Packages

- Rule ID and description

- MITRE technique and tactic| **Configuration** | Current agent configuration |

- Agent details

- Raw log data![System Inventory Packages](./images/17.png)



------



## Step 25 — Vulnerability Detection: Dashboard### What We're Seeing



![Vulnerability Detection — Dashboard Overview](./images/25.png)## Step 14 — AWS EC2: Managing InstancesThe **Packages** view within IT Hygiene — every installed package on the selected agent.



### What's Happening Here



We're in the **Vulnerability Detection** module — **Dashboard** tab. This shows all known CVEs (Common Vulnerabilities and Exposures) detected on our agents.![AWS EC2 — Instances Dashboard Sidebar](./images/14.png)### Why This Is Useful



### Vulnerability Counts- **Find vulnerable software:** Search for a specific package name/version



| Severity | Count |### What's Happening Here- **Shadow IT:** Detect unauthorized software installations

|---|---|

| 🔴 **Critical** | 18 |- **Compliance:** Verify only approved software is installed

| 🟠 **High** | 344 |

| 🟡 **Medium** | 758 |We're back in the **AWS EC2 Console**, preparing to launch or manage our second EC2 instance (for the second Wazuh Agent).- **Patch management:** Compare versions across agents

| 🔵 **Low** | 134 |

| ⚪ **Pending Evaluation** | 1,566 |



### Top Sections### EC2 Sidebar Sections Visible---



| Section | What It Shows |

|---|---|

| **Top 5 Vulnerabilities** | `CVE-2022-3219` (22 occurrences), `CVE-2025-68972` (22), `CVE-2025-68973` || Section | Purpose |## 🌐 Step 18 — System Inventory: Network / Ports

| **Top OS** | `Ubuntu 24.04.3 LTS (Noble Numbat)` — 2,820 vulnerabilities |

| **Top 5 Agents** | `test-server2` — 1,420 vulnerabilities, `ip-172-31-27-237` — 1,400 ||---|---|

| **Top 5 Packages** | `linux-aws` (2,352), `glib-2.0` (20) |

| **Dashboard** | EC2 overview |![System Inventory Network](./images/18.png)

### How It Works

| **Instances** | Running instances |

1. The `<syscollector>` module inventories all installed packages

2. The `<vulnerability-detection>` module cross-references package versions against CVE databases| **Instance Types** | Available machine types |### What We're Seeing

3. Results appear here with severity ratings

| **Launch Templates** | Saved instance configurations |The **Network interfaces and open ports** view.

> This is powered by `<vulnerability-detection><enabled>yes</enabled></vulnerability-detection>` in `ossec.conf`.

| **Spot Requests** | Spot instance management |

---

| **Savings Plans** | Cost optimization |### Key Things to Look For

## Step 26 — Vulnerability Detection: Events (Filtered by Agent)

| **Reserved Instances** | Long-term reservations |

![Vulnerability Detection — Events for test-server2](./images/26.png)

| **Dedicated Hosts** | Dedicated hardware || Red Flag | What It Means |

### What's Happening Here

| **Capacity Reservations** | Reserved capacity ||---|---|

We're in the **Vulnerability Detection → Events** tab, filtered to show vulnerabilities for **test-server2** specifically.

| **Unexpected open port** | A service is running that shouldn't be (backdoor?) |

### What We See

> We're launching a second instance to demonstrate monitoring **multiple agents** in Wazuh.| **Port 0.0.0.0:XXXX** | Service listening on ALL interfaces (should it be?) |

| Element | Description |

|---|---|| **Unknown process on a port** | Unfamiliar process bound to a network port |

| **Filter** | `agent.name: test-server2` and `Evaluated / Under evaluation` |

| **Total hits** | 1,420 vulnerabilities on this agent |---| **New interface** | New network interface appeared (VM escape? tunnel?) |

| **Columns** | Agent name, Package name, Package version, Vulnerability description, Severity, Vulnerability ID |



### Sample Vulnerability Visible

## Step 15 — Wazuh Dashboard: Two Agents Enrolled---

- **Package:** `libgnutls30t64` version `3.8.3-1.1ubuntu3.4`

- **Description:** "A flaw was found in the GnuTLS library..."

- **Agent:** `test-server2`

![Wazuh Agents — 2 Agents Active](./images/15.png)## 💳 Step 19 — PCI DSS Compliance Module

### Why This View Matters



This is where you do **per-agent vulnerability assessment**:

- Which packages on this specific server are vulnerable?### What's Happening Here![PCI DSS](./images/19.png)

- What's the severity of each vulnerability?

- What should be patched first? (Critical → High → Medium → Low)



> 💡 **Pro Tip:** Filter by `vulnerability.severity: Critical` to focus on the most dangerous ones first.Now we have **2 agents** enrolled and active in Wazuh:### What We're Seeing



---The **PCI DSS compliance** module showing how your environment aligns with Payment Card Industry standards.



## Step 27 — Configuration Assessment: CIS Benchmark Results| Agent ID | Name |



![Configuration Assessment — CIS Ubuntu 24.04 Benchmark](./images/27.png)|---|---|### Dashboard Elements



### What's Happening Here| **001** | `ip-172-31-27-237` |



We're in the **Configuration Assessment (SCA)** module for `test-server2`, viewing the **CIS Ubuntu Linux 24.04 LTS Benchmark v1.0.0** results.| **002** | `test-server2` || Element | Description |



### Scan Results|---|---|



| Status | Count |Both show as **Active** 🟢. We deployed the Wazuh Agent on the second EC2 instance using the same wizard (Steps 7-9) and it connected successfully.| **Requirements list** | All PCI DSS requirements with alert counts |

|---|---|

| ✅ **Passed** | 120 checks || **Compliance trend** | Improving or declining over time |

| ❌ **Failed** | 117 checks |

| ⬜ **Not Applicable** | 42 checks |---| **Top requirements triggered** | Which PCI DSS requirements have the most violations |



### Compliance Score| **Alert details** | Specific events tagged with PCI DSS mappings |



With 120 passed out of 237 applicable checks, the score is approximately **50%**.## Step 16 — Agents Detailed List View



### What This Means### Common PCI DSS Requirements in Wazuh



- **120 passed** — these security configurations are correctly set![Agents Detailed List — Both Active](./images/16.png)

- **117 failed** — these are **misconfigurations** that need to be fixed for hardening

- **42 not applicable** — checks that don't apply to this system| Requirement | What It Covers |



### Sample Check Visible### What's Happening Here|---|---|



- **ID:** 35500| **10.2.4** | Invalid logical access attempts |

- **Title:** "Ensure mounting of cramfs filesystems is disabled"

- **Policy:** CIS Ubuntu Linux 24.04 LTS Benchmark v1.0.0The **expanded agents table** showing full details for both agents:| **10.2.5** | Changes to identification/authentication mechanisms |



### What To Do With Failed Checks| **10.6.1** | Review all security events daily |



Click on any **failed check** to see:| Field | Agent 001 | Agent 002 || **11.2.1** | Run quarterly internal vulnerability scans |

- **What's wrong** — description of the misconfiguration

- **Rationale** — why it matters for security|---|---|---|| **11.4** | Intrusion detection and prevention techniques |

- **Remediation** — exact commands to fix it

- **Compliance mapping** — which standards it violates (PCI DSS, HIPAA, etc.)| **Name** | ip-172-31-27-237 | test-server2 |



---| **IP Address** | 172.31.27.237 | 172.31.25.172 |---



## Step 28 — Configuration Assessment: Events Tab| **Group** | default | default |



![Configuration Assessment — Events](./images/28.png)| **OS** | Ubuntu 24.04.3 LTS | Ubuntu 24.04.3 LTS |## 🇪🇺 Step 20 — GDPR Compliance Module



### What's Happening Here| **Cluster Node** | node01 | node01 |



We're in the **Configuration Assessment → Events** tab for `test-server2`. This shows the **raw SCA events** as a timeline.| **Version** | v4.14.3 | v4.14.3 |![GDPR](./images/20.png)



### What We See| **Status** | 🟢 Active | 🟢 Active |



| Element | Description |### What We're Seeing

|---|---|

| **Filter** | `manager.name: ip-172-31-17-228` and `rule.groups: sca` and `agent.id: 002` |### Key TakeawayThe **GDPR compliance** module — relevant if you handle EU citizen data.

| **Columns** | Timestamp, SCA event details |

| **Timeline** | Events plotted over time showing when SCA scans ran |

| **Timestamps** | Events clustered around `Feb 20, 2026 @ 11:22` |

Both agents are:### GDPR Articles Tracked

### How SCA Events Work

- Same OS (Ubuntu 24.04.3 LTS)

Every time the SCA module runs a scan (by default every 12 hours), it generates events for each check:

- Check passed → event logged- Same Wazuh version (v4.14.3)| Article | Focus |

- Check failed → event logged with alert

- Check status changed (was passing, now failing) → higher priority alert- In the same group (`default`)|---|---|



> These events feed into the SCA Dashboard (Step 27) to calculate the pass/fail/not-applicable counts.- Both reporting to the same cluster node (`node01`)| **Art. 5** | Principles relating to processing of personal data |



---- Both have action buttons for management (restart, upgrade, etc.)| **Art. 32** | Security of processing |



---| **Art. 33** | Notification of a personal data breach (within 72 hours!) |



# 📋 Lab Summary---| **Art. 35** | Data protection impact assessment |



## What We Did (Complete Flow)



### Phase 1: Infrastructure Setup (AWS)## Step 17 — Configuration Assessment (SCA) Dashboard---



| Step | Screenshot | What We Did |

|---|---|---|

| 1 | `1.png` | SSH into Ubuntu EC2 instance |![Configuration Assessment — test-server2](./images/17.png)## 🏥 Step 21 — HIPAA Compliance Module

| 2 | `2.png` | Ran Wazuh quickstart installation script |

| 3 | `3.png` | Accessed Dashboard — no agents yet |

| 4 | `4.png` | Explored Dashboard modules (sidebar) |

### What's Happening Here![HIPAA](./images/21.png)

### Phase 2: Initial Dashboard Exploration



| Step | Screenshot | What We Did |

|---|---|---|We're in the **Configuration Assessment (SCA)** module for `test-server2`. This module runs **CIS benchmark scans** to check if the system is configured securely.### What We're Seeing

| 5 | `5.png` | Explored Discover tab — `wazuh-alerts-*` index patterns, 404 hits |

| 6 | `6.png` | Threat Hunting — 78 authentication failure events |The **HIPAA compliance** module — for healthcare organizations protecting patient data (PHI).



### Phase 3: Agent Deployment### What We See



| Step | Screenshot | What We Did |### Key HIPAA Sections

|---|---|---|

| 7 | `7.png` | Deploy Agent wizard — selected Linux DEB amd64, server `34.224.212.65` || Element | Description |

| 8 | `8.png` | Confirmed package selection, updated server IP to `98.81.179.28` |

| 9 | `9.png` | Got the install command with `WAZUH_MANAGER='98.81.179.28'` ||---|---|| Section | What It Covers |

| 10 | `10.png` | Configured AWS Security Group `sg-0409dfe0b07ea77dd` (6 inbound rules) |

| 11 | `11.png` | Verified 2 EC2 instances running (`wazuh` = t2.medium, 2/2 checks passed) || **Dashboard tab** | Overview charts and metrics ||---|---|

| 12 | `12.png` | First agent enrolled — `001 ip-172-31-27-237` active |

| 13 | `13.png` | Viewed individual agent details (2 cores, 3.868 GB RAM, Intel Xeon) || **Inventory tab** | Detailed check-by-check results || **§164.308** | Administrative safeguards — risk analysis, access management |



### Phase 4: Second Agent Deployment| **Events tab** | Raw SCA events || **§164.310** | Physical safeguards — facility access, device controls |



| Step | Screenshot | What We Did || **Filter** | `manager.name: ip-172-31-17-228` and `rule.groups: sca` and `agent.id: 002` || **§164.312** | Technical safeguards — access control, audit controls, encryption |

|---|---|---|

| 14 | `14.png` | AWS EC2 console — managing instances (full sidebar visible) || **Time range** | Last 24 hours |

| 15 | `15.png` | Second agent enrolled — `002 test-server2` (never connected) |

| 16 | `16.png` | Verified both agents active — `172.31.27.237` and `172.31.25.172` |---



### Phase 5: Security Module Exploration> SCA scans run automatically when the agent is enrolled. It checks the system against CIS benchmarks and reports which checks **passed**, **failed**, or are **not applicable**.



| Step | Screenshot | What We Did |## 🇺🇸 Step 22 — NIST 800-53 Compliance Module

|---|---|---|

| 17 | `17.png` | SCA events for test-server2 — 280 hits, CIS checks |---

| 18 | `18.png` | Overview page — 2 active agents with data |

| 19 | `19.png` | IT Hygiene — system/platforms, Ubuntu 24.04.3 LTS, Kernel 6.14.0-1018-aws |![NIST 800-53](./images/22.png)

| 20 | `20.png` | IT Hygiene — network (top ports: 443, 1514, 80, 34007) |

| 21 | `21.png` | IT Hygiene — services (179 unique services, 356 total entries) |## Step 18 — Dashboard Overview with 2 Active Agents

| 22 | `22.png` | MITRE ATT&CK — dashboard, top tactics: Valid Accounts, Defense Evasion |

| 23 | `23.png` | MITRE ATT&CK — intelligence (150+ threat groups: admin@338, Ajax Security Team, Andariel) |### What We're Seeing

| 24 | `24.png` | MITRE ATT&CK — raw events with timestamps |

| 25 | `25.png` | Vulnerability Detection — 18 Critical, 344 High, 758 Medium, 134 Low, 1566 Pending |![Wazuh Overview — 2 Active Agents](./images/18.png)The **NIST 800-53** compliance module — the US federal security framework.

| 26 | `26.png` | Vulnerability events for test-server2 — 1,420 hits, libgnutls30t64 flaw |

| 27 | `27.png` | SCA CIS Ubuntu 24.04 Benchmark — 120 pass / 117 fail / 42 N/A (50% score) |

| 28 | `28.png` | SCA events timeline — clustered around 11:22 AM |

### What's Happening Here### Control Families Shown

---



## Key Takeaways

Back to the **Overview** page. Now the dashboard has data because we have **2 active agents**:| Code | Family | Wazuh Coverage |

1. **Wazuh needs agents to show data** — an empty dashboard just means no agents are enrolled yet

2. **AWS Security Groups are critical** — port 1514/1515 must be open for agents to connect|---|---|---|

3. **Agent deployment is simple** — the Dashboard wizard generates the exact install command

4. **IT Hygiene gives full inventory** — packages, ports, services, network interfaces### Agents Summary| **AC** | Access Control | Authentication monitoring |

5. **MITRE ATT&CK is built-in** — alerts are automatically mapped to tactics and techniques

6. **Vulnerability Detection found 1,000+ CVEs** — even a fresh Ubuntu server has hundreds of known vulnerabilities| **AU** | Audit & Accountability | Log collection, audit trails |

7. **CIS Benchmark score ~50%** — a default Ubuntu installation fails about half the security checks

8. **Everything connects back to `ossec.conf`** — modules must be enabled in config to generate data| Status | Count || **CM** | Configuration Management | SCA, FIM |



---|---|---|| **IR** | Incident Response | Active Response |



## References| 🟢 Active | 2 || **SI** | System & Info Protection | Malware detection |



- Wazuh Quick Start: https://documentation.wazuh.com/current/quickstart.html| 🔴 Disconnected | 0 |

- Wazuh Agent Deployment: https://documentation.wazuh.com/current/installation-guide/wazuh-agent/index.html

- Wazuh Dashboard: https://documentation.wazuh.com/current/getting-started/components/wazuh-dashboard.html---

- Capabilities: https://documentation.wazuh.com/current/user-manual/capabilities/index.html

- ossec.conf Reference: https://documentation.wazuh.com/current/user-manual/reference/ossec-conf/index.html### Modules Listed


## 🔐 Step 23 — TSC (SOC 2) Compliance Module

Now the Endpoint Security modules are visible and contain data:

- **Configuration Assessment** — SCA scan results available![TSC](./images/23.png)

- **File Integrity Monitoring** — File change events being collected

- **Malware Detection** — Rootcheck running### What We're Seeing

The **TSC (Trust Services Criteria)** module — used for **SOC 2 audits**, essential for SaaS companies.

> Compare this to Step 3 — the dashboard was empty. Now with 2 agents sending data, all modules are populated.

### Five Trust Principles

---

| Principle | Wazuh Coverage |

## Step 19 — IT Hygiene: System Dashboard|---|---|

| **Security** | FIM, authentication monitoring, malware detection |

![IT Hygiene — Dashboard, Platforms](./images/19.png)| **Availability** | System monitoring, agent status |

| **Processing Integrity** | Log integrity, FIM |

### What's Happening Here| **Confidentiality** | Access monitoring, encryption checks |

| **Privacy** | Data access monitoring |

We're exploring the **IT Hygiene** module (System Inventory). This shows the state of all monitored endpoints.

---

### Tabs Available

## 🐳 Step 24 — Docker (Container Security)

| Tab | What It Shows |

|---|---|![Docker Monitoring](./images/24.png)

| **Dashboard** | Overview with charts ← We're here |

| **OS / Hardware** | Operating system and hardware info |### What We're Seeing

| **Platform** | Platform distribution |The **Docker** module showing container activity from hosts running Docker.

| **System** | System-level details |

| **Software** | Installed packages |### Events Monitored



### What We See| Event | Why It Matters |

|---|---|

| Element | Value || **Container created/started** | Tracking container lifecycle |

|---|---|| **Container running privileged** | ⚠️ Full host access — serious security risk |

| **Cluster** | `ip-172-31-17-228` || **Exec into container** | Someone running commands inside — could be attacker |

| **Top 5 Platforms** | `ubuntu` (both agents run Ubuntu) || **Image pulled** | New image downloaded — is it trusted? |

| **Network changes** | Container networking modified |

> This data is collected by the `<syscollector>` module in `ossec.conf`. It inventories all software, hardware, network, and processes on each agent.

---

---

## ☁️ Step 25 — AWS Security Module

## Step 20 — IT Hygiene: Network Tab

![AWS Monitoring](./images/25.png)

![IT Hygiene — Network (Ports & Destinations)](./images/20.png)

### What We're Seeing

### What's Happening HereThe **Amazon Web Services** module showing events collected from AWS APIs (CloudTrail, etc.).



We're in the **IT Hygiene → Network** tab. This shows network-related inventory data.### Key AWS Events to Watch



### Tabs in Network Section| Event | Risk Level |

|---|---|

`Addresses` → `Interfaces` → `Protocols` → `Listeners` → `Traffic`| IAM user created with admin access | 🔴 Critical |

| S3 bucket made public | 🔴 Critical |

### What We See| Security group opened to 0.0.0.0/0 | 🔴 Critical |

| Root account login | 🟠 High |

| Element | Details || Console login without MFA | 🟠 High |

|---|---|| API call from unusual region | 🟡 Medium |

| **Filter** | `NOT destination_port:0` (filtering out port 0 noise) |

| **Top 5 Destination Ports** | **443** (HTTPS) and **1514** (Wazuh agent communication) |---



### Why Ports 443 and 1514?## 🔵 Step 26 — Google Cloud (GCP) Module



| Port | Why It Appears |![GCP Monitoring](./images/26.png)

|---|---|

| **443** | Agents/browsers connecting to the Wazuh Dashboard |### What We're Seeing

| **1514** | Agents sending event data to the Wazuh Manager |The **Google Cloud** module showing security events from GCP services.



These are the two most critical ports in a Wazuh deployment. Seeing them here confirms the agents are communicating properly.### GCP Events Tracked

- **Audit logs** — Admin activity, data access

---- **IAM changes** — User/service account modifications

- **Compute Engine** — Instance lifecycle events

## Step 21 — IT Hygiene: Services Tab- **Cloud Storage** — Bucket access and policy changes

- **VPC** — Network configuration changes

![IT Hygiene — Services](./images/21.png)

---

### What's Happening Here

## 📫 Step 27 — Office 365 / GitHub / Additional Cloud

We're in the **IT Hygiene → Identity → Services** tab. This shows all running services discovered on the monitored agents.

![Office 365 or GitHub](./images/27.png)

### What We See

### What We're Seeing

| Element | Details |Cloud service monitoring for **SaaS platforms** — Office 365, GitHub, or Microsoft Graph API.

|---|---|

| **Filter** | `wazuh.cluster.name: ip-172-31-17-228` |### Office 365 Events

| **Top 5 Services** | Listed services running across agents || Event | Why It Matters |

| **Service: apport** | Ubuntu crash reporting service ||---|---|

| **Total Unique Services** | 179 services discovered || Suspicious sign-in | Compromised account |

| Mail forwarding rule created | Data exfiltration via email |

### Why This Matters| External sharing enabled | Sensitive files exposed |

| Admin role assigned | Privilege escalation |

- **Unexpected services** = potential security risk (backdoor, unauthorized software)

- **Missing services** = something critical might not be running### GitHub Events

- Useful for **compliance audits** — verify only approved services are running| Event | Why It Matters |

|---|---|

---| Repository made public | Code exposure |

| Deploy key added | Persistent access |

## Step 22 — MITRE ATT&CK: Dashboard| Branch protection removed | Code integrity risk |

| External collaborator added | Unauthorized access |

![MITRE ATT&CK — Dashboard & Tactics](./images/22.png)

---

### What's Happening Here

## ⚙️ Step 28 — Server Management / Settings

We're in the **MITRE ATT&CK** module — **Dashboard** tab. This maps Wazuh alerts to the MITRE ATT&CK framework.

![Server Management](./images/28.png)

### What We See

### What We're Seeing

| Element | Description |The **Server Management** or **Settings** section of the Dashboard — where you manage the Wazuh platform itself.

|---|---|

| **Alerts evolution over time** | Timeline chart showing ATT&CK-tagged alerts |### What You Can Do Here

| **Top tactics** | Most detected tactics (e.g., Valid Accounts, Defense Evasion) |

| **Filter** | `manager.name: ip-172-31-17-228` and `rule.mitre.id: exists` || Section | Purpose |

| **Time range** | Last 24 hours ||---|---|

| **Status** | Check if all Wazuh services are running |

### What Are MITRE Tactics?| **Logs** | View server logs for troubleshooting |

| **Statistics** | Events per second, queue sizes, performance metrics |

| Tactic | What It Means || **Configuration** | View/edit server configuration |

|---|---|| **Rules** | Browse default and custom detection rules |

| **Valid Accounts** | Attackers using legitimate credentials || **Decoders** | Browse default and custom log decoders |

| **Defense Evasion** | Techniques to avoid detection || **CDB Lists** | Manage IOC databases (malware hashes, bad IPs) |

| **Persistence** | Maintaining access after initial compromise || **Groups** | Manage agent groups and shared configuration |

| **Privilege Escalation** | Gaining higher-level permissions || **API Console** | Direct API access for automation |

| **Ruleset Test** | Test how logs get decoded and which rules trigger |

> Every Wazuh rule can be mapped to MITRE ATT&CK tactics and techniques. This turns isolated alerts into a **coherent attack narrative**.

---

---

---

## Step 23 — MITRE ATT&CK: Intelligence Tab

# 📋 Complete Lab Summary

![MITRE ATT&CK — Intelligence (Groups, Mitigations, Software)](./images/23.png)

## What We Did (Step by Step)

### What's Happening Here

### Phase 1: Setup & Configuration

We're exploring the **MITRE ATT&CK → Intelligence** tab. This is a built-in reference database of threat intelligence.1. **Installed Wazuh** All-in-One on a single node (quickstart)

2. **Logged into the Dashboard** at `https://<ip>:443`

### Sections Available3. **Configured `ossec.conf`** to enable key modules:

   - `<global>` → Enabled JSON output + archives (`logall`)

| Section | What It Contains |   - `<rootcheck>` → Enabled rootkit/trojan detection

|---|---|   - `<syscollector>` → Enabled full system inventory

| **Groups** | 150+ known threat actor groups (APT28, Lazarus, etc.) |   - `<sca>` → Enabled CIS benchmark scanning

| **Mitigations** | Recommended countermeasures for each technique |   - `<vulnerability-detection>` → Enabled CVE detection

| **Software** | Known malware and tools used by threat actors |   - `<syscheck>` → Enabled File Integrity Monitoring

| **Tactics** | The 14 ATT&CK tactic categories |4. **Restarted the manager** → `sudo systemctl restart wazuh-manager`

| **Techniques** | Specific attack techniques with descriptions |

### Phase 2: Dashboard Exploration

### Why This Is Useful

| Screenshot | Module | Category | What We Explored |

You can **search for any threat group** and see:|---|---|---|---|

- What techniques they use| 1-2 | **Overview** | Home | Security events summary, alert levels, top agents |

- What software/malware they deploy| 3-4 | **Agents** | Management | Agent list, status, individual agent dashboard |

- What mitigations defend against them| 5-6 | **Configuration Assessment** | Endpoint Security | CIS benchmark results, pass/fail, remediation |

| 7 | **Malware Detection** | Endpoint Security | Rootkit alerts, IOC matches, suspicious files |

> This is built into Wazuh — no extra tools needed. It's like having a MITRE ATT&CK encyclopedia inside your SIEM.| 8-9 | **File Integrity Monitoring** | Endpoint Security | File changes, permissions, who/what/when |

| 10-11 | **Threat Hunting** | Threat Intelligence | Alert search, filtering, JSON drill-down |

---| 12-13 | **Vulnerability Detection** | Threat Intelligence | CVE inventory, severity, remediation |

| 14-15 | **MITRE ATT&CK** | Threat Intelligence | Tactics/techniques mapping, attack context |

## Step 24 — MITRE ATT&CK: Events Tab| 16-18 | **IT Hygiene** | Security Operations | System inventory — packages, ports, network |

| 19 | **PCI DSS** | Security Operations | Payment card compliance dashboard |

![MITRE ATT&CK — Events](./images/24.png)| 20 | **GDPR** | Security Operations | EU data privacy compliance |

| 21 | **HIPAA** | Security Operations | Healthcare data security compliance |

### What's Happening Here| 22 | **NIST 800-53** | Security Operations | Federal security controls compliance |

| 23 | **TSC** | Security Operations | SOC 2 audit readiness |

We're in the **MITRE ATT&CK → Events** tab. This shows the **raw alert data** for events that matched MITRE ATT&CK rules.| 24 | **Docker** | Cloud Security | Container lifecycle and security events |

| 25 | **AWS** | Cloud Security | CloudTrail, IAM, S3, security group events |

### What We See| 26 | **Google Cloud** | Cloud Security | GCP audit logs, IAM, compute events |

| 27 | **Office 365 / GitHub** | Cloud Security | SaaS platform security events |

| Element | Description || 28 | **Server Management** | Settings | Platform management, rules, decoders, API |

|---|---|

| **Filter** | `manager.name: ip-172-31-17-228` |---

| **Columns** | Timestamp, alert details |

| **Export** | `Export Formatted` button to download results |## 🧩 How the Config Maps to What We Saw

| **Timestamps** | Events from `Feb 20, 2026 @ 11:12` to `11:21` |

```

Each row is an individual alert that has been tagged with a MITRE ATT&CK technique ID. You can click any row to see the full JSON with:  ossec.conf                   Dashboard Module            Screenshots

- Rule ID and description  ──────────                  ──────────────              ────────────

- MITRE technique and tactic  <global>

- Agent details    logall=yes          ──▶   Threat Hunting              10-11

- Raw log data    alerts_log=yes      ──▶   All alert-based modules



---  <rootcheck>           ──▶   Malware Detection           7



## Step 25 — Vulnerability Detection: Dashboard  <syscollector>        ──▶   IT Hygiene (Inventory)      16-18

                        ──▶   Vulnerability Detection     12-13

![Vulnerability Detection — Dashboard Overview](./images/25.png)

  <sca>                 ──▶   Configuration Assessment    5-6

### What's Happening Here

  <vulnerability-       ──▶   Vulnerability Detection     12-13

We're in the **Vulnerability Detection** module — **Dashboard** tab. This shows all known CVEs (Common Vulnerabilities and Exposures) detected on our agents.    detection>



### Vulnerability Counts  <syscheck>            ──▶   File Integrity Monitoring   8-9



| Severity | Count |  <indexer>             ──▶   (Stores all indexed data)   All modules

|---|---|```

| 🔴 **Critical** | 18 |

| 🟠 **High** | 344 |---

| 🟡 **Medium** | 758 |

| 🔵 **Low** | 134 |## 💡 Key Takeaways from This Lab

| ⚪ **Pending Evaluation** | 1,566 |

### 1. Everything starts at `ossec.conf`

### Top Sections> If a module isn't enabled in the config, you'll see **empty dashboards** with no data. Always verify your config before troubleshooting "missing data."



| Section | What It Shows |### 2. The Dashboard has 4 clear categories

|---|---|> **Endpoint Security** (what's on your machines) → **Threat Intelligence** (what threats exist) → **Security Operations** (are you compliant?) → **Cloud Security** (what's in the cloud?)

| **Top 5 Vulnerabilities** | Most common CVEs (e.g., `CVE-2022-3219` — 22 occurrences) |

| **Top OS** | `Ubuntu 24.04.3 LTS (Noble Numbat)` — 2,820 vulnerabilities |### 3. Every alert answers the 5 WH questions

| **Top 5 Agents** | `test-server2` has the most vulnerabilities |> WHO, WHAT, WHERE, WHEN, HOW — if your alert doesn't answer these, it's a notification, not an alert.

| **Top 5 Packages** | Most vulnerable installed packages |

### 4. Compliance comes free

### How It Works> Every alert is automatically tagged with PCI DSS, GDPR, HIPAA, NIST, TSC mappings. You don't configure this separately — it's built into the rules.



1. The `<syscollector>` module inventories all installed packages### 5. The Investigation Workflow

2. The `<vulnerability-detection>` module cross-references package versions against CVE databases```

3. Results appear here with severity ratingsOverview (spot the spike) 

  → Threat Hunting (search & filter) 

> This is powered by `<vulnerability-detection><enabled>yes</enabled></vulnerability-detection>` in `ossec.conf`.    → Alert JSON (full context) 

      → MITRE ATT&CK (attack narrative) 

---        → Incident Response (take action)

```

## Step 26 — Vulnerability Detection: Events (Filtered by Agent)

### 6. Restart is mandatory

![Vulnerability Detection — Events for test-server2](./images/26.png)> `sudo systemctl restart wazuh-manager` — Nothing in `ossec.conf` takes effect without this step. Ever.



### What's Happening Here---



We're in the **Vulnerability Detection → Events** tab, filtered to show vulnerabilities for **test-server2** specifically.## 🔗 References

- Wazuh Dashboard: https://documentation.wazuh.com/current/getting-started/components/wazuh-dashboard.html

### What We See- Capabilities: https://documentation.wazuh.com/current/user-manual/capabilities/index.html

- ossec.conf Reference: https://documentation.wazuh.com/current/user-manual/reference/ossec-conf/index.html

| Element | Description |- SOC 201 Podcast: https://studywithurvesh.notion.site/SOC-201-Podcast-2cea317e9ac48071ab99c70e32795341

|---|---|
| **Filter** | `agent.name: test-server2` |
| **Status** | Evaluated / Under evaluation |
| **Total hits** | 1,420 vulnerabilities on this agent |
| **Columns** | Agent name, Package name, Package version, Vulnerability description, Severity |
| **Export** | Can export the full list as formatted data |

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

### What This Means

- **120 passed** — these security configurations are correctly set
- **117 failed** — these are **misconfigurations** that need to be fixed for hardening
- **42 not applicable** — checks that don't apply to this system

### Compliance Score

With 120 passed out of 237 applicable checks, the score is approximately **50.6%**. This means the system needs significant hardening.

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
| **Filter** | `manager.name: ip-172-31-17-228` |
| **Columns** | Timestamp, event details |
| **Timeline** | Events plotted over time showing when SCA scans ran |
| **Count** | Number of SCA events generated |

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
| 1 | `images/1.png` | SSH into Ubuntu EC2 instance |
| 2 | `images/2.png` | Ran Wazuh quickstart installation script |
| 3 | `images/3.png` | Accessed Dashboard — no agents yet |
| 4 | `images/4.png` | Explored Dashboard modules (sidebar) |

### Phase 2: Initial Dashboard Exploration

| Step | Screenshot | What We Did |
|---|---|---|
| 5 | `images/5.png` | Explored Discover tab — index patterns |
| 6 | `images/6.png` | Threat Hunting — authentication failure events |

### Phase 3: Agent Deployment

| Step | Screenshot | What We Did |
|---|---|---|
| 7 | `images/7.png` | Deploy Agent wizard — selected Linux |
| 8 | `images/8.png` | Selected DEB amd64 package type |
| 9 | `images/9.png` | Got the install command with server IP |
| 10 | `images/10.png` | Configured AWS Security Group (open ports 1514, 1515) |
| 11 | `images/11.png` | Verified 2 EC2 instances running |
| 12 | `images/12.png` | First agent enrolled and active |
| 13 | `images/13.png` | Viewed individual agent details |

### Phase 4: Second Agent Deployment

| Step | Screenshot | What We Did |
|---|---|---|
| 14 | `images/14.png` | AWS EC2 console — managing instances |
| 15 | `images/15.png` | Second agent enrolled (2 agents total) |
| 16 | `images/16.png` | Verified both agents active with details |

### Phase 5: Security Module Exploration

| Step | Screenshot | What We Did |
|---|---|---|
| 17 | `images/17.png` | Configuration Assessment (SCA) dashboard |
| 18 | `images/18.png` | Overview page — 2 active agents with data |
| 19 | `images/19.png` | IT Hygiene — system overview, platforms |
| 20 | `images/20.png` | IT Hygiene — network (ports 443, 1514) |
| 21 | `images/21.png` | IT Hygiene — services inventory |
| 22 | `images/22.png` | MITRE ATT&CK — dashboard, tactics |
| 23 | `images/23.png` | MITRE ATT&CK — intelligence database |
| 24 | `images/24.png` | MITRE ATT&CK — raw events |
| 25 | `images/25.png` | Vulnerability Detection — dashboard (18 Critical, 344 High) |
| 26 | `images/26.png` | Vulnerability Detection — events for test-server2 |
| 27 | `images/27.png` | Configuration Assessment — CIS Ubuntu benchmark (120 pass / 117 fail) |
| 28 | `images/28.png` | Configuration Assessment — events timeline |

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

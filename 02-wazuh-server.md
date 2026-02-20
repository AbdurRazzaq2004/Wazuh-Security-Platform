# 🧠 Wazuh Server — The Brain That Analyzes Everything

## What is the Wazuh Server?

The **Wazuh Server** is the **central component** that receives data from all agents, analyzes it, detects threats, and generates security alerts.

> **In simple words:** If the Wazuh Agents are security cameras scattered across your building, the **Wazuh Server is the security control room** where trained analysts watch all the feeds, spot suspicious activity, and sound the alarm. Except it's all automated and runs 24/7 without coffee breaks.

---

## 🎯 What Does It Do?

| Function | Description |
|---|---|
| **Analyzes data** | Processes events from agents using decoders and rules |
| **Detects threats** | Uses threat intelligence to find Indicators of Compromise (IOCs) |
| **Generates alerts** | Creates alerts when suspicious activity is found |
| **Manages agents** | Remotely configures, upgrades, and monitors agents |
| **Enriches data** | Maps events to MITRE ATT&CK, regulatory standards |
| **Integrates** | Connects with Slack, Jira, ServiceNow, PagerDuty, VirusTotal, etc. |
| **Forwards data** | Sends analyzed data to the Wazuh Indexer for storage |

---

## 🏗️ Server Architecture

The Wazuh Server runs on **Linux** (physical machines, VMs, containers, or cloud). On Windows, you deploy it using **Docker**.

![Wazuh Server Architecture](https://documentation.wazuh.com/current/_images/wazuh-server-architecture1.png)

---

## 🧩 Server Components Explained (One by One)

### 📝 1. Agent Enrollment Service
**What it does:** Registers new agents and gives them unique authentication keys.

> **Simple Analogy:** When a new employee joins your company, HR gives them an ID badge. The Agent Enrollment Service is the "HR" of Wazuh — it registers each new agent and gives it a unique identity.

- Listens on **port 1515/TCP** by default
- Supports **TLS/SSL** certificate-based authentication
- Also supports **password-based** enrollment
- Generates a **unique key** per agent for secure communication

**How it works:**
```
New Agent ──enrollment request──▶ Enrollment Service
                                        │
                                  Generates unique key
                                        │
         ◀──sends key back────────────────┘
```

---

### 🔌 2. Agent Connection Service
**What it does:** Manages the ongoing communication between agents and the server.

> **Simple Analogy:** Think of this as the **phone line operator**. Once an agent has its ID badge (enrollment key), the Connection Service keeps the phone line open, ensures the calls are encrypted, and manages what instructions to send back to the agent.

- Listens on **port 1514/TCP** by default
- **Validates agent identity** using enrollment keys
- **Encrypts** all communications (AES-256)
- Enables **centralized configuration management** — push new settings to agents remotely

---

### 🔍 3. Analysis Engine (The Core!)
**What it does:** This is the **heart** of the Wazuh Server. It processes all incoming security data and decides what's normal and what's a threat.

> **Simple Analogy:** Imagine a detective receiving thousands of tips per day. The Analysis Engine is that detective — it reads every tip (event), figures out what kind of tip it is (decoder), checks it against known crime patterns (rules), and raises the alarm when something matches.

The Analysis Engine works in **two steps**:

#### Step 1: Decoders (Classify & Extract)
Decoders figure out **what kind of log** it is and **extract the important fields**.

**Example:**
```
Raw log: "Feb 19 10:30:15 server sshd[12345]: Failed password for root from 192.168.1.100"

Decoder extracts:
  - Type: SSH authentication event
  - Username: root
  - Source IP: 192.168.1.100
  - Action: Failed password
  - Timestamp: Feb 19 10:30:15
```

#### Step 2: Rules (Match & Alert)
Rules check the decoded events against **known threat patterns**.

**Example:**
```
Rule: "If there are 5+ failed SSH login attempts from the same IP within 2 minutes"
Match: ✅ YES — 192.168.1.100 had 8 failed attempts in 90 seconds
Alert: "Possible SSH brute-force attack from 192.168.1.100"
Action: Block IP via Active Response
```

When a rule is triggered, the engine can:
- ✅ Generate an **alert**
- ✅ **Block** an IP address
- ✅ **Kill** a malicious process
- ✅ **Delete** a malware file

---

### 🌐 4. Wazuh Server API (RESTful API)
**What it does:** Provides a programmatic interface for interacting with the Wazuh Server.

> **Simple Analogy:** If the Wazuh Dashboard is the "steering wheel" of your car, the API is the **engine underneath**. The dashboard uses the API to talk to the server, but you can also talk to the API directly (via command line, scripts, or other tools).

- Listens on **port 55000/TCP**
- Uses **HTTPS** with username/password authentication
- Used by the **Wazuh Dashboard** under the hood

**What you can do via the API:**
| Action | Example |
|---|---|
| Manage agents | List all agents, restart an agent, change agent groups |
| Configure the server | Update decoders, rules, and settings |
| Monitor health | Check server status, cluster health, queue sizes |
| Query alerts | Search alerts by severity, time range, agent |
| Manage rules | Create, update, or delete detection rules |

**Quick API Example:**
```bash
# List all agents
curl -k -X GET "https://wazuh-server:55000/agents" \
  -H "Authorization: Bearer <token>"

# Get server status
curl -k -X GET "https://wazuh-server:55000/manager/status" \
  -H "Authorization: Bearer <token>"
```

---

### 🔗 5. Wazuh Cluster Daemon
**What it does:** Allows you to link **multiple Wazuh Servers** together into a cluster for scalability and high availability.

> **Simple Analogy:** If one security control room can monitor 1,000 cameras, what happens when you have 10,000 cameras? You add more control rooms and connect them together. That's what the Cluster Daemon does.

- **Horizontal scaling:** Add more servers as your environment grows
- **High availability:** If one server goes down, others keep working
- **Load balancing:** Distribute agent connections across servers
- Uses a **master-worker** architecture:
  - **Master node:** Manages the cluster configuration
  - **Worker nodes:** Process agent data

**Cluster Architecture:**
```
                   ┌─────────────────────┐
                   │   Load Balancer     │
                   └──────┬──────┬───────┘
                          │      │
              ┌───────────┘      └───────────┐
              ▼                              ▼
   ┌──────────────────┐          ┌──────────────────┐
   │  Server (Master)  │◀────────▶  Server (Worker)  │
   │  500 agents       │  sync   │  500 agents       │
   └──────────────────┘          └──────────────────┘
```

---

### 📤 6. Filebeat
**What it does:** Forwards analyzed events and alerts from the Wazuh Server to the Wazuh Indexer.

> **Simple Analogy:** Filebeat is the **delivery truck**. The Analysis Engine processes the events and puts them in a box. Filebeat picks up that box and delivers it to the Wazuh Indexer (the warehouse) where it's stored and made searchable.

- Reads the Wazuh Server's **output data** (alerts and events)
- Sends data to the Wazuh Indexer over **TLS-encrypted** connection
- Default destination: **port 9200/TCP** on the Indexer

**Data Flow:**
```
Analysis Engine ──generates alerts──▶ Filebeat ──TLS (9200)──▶ Wazuh Indexer
```

---

## 🛡️ Threat Intelligence Capabilities

The Wazuh Server doesn't just analyze — it **enriches** alerts with intelligence:

| Source | What It Adds |
|---|---|
| **MITRE ATT&CK** | Maps events to known attack tactics and techniques |
| **Wazuh CTI** | Checks against known CVEs and vulnerability data |
| **PCI DSS** | Tags alerts relevant to payment card security |
| **GDPR** | Tags alerts relevant to data privacy regulations |
| **HIPAA** | Tags alerts relevant to healthcare data security |
| **CIS Benchmarks** | Tags configuration compliance findings |
| **NIST 800-53** | Tags alerts against US government security controls |

---

## 🔗 Third-Party Integrations

The server can send alerts to external platforms:

| Platform | Use Case |
|---|---|
| **Slack** | Get security alerts in your team chat |
| **Jira** | Auto-create tickets for security incidents |
| **ServiceNow** | Track incidents in your ITSM |
| **PagerDuty** | Wake up on-call engineers for critical alerts |
| **VirusTotal** | Scan suspicious files against malware database |
| **MISP** | Enrich alerts with threat intelligence |

---

## 📏 Scalability

| Setup | Capacity |
|---|---|
| **Single server** | Hundreds to thousands of agents |
| **Server cluster** | Tens of thousands of agents |
| **With load balancer** | Virtually unlimited (add more workers) |

---

## 💡 Key Takeaway

> The Wazuh Server is **where the magic happens**. It takes raw data from agents, decodes it, matches it against threat rules, enriches it with intelligence, generates alerts, and forwards everything to the Indexer. A single server can handle thousands of agents, and you can cluster them for larger environments.

---

## 🔗 References
- Official Docs: https://documentation.wazuh.com/current/getting-started/components/wazuh-server.html
- Server API Docs: https://documentation.wazuh.com/current/user-manual/api/index.html
- Installation Guide: https://documentation.wazuh.com/current/installation-guide/wazuh-server/index.html

# 🏗️ Wazuh Architecture — How Everything Connects

## Architecture Overview

The Wazuh architecture consists of **two main areas**:

1. **Endpoint Security Agent** — deployed on every machine you want to monitor
2. **Central Components** — the server-side infrastructure that processes, stores, and visualizes data

> **In simple words:** Think of it like a hospital security system. The **agents** are security cameras and sensors in every room (endpoints). The **central components** are the security control room (server), the evidence locker (indexer), and the monitoring screens (dashboard).

---

## 🖼️ Architecture Diagram

The image below shows the complete Wazuh architecture with all components and how they connect:

![Wazuh Architecture](https://documentation.wazuh.com/current/_images/deployment-architecture1.png)

---

## 🔍 Detailed Architecture Breakdown

### Left Side: Endpoint Security Agent

The agent has two main parts:

#### Agent Modules (Data Collectors)
These are the "sensors" on each endpoint:

| Module | What It Watches |
|---|---|
| Active Response | Executes automated actions when threats are detected |
| Command Execution | Runs authorized commands and reports output |
| Configuration Assessment | Checks if system config follows security benchmarks |
| Container Security | Monitors Docker/container environments |
| Cloud Security | Watches AWS/Azure/GCP via their APIs |
| File Integrity Monitoring | Detects file changes (create/modify/delete) |
| Log Collector | Reads system and application logs |
| Malware Detection | Looks for rootkits, hidden processes, anomalies |
| System Inventory | Collects installed software, ports, processes |

#### Agent Daemon (The Manager)
The daemon is the "brain" of the agent:

| Function | What It Does |
|---|---|
| Data Encryption | Encrypts all data before sending (AES-256) |
| Modules Management | Controls which modules are active |
| Remote Configuration | Accepts configuration updates from the server |
| Server Authentication | Authenticates with the server using unique keys |

The daemon funnels all collected data through **Data Flow Control** — which handles queuing, compression, and flow control before sending to the server.

---

### Right Side: Central Components

#### Wazuh Server
The server receives and processes all data:

| Service | What It Does | Port |
|---|---|---|
| Agent Connection Service | Manages communication with agents | 1514/TCP |
| Agent Enrollment Service | Registers new agents, issues keys | 1515/TCP |
| Syslog Listener | Receives logs from agentless devices | 514/TCP/UDP |
| RESTful API Service | Programmatic access to the server | 55000/TCP |

**Analysis Engine** — the core of threat detection:
- **Decoding and rule matching** — Classifies logs and matches against threat rules
- **Correlation and enrichment** — Connects related events and adds context

**Outputs from the server:**
- 🔴 **Threat Intelligence** — Matches against known threats
- 🔵 **Vulnerability Detection** — Identifies known CVEs
- 🟣 **Regulatory Compliance** — Maps to PCI DSS, GDPR, HIPAA, etc.

#### Filebeat (Data Shipper)
Sits between the Server and Indexer. Picks up:
- **Raw data events** — All processed events
- **Security alerts** — Events that matched detection rules

Sends them to the Wazuh Indexer over TLS.

#### Wazuh Indexer
Stores and makes data searchable:

| Function | Details |
|---|---|
| Search Engine | Full-text search across all security data |
| Data Analytics | Powers all visualizations and statistics |
| Long-term Storage | Retains data for forensics and compliance |

Stores four types of data:
- Raw data events
- Security alerts
- Inventory data
- Agent monitoring data

#### Wazuh Dashboard
Your visual interface:

| Function | Details |
|---|---|
| Query Language | Search using full-text queries |
| Visualizations & Dashboards | Charts, graphs, maps, tables |
| Reports Engine | Generate PDF/CSV reports for audits |

---

## 🔄 Component Communication

### 1. Agent ↔ Server

```
┌──────────────┐                    ┌──────────────┐
│  Wazuh Agent │ ══AES-256══════▶  │ Wazuh Server │
│  (Endpoint)  │    TCP:1514       │  (Analysis)  │
└──────────────┘                    └──────────────┘
```

| Detail | Value |
|---|---|
| **Protocol** | TCP (default) or UDP |
| **Port** | 1514/TCP |
| **Encryption** | AES-256 (128-bit blocks, 256-bit keys) |
| **Alternative** | Blowfish (optional) |
| **Compression** | Real-time compression |
| **Flow Control** | Queues events to prevent flooding |

### 2. Server → Indexer

```
┌──────────────┐                    ┌──────────────┐
│ Wazuh Server │ ══Filebeat═════▶  │Wazuh Indexer │
│  + Filebeat  │   TLS:9200        │  (Storage)   │
└──────────────┘                    └──────────────┘
```

| Detail | Value |
|---|---|
| **Shipper** | Filebeat |
| **Protocol** | HTTPS/TLS |
| **Port** | 9200/TCP |
| **Data** | Alerts and events in JSON format |

### 3. Dashboard ↔ Server & Indexer

```
┌──────────────────┐
│  Wazuh Dashboard │
│    (Port 443)    │
└────────┬─────────┘
         │
    ┌────┴────┐
    ▼         ▼
┌────────┐ ┌────────┐
│ Server │ │Indexer │
│ :55000 │ │ :9200  │
└────────┘ └────────┘
```

| Connection | Port | Encryption | Auth | Purpose |
|---|---|---|---|---|
| Dashboard → Server API | 55000/TCP | TLS | Username/Password | Config, agent mgmt, status |
| Dashboard → Indexer API | 9200/TCP | TLS | Username/Password | Query alerts, visualizations |

---

## 📡 Agentless Monitoring

Not every device can run a Wazuh agent. For these, Wazuh supports **agentless monitoring**:

> **Simple Example:** Your network firewall (Cisco ASA) can't run a Wazuh agent, but it can send logs via Syslog. Wazuh Server listens on port 514 and receives those logs for analysis.

Supported methods:
- **Syslog** — Devices forward logs to port 514 on the server
- **SSH** — Server connects to devices periodically to pull configuration data
- **APIs** — Server queries device APIs for status and configuration

Agentless devices:
- 🔥 Firewalls
- 🔀 Network switches
- 🌐 Routers
- 📡 Access points
- 🖨️ Network IDS/IPS

---

## 🚀 Deployment Architectures

### Option 1: All-in-One (Lab/Testing)

```
┌─────────────────────────────────────┐
│         Single Machine              │
│                                     │
│  ┌─────────┐ ┌─────────┐ ┌──────┐  │
│  │ Server  │ │ Indexer │ │Dashb.│  │
│  └─────────┘ └─────────┘ └──────┘  │
│                                     │
└─────────────────────────────────────┘
```

| Pros | Cons |
|---|---|
| Easy to set up | No redundancy |
| Minimal resources needed | Performance limits |
| Great for learning/testing | Not for production |

**Best for:** Labs, PoCs, small environments (<50 agents)

---

### Option 2: Single-Node (Medium)

```
┌──────────┐    ┌──────────┐    ┌──────────┐
│  Server  │    │  Indexer  │    │Dashboard │
│ Machine 1│    │ Machine 2 │    │Machine 3 │
└──────────┘    └──────────┘    └──────────┘
```

| Pros | Cons |
|---|---|
| Better performance | Still single points of failure |
| Separated workloads | Not highly available |
| Easier to scale | Requires 3 machines |

**Best for:** Medium environments (50-500 agents)

---

### Option 3: Multi-Node Cluster (Enterprise)

```
                    ┌──────────────┐
                    │ Load Balancer│
                    └──────┬───────┘
                           │
              ┌────────────┼────────────┐
              ▼            ▼            ▼
        ┌──────────┐ ┌──────────┐ ┌──────────┐
        │ Server   │ │ Server   │ │ Server   │
        │ (Master) │ │ (Worker) │ │ (Worker) │
        └─────┬────┘ └─────┬────┘ └─────┬────┘
              │             │            │
              └──────┬──────┘────────────┘
                     ▼
        ┌──────────────────────────────────┐
        │      Indexer Cluster             │
        │  ┌────────┐┌────────┐┌────────┐ │
        │  │ Node 1 ││ Node 2 ││ Node 3 │ │
        │  └────────┘└────────┘└────────┘ │
        └──────────────────────────────────┘
                     │
                     ▼
              ┌──────────────┐
              │  Dashboard   │
              └──────────────┘
```

| Pros | Cons |
|---|---|
| High availability | More complex setup |
| Load balancing | More resources needed |
| Fault tolerance | Requires expertise to manage |
| Horizontal scaling | Higher cost |

**Best for:** Large environments (500+ agents), production, enterprise

---

## 🔌 Required Ports Summary

| Port | Protocol | Component | Direction | Purpose |
|---|---|---|---|---|
| 1514 | TCP | Server | Agent → Server | Agent data transmission |
| 1515 | TCP | Server | Agent → Server | Agent enrollment |
| 514 | TCP/UDP | Server | Device → Server | Syslog from agentless devices |
| 55000 | TCP | Server API | Dashboard → Server | Management & status |
| 9200 | TCP | Indexer API | Server/Dashboard → Indexer | Data indexing & search |
| 443 | TCP | Dashboard | User → Dashboard | Web UI access |

---

## 🛡️ Wazuh CTI (Cyber Threat Intelligence)

Wazuh has its own **Cyber Threat Intelligence** service:

> **In simple words:** Think of it as Wazuh's own **intelligence agency** that constantly monitors the threat landscape, collecting information about new vulnerabilities (CVEs), their severity, and how to fix them.

| Feature | Details |
|---|---|
| **Focus** | Vulnerability intelligence |
| **Data sources** | OS vendors, major vulnerability databases |
| **Updates** | Timely CVE updates, severity scores, exploitability insights |
| **Integration** | Built into the Vulnerability Detection module |
| **Public access** | Available at https://cti.wazuh.com/ |

---

## 🔒 Security Throughout

Every connection in the Wazuh architecture is **encrypted and authenticated**:

| Connection | Encryption | Authentication |
|---|---|---|
| Agent → Server | AES-256 | Per-agent unique keys |
| Server → Indexer (Filebeat) | TLS | Certificate-based |
| Dashboard → Server API | TLS | Username/Password |
| Dashboard → Indexer | TLS | Username/Password |
| Cluster node ↔ node | TLS | Certificate-based |

---

## 💡 Key Takeaway

> The Wazuh architecture is designed for **security, scalability, and flexibility**. Data flows from agents through encrypted channels to the server for analysis, then to the indexer for storage, and finally to the dashboard for visualization. You can start with a simple all-in-one setup and scale to a multi-node cluster as your environment grows. Every component communicates securely, and agentless monitoring covers devices that can't run an agent.

---

## 🔗 References
- Architecture Docs: https://documentation.wazuh.com/current/getting-started/architecture.html
- Components Overview: https://documentation.wazuh.com/current/getting-started/components/index.html
- Installation Guide: https://documentation.wazuh.com/current/installation-guide/index.html
- Deployment Options: https://documentation.wazuh.com/current/deployment-options/index.html

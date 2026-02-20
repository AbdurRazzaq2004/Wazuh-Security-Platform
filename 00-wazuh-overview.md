# 🛡️ Wazuh — Complete Overview

## What is Wazuh?

**Wazuh** is a **free and open-source security platform** that combines **XDR** (Extended Detection & Response) and **SIEM** (Security Information & Event Management) capabilities into one unified solution.

> **In simple words:** Think of Wazuh as a **security guard system for your entire IT infrastructure** — it watches over your servers, laptops, cloud instances, and containers 24/7, looking for anything suspicious, and alerts you when something goes wrong.

---

## 🎯 What Does Wazuh Do?

| Capability | What It Means |
|---|---|
| **Log Data Analysis** | Reads and analyzes logs from all your systems to find suspicious activity |
| **Intrusion Detection** | Detects if someone is trying to break into your systems |
| **Malware Detection** | Finds malicious software on your endpoints |
| **File Integrity Monitoring** | Alerts you if important files are changed, deleted, or created |
| **Configuration Assessment** | Checks if your systems are configured securely (following CIS benchmarks) |
| **Vulnerability Detection** | Scans for known software vulnerabilities (CVEs) on your systems |
| **Regulatory Compliance** | Helps you meet standards like PCI DSS, HIPAA, GDPR, NIST 800-53 |
| **Cloud Security** | Monitors AWS, Azure, and GCP cloud environments |
| **Container Security** | Monitors Docker containers and Kubernetes environments |
| **Active Response** | Automatically blocks threats (e.g., blocking an attacker's IP) |

---

## 🏗️ Architecture at a Glance

Wazuh has **two main parts**:

### 1️⃣ Endpoint Security Agent (runs on your machines)
- **Agent Modules** — The "eyes and ears" on each machine (collects data)
- **Agent Daemon** — The "brain" on each machine (encrypts, manages, sends data)

### 2️⃣ Central Components (your security headquarters)
- **Wazuh Server** — The "analyst" that processes and analyzes all data
- **Wazuh Indexer** — The "database" that stores all security events and alerts
- **Wazuh Dashboard** — The "control panel" where you see everything visually

![Wazuh Architecture](https://documentation.wazuh.com/current/_images/deployment-architecture1.png)

---

## 🔄 How Data Flows Through Wazuh

```
  Your Machines                    Security HQ
┌─────────────┐               ┌──────────────────┐
│ Wazuh Agent │──encrypted──▶│  Wazuh Server    │
│ (collects   │  TCP:1514    │  (analyzes data)  │
│  data)      │               │                  │
└─────────────┘               └────────┬─────────┘
                                       │
                                       │ Filebeat (TCP:9200)
                                       ▼
                              ┌──────────────────┐
                              │  Wazuh Indexer   │
                              │  (stores data)   │
                              └────────┬─────────┘
                                       │
                                       ▼
                              ┌──────────────────┐
                              │ Wazuh Dashboard  │
                              │ (visualizes data)│
                              └──────────────────┘
```

---

## 🔌 Default Ports

| Port | Service | Purpose |
|---|---|---|
| **1514/TCP** | Agent Connection | Agents send data to the server |
| **1515/TCP** | Agent Enrollment | New agents register with the server |
| **514/TCP/UDP** | Syslog | Receives syslog from agentless devices |
| **55000/TCP** | Wazuh Server API | Dashboard talks to the server |
| **9200/TCP** | Wazuh Indexer API | Server sends data to the indexer |
| **443/TCP** | Wazuh Dashboard | You access the web UI |

---

## 🚀 Deployment Options

| Deployment Type | Description | Best For |
|---|---|---|
| **All-in-one** | Server + Indexer + Dashboard on ONE machine | Labs, small environments |
| **Single-node** | Each component on its own server | Medium environments |
| **Multi-node** | Clustered servers and indexers across many machines | Large enterprises, high availability |

---

## 📂 Documentation Files in This Folder

| File | Description |
|---|---|
| [01-wazuh-agent.md](./01-wazuh-agent.md) | The Wazuh Agent — your eyes on every endpoint |
| [02-wazuh-server.md](./02-wazuh-server.md) | The Wazuh Server — the brain that analyzes everything |
| [03-wazuh-indexer.md](./03-wazuh-indexer.md) | The Wazuh Indexer — the database that stores it all |
| [04-wazuh-dashboard.md](./04-wazuh-dashboard.md) | The Wazuh Dashboard — your visual control panel |
| [05-wazuh-architecture.md](./05-wazuh-architecture.md) | Full architecture & communication details |
| [06-wazuh-use-cases.md](./06-wazuh-use-cases.md) | Real-world use cases and capabilities |

---

## 🔗 Useful Links

- **Official Docs:** https://documentation.wazuh.com/current/getting-started/index.html
- **Components:** https://documentation.wazuh.com/current/getting-started/components/index.html
- **Architecture:** https://documentation.wazuh.com/current/getting-started/architecture.html
- **Use Cases:** https://documentation.wazuh.com/current/getting-started/use-cases/index.html
- **Quickstart:** https://documentation.wazuh.com/current/quickstart.html
- **GitHub:** https://github.com/wazuh

---

> 📝 **Author's Note:** These docs are written from the perspective of a Senior DevOps Engineer & Security Specialist to help you understand Wazuh's components without drowning in jargon. Each component has its own detailed file — start with whichever interests you most!

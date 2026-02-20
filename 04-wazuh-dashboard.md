# 📊 Wazuh Dashboard — Your Visual Security Command Center

## What is the Wazuh Dashboard?

The **Wazuh Dashboard** is the **web-based user interface** where you see, search, analyze, and manage all your security data.

> **In simple words:** If Wazuh were a car, the Dashboard is the **windshield and steering wheel**. It's where you see everything that's happening (alerts, vulnerabilities, compliance) and where you control the system (configure agents, manage rules, generate reports). Everything else (Agent, Server, Indexer) works behind the scenes — the Dashboard is what **you** interact with.

---

## 🎯 What Does It Do?

| Function | Description |
|---|---|
| **Visualize alerts** | See security events in beautiful dashboards and charts |
| **Search events** | Full-text search across all indexed security data |
| **Manage agents** | Monitor agent status, configure modules, push settings |
| **Manage the platform** | Configure the server, check logs, view cluster health |
| **Regulatory compliance** | Pre-built dashboards for PCI DSS, GDPR, HIPAA, CIS, NIST |
| **Threat hunting** | Investigate suspicious activity across your environment |
| **Generate reports** | Export PDF/CSV reports for audits and management |
| **Developer tools** | Test rules/decoders, interact with APIs directly |
| **Access control** | RBAC (Role-Based Access Control) and SSO (Single Sign-On) |

---

## 🖥️ Accessing the Dashboard

The Dashboard is a web application you access via your browser:

```
https://<your-wazuh-dashboard>:443
```

- Default port: **443/TCP** (HTTPS)
- Requires **username and password** authentication
- Supports **RBAC** — different users see different things
- Supports **SSO** — integrate with your identity provider

---

## 🏗️ Dashboard Features (One by One)

### 📈 1. Data Visualization & Analysis

This is the **main feature** — gorgeous dashboards that show your security posture at a glance.

> **Simple Analogy:** It's like the dashboard of your car but for security. Instead of speed and fuel, you see "threats detected," "vulnerable systems," and "compliance score."

**Pre-built dashboards include:**

| Dashboard | What It Shows |
|---|---|
| **Threat Hunting** | All security events, filterable by agent, rule, severity |
| **Vulnerability Detection** | Known CVEs across all your endpoints |
| **File Integrity Monitoring** | Files created, modified, or deleted across systems |
| **Security Configuration Assessment** | CIS benchmark compliance per agent |
| **Malware Detection** | Detected malware and rootkit activity |
| **System Inventory** | Software, ports, processes, interfaces per agent |
| **MITRE ATT&CK** | Events mapped to attack tactics and techniques |
| **PCI DSS** | Payment card industry compliance status |
| **GDPR** | EU data protection compliance status |
| **HIPAA** | Healthcare data security compliance status |
| **NIST 800-53** | US government security controls compliance |
| **Cloud Security** | AWS, Azure, GCP security events |

**What you can do:**
- 🔍 **Search** any field across millions of events
- 📊 **Create custom visualizations** (bar charts, pie charts, tables, maps)
- 📋 **Build custom dashboards** tailored to your needs
- 📄 **Generate reports** as PDF or CSV

![Data Visualization](https://documentation.wazuh.com/current/_images/data-visualization1.png)

---

### 🖥️ 2. Agent Monitoring & Configuration

Manage all your agents from one place — no need to SSH into every machine.

> **Simple Example:** You have 500 servers. You want to enable File Integrity Monitoring on all of them and add `/etc/shadow` to the monitored files list. Instead of logging into 500 machines, you do it from the Dashboard.

**What you can do:**
- ✅ See **all agents** and their status (active, disconnected, never connected)
- ✅ View agent **details** (OS, IP, version, last check-in)
- ✅ **Configure** which modules are enabled per agent
- ✅ **Define** which log files to read
- ✅ **Set** which files/directories to monitor with FIM
- ✅ **Choose** which SCA checks to perform

![Agents Monitoring](https://documentation.wazuh.com/current/_images/agents-monitoring1.png)

---

### ⚙️ 3. Platform Management

Manage the entire Wazuh platform from the Dashboard.

> **Simple Example:** Want to check if the Wazuh Server is healthy? Want to see how many events per second it's processing? Want to add a new decoder? All from the Dashboard.

**What you can manage:**

| Area | What You Can Do |
|---|---|
| **Server status** | Monitor server health, logs, and statistics |
| **Cluster health** | See all nodes in your cluster and their status |
| **Rules** | View, create, and edit detection rules |
| **Decoders** | View, create, and edit log decoders |
| **CDB Lists** | Manage Constant DataBase lists (IOC lists) |
| **Configuration** | View and modify server settings |

![Platform Management](https://documentation.wazuh.com/current/_images/platform-management1.png)

---

### 🛠️ 4. Developer Tools

Built-in tools for testing and debugging — super useful for DevOps engineers.

#### Ruleset Test Tool
Test how a log message will be decoded and which rules it will match — **without** deploying anything.

> **Simple Example:** You wrote a custom rule to detect a specific attack pattern. Before deploying it to production, you paste a sample log into the Ruleset Test tool. It shows you exactly how it's decoded and whether your rule triggers.

![Ruleset Test](https://documentation.wazuh.com/current/_images/ruleset-test1.png)

#### API Consoles
Interact with both the **Wazuh Server API** and the **Wazuh Indexer API** directly from the browser.

> **Simple Example:** Instead of opening a terminal and running `curl` commands, you use the built-in API console to query agent status, search alerts, or manage rules.

**Wazuh Server API Console:**
```
GET /agents?status=active
GET /rules?search=ssh
PUT /agents/003/restart
```

![Server API Console](https://documentation.wazuh.com/current/_images/server-api-console1.png)

**Wazuh Indexer API Console:**
```
GET /wazuh-alerts-*/_search
GET /_cat/indices
GET /_cluster/health
```

![Indexer API Console](https://documentation.wazuh.com/current/_images/indexer-api-console1.png)

---

## 🔐 Security & Access Control

### Role-Based Access Control (RBAC)
Not everyone should see everything. RBAC lets you control access:

| Role | Access Level |
|---|---|
| **Admin** | Full access to everything |
| **Analyst** | View alerts, search events, but can't change config |
| **Auditor** | View compliance dashboards and generate reports only |
| **Custom roles** | Define exactly what each role can see and do |

### Single Sign-On (SSO)
Integrate with your identity provider:
- **SAML 2.0** (Okta, Azure AD, OneLogin)
- **OpenID Connect** (Keycloak, Auth0)

---

## 🔄 How the Dashboard Communicates

```
                    ┌────────────────────┐
                    │   Wazuh Dashboard  │
                    │    (Port 443)      │
                    └────────┬───────────┘
                             │
              ┌──────────────┼──────────────┐
              ▼                             ▼
   ┌──────────────────┐         ┌──────────────────┐
   │  Wazuh Server    │         │  Wazuh Indexer   │
   │  API (55000)     │         │  API (9200)      │
   │                  │         │                  │
   │  Config, status, │         │  Alerts, events, │
   │  agent mgmt      │         │  search, charts  │
   └──────────────────┘         └──────────────────┘
```

- **Dashboard → Server API (55000):** For configuration, agent management, server status
- **Dashboard → Indexer API (9200):** For querying alerts, visualizations, search
- All communications use **TLS encryption**

---

## 🎨 Out-of-the-Box Compliance Dashboards

One of the biggest selling points for enterprises — ready-made compliance views:

### PCI DSS (Payment Card Industry)
If you handle credit card data, PCI DSS compliance is mandatory. The Dashboard shows which requirements you're meeting and which need attention.

### GDPR (EU Data Protection)
For organizations handling EU citizen data. See which GDPR articles your security controls address.

### HIPAA (Healthcare)
For healthcare organizations. Monitor compliance with patient data security requirements.

### CIS Benchmarks
See how well your systems are configured according to CIS hardening guides.

### NIST 800-53
For US government agencies and contractors. Map security events to NIST controls.

---

## 💡 Key Takeaway

> The Wazuh Dashboard is your **single pane of glass** for everything security. It turns raw security data into actionable insights through visualizations, provides tools to manage your entire deployment, and includes developer tools for testing and debugging. If you're a DevOps engineer or security analyst, this is where you'll spend most of your time.

---

## 🔗 References
- Official Docs: https://documentation.wazuh.com/current/getting-started/components/wazuh-dashboard.html
- Installation Guide: https://documentation.wazuh.com/current/installation-guide/wazuh-dashboard/index.html

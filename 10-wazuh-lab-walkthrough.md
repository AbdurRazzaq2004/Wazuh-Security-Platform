# 🧪 Wazuh Dashboard Lab Walkthrough — Complete Hands-On Guide

> **Lab Environment:** Wazuh All-in-One (Single Node) Deployment
> **Purpose:** Explore every section of the Wazuh Dashboard, understand what each module does, and learn how to navigate as a SOC Analyst / DevOps Engineer.

---

## 📌 What This Lab Covers

In this lab, we installed Wazuh on a single node, enrolled agents, configured `ossec.conf` to enable key security modules, and then explored **every major section** of the Wazuh Dashboard. Below is a walkthrough of each step with screenshots.

---

---

## 🖥️ Step 1 — Wazuh Dashboard Login & Overview

![Wazuh Dashboard Login / Overview](./images/1.png)

### What We're Seeing
When you first access the Wazuh Dashboard (via `https://<your-ip>:443`), you land on the **Overview** page. This is your **security command center** — a bird's-eye view of your entire monitored environment.

### Key Things on the Overview Page

| Element | What It Shows |
|---|---|
| **Total Agents** | How many agents are enrolled (active, disconnected, never connected) |
| **Security Events** | Total number of security events over the selected time range |
| **Alert Level Distribution** | Breakdown of alerts by severity (low, medium, high, critical) |
| **Top MITRE ATT&CK Tactics** | Most common attack techniques detected |
| **Top 5 Agents** | Agents generating the most alerts |
| **Top 5 Rules** | Most frequently triggered detection rules |

> 💡 *"The overview tab is self-explanatory, yet... Dashboards are for visibility, not for in-depth investigation."* — SOC 201 Podcast

---

## 📊 Step 2 — Security Events Overview

![Security Events Overview](./images/2.png)

### What We're Seeing
The security events timeline and distribution. This shows the **volume and pattern** of alerts over time.

### Why This Matters
- **Spikes** in the timeline = something happened (attack, misconfiguration, new agent online)
- **Flat lines** at zero = either nothing is happening (good) or your detection isn't working (bad!)
- Use the **time picker** (top right) to zoom into specific time ranges

### SOC Analyst Workflow
```
1. Look at the timeline → Any unusual spikes?
2. Check alert levels → How many high/critical?
3. Check top agents → Which machines are noisiest?
4. Check top rules → What's being detected most?
5. Drill into specifics → Click to investigate
```

---

## 🔌 Step 3 — Agents Overview

![Agents Overview](./images/3.png)

### What We're Seeing
The **Agents** section shows all enrolled Wazuh agents with their status, OS, IP address, and agent version.

### Agent Statuses

| Status | Color | Meaning |
|---|---|---|
| **Active** | 🟢 Green | Agent is connected and sending data |
| **Disconnected** | 🔴 Red | Agent was connected but lost connection |
| **Never Connected** | ⚫ Grey | Agent was enrolled but never established a connection |
| **Pending** | 🟡 Yellow | Agent is waiting for enrollment |

### What You Can Do Here
- Click on any agent to see its **detailed view** (events, inventory, config)
- Filter agents by **OS, status, group**
- See agent **version** (check if any agents need upgrading)
- Check **last keep-alive** time (when did the agent last check in?)

---

## 🖥️ Step 4 — Individual Agent Dashboard

![Individual Agent Details](./images/4.png)

### What We're Seeing
When you click on a specific agent, you get its **dedicated dashboard** — everything about that one endpoint.

### Sections Available Per Agent

| Section | What It Shows |
|---|---|
| **Overview** | Summary of events, alert levels, top rules for this agent |
| **Configuration** | Current agent configuration (which modules are enabled) |
| **Inventory** | OS, packages, processes, ports, network interfaces |
| **SCA** | Security Configuration Assessment results for this agent |
| **FIM** | File Integrity Monitoring events on this agent |
| **Vulnerabilities** | Known CVEs affecting this specific agent |

---

## ✅ Step 5 — Configuration Assessment (SCA)

![SCA Module Overview](./images/5.png)

### What We're Seeing
The **Security Configuration Assessment** module showing CIS benchmark scan results.

### What You See

| Column | Description |
|---|---|
| **Policy** | The benchmark being checked (e.g., CIS Ubuntu 22.04) |
| **Score** | Compliance percentage (higher is better) |
| **Passed** | Number of checks that passed |
| **Failed** | Number of checks that failed (these need fixing!) |
| **Not Applicable** | Checks that don't apply to this system |

### How to Read It
- **Score 39%** = Only 39% of CIS checks pass. Your system needs hardening.
- Click on **Failed** checks to see exactly what's wrong and how to fix it.
- Each failed check includes **Remediation** steps.

> This module is powered by `<sca><enabled>yes</enabled></sca>` in `ossec.conf`.

---

## ✅ Step 6 — SCA Policy Details

![SCA Policy Details](./images/6.png)

### What We're Seeing
Drilling into a specific SCA policy to see **individual check results**.

### Check Result Details

| Field | What It Shows |
|---|---|
| **ID** | Unique check identifier (e.g., 28500) |
| **Title** | What's being checked (e.g., "Ensure /tmp is a separate partition") |
| **Status** | Pass ✅ / Fail ❌ / Not Applicable ⬜ |
| **Rationale** | Why this check matters for security |
| **Remediation** | Step-by-step instructions to fix the issue |
| **Compliance** | Which standards this maps to (CIS, PCI DSS, HIPAA, etc.) |

> 💡 **Pro Tip:** Sort by **Failed** first. These are your actionable items. Fix the easy ones first to quickly improve your score.

---

## 🦠 Step 7 — Malware Detection Module

![Malware Detection](./images/7.png)

### What We're Seeing
The **Malware Detection** module showing alerts related to malware activity, rootkits, and IOCs.

### Types of Alerts You'll See Here

| Alert Type | Description |
|---|---|
| **Rootkit detection** | Known rootkit files or behaviors found |
| **Trojaned binary** | System binary replaced with malicious version |
| **Hidden process** | Process running but invisible to `ps` |
| **Hidden port** | Network port open but invisible to `netstat` |
| **Suspicious file** | Executable dropped in commonly abused directories |
| **Threat intelligence match** | File hash/IP matches known malware database |

> This module is powered by `<rootcheck><disabled>no</disabled></rootcheck>` in `ossec.conf`.

---

## 📁 Step 8 — File Integrity Monitoring (FIM)

![FIM Overview](./images/8.png)

### What We're Seeing
The **File Integrity Monitoring** module showing file change events across monitored endpoints.

### What FIM Tracks

| Event Type | Icon | Description |
|---|---|---|
| **Added** | ➕ | New file was created |
| **Modified** | ✏️ | Existing file was changed |
| **Deleted** | 🗑️ | File was removed |

### Dashboard Elements
- **Events over time** — Timeline of file changes
- **Top agents** — Which endpoints have the most file changes
- **Top files** — Most frequently modified files
- **Actions summary** — Count of added/modified/deleted

> This module is powered by `<syscheck><disabled>no</disabled></syscheck>` in `ossec.conf`.

---

## 📁 Step 9 — FIM Event Details

![FIM Event Details](./images/9.png)

### What We're Seeing
Drilling into a specific FIM event to see **exactly what changed**.

### Event Detail Fields

| Field | What It Shows |
|---|---|
| **File path** | `/etc/passwd`, `/etc/shadow`, etc. |
| **Event type** | Modified / Added / Deleted |
| **Changed attributes** | Which attributes changed (content, permissions, owner, etc.) |
| **Old value → New value** | What the attribute was before and after |
| **User** | Who made the change |
| **Process** | Which process made the change |
| **Timestamp** | Exactly when the change occurred |
| **MD5 / SHA1 / SHA256** | File hash before and after (proves content changed) |

> 💡 **This answers the 5 WH questions:** WHO changed it, WHAT was changed, WHERE (file path), WHEN (timestamp), and HOW (which process).

---

## 🔎 Step 10 — Threat Hunting Module

![Threat Hunting](./images/10.png)

### What We're Seeing
The **Threat Hunting** module — the SOC analyst's primary investigation workspace.

### How to Use Threat Hunting
This is a **raw alert explorer** with powerful search and filtering:

| Feature | How to Use It |
|---|---|
| **Search bar** | Type any keyword — IP, username, rule ID, file path |
| **Time range** | Use the date picker to narrow down to a specific window |
| **Filters** | Add filters for agent, rule level, rule group, MITRE technique |
| **Columns** | Customize which columns are shown in the results table |
| **Alert details** | Click any row to expand the full JSON alert |

### SOC Investigation Flow
```
1. Start with a time range (when did the incident happen?)
2. Filter by alert level ≥ 10 (focus on high severity)
3. Search for the IOC (IP, hash, domain, username)
4. Expand alerts to read full context
5. Correlate — are multiple agents affected?
6. Document findings for the incident report
```

---

## 🔎 Step 11 — Threat Hunting: Alert Details

![Threat Hunting Alert Details](./images/11.png)

### What We're Seeing
Expanding a specific alert in the Threat Hunting module to see its **full JSON payload**.

### What the JSON Contains

```json
{
  "agent": { "id": "001", "name": "ubuntu-server", "ip": "10.0.1.50" },
  "rule": {
    "id": "5710",
    "level": 10,
    "description": "SSH brute-force attack",
    "mitre": { "id": ["T1110"], "tactic": ["Credential Access"] },
    "pci_dss": ["10.2.4"],
    "gdpr": ["IV_35.7.d"]
  },
  "data": {
    "srcip": "192.168.1.100",
    "dstuser": "root"
  },
  "full_log": "sshd[12345]: Failed password for root from 192.168.1.100..."
}
```

> Every field is searchable. Every field can be used as a filter. This is the power of Wazuh + Indexer.

---

## 💀 Step 12 — Vulnerability Detection Overview

![Vulnerability Detection Overview](./images/12.png)

### What We're Seeing
The **Vulnerability Detection** module showing known CVEs across all monitored endpoints.

### Dashboard Elements

| Element | What It Shows |
|---|---|
| **Total vulnerabilities** | Count across all agents |
| **Severity chart** | Critical / High / Medium / Low breakdown |
| **Top vulnerable agents** | Which endpoints need patching first |
| **Top CVEs** | Most common vulnerabilities |
| **Vulnerability table** | Detailed list with CVE ID, package, severity, agent |

> This module is powered by `<vulnerability-detection><enabled>yes</enabled></vulnerability-detection>` in `ossec.conf` and requires `<syscollector>` to be enabled.

---

## 💀 Step 13 — Vulnerability Alert Details

![Vulnerability Alert Details](./images/13.png)

### What We're Seeing
Drilling into a specific vulnerability to see its **full details**.

### Vulnerability Detail Fields

| Field | Example |
|---|---|
| **CVE** | CVE-2024-1234 |
| **Severity** | Critical (CVSS 9.8) |
| **Package** | openssl 3.0.2-0ubuntu1 |
| **Status** | Active / Fixed |
| **Description** | Buffer overflow allowing remote code execution... |
| **References** | Links to NVD, vendor advisories |
| **Remediation** | Upgrade to version X.Y.Z |
| **Affected agent** | web-server-01 |

---

## ⚔️ Step 14 — MITRE ATT&CK Module

![MITRE ATT&CK Overview](./images/14.png)

### What We're Seeing
The **MITRE ATT&CK** module showing security events mapped to adversary tactics and techniques.

### What the MITRE View Shows

| Element | Description |
|---|---|
| **Tactics bar chart** | Which of the 14 ATT&CK tactics have been observed |
| **Techniques table** | Specific techniques with alert counts |
| **Heatmap** | Visual representation of technique frequency |

### Reading the MITRE Dashboard
- **Most common tactics** = where your environment is being targeted most
- **Rare tactics** = either you're not being targeted OR your detection has gaps
- Click any technique to see the **specific alerts** that triggered it

---

## ⚔️ Step 15 — MITRE ATT&CK Technique Details

![MITRE ATT&CK Technique Details](./images/15.png)

### What We're Seeing
Drilling into a specific MITRE ATT&CK technique to see which **alerts and agents** are associated.

### Why This Matters
Instead of seeing isolated alerts, you now see them in the **context of an attack chain**:
```
T1110 (Brute Force) → Multiple failed SSH logins from same IP
T1078 (Valid Accounts) → Successful login after brute force
T1059 (Command Execution) → Suspicious commands run post-login
T1547 (Persistence) → New startup script created
```

> 💡 This turns random alerts into a **coherent attack narrative**.

---

## 🔧 Step 16 — IT Hygiene / System Inventory

![IT Hygiene](./images/16.png)

### What We're Seeing
The **IT Hygiene** module (also known as System Inventory) showing the state of your endpoints.

### Inventory Categories

| Category | What It Shows |
|---|---|
| **Packages** | All installed software with versions |
| **Processes** | Currently running processes |
| **Ports** | Open/listening ports and associated services |
| **Network** | Network interfaces, IPs, MACs |
| **OS** | Operating system name, version, kernel |
| **Hardware** | CPU, RAM, serial numbers |

> This data is collected by `<wodle name="syscollector">` in `ossec.conf`.

---

## 📦 Step 17 — System Inventory: Packages

![System Inventory Packages](./images/17.png)

### What We're Seeing
The **Packages** view within IT Hygiene — every installed package on the selected agent.

### Why This Is Useful
- **Find vulnerable software:** Search for a specific package name/version
- **Shadow IT:** Detect unauthorized software installations
- **Compliance:** Verify only approved software is installed
- **Patch management:** Compare versions across agents

---

## 🌐 Step 18 — System Inventory: Network / Ports

![System Inventory Network](./images/18.png)

### What We're Seeing
The **Network interfaces and open ports** view.

### Key Things to Look For

| Red Flag | What It Means |
|---|---|
| **Unexpected open port** | A service is running that shouldn't be (backdoor?) |
| **Port 0.0.0.0:XXXX** | Service listening on ALL interfaces (should it be?) |
| **Unknown process on a port** | Unfamiliar process bound to a network port |
| **New interface** | New network interface appeared (VM escape? tunnel?) |

---

## 💳 Step 19 — PCI DSS Compliance Module

![PCI DSS](./images/19.png)

### What We're Seeing
The **PCI DSS compliance** module showing how your environment aligns with Payment Card Industry standards.

### Dashboard Elements

| Element | Description |
|---|---|
| **Requirements list** | All PCI DSS requirements with alert counts |
| **Compliance trend** | Improving or declining over time |
| **Top requirements triggered** | Which PCI DSS requirements have the most violations |
| **Alert details** | Specific events tagged with PCI DSS mappings |

### Common PCI DSS Requirements in Wazuh

| Requirement | What It Covers |
|---|---|
| **10.2.4** | Invalid logical access attempts |
| **10.2.5** | Changes to identification/authentication mechanisms |
| **10.6.1** | Review all security events daily |
| **11.2.1** | Run quarterly internal vulnerability scans |
| **11.4** | Intrusion detection and prevention techniques |

---

## 🇪🇺 Step 20 — GDPR Compliance Module

![GDPR](./images/20.png)

### What We're Seeing
The **GDPR compliance** module — relevant if you handle EU citizen data.

### GDPR Articles Tracked

| Article | Focus |
|---|---|
| **Art. 5** | Principles relating to processing of personal data |
| **Art. 32** | Security of processing |
| **Art. 33** | Notification of a personal data breach (within 72 hours!) |
| **Art. 35** | Data protection impact assessment |

---

## 🏥 Step 21 — HIPAA Compliance Module

![HIPAA](./images/21.png)

### What We're Seeing
The **HIPAA compliance** module — for healthcare organizations protecting patient data (PHI).

### Key HIPAA Sections

| Section | What It Covers |
|---|---|
| **§164.308** | Administrative safeguards — risk analysis, access management |
| **§164.310** | Physical safeguards — facility access, device controls |
| **§164.312** | Technical safeguards — access control, audit controls, encryption |

---

## 🇺🇸 Step 22 — NIST 800-53 Compliance Module

![NIST 800-53](./images/22.png)

### What We're Seeing
The **NIST 800-53** compliance module — the US federal security framework.

### Control Families Shown

| Code | Family | Wazuh Coverage |
|---|---|---|
| **AC** | Access Control | Authentication monitoring |
| **AU** | Audit & Accountability | Log collection, audit trails |
| **CM** | Configuration Management | SCA, FIM |
| **IR** | Incident Response | Active Response |
| **SI** | System & Info Protection | Malware detection |

---

## 🔐 Step 23 — TSC (SOC 2) Compliance Module

![TSC](./images/23.png)

### What We're Seeing
The **TSC (Trust Services Criteria)** module — used for **SOC 2 audits**, essential for SaaS companies.

### Five Trust Principles

| Principle | Wazuh Coverage |
|---|---|
| **Security** | FIM, authentication monitoring, malware detection |
| **Availability** | System monitoring, agent status |
| **Processing Integrity** | Log integrity, FIM |
| **Confidentiality** | Access monitoring, encryption checks |
| **Privacy** | Data access monitoring |

---

## 🐳 Step 24 — Docker (Container Security)

![Docker Monitoring](./images/24.png)

### What We're Seeing
The **Docker** module showing container activity from hosts running Docker.

### Events Monitored

| Event | Why It Matters |
|---|---|
| **Container created/started** | Tracking container lifecycle |
| **Container running privileged** | ⚠️ Full host access — serious security risk |
| **Exec into container** | Someone running commands inside — could be attacker |
| **Image pulled** | New image downloaded — is it trusted? |
| **Network changes** | Container networking modified |

---

## ☁️ Step 25 — AWS Security Module

![AWS Monitoring](./images/25.png)

### What We're Seeing
The **Amazon Web Services** module showing events collected from AWS APIs (CloudTrail, etc.).

### Key AWS Events to Watch

| Event | Risk Level |
|---|---|
| IAM user created with admin access | 🔴 Critical |
| S3 bucket made public | 🔴 Critical |
| Security group opened to 0.0.0.0/0 | 🔴 Critical |
| Root account login | 🟠 High |
| Console login without MFA | 🟠 High |
| API call from unusual region | 🟡 Medium |

---

## 🔵 Step 26 — Google Cloud (GCP) Module

![GCP Monitoring](./images/26.png)

### What We're Seeing
The **Google Cloud** module showing security events from GCP services.

### GCP Events Tracked
- **Audit logs** — Admin activity, data access
- **IAM changes** — User/service account modifications
- **Compute Engine** — Instance lifecycle events
- **Cloud Storage** — Bucket access and policy changes
- **VPC** — Network configuration changes

---

## 📫 Step 27 — Office 365 / GitHub / Additional Cloud

![Office 365 or GitHub](./images/27.png)

### What We're Seeing
Cloud service monitoring for **SaaS platforms** — Office 365, GitHub, or Microsoft Graph API.

### Office 365 Events
| Event | Why It Matters |
|---|---|
| Suspicious sign-in | Compromised account |
| Mail forwarding rule created | Data exfiltration via email |
| External sharing enabled | Sensitive files exposed |
| Admin role assigned | Privilege escalation |

### GitHub Events
| Event | Why It Matters |
|---|---|
| Repository made public | Code exposure |
| Deploy key added | Persistent access |
| Branch protection removed | Code integrity risk |
| External collaborator added | Unauthorized access |

---

## ⚙️ Step 28 — Server Management / Settings

![Server Management](./images/28.png)

### What We're Seeing
The **Server Management** or **Settings** section of the Dashboard — where you manage the Wazuh platform itself.

### What You Can Do Here

| Section | Purpose |
|---|---|
| **Status** | Check if all Wazuh services are running |
| **Logs** | View server logs for troubleshooting |
| **Statistics** | Events per second, queue sizes, performance metrics |
| **Configuration** | View/edit server configuration |
| **Rules** | Browse default and custom detection rules |
| **Decoders** | Browse default and custom log decoders |
| **CDB Lists** | Manage IOC databases (malware hashes, bad IPs) |
| **Groups** | Manage agent groups and shared configuration |
| **API Console** | Direct API access for automation |
| **Ruleset Test** | Test how logs get decoded and which rules trigger |

---

---

# 📋 Complete Lab Summary

## What We Did (Step by Step)

### Phase 1: Setup & Configuration
1. **Installed Wazuh** All-in-One on a single node (quickstart)
2. **Logged into the Dashboard** at `https://<ip>:443`
3. **Configured `ossec.conf`** to enable key modules:
   - `<global>` → Enabled JSON output + archives (`logall`)
   - `<rootcheck>` → Enabled rootkit/trojan detection
   - `<syscollector>` → Enabled full system inventory
   - `<sca>` → Enabled CIS benchmark scanning
   - `<vulnerability-detection>` → Enabled CVE detection
   - `<syscheck>` → Enabled File Integrity Monitoring
4. **Restarted the manager** → `sudo systemctl restart wazuh-manager`

### Phase 2: Dashboard Exploration

| Screenshot | Module | Category | What We Explored |
|---|---|---|---|
| 1-2 | **Overview** | Home | Security events summary, alert levels, top agents |
| 3-4 | **Agents** | Management | Agent list, status, individual agent dashboard |
| 5-6 | **Configuration Assessment** | Endpoint Security | CIS benchmark results, pass/fail, remediation |
| 7 | **Malware Detection** | Endpoint Security | Rootkit alerts, IOC matches, suspicious files |
| 8-9 | **File Integrity Monitoring** | Endpoint Security | File changes, permissions, who/what/when |
| 10-11 | **Threat Hunting** | Threat Intelligence | Alert search, filtering, JSON drill-down |
| 12-13 | **Vulnerability Detection** | Threat Intelligence | CVE inventory, severity, remediation |
| 14-15 | **MITRE ATT&CK** | Threat Intelligence | Tactics/techniques mapping, attack context |
| 16-18 | **IT Hygiene** | Security Operations | System inventory — packages, ports, network |
| 19 | **PCI DSS** | Security Operations | Payment card compliance dashboard |
| 20 | **GDPR** | Security Operations | EU data privacy compliance |
| 21 | **HIPAA** | Security Operations | Healthcare data security compliance |
| 22 | **NIST 800-53** | Security Operations | Federal security controls compliance |
| 23 | **TSC** | Security Operations | SOC 2 audit readiness |
| 24 | **Docker** | Cloud Security | Container lifecycle and security events |
| 25 | **AWS** | Cloud Security | CloudTrail, IAM, S3, security group events |
| 26 | **Google Cloud** | Cloud Security | GCP audit logs, IAM, compute events |
| 27 | **Office 365 / GitHub** | Cloud Security | SaaS platform security events |
| 28 | **Server Management** | Settings | Platform management, rules, decoders, API |

---

## 🧩 How the Config Maps to What We Saw

```
  ossec.conf                   Dashboard Module            Screenshots
  ──────────                  ──────────────              ────────────
  <global>
    logall=yes          ──▶   Threat Hunting              10-11
    alerts_log=yes      ──▶   All alert-based modules

  <rootcheck>           ──▶   Malware Detection           7

  <syscollector>        ──▶   IT Hygiene (Inventory)      16-18
                        ──▶   Vulnerability Detection     12-13

  <sca>                 ──▶   Configuration Assessment    5-6

  <vulnerability-       ──▶   Vulnerability Detection     12-13
    detection>

  <syscheck>            ──▶   File Integrity Monitoring   8-9

  <indexer>             ──▶   (Stores all indexed data)   All modules
```

---

## 💡 Key Takeaways from This Lab

### 1. Everything starts at `ossec.conf`
> If a module isn't enabled in the config, you'll see **empty dashboards** with no data. Always verify your config before troubleshooting "missing data."

### 2. The Dashboard has 4 clear categories
> **Endpoint Security** (what's on your machines) → **Threat Intelligence** (what threats exist) → **Security Operations** (are you compliant?) → **Cloud Security** (what's in the cloud?)

### 3. Every alert answers the 5 WH questions
> WHO, WHAT, WHERE, WHEN, HOW — if your alert doesn't answer these, it's a notification, not an alert.

### 4. Compliance comes free
> Every alert is automatically tagged with PCI DSS, GDPR, HIPAA, NIST, TSC mappings. You don't configure this separately — it's built into the rules.

### 5. The Investigation Workflow
```
Overview (spot the spike) 
  → Threat Hunting (search & filter) 
    → Alert JSON (full context) 
      → MITRE ATT&CK (attack narrative) 
        → Incident Response (take action)
```

### 6. Restart is mandatory
> `sudo systemctl restart wazuh-manager` — Nothing in `ossec.conf` takes effect without this step. Ever.

---

## 🔗 References
- Wazuh Dashboard: https://documentation.wazuh.com/current/getting-started/components/wazuh-dashboard.html
- Capabilities: https://documentation.wazuh.com/current/user-manual/capabilities/index.html
- ossec.conf Reference: https://documentation.wazuh.com/current/user-manual/reference/ossec-conf/index.html
- SOC 201 Podcast: https://studywithurvesh.notion.site/SOC-201-Podcast-2cea317e9ac48071ab99c70e32795341

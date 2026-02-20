# 🧪 Wazuh Lab Walkthrough

> **Environment:** Wazuh 4.14.3 All-in-One · AWS EC2 · Ubuntu 24.04.3 LTS · Region `us-east-1`
>
> **Setup:** 2 EC2 instances — 1 Wazuh Server (Manager + Indexer + Dashboard) + 1 Agent
>
> **Goal:** Install Wazuh, enroll agents, and explore every Dashboard module.

---

## Lab Overview

| Phase | Steps | What We Do |
|:---:|:---:|:---|
| **1** | 1–4 | Install Wazuh & explore the empty Dashboard |
| **2** | 5–6 | Discover tab & Threat Hunting |
| **3** | 7–13 | Deploy first agent & verify enrollment |
| **4** | 14–16 | Deploy second agent |
| **5** | 17–28 | Explore all security modules |

---

## Phase 1 — Installation & First Look

### Step 1 · SSH into the EC2 Instance

![Step 1](./images/1.png)

We SSH into our Ubuntu EC2 instance — this is where Wazuh will be installed.

```bash
ssh -i "your-key.pem" ubuntu@<ec2-public-ip>
```

The terminal shows the standard Ubuntu welcome: ESM update notice, warranty disclaimer, and system stats.

---

### Step 2 · Running the Wazuh Install Script

![Step 2](./images/2.png)

We run the **all-in-one quickstart** installer:

```bash
curl -sO https://packages.wazuh.com/4.x/wazuh-install.sh
sudo bash ./wazuh-install.sh -a
```

The terminal shows progress — installing dependencies, adding the Wazuh repo, generating TLS certificates (root, admin, indexer, dashboard). The `-a` flag installs all three components on one machine:

| Component | Role |
|:---|:---|
| **Wazuh Manager** | Receives & analyzes agent events |
| **Wazuh Indexer** | Stores alerts as searchable documents |
| **Wazuh Dashboard** | Web UI on port 443 |

> ⚠️ The installer prints an **admin password** at the end — save it!

---

### Step 3 · Dashboard First Login — No Agents

![Step 3](./images/3.png)

We open `https://<ip>:443` and log in. The Overview page shows **0 agents registered** — all modules (Configuration Assessment, Malware Detection, FIM) are listed but empty.

This is expected: Wazuh only shows data from enrolled agents.

---

### Step 4 · Dashboard Sidebar — All Modules

![Step 4](./images/4.png)

The left sidebar lists every available module:

| Category | Modules |
|:---|:---|
| **Endpoint Security** | Configuration Assessment · FIM · Malware Detection |
| **Security Operations** | IT Hygiene · GDPR · NIST 800-53 · PCI DSS · HIPAA · TSC |
| **Threat Intelligence** | Threat Hunting · MITRE ATT&CK · Vulnerability Detection |

> These stay empty until agents are deployed and `ossec.conf` modules are enabled.

---

## Phase 2 — Initial Dashboard Exploration

### Step 5 · Discover Tab — Raw Alert Data

![Step 5](./images/5.png)

The **Discover** tab is the raw data explorer (like Kibana). We see:

- **Index patterns:** `wazuh-alerts-*` · `wazuh-monitoring-*` · `wazuh-statistics-*`
- **404 hits** in the last 24 hours
- **DQL search bar** for filtering alerts by any field
- Available fields: `index`, `agent.id`, `data.command`, etc.

This is where SOC analysts do deep-dive investigations.

---

### Step 6 · Threat Hunting — Auth Failures

![Step 6](./images/6.png)

The **Threat Hunting** module filtered to `Authentication failure` shows **78 failed login events** against manager `ip-172-31-28-40`.

Why this matters:
- **Brute force** — repeated password guessing
- **Misconfig** — wrong creds in automation
- **Recon** — unauthorized access attempts

> In a real SOC, you'd check the source IP, attempt count, and possibly trigger Active Response to block the attacker.

---

## Phase 3 — Agent Deployment

### Step 7 · Deploy Agent Wizard — Select OS

![Step 7](./images/7.png)

The Dashboard has a built-in **Deploy New Agent** wizard. We select:

- **OS:** Linux → **DEB amd64** (for Ubuntu)
- **Server address:** `34.224.212.65` (Wazuh Manager public IP)

Other options available: RPM (amd64/aarch64), Windows MSI, macOS (Intel/Apple Silicon).

---

### Step 8 · Deploy Agent — Confirm Package

![Step 8](./images/8.png)

Confirming the package selection — **DEB amd64** with updated server address `98.81.179.28`.

> The IP changed from Step 7 because stopping/starting an EC2 instance reassigns its public IP (use an Elastic IP to keep it static).

---

### Step 9 · Deploy Agent — Install Command

![Step 9](./images/9.png)

The wizard generates the exact install command:

```bash
wget https://packages.wazuh.com/4.x/apt/pool/main/w/wazuh-agent/wazuh-agent_4.14.3-1_amd64.deb
sudo WAZUH_MANAGER='98.81.179.28' WAZUH_AGENT_GROUP='default' WAZUH_AGENT_NAME='ubuntu' \
     dpkg -i ./wazuh-agent_4.14.3-1_amd64.deb
```

| Parameter | Value |
|:---|:---|
| `WAZUH_MANAGER` | `98.81.179.28` — Server IP |
| `WAZUH_AGENT_GROUP` | `default` |
| `WAZUH_AGENT_NAME` | `ubuntu` |

> ⚠️ Ports **1514** (data) and **1515** (enrollment) must be open on the server's Security Group first!

---

### Step 10 · AWS Security Group — Open Ports

![Step 10](./images/10.png)

The AWS **Security Group** (`sg-0409dfe0b07ea77dd`) for our Wazuh server with **6 inbound rules**:

| Port | Purpose |
|:---|:---|
| **22** | SSH access |
| **443** | Dashboard (HTTPS) |
| **1514** | Agent data transmission |
| **1515** | Agent enrollment |

> This is the **#1 deployment issue** on AWS. Without these ports open, agents get "Connection refused."

---

### Step 11 · AWS EC2 — Two Instances Running

![Step 11](./images/11.png)

The EC2 dashboard shows **2 running instances**:

| Instance | Type | Role |
|:---|:---|:---|
| **wazuh** | `t2.medium` | Server (Manager + Indexer + Dashboard) |
| **Instance 2** | — | Agent endpoint |

Both in `us-east-1`, status checks passing (`2/2`).

---

### Step 12 · First Agent Enrolled

![Step 12](./images/12.png)

Back in Wazuh — the **Endpoints** page now shows **1 active agent**:

| Field | Value |
|:---|:---|
| **ID** | `001` |
| **Name** | `ip-172-31-27-237` |
| **Status** | 🟢 Active |
| **OS** | Ubuntu 24.04.3 LTS |
| **Version** | v4.14.3 |
| **Node** | node01 |

Compare to Step 3: the dashboard was empty. Now with an agent enrolled, data starts flowing.

---

### Step 13 · Individual Agent Details

![Step 13](./images/13.png)

Clicking on Agent 001 opens its dedicated dashboard:

| Detail | Value |
|:---|:---|
| **IP** | 172.31.27.237 |
| **Registered** | Feb 20, 2026 @ 11:10:20 |
| **Last Alive** | Feb 20, 2026 @ 11:18:20 |
| **CPU** | Intel Xeon · 2 cores |
| **Memory** | 3.868 GB |

**Available tabs:** Threat Hunting · FIM · SCA · MITRE ATT&CK · Vulnerability Detection · Stats · Configuration

---

## Phase 4 — Second Agent Deployment

### Step 14 · AWS EC2 Console — Managing Instances

![Step 14](./images/14.png)

Back in the AWS console. The EC2 sidebar shows all management sections: Instances, AMIs, Volumes, Snapshots, Security Groups, Elastic IPs, Key Pairs, etc. We're preparing the second EC2 instance for Agent 002.

---

### Step 15 · Two Agents Enrolled

![Step 15](./images/15.png)

Now **2 agents** are enrolled:

| Agent | Name | Status |
|:---|:---|:---|
| 001 | `ip-172-31-27-237` | 🟢 Active |
| 002 | `test-server2` | ⚫ Never Connected |

Agent 002 is registered but hasn't started its service yet.

---

### Step 16 · Both Agents Active

![Step 16](./images/16.png)

After starting the agent service on the second machine, **both are now active**:

| Field | Agent 001 | Agent 002 |
|:---|:---|:---|
| **Name** | ip-172-31-27-237 | test-server2 |
| **IP** | 172.31.27.237 | 172.31.25.172 |
| **OS** | Ubuntu 24.04.3 LTS | Ubuntu 24.04.3 LTS |
| **Version** | v4.14.3 | v4.14.3 |
| **Status** | 🟢 Active | 🟢 Active |

---

## Phase 5 — Security Module Exploration

### Step 17 · Configuration Assessment (SCA) — Events

![Step 17](./images/17.png)

The **SCA Events** tab for `test-server2` — filtered by `rule.groups: sca` and `agent.id: 002`. Shows **280 SCA events** with columns: timestamp, check title, config file evaluated.

SCA automatically runs CIS benchmark scans when an agent enrolls.

---

### Step 18 · Overview — 2 Active Agents

![Step 18](./images/18.png)

The **Overview** page now populated — **2 active agents**, and all modules show data:

- Configuration Assessment ✅
- File Integrity Monitoring ✅
- Malware Detection ✅
- IT Hygiene ✅
- GDPR ✅

> Compare to Step 3 — the dashboard was empty. Agents make everything come alive.

---

### Step 19 · IT Hygiene — Platforms

![Step 19](./images/19.png)

The **IT Hygiene** module shows a full system inventory. Under the **Platform** tab:

- **Both agents** run `Ubuntu 24.04.3 LTS (Noble Numbat)`
- **Kernel:** `6.14.0-1018-aws`
- **Tabs available:** Dashboard · OS/Hardware · Platform · Software · Processes · Network · Identity · Services

> Powered by `<syscollector>` in `ossec.conf`.

---

### Step 20 · IT Hygiene — Network

![Step 20](./images/20.png)

The **Network** tab shows connection data across all agents. Top destination ports:

| Port | Service |
|:---|:---|
| **443** | Wazuh Dashboard (HTTPS) |
| **1514** | Agent → Manager communication |
| **80** | HTTP |

Both agents (`172.31.27.237` and `172.31.25.172`) show active connections — confirming proper communication.

---

### Step 21 · IT Hygiene — Services

![Step 21](./images/21.png)

The **Services** tab discovered **179 unique services** (356 total entries) across both agents. Examples: `apport`, `ModemManager`, `networking`, `ldconfig`, `systemd-tpm2-setup-early`.

Unexpected services could indicate a backdoor or unauthorized software — critical for compliance audits.

---

### Step 22 · MITRE ATT&CK — Dashboard

![Step 22](./images/22.png)

The **MITRE ATT&CK** module maps every Wazuh alert to the ATT&CK framework. Top tactics detected:

| Tactic | Meaning |
|:---|:---|
| Valid Accounts | Legitimate credential abuse |
| Defense Evasion | Avoiding detection |
| Privilege Escalation | Gaining higher access (sudo) |
| Persistence | Maintaining access |
| Initial Access | First entry point |

Additional views: attacks by technique, top tactics per agent, MITRE techniques per agent.

> This turns isolated alerts into a **coherent attack narrative**.

---

### Step 23 · MITRE ATT&CK — Intelligence

![Step 23](./images/23.png)

The **Intelligence** tab is a built-in threat encyclopedia — **150+ threat actor groups** with their techniques, tools, and mitigations:

| Group ID | Name | Origin |
|:---|:---|:---|
| G0018 | admin@338 | China |
| G0130 | Ajax Security Team | Iran |
| G0138 | Andariel | North Korea (Lazarus sub-group) |
| G1007 | Aoqin Dragon | China |

Also available: Mitigations, Software, Tactics, and Techniques databases.

---

### Step 24 · MITRE ATT&CK — Events

![Step 24](./images/24.png)

The **Events** tab shows raw alerts tagged with MITRE technique IDs. Filtered by `rule.mitre.id: exists`, with events spanning `10:55 – 11:21 AM`. Each row can be expanded to see the full JSON: rule ID, technique, tactic, agent details, and raw log.

---

### Step 25 · Vulnerability Detection — Dashboard

![Step 25](./images/25.png)

The **Vulnerability Detection** module scans agent packages against CVE databases:

| Severity | Count |
|:---|:---:|
| 🔴 Critical | 18 |
| 🟠 High | 344 |
| 🟡 Medium | 758 |
| 🔵 Low | 134 |
| ⚪ Pending | 1,566 |

**Top findings:**
- **Top CVE:** `CVE-2022-3219` (22 occurrences)
- **Top package:** `linux-aws` (2,352 vulns)
- **Top agent:** `test-server2` (1,420 vulns)

> Powered by `<vulnerability-detection>` + `<syscollector>` in `ossec.conf`.

---

### Step 26 · Vulnerability Detection — Agent Events

![Step 26](./images/26.png)

Filtering vulnerabilities for **test-server2** specifically — **1,420 hits**. Example: `libgnutls30t64` v3.8.3 with a GnuTLS library flaw.

Columns: Agent name · Package · Version · Description · Severity · CVE ID.

> **Tip:** Filter by `vulnerability.severity: Critical` to prioritize patching.

---

### Step 27 · Configuration Assessment — CIS Benchmark

![Step 27](./images/27.png)

The **SCA Inventory** for `test-server2` showing **CIS Ubuntu Linux 24.04 LTS Benchmark v1.0.0** results:

| Result | Count |
|:---|:---:|
| ✅ Passed | 120 |
| ❌ Failed | 117 |
| ⬜ N/A | 42 |
| **Score** | **~50%** |

Each failed check includes: description, rationale, remediation commands, and compliance mapping (PCI DSS, HIPAA, etc.).

> A default Ubuntu install fails ~half the CIS checks — hardening is essential.

---

### Step 28 · Configuration Assessment — Events Timeline

![Step 28](./images/28.png)

The **SCA Events** tab shows when scans ran — events clustered around `11:22 AM`. Filtered by `rule.groups: sca` and `agent.id: 002`.

SCA scans run every 12 hours by default. Status changes (pass → fail) generate higher-priority alerts.

---

---

# 📋 Summary

## Complete Step Map

| Phase | Step | Screenshot | Description |
|:---:|:---:|:---:|:---|
| **1** | 1 | 1.png | SSH into Ubuntu EC2 |
| | 2 | 2.png | Run Wazuh install script |
| | 3 | 3.png | Dashboard — no agents |
| | 4 | 4.png | Sidebar modules overview |
| **2** | 5 | 5.png | Discover tab — 404 hits, 3 index patterns |
| | 6 | 6.png | Threat Hunting — 78 auth failures |
| **3** | 7 | 7.png | Deploy Agent — select Linux DEB amd64 |
| | 8 | 8.png | Confirm package, server IP `98.81.179.28` |
| | 9 | 9.png | Install command generated |
| | 10 | 10.png | Security Group — 6 inbound rules |
| | 11 | 11.png | 2 EC2 instances running |
| | 12 | 12.png | Agent 001 active |
| | 13 | 13.png | Agent details — 2 cores, 3.9 GB RAM |
| **4** | 14 | 14.png | AWS EC2 console sidebar |
| | 15 | 15.png | Agent 002 enrolled (never connected) |
| | 16 | 16.png | Both agents active |
| **5** | 17 | 17.png | SCA events — 280 hits |
| | 18 | 18.png | Overview — 2 active agents |
| | 19 | 19.png | IT Hygiene — Ubuntu 24.04, Kernel 6.14 |
| | 20 | 20.png | IT Hygiene — ports 443, 1514, 80 |
| | 21 | 21.png | IT Hygiene — 179 services |
| | 22 | 22.png | MITRE ATT&CK dashboard |
| | 23 | 23.png | MITRE Intelligence — 150+ groups |
| | 24 | 24.png | MITRE events |
| | 25 | 25.png | Vuln Detection — 18 Crit, 344 High |
| | 26 | 26.png | Vuln events — 1,420 on test-server2 |
| | 27 | 27.png | CIS Benchmark — 120 pass / 117 fail |
| | 28 | 28.png | SCA events timeline |

---

## Key Takeaways

1. **No agents = no data.** The dashboard only shows what agents report.
2. **Ports 1514/1515 must be open** in the AWS Security Group — this is the #1 deployment issue.
3. **Agent deployment is wizard-driven** — the Dashboard generates the exact install command.
4. **IT Hygiene = full asset inventory** — packages, ports, services, network interfaces.
5. **MITRE ATT&CK is built-in** — every alert maps to tactics and techniques automatically.
6. **1,000+ CVEs on a fresh Ubuntu server** — vulnerability scanning is essential, not optional.
7. **CIS score ~50%** — a default install needs significant hardening.
8. **Everything starts at `ossec.conf`** — if a module isn't enabled, its dashboard stays empty.

---

## References

| Resource | Link |
|:---|:---|
| Quick Start Guide | https://documentation.wazuh.com/current/quickstart.html |
| Agent Deployment | https://documentation.wazuh.com/current/installation-guide/wazuh-agent/index.html |
| Dashboard Docs | https://documentation.wazuh.com/current/getting-started/components/wazuh-dashboard.html |
| Capabilities | https://documentation.wazuh.com/current/user-manual/capabilities/index.html |
| ossec.conf Reference | https://documentation.wazuh.com/current/user-manual/reference/ossec-conf/index.html |

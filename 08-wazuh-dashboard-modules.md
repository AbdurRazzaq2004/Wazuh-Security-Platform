# 🖥️ Wazuh Dashboard Modules — Every Tool Explained

> *"Dashboards are for visibility, not for in-depth investigation."*
> — SOC 201 Podcast

---

## 📌 What Is This File?

When you log into the Wazuh Dashboard, you see **four main categories** of modules:

1. 🔒 **Endpoint Security** — What's happening on your machines?
2. 🧠 **Threat Intelligence** — What threats exist in your environment?
3. 🛡️ **Security Operations** — Are you meeting compliance & operational standards?
4. ☁️ **Cloud Security** — What's happening in your cloud & SaaS platforms?

This file explains **every single module** in each category — what it does, why it matters, and how a SOC analyst or DevOps engineer uses it in the real world.

---

## 🔑 SOC Mindset Before We Begin

From the SOC 201 Podcast — remember these golden rules:

> **ONE EVENT = NOISE. PATTERNS = SIGNALS.**

> **CORRELATION = Multiple failed logins from the same user**

> **CONTEXT = WHO did it, FROM WHERE, WHAT system, HOW?**

> 💡 *"If your alert doesn't answer WHO, WHAT, WHERE, HOW, WHEN — it's not an alert, it's a notification!"*

The data flow you must memorize:
```
A. Endpoint generates event
B. Agent collects it
C. Manager (Server) analyzes it
D. Indexer stores it
E. Dashboard displays it

IF YOU UNDERSTAND THIS FLOW, WAZUH STOPS FEELING COMPLICATED.
```

Every module below is the **"E" step** — the Dashboard displaying what the rest of the pipeline collected, analyzed, and stored.

---

---

# 🔒 ENDPOINT SECURITY

> These modules show you what's happening **directly on your monitored endpoints** (servers, laptops, VMs). This is where the Wazuh Agent shines — it collects data from the machine it lives on and ships it to the server.

---

## 1. ✅ Configuration Assessment

**Dashboard Label:** *"Scan your assets as part of a configuration assessment audit."*

### What Is It?
This module shows the results of **Security Configuration Assessment (SCA)** scans. It checks whether your endpoints are configured securely according to **industry benchmarks** (CIS, NIST, etc.).

### Why It Matters
> Misconfiguration is the **#1 cause of breaches**. Not zero-days. Not sophisticated APTs. Misconfiguration. A default password. An open port. SSH root login enabled. This module catches all of that.

### What You See on the Dashboard

| View | Description |
|---|---|
| **Policy list** | All SCA policies running on an agent (e.g., CIS Ubuntu 22.04, CIS Windows 2022) |
| **Score** | Compliance percentage (e.g., 39% — meaning 61% of checks failed!) |
| **Pass / Fail / N/A** | How many checks passed, failed, or are not applicable |
| **Individual checks** | Click any check to see description, rationale, remediation steps, and compliance mappings |

### Real Example
```
Policy: CIS Ubuntu 22.04
Score:  39%
Passed: 56  |  Failed: 87  |  N/A: 48

Check ID: 3003
Title:    "Ensure SSH public key authentication is enabled"
Status:   ❌ FAILED
Remediation: Set "PubkeyAuthentication yes" in /etc/ssh/sshd_config
```

### Who Uses This?
- **DevOps Engineers** — Ensure all servers meet hardening baseline before production
- **Compliance Teams** — Prove CIS/NIST compliance during audits
- **SOC Analysts** — Identify weak endpoints that attackers could exploit

---

## 2. 🦠 Malware Detection

**Dashboard Label:** *"Check indicators of compromise triggered by malware infections or cyberattacks."*

### What Is It?
This module shows alerts related to **malware activity** detected on your endpoints. It covers:
- Behavior-based malware detection (suspicious actions)
- Rootkit detection (hidden processes, files, ports)
- Threat intelligence matches (known malware hashes, IPs, domains)
- Antivirus integration alerts (VirusTotal, ClamAV, Windows Defender)

### Why It Matters
> Traditional antivirus looks for known signatures. Wazuh goes further — it watches **behavior**. Did an executable drop in `AppData\Roaming`? Did it add itself to the Windows registry for persistence? Did it make a suspicious DNS query? Those are **malware behaviors**, and Wazuh catches them even if no antivirus has seen this specific malware before.

### What You See on the Dashboard

| View | Description |
|---|---|
| **Alert timeline** | When malware activity was detected |
| **Rule details** | Which detection rule triggered (rule ID, description, MITRE technique) |
| **Affected agents** | Which endpoints have malware alerts |
| **IOCs** | Indicators of Compromise — file hashes, IPs, domains |
| **Severity levels** | How critical the finding is |

### Real Example
```
Alert:    "Executable dropped in folder commonly used by malware"
Rule ID:  92213
Level:    12 (High)
Agent:    win-server-03
File:     C:\Users\admin\AppData\Roaming\checker netflix.exe
MITRE:    T1036 (Masquerading)
```

### SOC 201 Insight
> *"Wazuh rules use multiple fields that denote IOCs to reduce false positives. These rules can connect related malware activities — intrusion, privilege escalation, lateral movement, obfuscation, and exfiltration — for comprehensive detection."*

---

## 3. 📁 File Integrity Monitoring (FIM)

**Dashboard Label:** *"Alerts related to file changes, including permissions, content, ownership, and attributes."*

### What Is It?
FIM monitors **critical files and directories** on your endpoints. When a file is created, modified, or deleted — FIM alerts you with full context.

### Why It Matters
> Attackers **always** touch the filesystem. They drop web shells, modify config files, plant backdoors, or delete logs to cover their tracks. FIM is your **tripwire** — invisible to the attacker, but it catches every filesystem change.

### What You See on the Dashboard

| View | Description |
|---|---|
| **Events timeline** | File change events over time |
| **Top agents** | Which agents have the most file changes |
| **Top files** | Most frequently modified files |
| **Actions** | Created / Modified / Deleted breakdown |
| **Event details** | Full details — file path, what changed, who changed it, when, process that made the change |

### What FIM Tracks

| Attribute | Example |
|---|---|
| **Content** | File content was changed (with diff!) |
| **Permissions** | File went from 600 to 777 |
| **Ownership** | File owner changed from root to www-data |
| **Attributes** | File attributes modified |
| **Size** | File size increased/decreased |
| **MD5/SHA1/SHA256** | File hash changed (content modification proof) |

### Real Example
```
Alert:    "File modified: /etc/shadow"
Action:   Modified
Changed:  Content, Permissions
Old perm: 640
New perm: 644
User:     root
Process:  /usr/bin/vim
Time:     2026-02-20 03:14:22 UTC
```

### SOC 201 Insight
> *FIM + custom rules is a powerful combo. When FIM detects a new file in your web root with a `.php` extension created by a non-standard user — that's likely a web shell. Create a custom rule that correlates FIM events with specific IOCs.*

### Windows Registry Monitoring
FIM also monitors the **Windows Registry** — malware's favorite hiding spot:
- Startup entries (`Run`, `RunOnce`)
- Service configurations
- Security policy modifications
- Browser helper objects

---

---

# 🧠 THREAT INTELLIGENCE

> These modules help you **understand the threats** in your environment — not just detect them, but contextualize and prioritize them. This is where you move from "something happened" to "here's what it means and how dangerous it is."

---

## 4. 🔎 Threat Hunting

**Dashboard Label:** *"Browse through your security alerts, identifying issues and threats in your environment."*

### What Is It?
This is the **primary investigation module** for SOC analysts. It gives you a searchable, filterable view of **all security alerts** across your entire environment.

### Why It Matters
> This is where you **do the actual detective work**. All other modules are specialized views — Threat Hunting is the general-purpose investigation tool. If you're not sure which module to use, start here.

### What You See on the Dashboard

| View | Description |
|---|---|
| **Alert table** | All alerts with timestamp, agent, rule, level, description |
| **Filters** | Filter by agent, rule group, level, time range, specific fields |
| **Full-text search** | Search any keyword across all alert fields |
| **Alert details** | Click any alert for full JSON — every field the decoder extracted |
| **Timeline** | Visual timeline of alert frequency |

### How to Use It (SOC Analyst Workflow)
```
1. Start broad → Filter by time range (last 24 hours)
2. Sort by level → High severity first (level 10+)
3. Group by rule → See which rules fire most
4. Filter by agent → Focus on specific endpoint
5. Drill into an alert → Read full JSON
6. Ask the WH questions:
   - WHO triggered this? (user, process)
   - WHAT happened? (rule description, action)
   - WHERE did it happen? (agent, file path, IP)
   - WHEN did it happen? (timestamp)
   - HOW did it happen? (technique, process chain)
```

### SOC 201 Insight
> *"ONE EVENT = NOISE, PATTERNS = SIGNALS. CORRELATION = Multiple failed logins from the same user/IP. CONTEXT = Who did it, from where, what system, how — Ask WH Questions to your logs."*

---

## 5. 💀 Vulnerability Detection

**Dashboard Label:** *"Discover what applications in your environment are affected by well-known vulnerabilities."*

### What Is It?
This module shows **known software vulnerabilities (CVEs)** on all your monitored endpoints. It correlates the installed software inventory with vulnerability databases (Wazuh CTI).

### Why It Matters
> You have 500 servers. Which ones are running an OpenSSL version with a critical RCE vulnerability? This module tells you **instantly** — no need to SSH into each server and check manually.

### What You See on the Dashboard

| View | Description |
|---|---|
| **Vulnerability inventory** | All vulnerabilities found across all agents |
| **Severity breakdown** | Critical / High / Medium / Low counts |
| **Top vulnerable agents** | Which endpoints have the most vulnerabilities |
| **Top CVEs** | Most common vulnerabilities |
| **Package details** | Affected package name, installed version, fixed version |
| **Alert details** | CVE ID, CVSS score, description, references, remediation |

### Real Example
```
CVE:         CVE-2024-1234
Severity:    Critical (CVSS 9.8)
Package:     openssl 1.1.1k
Agent:       web-server-01
Description: Buffer overflow allowing remote code execution
Fix:         Upgrade to openssl 1.1.1l
Status:      Active (unpatched)
References:  https://nvd.nist.gov/vuln/detail/CVE-2024-1234
```

### Key Feature: Remediation Tracking
When you patch a vulnerability, Wazuh detects it and updates the status:
```
Before: CVE-2024-1234 → Active (openssl 1.1.1k installed)
After:  CVE-2024-1234 → Fixed  (openssl 1.1.1l installed after upgrade)
```

---

## 6. ⚔️ MITRE ATT&CK

**Dashboard Label:** *"Explore security alerts mapped to adversary tactics and techniques for better threat understanding."*

### What Is It?
This module maps every Wazuh alert to the **MITRE ATT&CK framework** — a globally recognized knowledge base of adversary tactics, techniques, and procedures (TTPs).

### Why It Matters
> MITRE ATT&CK answers the question: **"What is the attacker TRYING to do?"** Instead of seeing isolated alerts, you see them in the context of an attack lifecycle — initial access → execution → persistence → privilege escalation → lateral movement → exfiltration.

### What You See on the Dashboard

| View | Description |
|---|---|
| **Tactics overview** | All 14 MITRE ATT&CK tactics with alert counts |
| **Techniques heatmap** | Which techniques are most active in your environment |
| **Technique details** | Click any technique for related alerts, descriptions, mitigations |
| **Timeline** | When specific techniques were observed |
| **Agent correlation** | Which agents triggered which techniques |

### The 14 MITRE ATT&CK Tactics

| # | Tactic | What It Means | Example Alert |
|---|---|---|---|
| 1 | **Reconnaissance** | Attacker gathering info | Port scan detected |
| 2 | **Resource Development** | Attacker setting up tools | — |
| 3 | **Initial Access** | Getting into the system | SSH brute-force, phishing |
| 4 | **Execution** | Running malicious code | Suspicious PowerShell |
| 5 | **Persistence** | Staying after reboot | Registry run key added |
| 6 | **Privilege Escalation** | Getting higher permissions | sudo abuse, exploit |
| 7 | **Defense Evasion** | Hiding from security tools | Log deletion, timestomping |
| 8 | **Credential Access** | Stealing passwords | Mimikatz, credential dump |
| 9 | **Discovery** | Mapping the network | Internal scanning |
| 10 | **Lateral Movement** | Moving to other systems | PsExec, RDP |
| 11 | **Collection** | Gathering target data | File archiving |
| 12 | **Command & Control** | Communicating with attacker | C2 beacon detected |
| 13 | **Exfiltration** | Stealing data out | Large outbound transfer |
| 14 | **Impact** | Damage/disruption | Ransomware, data wipe |

### SOC 201 Insight
> *When writing custom rules, always include MITRE mappings:*
```xml
<rule id="100200" level="12" frequency="3" timeframe="120">
    <if_matched_group>authentication_failed</if_matched_group>
    <same_field>win.eventdata.ipAddress</same_field>
    <description>Custom: Windows brute-force detected</description>
    <mitre>
      <id>T1110</id>  <!-- Brute Force technique -->
    </mitre>
</rule>
```
> *This ensures your alert shows up in the MITRE ATT&CK module mapped to the correct technique.*

---

---

# 🛡️ SECURITY OPERATIONS

> These modules focus on **operational security, compliance monitoring, and regulatory standards**. This is what auditors ask about, and what management cares about during board meetings.

---

## 7. 🔧 IT Hygiene

**Dashboard Label:** *"Assess system, software, processes, and network layers to detect misconfigurations, unauthorized changes, and anomalies."*

### What Is It?
IT Hygiene gives you a **holistic view** of the health and state of your IT environment. Think of it as your **system inventory + anomaly detection** combined.

### Why It Matters
> You can't protect what you don't know you have. IT Hygiene answers: "What's running on my systems? What software is installed? What ports are open? Did anything change unexpectedly?"

### What You See on the Dashboard

| View | Description |
|---|---|
| **System inventory** | OS versions, hardware info, hostnames |
| **Installed software** | All packages/applications with versions |
| **Running processes** | Currently active processes on each endpoint |
| **Open ports** | Listening ports and associated services |
| **Network interfaces** | IP addresses, MAC addresses, network config |
| **Anomalies** | Unauthorized changes, unexpected software, rogue processes |

### Real-World Use Cases

| Scenario | How IT Hygiene Helps |
|---|---|
| **Shadow IT** | Detect unauthorized software installed by employees |
| **Asset management** | Know every OS version across all endpoints |
| **Attack surface** | Identify unnecessary open ports and services |
| **Patch management** | Find all endpoints running a specific vulnerable version |
| **Change detection** | Alert when new software or processes appear |

---

## 8. 💳 PCI DSS

**Dashboard Label:** *"Global security standard for entities that process, store, or transmit payment cardholder data."*

### What Is It?
A **compliance dashboard** that shows your alignment with the **Payment Card Industry Data Security Standard (PCI DSS)**.

### Why It Matters
> If your company processes, stores, or transmits **credit card data**, you **MUST** comply with PCI DSS. Non-compliance means fines of **$5,000 – $100,000 per month**, and if you suffer a breach — goodbye business.

### PCI DSS Requirements Covered by Wazuh

| Requirement | Description | How Wazuh Helps |
|---|---|---|
| **Req 1** | Install & maintain a firewall | Monitor firewall configuration changes |
| **Req 2** | Don't use vendor-supplied defaults | SCA checks for default passwords/configs |
| **Req 5** | Protect against malware | Malware detection module |
| **Req 6** | Develop secure systems | Vulnerability detection |
| **Req 7** | Restrict access to cardholder data | FIM on sensitive files |
| **Req 8** | Identify and authenticate access | Monitor authentication events |
| **Req 10** | Track & monitor all access | Log collection and analysis |
| **Req 11** | Regularly test security systems | SCA, vulnerability scans |

### What You See
- Alerts **tagged** with specific PCI DSS requirements
- **Compliance score** per requirement
- **Trending** — are you getting more or less compliant over time?
- **Exportable reports** for auditors

---

## 9. 🇪🇺 GDPR

**Dashboard Label:** *"General Data Protection Regulation (GDPR) sets guidelines for processing of personal data."*

### What Is It?
A compliance dashboard for the **EU General Data Protection Regulation** — the world's strictest data privacy law.

### Why It Matters
> If you handle **any personal data of EU citizens** (names, emails, IPs, health data), GDPR applies to you — **regardless of where your company is located**. Fines can be up to **€20 million or 4% of annual global revenue**, whichever is higher.

### GDPR Articles Covered by Wazuh

| Article | Topic | How Wazuh Helps |
|---|---|---|
| **Art. 5** | Data processing principles | Log monitoring for unauthorized data access |
| **Art. 25** | Data protection by design | SCA ensures secure configurations |
| **Art. 32** | Security of processing | FIM, malware detection, vulnerability detection |
| **Art. 33** | Breach notification | Real-time alerting for data breaches |
| **Art. 35** | Data protection impact assessment | Comprehensive security visibility |

---

## 10. 🏥 HIPAA

**Dashboard Label:** *"Health Insurance Portability and Accountability Act of 1996 (HIPAA) provides data privacy and security provisions for safeguarding medical information."*

### What Is It?
A compliance dashboard for **HIPAA** — the US law that protects patient health information (PHI).

### Why It Matters
> If you're a **healthcare organization, health insurer, or any company handling patient data**, HIPAA compliance is mandatory. Violations range from **$100 to $50,000 per violation**, up to **$1.5 million per year**.

### HIPAA Safeguards Covered by Wazuh

| Safeguard | Section | How Wazuh Helps |
|---|---|---|
| **Administrative** | §164.308 | Access control monitoring, risk assessment, security management |
| **Physical** | §164.310 | FIM for physical access logs and device controls |
| **Technical** | §164.312 | Encryption monitoring, audit controls, access controls, integrity controls |

### What You See
- Alerts mapped to specific **HIPAA sections**
- Dashboard showing **compliance status** per safeguard
- **Trend analysis** — improving or declining over time
- **Reports** for HHS auditors

---

## 11. 🇺🇸 NIST 800-53

**Dashboard Label:** *"National Institute of Standards and Technology Special Publication 800-53 (NIST 800-53) sets guidelines for federal information systems."*

### What Is It?
A compliance dashboard for **NIST 800-53** — the gold standard security framework used by US government agencies and their contractors.

### Why It Matters
> If you work with the **US federal government** (or want to), NIST 800-53 compliance is required. Even non-government organizations use it as a best-practice security framework because it's the **most comprehensive** control catalog in existence.

### NIST 800-53 Control Families Covered

| Family | Code | How Wazuh Helps |
|---|---|---|
| **Access Control** | AC | Authentication monitoring, RBAC |
| **Audit & Accountability** | AU | Log collection, audit trails |
| **Configuration Management** | CM | SCA, FIM |
| **Incident Response** | IR | Active Response, alerting |
| **Risk Assessment** | RA | Vulnerability detection |
| **System & Info Protection** | SI | Malware detection, FIM |
| **System & Communications Protection** | SC | Encryption monitoring |

---

## 12. 🔐 TSC (Trust Services Criteria)

**Dashboard Label:** *"Trust Services Criteria for Security, Availability, Processing Integrity, Confidentiality, and Privacy."*

### What Is It?
A compliance dashboard for **TSC** — the criteria used in **SOC 2 audits**. If your company is a SaaS provider, your customers will ask for your SOC 2 report.

### Why It Matters
> **SOC 2 is the gold standard for SaaS companies.** Your enterprise customers won't buy your product without a SOC 2 Type II report. Wazuh helps you meet the security criteria continuously.

### TSC Principles Covered

| Principle | What It Means | How Wazuh Helps |
|---|---|---|
| **Security** | Protection against unauthorized access | FIM, authentication monitoring, malware detection |
| **Availability** | System uptime and performance | System monitoring, agent status |
| **Processing Integrity** | Data processed correctly | Log integrity, FIM |
| **Confidentiality** | Data protected from disclosure | Access monitoring, encryption checks |
| **Privacy** | Personal data handled properly | Data access monitoring, RBAC |

---

---

# ☁️ CLOUD SECURITY

> These modules monitor your **cloud infrastructure and SaaS platforms**. In today's world, your data isn't just on-premises — it's in AWS, Azure, GCP, Office 365, and GitHub. These modules extend Wazuh's visibility into the cloud.

---

## 13. 🐳 Docker

**Dashboard Label:** *"Monitor and collect the activity from Docker containers such as creation, running, starting, stopping or pausing events."*

### What Is It?
Monitors all **Docker container activity** on hosts running the Wazuh agent.

### Why It Matters
> Containers are ephemeral — they spin up and die in seconds. Without monitoring, you have **zero visibility** into what's happening inside your containerized workloads. An attacker could compromise a container, pivot to the host, and you'd never know.

### What You See on the Dashboard

| Event | Description |
|---|---|
| **Container created** | A new container was started |
| **Container stopped** | A container was stopped or killed |
| **Container paused** | A container was paused |
| **Image pulled** | A new Docker image was downloaded |
| **Privileged container** | ⚠️ Container running with `--privileged` flag |
| **Exec in container** | Someone executed a command inside a running container |
| **Network changes** | Container network configuration modified |
| **Volume changes** | Data volume mounted or modified |

### Critical Alerts to Watch

| Alert | Why It's Dangerous |
|---|---|
| **Privileged container** | Has full access to the host — could escape the container |
| **Exec into container** | Someone is running commands inside — could be an attacker |
| **Unknown image pulled** | Untrusted image could contain malware |
| **Container running as root** | Compromised container = root on the host |

---

## 14. 🟠 Amazon Web Services (AWS)

**Dashboard Label:** *"Security events related to your Amazon AWS services, collected directly via AWS API."*

### What Is It?
Monitors your **AWS infrastructure** by collecting events directly from AWS APIs — primarily **CloudTrail**.

### Why It Matters
> AWS is the most popular cloud platform. If you're running workloads on AWS, you MUST monitor the control plane (IAM, security groups, S3, etc.) — not just the instances. Most AWS breaches happen because of **misconfigurations**, not because someone hacked an EC2 instance.

### What AWS Services Are Monitored

| AWS Service | What Wazuh Collects |
|---|---|
| **CloudTrail** | Every API call made in your AWS account — who did what, when, from where |
| **VPC Flow Logs** | Network traffic metadata — source/dest IPs, ports, protocols |
| **GuardDuty** | AWS's own threat detection findings |
| **S3** | Bucket access, policy changes, data events |
| **IAM** | User creation, policy changes, role assumptions |
| **Security Groups** | Inbound/outbound rule changes |
| **EC2** | Instance start/stop, AMI creation, EBS changes |

### Critical Alerts to Watch

| Alert | Why It's Dangerous |
|---|---|
| **New IAM user with admin access** | Could be an attacker creating a backdoor account |
| **S3 bucket made public** | Your data is now exposed to the entire internet |
| **Security group opened to 0.0.0.0/0** | Anyone on the internet can reach your instances |
| **Root account used** | AWS root should NEVER be used for daily operations |
| **Console login without MFA** | Weak authentication = easy target |
| **API calls from unusual IP/region** | Could indicate stolen credentials |

---

## 15. 🔵 Google Cloud (GCP)

**Dashboard Label:** *"Security events related to your Google Cloud Platform services, collected directly via GCP API."*

### What Is It?
Monitors your **Google Cloud Platform** infrastructure through GCP APIs and Pub/Sub integration.

### What GCP Services Are Monitored

| Service | What Wazuh Collects |
|---|---|
| **Audit Logs** | Admin activity, data access, system events |
| **Pub/Sub** | Real-time event streaming from GCP services |
| **IAM** | User and service account changes |
| **Compute Engine** | Instance lifecycle events |
| **Cloud Storage** | Bucket access and policy changes |
| **VPC** | Network configuration changes |

### Cloud Security Posture Management (CSPM)
Wazuh also provides **CSPM** for GCP — continuously assessing your GCP configuration against security best practices.

---

## 16. 📫 Office 365

**Dashboard Label:** *"Security events related to your Office 365 services."*

### What Is It?
Monitors **Microsoft Office 365** audit logs for security-relevant events.

### Why It Matters
> Office 365 is where your company's **email, files, and collaboration** live. Compromised O365 accounts are one of the **top initial access vectors** for attackers (business email compromise, phishing, data theft).

### What O365 Events Are Monitored

| Event Type | Examples |
|---|---|
| **User activity** | Logins, failed logins, suspicious sign-ins |
| **Mailbox activity** | Email forwarding rules created, mailbox delegation changes |
| **SharePoint/OneDrive** | File sharing, external sharing, permission changes |
| **Admin activity** | User creation/deletion, license changes, role assignments |
| **DLP events** | Data Loss Prevention policy violations |
| **eDiscovery** | Search and export activities |

### Critical Alerts

| Alert | Why It's Dangerous |
|---|---|
| **Impossible travel** | User logged in from US and Russia within 5 minutes |
| **Mail forwarding rule** | Attacker set up auto-forward to exfiltrate emails |
| **External sharing enabled** | Sensitive files shared outside the organization |
| **Admin role assigned** | Unexpected privilege escalation |

---

## 17. 🐙 GitHub

**Dashboard Label:** *"Monitoring events from audit logs of your GitHub organizations."*

### What Is It?
Monitors **GitHub organization audit logs** for security and compliance events.

### Why It Matters
> Your **source code is your most valuable asset**. GitHub audit logs tell you who accessed your repositories, who changed permissions, who added deploy keys, and who cloned your code. If an attacker compromises a developer's GitHub account, you need to know immediately.

### What GitHub Events Are Monitored

| Event Type | Examples |
|---|---|
| **Repository access** | Clone, fork, visibility change (public/private) |
| **Member management** | Users added/removed from org, team changes |
| **Permission changes** | Role changes, deploy key additions |
| **Webhook management** | New webhooks added (potential data exfiltration) |
| **OAuth/PAT activity** | New OAuth apps authorized, personal access tokens created |
| **Branch protection** | Protection rules added, modified, or removed |

### Critical Alerts

| Alert | Why It's Dangerous |
|---|---|
| **Repo made public** | Private code exposed to the world |
| **Deploy key added** | Could be an attacker gaining persistent access |
| **Branch protection removed** | Anyone can now push to main/master |
| **New admin added** | Unexpected privilege escalation |
| **External collaborator added** | Non-org member given access |

---

## 18. 🟦 Microsoft Graph API

**Dashboard Label:** *"Security events related to your Microsoft Graph services, collected directly via Microsoft Graph API."*

### What Is It?
Monitors **Microsoft 365 ecosystem** through the Microsoft Graph API — a unified API endpoint for Azure AD, Intune, Teams, and other Microsoft services.

### What's Monitored

| Service | What Wazuh Collects |
|---|---|
| **Azure Active Directory** | User sign-ins, MFA events, conditional access, risky users |
| **Microsoft Intune** | Device compliance, app deployments, policy violations |
| **Microsoft Teams** | Team creation, guest access, policy changes |
| **Azure AD Identity Protection** | Risky sign-ins, compromised credentials |

### Why Graph API vs Office 365?
- **Office 365 module** → Focuses on O365 workload-specific audit logs (Exchange, SharePoint, etc.)
- **Graph API module** → Focuses on **identity, access, and device management** (Azure AD, Intune, etc.)
- Together, they provide **complete Microsoft ecosystem visibility**

---

---

# 🎓 Detection Engineering Notes (from SOC 201 Podcast)

These insights apply to **ALL modules above**. When you create custom rules for any module, follow these principles:

## Golden Rule #1: Never Edit Default Rules
```
❌ NEVER edit /var/ossec/ruleset/rules/
   - These are vendor-provided rules
   - They get overwritten during upgrades
   - You lose version tracking

✅ ALWAYS work in /var/ossec/etc/rules/local_rules.xml
   - Your custom rules live here
   - Survive upgrades
   - Wazuh evaluates local rules after default rules
```

## Custom Rule ID Convention
```
All custom rules → 100xxx
Brute-force     → 1002xx
Malware         → 1003xx
Cloud           → 1004xx
Compliance      → 1005xx
```

## Example Custom Rule: Windows Brute-Force
```xml
<group name="windows,authentication,custom,">
  <rule id="100200" level="12" frequency="3" timeframe="120">
    <if_matched_group>authentication_failed</if_matched_group>
    <same_field>win.eventdata.ipAddress</same_field>
    <description>
      Custom: Windows brute-force detected (3 failures in 2 minutes)
    </description>
    <mitre>
      <id>T1110</id>
    </mitre>
  </rule>
</group>
```

### Line-by-Line Breakdown:
| Line | What It Does |
|---|---|
| `frequency="3"` | Trigger when 3 matching events occur |
| `timeframe="120"` | ...within 120 seconds (2 minutes) |
| `<if_matched_group>authentication_failed` | Reuses existing Windows failure detection (no need to decode logs again!) |
| `<same_field>win.eventdata.ipAddress` | Correlates failures from the **same source IP** |
| `<mitre><id>T1110</id></mitre>` | Maps to MITRE ATT&CK "Brute Force" technique |

> *"Every failed Windows login already triggers a base rule. This custom rule doesn't look at logs — it looks at previous alerts."*

## About Rule IDs
```
60105, 60204 — These are vendor-managed, internally organized rule IDs.
You DON'T need to follow their numbering.

"Rule IDs are for humans, not for the engine."
They help with: Dashboards, Runbooks, Incident references, SOC communication
```

---

## 💡 Final Summary Table

| Category | Module | One-Line Purpose |
|---|---|---|
| **Endpoint Security** | Configuration Assessment | Are my systems configured securely? |
| | Malware Detection | Is there malware on my systems? |
| | File Integrity Monitoring | Did someone tamper with critical files? |
| **Threat Intelligence** | Threat Hunting | Investigate all security alerts |
| | Vulnerability Detection | What known CVEs affect my systems? |
| | MITRE ATT&CK | What attack techniques are active? |
| **Security Operations** | IT Hygiene | What's the state of my IT environment? |
| | PCI DSS | Am I compliant with payment card security? |
| | GDPR | Am I compliant with EU data privacy? |
| | HIPAA | Am I compliant with healthcare data security? |
| | NIST 800-53 | Am I compliant with federal security controls? |
| | TSC | Am I ready for SOC 2 audit? |
| **Cloud Security** | Docker | What's happening in my containers? |
| | AWS | What's happening in my AWS account? |
| | Google Cloud | What's happening in my GCP project? |
| | Office 365 | What's happening in my O365 tenant? |
| | GitHub | What's happening in my GitHub org? |
| | Microsoft Graph API | What's happening in my Azure AD/Intune? |

---

> 🧠 *"SHARPEN YOUR MINDSET, SKILLS DO FOLLOW"* — SOC 201 Podcast

---

## 🔗 References
- SOC 201 Podcast Notes: https://studywithurvesh.notion.site/SOC-201-Podcast-2cea317e9ac48071ab99c70e32795341
- Wazuh Capabilities: https://documentation.wazuh.com/current/user-manual/capabilities/index.html
- Cloud Security: https://documentation.wazuh.com/current/cloud-security/monitoring.html
- MITRE ATT&CK: https://attack.mitre.org/
- PCI DSS: https://www.pcisecuritystandards.org/
- HIPAA: https://www.hhs.gov/hipaa/
- GDPR: https://gdpr.eu/
- NIST 800-53: https://csf.tools/reference/nist-sp-800-53/

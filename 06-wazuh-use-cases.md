# 🎯 Wazuh Use Cases — Real-World Security Scenarios

## Overview

Wazuh isn't just a tool — it's a platform that covers a **wide range of security use cases**. Below, we break down each major use case with real-world examples that a DevOps engineer or security specialist would encounter daily.

> **In simple words:** These are the actual problems Wazuh solves for you in the real world. Each section answers: "What's the problem?" and "How does Wazuh fix it?"

---

## 📋 Table of Contents

| # | Use Case | What It Solves |
|---|---|---|
| 1 | [Configuration Assessment](#-1-configuration-assessment) | Are my systems configured securely? |
| 2 | [Malware Detection](#-2-malware-detection) | Is there malicious software on my systems? |
| 3 | [File Integrity Monitoring](#-3-file-integrity-monitoring) | Did someone tamper with critical files? |
| 4 | [Threat Hunting](#-4-threat-hunting) | Are there hidden threats in my environment? |
| 5 | [Log Data Analysis](#-5-log-data-analysis) | What do all these logs actually mean? |
| 6 | [Vulnerability Detection](#-6-vulnerability-detection) | What vulnerable software is running on my systems? |
| 7 | [Incident Response](#-7-incident-response) | A threat was detected — now what? |
| 8 | [Regulatory Compliance](#-8-regulatory-compliance) | Am I meeting security standards? |
| 9 | [Cloud Security](#-9-cloud-security) | Is my cloud infrastructure secure? |
| 10 | [Container Security](#-10-container-security) | Are my containers safe? |

---

## ✅ 1. Configuration Assessment

### The Problem
> Your team deploys 100 Ubuntu servers. But are they all configured securely? Is SSH root login disabled? Are unnecessary services turned off? Is the firewall configured correctly?

**Misconfiguration is the #1 cause of security breaches.** Most breaches aren't caused by sophisticated attacks — they happen because someone left a door unlocked.

### How Wazuh Fixes It

Wazuh's **Security Configuration Assessment (SCA)** module scans every endpoint against industry benchmarks.

#### What happens:
1. The Wazuh agent reads **SCA policy files** (YAML format)
2. Each policy contains **checks** (e.g., "Is SSH root login disabled?")
3. Each check runs **rules** against the system
4. Results are reported as **Pass**, **Fail**, or **Not Applicable**
5. Dashboard shows a **compliance score** (e.g., 39% — meaning 61% needs fixing!)

#### Built-in Benchmarks:
| Benchmark | What It Checks |
|---|---|
| CIS Ubuntu 22.04 | Ubuntu-specific hardening rules |
| CIS Windows Server 2022 | Windows hardening rules |
| CIS Amazon Linux 2 | AWS AMI hardening |
| CIS macOS | Mac hardening rules |
| Custom policies | Your own organization's rules |

#### Real-World Example:
```yaml
# SCA Check: Is /tmp on a separate partition?
- id: 28500
  title: "Ensure /tmp is a separate partition."
  description: "The /tmp directory is world-writable..."
  remediation: "Mount tmpfs to /tmp or create a dedicated partition..."
  compliance:
    - cis: ["1.1.2.1"]
    - pci_dss: ["7.1"]
    - hipaa: ["164.312(a)(1)"]
  rules:
    - 'c:findmnt --kernel /tmp -> r:\s*/tmp\s'
```

**Result on Dashboard:**
- ✅ **56 checks passed**
- ❌ **87 checks failed** (each with remediation steps!)
- ⬜ **48 not applicable**
- 📊 **Score: 39%**

> **DevOps Pro Tip:** Run SCA scans in your CI/CD pipeline. Before deploying a new server image, verify it passes your security baseline. Automate remediation using Ansible/Chef/Puppet based on SCA findings.

---

## 🦠 2. Malware Detection

### The Problem
> A developer accidentally downloads a trojanized npm package. An attacker drops a web shell on your web server. A rootkit hides itself on a compromised machine. How do you detect any of this?

### How Wazuh Fixes It

Wazuh uses **multiple approaches** for malware detection — not just signature-based:

#### Approach 1: Behavior-Based Detection (Threat Rules)
Instead of looking for known malware files, Wazuh watches for **malware behaviors**:

| Behavior | What Wazuh Detects |
|---|---|
| Suspicious file creation | Executable dropped in temp/appdata folders |
| Registry persistence | Malware adding itself to startup |
| Suspicious DNS queries | C2 (command & control) communication |
| Service creation | Malware installing itself as a service |
| Privilege escalation | Unauthorized elevation of privileges |

**Example:** Wazuh detects LimeRAT malware by watching for:
1. `checker netflix.exe` created in `AppData\Roaming` → Alert!
2. Same file added to `Registry\Run` for persistence → Alert!
3. Same file making suspicious DNS queries → Alert!
4. New service created under `HKLM\System\Services` → Alert!

#### Approach 2: File Integrity Monitoring (FIM)
- Detects when a **web shell** is dropped on a web server
- Monitors **Windows Registry** for malware persistence
- Custom rules can look for specific **file extensions** or **content patterns**

#### Approach 3: Threat Intelligence Integration
- Integrates with **VirusTotal** — scan suspicious files against 70+ antivirus engines
- Uses **CDB Lists** — databases of known malware hashes, IPs, domains
- Connects with **MISP** — open-source threat intelligence platform

#### Approach 4: Rootkit Detection
- Scans for **known rootkit patterns** (files, directories, processes)
- Detects **hidden processes** that don't appear in `ps`
- Finds **hidden files** not visible in `ls`
- Discovers **hidden ports** not shown by `netstat`

#### Approach 5: System Call Monitoring
- Uses **Linux Audit** to monitor system calls
- Detects privilege abuse, unauthorized file access, suspicious command execution

> **DevOps Pro Tip:** Combine FIM + VirusTotal integration. When FIM detects a new executable on your server, automatically hash it and check it against VirusTotal. If it's malware, Active Response deletes it instantly.

---

## 📁 3. File Integrity Monitoring

### The Problem
> An attacker modifies `/etc/passwd` to add a backdoor user. A developer accidentally overwrites a production config file. A web shell is dropped in your web root. How do you know?

### How Wazuh Fixes It

Wazuh's **FIM module** monitors files and directories in real-time:

#### What It Tracks:

| Change Type | Example |
|---|---|
| **File created** | New script appears in `/tmp` |
| **File modified** | `/etc/shadow` permissions changed |
| **File deleted** | Important log file removed |
| **Permissions changed** | Config file made world-writable |
| **Ownership changed** | System file ownership changed to non-root |
| **Content changed** | Code injected into a PHP file |

#### What You See in Alerts:
- **Who** made the change (user/process)
- **What** was changed (content diff, permissions, ownership)
- **When** the change happened (timestamp)
- **Where** the change occurred (file path)

#### Windows Registry Monitoring
FIM also monitors **Windows Registry** — a favorite target for malware:
- Startup entries (`HKCU\Software\Microsoft\Windows\CurrentVersion\Run`)
- Service configurations
- Security policy settings

> **DevOps Pro Tip:** Monitor these critical paths:
> - Linux: `/etc/passwd`, `/etc/shadow`, `/etc/sudoers`, `/etc/ssh/sshd_config`, web roots
> - Windows: `C:\Windows\System32`, `HKLM\SOFTWARE`, startup registry keys

---

## 🔎 4. Threat Hunting

### The Problem
> Your automated rules detect known attack patterns. But what about unknown threats? What about an attacker who's already inside your network, moving slowly to avoid detection?

### How Wazuh Fixes It

Wazuh provides several capabilities for **proactive threat hunting**:

#### Capability 1: Log Data Analysis
- Centralized collection from **all sources** (endpoints, network, applications)
- **Decoders** extract meaningful fields from raw logs
- **Full-text search** across millions of events

#### Capability 2: Wazuh Archives
- Stores **ALL logs**, not just those that triggered alerts
- Disabled by default (enable for comprehensive hunting)
- Search through historical data for patterns you missed

> **Simple Example:** You discover a new attack technique. You search your archived logs going back 90 days to check if you've already been targeted.

#### Capability 3: MITRE ATT&CK Mapping
- Events mapped to **tactics and techniques** from the MITRE ATT&CK framework
- Visual dashboard showing which techniques are most common in your environment
- Helps identify **attack patterns** and **gaps in detection**

#### Capability 4: Third-Party Integrations
| Tool | How It Helps Threat Hunting |
|---|---|
| **VirusTotal** | Scan files and URLs against 70+ AV engines |
| **URLHaus** | Detect and block known malicious URLs |
| **MISP** | Cross-reference with community threat intelligence |
| **osquery** | SQL-like queries against system state |

#### Capability 5: Custom Rules & Decoders
- Write **your own detection rules** for threats specific to your environment
- Create **custom decoders** for proprietary log formats
- **Test rules** in the Dashboard's Ruleset Test tool before deploying

> **DevOps Pro Tip:** Schedule weekly threat hunting sessions. Use the MITRE ATT&CK dashboard to identify the most common techniques in your environment, then create custom rules to improve detection for your top 10 techniques.

---

## 📊 5. Log Data Analysis

### The Problem
> You have 500 servers, each generating thousands of log lines per minute. That's millions of events per day. How do you make sense of it all?

### How Wazuh Fixes It

Wazuh acts as a **centralized SIEM** for all your logs:

#### Collection
- **Agent-based:** Agents read local log files and Windows events
- **Agentless:** Syslog listener receives logs from firewalls, routers, switches
- **Cloud:** Native API integration with AWS CloudTrail, Azure, GCP

#### Decoding
Wazuh has **thousands of built-in decoders** for common log formats:
- SSH/PAM authentication
- Apache/Nginx web server
- MySQL/PostgreSQL databases
- Windows Event Logs
- AWS CloudTrail
- Docker/Kubernetes
- And many more...

#### Analysis
The **Analysis Engine** matches decoded events against **15,000+ built-in rules** covering:
- Authentication attacks (brute force, credential stuffing)
- Web attacks (SQL injection, XSS, directory traversal)
- Privilege escalation
- Data exfiltration
- Network scanning
- Malware indicators

#### Alerting
When a rule matches, you get:
- **Real-time alerts** on the Dashboard
- **Email notifications**
- **Slack/Teams messages**
- **Jira tickets** (via integrations)

> **DevOps Pro Tip:** Don't just collect logs — tune your rules. Start with Wazuh's default ruleset, then gradually add custom rules for your specific applications and suppress noisy false positives.

---

## 🔓 6. Vulnerability Detection

### The Problem
> You have 200 servers running various software. Which ones have known vulnerabilities (CVEs)? Which need urgent patching?

### How Wazuh Fixes It

Wazuh's **Vulnerability Detection** module automatically scans all your endpoints:

#### How It Works:
1. **Agent** collects **software inventory** (every installed package + version)
2. **Server** correlates inventory with **Wazuh CTI** vulnerability database
3. **Alerts** generated for each vulnerability found
4. **Dashboard** shows a comprehensive vulnerability view

#### What You See:
| Info | Example |
|---|---|
| **CVE ID** | CVE-2024-1234 |
| **Affected package** | openssl 1.1.1k |
| **Severity** | Critical (CVSS 9.8) |
| **Description** | Buffer overflow in... |
| **Remediation** | Upgrade to openssl 1.1.1l |
| **References** | Links to NVD, vendor advisories |
| **Affected agents** | web-server-01, web-server-02 |

#### Key Features:
- **Comprehensive visibility** — See ALL vulnerabilities across ALL endpoints
- **Actionable intelligence** — Each alert includes remediation steps and references
- **Remediation tracking** — Detects when a vulnerability is resolved after patching
- **Reporting** — Generate vulnerability reports for management/auditors
- **Multi-OS support** — Windows, Linux (multiple distros), macOS

#### Supported Operating Systems:
Windows, CentOS, RHEL, Ubuntu, Debian, Amazon Linux, Arch Linux, macOS

> **DevOps Pro Tip:** Integrate vulnerability reports into your patch management workflow. Prioritize by CVSS score — fix Critical and High first. Use Wazuh's API to automate vulnerability reporting in your CI/CD pipeline.

---

## 🚨 7. Incident Response

### The Problem
> A threat has been detected — an attacker is brute-forcing SSH, malware was found, or a web shell was dropped. What happens next? Every second counts.

### How Wazuh Fixes It

Wazuh provides **automated incident response** through **Active Response**:

#### Built-in Responses:
| Trigger | Automated Action |
|---|---|
| SSH brute-force detected | Block attacker's IP with iptables/firewalld |
| Malware file detected | Delete the malicious file |
| Malicious process found | Kill the process |
| Web attack detected | Block the source IP |

#### Custom Responses (You Build):
| Trigger | Custom Action |
|---|---|
| Suspicious binary found | Run it in a sandbox for analysis |
| Data exfiltration detected | Capture network traffic for forensics |
| Unauthorized USB device | Disable the USB port |
| Suspicious file detected | Scan with external antivirus |

#### Integration for IR:
| Tool | How It Helps |
|---|---|
| **Jira** | Auto-create incident tickets |
| **ServiceNow** | Track incidents in ITSM |
| **PagerDuty** | Alert on-call engineers |
| **Slack/Teams** | Notify the security team in real-time |
| **TheHive** | Full incident response platform integration |

> **DevOps Pro Tip:** Start with conservative Active Response rules. Block known-bad IPs automatically, but for more complex responses (killing processes, deleting files), start with alerts-only mode and add automation after testing.

---

## 📜 8. Regulatory Compliance

### The Problem
> Your company needs to comply with PCI DSS, HIPAA, GDPR, or other regulations. Auditors want proof. How do you demonstrate continuous compliance?

### How Wazuh Fixes It

Wazuh has **built-in compliance dashboards** and **automatic tagging**:

#### Supported Standards:
| Standard | Who Needs It | What Wazuh Covers |
|---|---|---|
| **PCI DSS** | Anyone handling credit cards | Requirements 1-12 |
| **GDPR** | Companies with EU customer data | Articles 5, 25, 32, 33, 35 |
| **HIPAA** | Healthcare organizations | Administrative, Physical, Technical safeguards |
| **NIST 800-53** | US government & contractors | All control families |
| **CIS Benchmarks** | Everyone (best practice) | OS and application hardening |
| **SOC 2** | SaaS companies | Trust Service Criteria |
| **TSC** | Service organizations | Trust Service Criteria |

#### How It Works:
1. Every Wazuh rule is **tagged** with relevant compliance mappings
2. When a rule triggers, the alert includes **compliance context**
3. Dashboard shows **dedicated compliance views** per standard
4. You can **generate reports** proving compliance to auditors

**Example alert compliance tags:**
```json
{
  "rule": {
    "description": "SSH brute-force attack detected",
    "pci_dss": ["10.2.4", "10.2.5", "11.4"],
    "gdpr": ["IV_35.7.d", "IV_32.2"],
    "hipaa": ["164.312(b)"],
    "nist_800_53": ["AU-6", "SI-4"]
  }
}
```

> **DevOps Pro Tip:** Before an audit, generate compliance reports from the Dashboard. Each report shows which controls are met, which have violations, and evidence for each. This turns a weeks-long audit prep into hours.

---

## ☁️ 9. Cloud Security

### The Problem
> Your infrastructure is on AWS/Azure/GCP. Someone creates an S3 bucket with public access. A new IAM user is created at 3 AM. A security group is opened to 0.0.0.0/0. How do you know?

### How Wazuh Fixes It

Wazuh monitors cloud infrastructure at **two levels**:

#### Level 1: System-Level (Agent on VMs)
Install the Wazuh agent on cloud instances (EC2, Azure VMs, GCP Compute) for:
- Log collection
- FIM
- Vulnerability detection
- SCA
- Malware detection

#### Level 2: Infrastructure-Level (Cloud APIs)
Wazuh **natively communicates** with cloud provider APIs:

| Cloud | Data Source | What It Monitors |
|---|---|---|
| **AWS** | CloudTrail | API calls, IAM changes, S3 access, security group changes |
| **AWS** | VPC Flow Logs | Network traffic metadata |
| **AWS** | GuardDuty | AWS threat detection findings |
| **Azure** | Activity Logs | Resource changes, sign-ins, policy changes |
| **Azure** | Active Directory | User management, authentication events |
| **GCP** | Pub/Sub | Audit logs, admin activity, data access |

> **DevOps Pro Tip:** Create custom Wazuh rules for your cloud-specific concerns. For example: alert if anyone creates an IAM user with admin privileges, or if an S3 bucket policy is changed to public.

---

## 🐳 10. Container Security

### The Problem
> You run 500 Docker containers across a Kubernetes cluster. How do you monitor what's happening inside them? How do you detect if a container is compromised?

### How Wazuh Fixes It

Wazuh provides **cloud-native runtime security** for containers:

#### Docker Monitoring:
| What It Monitors | Example |
|---|---|
| Container lifecycle | Container started, stopped, destroyed |
| Privileged mode | Alert: Container running with `--privileged` |
| Image changes | Container image was modified |
| Network config | Container network settings changed |
| Data volumes | New volume mounted or changed |
| Commands | User executed commands inside a running container |

#### How It Works:
1. Install Wazuh agent on the **Docker host**
2. Agent integrates with the **Docker Engine API**
3. Agent monitors all container activity
4. Events sent to Wazuh Server for analysis

#### Kubernetes:
- Integrates with the **Kubernetes API**
- Monitors pod creation, deletion, and modification
- Detects privilege escalation in pods
- Monitors RBAC changes

> **DevOps Pro Tip:** Set up alerts for containers running in privileged mode, containers with excessive capabilities, and any exec commands into production containers. These are the top container security red flags.

---

## 💡 Summary: Why This Matters for DevOps & Security

| Traditional Approach | With Wazuh |
|---|---|
| Check configs manually | Automated SCA scans across all endpoints |
| No malware visibility | Multi-layered malware detection |
| Reactive security | Proactive threat hunting |
| Logs scattered everywhere | Centralized log analysis |
| Manual vulnerability scanning | Continuous automated detection |
| Slow incident response | Automated Active Response |
| Audit prep takes weeks | Compliance dashboards and reports |
| Cloud security is a blind spot | Native cloud API monitoring |
| Containers are unmonitored | Docker & Kubernetes security |

---

## 🔗 References
- Use Cases Overview: https://documentation.wazuh.com/current/getting-started/use-cases/index.html
- Configuration Assessment: https://documentation.wazuh.com/current/getting-started/use-cases/configuration-assessment.html
- Malware Detection: https://documentation.wazuh.com/current/getting-started/use-cases/malware-detection.html
- Vulnerability Detection: https://documentation.wazuh.com/current/getting-started/use-cases/vulnerability-detection.html
- Threat Hunting: https://documentation.wazuh.com/current/getting-started/use-cases/threat-hunting.html

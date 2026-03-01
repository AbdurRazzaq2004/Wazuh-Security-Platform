# Wazuh + OpenClaw: AI-Powered Security Monitoring

> **Automated threat detection, compliance monitoring, and incident response — managed through natural language via Telegram.**

---

## What Is This Project?

This project combines **Wazuh** (an open-source security platform) with **OpenClaw** (an AI assistant framework) to create a fully automated security monitoring system that anyone can interact with using plain English — no dashboards, no complex queries, no security expertise required.

Instead of staring at dashboards or writing search queries, you simply message a Telegram bot:

> *"Are there any security threats on our servers?"*

And get back a detailed, human-readable answer — sourced directly from real-time security data.

---

## The Problem It Solves

| Challenge | How This Project Addresses It |
|-----------|-------------------------------|
| Security tools generate thousands of alerts — most are noise | AI filters and prioritizes, Slack fires **only when real issues are found** |
| Dashboards require expertise to interpret | Telegram bot explains findings in plain language |
| Compliance audits are manual, slow, and expensive | Automated checks against PCI DSS, HIPAA, NIST 800-53, SOC 2, ISO 27001, CMMC |
| Brute-force attacks go unnoticed until it's too late | Real-time detection + automatic IP blocking (active response) |
| File tampering is hard to catch | Real-time File Integrity Monitoring (FIM) on critical system paths |
| Small teams can't afford 24/7 security operations | Cron-based automation runs health checks, reports, and audits around the clock |

---

## How It Works

```
┌─────────────┐    reports to     ┌─────────────────┐    stores in    ┌──────────────┐
│ Wazuh Agent │ ───────────────▶  │  Wazuh Manager  │ ────────────▶  │  OpenSearch   │
│ (on server) │                   │  (Docker)       │                │  (Indexer)    │
└─────────────┘                   └────────┬────────┘                └──────┬───────┘
                                           │                                │
                                    auto-blocks                      queries alerts
                                    attackers                        & analytics
                                           │                                │
                                  ┌────────▼────────┐               ┌──────▼───────┐
                                  │ Active Response  │               │   OpenClaw    │
                                  │ (firewall-drop)  │               │  (AI Engine)  │
                                  └─────────────────┘               └──────┬───────┘
                                                                           │
                                                              ┌────────────┼────────────┐
                                                              │            │            │
                                                        ┌─────▼──┐  ┌─────▼──┐  ┌─────▼──┐
                                                        │Telegram│  │ Slack  │  │Control │
                                                        │  Bot   │  │Alerts  │  │  UI    │
                                                        └────────┘  └────────┘  └────────┘
```

**In simple terms:**

1. **Wazuh Agents** sit on your servers and watch everything — logins, file changes, processes, network connections, vulnerabilities, compliance posture.
2. **Wazuh Manager** collects all that data, correlates events into meaningful alerts, and auto-blocks attackers.
3. **OpenClaw** connects to Wazuh's API and alert database, understands security data through AI, and translates it into actionable insights.
4. **You** ask questions via Telegram or check Slack for automated notifications.

---

## What It Monitors

### Endpoint Security
- **Security Configuration Assessment (SCA)** — Checks servers against CIS benchmarks (e.g., CIS Ubuntu 24.04). Identifies misconfigurations like weak SSH settings, missing firewall rules, or improper file permissions.
- **Rootkit & Malware Detection** — Scans for hidden processes, suspicious files, trojans, and known rootkit signatures.
- **File Integrity Monitoring (FIM)** — Watches critical system files (`/etc`, `/bin`, `/usr/bin`, SSH keys, cron jobs) in real-time. Any unauthorized change triggers an alert.

### Threat Intelligence
- **Threat Hunting** — Tracks SSH brute-force attacks, failed authentication attempts, suspicious source IPs, and unusual login patterns.
- **Vulnerability Detection** — Identifies known CVEs on monitored servers by cross-referencing installed packages.
- **MITRE ATT&CK Mapping** — Maps detected threats to the MITRE ATT&CK framework (tactics, techniques) for structured threat analysis.

### Compliance Monitoring
Automated checks against **7 major frameworks**:

| Framework | Coverage |
|-----------|----------|
| PCI DSS v3.2.1 & v4.0 | Payment card security |
| HIPAA | Healthcare data protection |
| NIST SP 800-53 | US federal information security |
| SOC 2 / TSC | Service organization controls |
| ISO 27001:2013 | International information security |
| CMMC v2.0 | Defense contractor cybersecurity |

### Active Response
- Auto-blocks SSH brute-force attackers using `firewall-drop` (10 min) and `host-deny` (30 min)
- Triggers on Wazuh rules 5763 (multiple auth failures) and 5720 (SSH brute force)

---

## Why Wazuh?

| Criteria | Wazuh | Alternatives |
|----------|-------|-------------|
| **Cost** | Free & open-source | Splunk, CrowdStrike, SentinelOne — $10K-$100K+/year |
| **Capabilities** | SIEM + XDR + compliance in one platform | Often need 3-4 separate tools |
| **Agent support** | Linux, Windows, macOS, containers | Varies |
| **Compliance** | Built-in PCI DSS, HIPAA, NIST, SOC2, GDPR, ISO 27001 | Often add-on modules |
| **Community** | 20K+ GitHub stars, active development | Proprietary / closed source |
| **Scalability** | Single-node to multi-cluster | Some tools don't scale without expensive licenses |
| **Integration** | REST API, OpenSearch, Slack, Syslog, and more | Vendor lock-in common |

**Bottom line:** Wazuh gives enterprise-grade security monitoring at zero licensing cost.

---

## Why OpenClaw?

OpenClaw turns Wazuh from a "tool for security engineers" into a "tool for everyone."

- **Natural language interface** — Ask security questions in plain English via Telegram
- **AI-powered analysis** — LLM interprets raw security data and explains findings in context
- **Skill-based architecture** — Custom skills teach the AI exactly how to query Wazuh APIs and interpret results
- **Script execution** — AI can run health checks, audits, and reports directly on the server
- **Multi-channel** — Telegram bot, Slack notifications, and web-based Control UI

Without OpenClaw, you'd need a trained security analyst watching dashboards. With OpenClaw, anyone on the team can ask *"Are we secure?"* and get a real answer.

---

## Automation Schedule

| Frequency | What Runs | Output |
|-----------|-----------|--------|
| Every 15 minutes | Alert check (7 categories) | Slack notification **only if issues found** |
| Every 6 hours | Security report (MITRE, FIM, SCA) | Log file + available via Telegram |
| Daily at 08:00 UTC | Comprehensive daily report | Log file + available via Telegram |
| Real-time | Wazuh integratord | Slack alert for any level 7+ event |
| On-demand | Deep security audit (10 sections) | Full audit report via Telegram |

---

## Key Benefits

### 1. Zero Alert Fatigue
The system uses **alert-only mode** — Slack notifications fire only when actual security issues are detected. No more "all clear" spam every 15 minutes.

### 2. Enterprise Compliance Without Enterprise Cost
Automated compliance checks against 7 frameworks, with specific control-level mapping (e.g., PCI DSS Requirement 2.2, HIPAA §164.312(a)(1)). A compliance audit that would take days is reduced to a single Telegram message.

### 3. Proactive Defense
Active response automatically blocks brute-force attackers. FIM detects unauthorized file changes in real-time. You don't wait for a breach — the system acts immediately.

### 4. Accessible Security
No need to learn OpenSearch query syntax, Wazuh API endpoints, or security frameworks. Just ask the Telegram bot in natural language. The AI handles the complexity.

### 5. Full Visibility
10-section deep audit covers: system health, alert analysis, file integrity, malware, vulnerabilities, threat hunting, SCA compliance, regulatory compliance, IT hygiene, and active response — all in one report.

### 6. Cost Effective
The entire stack runs on a single EC2 instance. All components are open-source or free-tier. No per-agent licensing, no per-GB ingestion fees.

---

## Tech Stack

| Component | Technology | Role |
|-----------|-----------|------|
| SIEM / XDR | Wazuh 4.14.3 | Security event collection, correlation, and response |
| Data Store | OpenSearch (Wazuh Indexer) | Alert storage and search |
| AI Engine | OpenClaw 2026.2.26 | Natural language interface and automation |
| LLM | Configurable (GPT, Claude, etc.) | Powers AI analysis and responses |
| Containers | Docker Compose | Runs Wazuh single-node stack |
| Notifications | Slack (webhook + native integration) | Real-time alerts and reports |
| Chat Interface | Telegram Bot | User interaction via natural language |
| Infrastructure | AWS EC2 (Ubuntu) | Hosting |
| Scripting | Bash + curl + jq | Health checks, reports, audits |

---

## Deployed Components

```
~/.openclaw/skills/wazuh-monitor/
├── SKILL.md                  # AI knowledge base (333 lines, 4 modules)
├── wazuh-health-check.sh     # Quick system health overview
├── wazuh-alert-check.sh      # 15-min alert check (alert-only mode)
├── wazuh-report.sh           # Periodic report (MITRE, FIM, SCA)
└── wazuh-deep-audit.sh       # Comprehensive 10-section audit (280 lines)
```

---

## Example Interactions

**Via Telegram:**

> **You:** "Check if any servers have been attacked in the last 24 hours."
>
> **Bot:** "In the last 24 hours, I found 496 security alerts across 1 active agent. The most notable findings: 2 unique source IPs attempted SSH brute-force attacks (223.x.x.x and 218.x.x.x). Active response has automatically blocked these IPs. Additionally, 117 CIS benchmark checks are failing on test-server1..."

> **You:** "Run a full security audit."
>
> **Bot:** *(runs wazuh-deep-audit.sh and analyzes the output — returns a structured 10-section report covering health, alerts, FIM, malware, vulnerabilities, threat hunting, SCA, compliance, IT hygiene, and active response)*

> **You:** "Are we HIPAA compliant?"
>
> **Bot:** "Based on the current SCA results mapped to HIPAA controls, there are 45 findings related to HIPAA requirements. Key areas needing attention: access controls (§164.312(a)), audit controls (§164.312(b)), and transmission security (§164.312(e))..."

**Via Slack (automatic):**

> 🚨 **Security Alert** — 3 issues detected
> - High-severity alerts (level 12+): 2 events
> - SSH brute force from 223.x.x.x: 47 attempts in 15 min
> - FIM change detected: /etc/passwd modified

---

## Who Is This For?

| Audience | Use Case |
|----------|----------|
| **Small/medium teams** | Enterprise-grade security without hiring a SOC team |
| **DevOps engineers** | Integrate security monitoring into existing workflows |
| **Compliance officers** | Automated evidence collection for audits |
| **Startups** | Ship with security from day one — at zero cost |
| **Students & learners** | Hands-on SIEM/XDR experience with AI integration |
| **MSPs / consultants** | Offer security monitoring as a managed service |

---

## Limitations & Considerations

- **Single-node deployment** — Suitable for small to medium environments. For large-scale, Wazuh supports multi-node clusters.
- **AI accuracy** — LLM responses are based on the data available. Complex edge cases may need human review.
- **Default credentials** — Must be changed before production use.
- **Agent coverage** — Only monitors servers with Wazuh agents installed. Agentless monitoring is limited.
- **Internet dependency** — OpenClaw requires internet access for LLM API calls (unless using a local model).

---

## Summary

This project delivers a **complete, AI-powered security operations center** using entirely open-source tools:

- **Wazuh** handles the heavy lifting — agent-based monitoring, alert correlation, compliance checking, and active response.
- **OpenClaw** makes it accessible — turning complex security data into conversational insights via Telegram.
- **Automation** keeps it running — cron jobs, alert-only notifications, and real-time integrations ensure nothing is missed.

The result: **24/7 security monitoring, automated compliance, proactive threat response — all manageable from a chat window.**

---

*Built with Wazuh 4.14.3 + OpenClaw 2026.2.26 | Deployed on AWS EC2 | March 2026*

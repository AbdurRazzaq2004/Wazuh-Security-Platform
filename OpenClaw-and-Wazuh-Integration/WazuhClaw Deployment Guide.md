# OpenClaw + Wazuh — Deployment & Integration Guide (Updated)

> **Document Type:** Step-by-Step Deployment Guide  
> **Version:** 3.0 (Advanced Security Monitoring Edition)  
> **Date:** 2026-03-01  
> **Status:** Execution-Ready — All steps verified on Ubuntu 24.04 (AWS EC2)  
> **Original Guide:** [dep-guide.md](https://github.com/AbdurRazzaq2004/Wazuh-Security-Platform/blob/main/OpenClaw-and-Wazuh-Integration/dep-guide.md)

---

## What Changed From the Original Guide

This document is a **corrected and updated** version of the original deployment guide. The following critical issues were discovered and fixed during production deployment:

| # | Issue | Fix Applied |
|---|-------|-------------|
| 1 | Wazuh API default credentials `wazuh:wazuh` don't work | Correct user is **`wazuh-wui`** with password from `docker-compose.yml` (`MyS3cr37P450r.*-`) |
| 2 | Wazuh API `/alerts` endpoint returns 404 | Alerts must be queried from **OpenSearch/Indexer** (port 9200), not the Wazuh API |
| 3 | OpenClaw `channels.telegram.token` is invalid config key | Correct key is **`channels.telegram.botToken`** |
| 4 | OpenClaw `tools.exec.security "none"` is invalid | Valid values are **`deny`**, **`allowlist`**, or **`full`** |
| 5 | OpenClaw `tools.exec.allowlist` key doesn't exist | Use `tools.exec.security "full"` to allow all exec commands |
| 6 | `openclaw onboard` may skip model & channel configuration | Run `openclaw configure --section model` and `openclaw configure --section telegram` separately |
| 7 | Gateway mode unset after install blocks gateway start | Run `openclaw config set gateway.mode local` |
| 8 | Telegram bot requires pairing approval after `/start` | Run `openclaw pairing approve telegram <PAIRING_CODE>` |
| 9 | Wazuh Slack `<integration>` block added outside `</ossec_config>` causes config crash | Must be placed **inside** the `</ossec_config>` closing tag |
| 10 | Heredocs with backtick code blocks break when pasted into terminal | Removed backtick code fences from SKILL.md heredoc content |
| 11 | Report script fails silently when `jq` receives invalid JSON from 404 endpoints | Rewrote report script to use OpenSearch and added error handling |
| 12 | Wazuh API password contains special chars (`.*`) breaking shell expansion | Must use **single quotes** around the password in curl commands |

### v3.0 Changes (Advanced Security Monitoring)

| # | Enhancement | Details |
|---|-------------|----------|
| 13 | Alert script switched to **alert-only mode** | Removed "always send good status" — Slack notifications only fire when actual issues are detected. Quiet when healthy. |
| 14 | Alert script expanded to **7 detection categories** | Now checks: agent health, high-severity alerts, FIM changes, brute force/auth failures, rootcheck/malware, indexer health, vulnerability alerts |
| 15 | FIM enhanced with **real-time monitoring** | Manager frequency reduced 12h→2h, agent frequency 1h, `realtime="yes"` on critical dirs, `report_changes="yes"` |
| 16 | **Active Response** enabled | Auto-blocks SSH brute force IPs via `firewall-drop` (10 min) + `host-deny` (30 min) for rules 5763/5720 |
| 17 | Slack integration level lowered **10→7** | Captures medium-severity threats, not just critical |
| 18 | `log_alert_level` lowered **3→1** | Stores all alert levels for comprehensive analysis |
| 19 | **Advanced SKILL.md** rewritten (333 lines) | 4 modules: Endpoint Security, Threat Intelligence, Security Operations & Compliance, Manager Health |
| 20 | **Deep Security Audit script** created | New `wazuh-deep-audit.sh` — 280 lines, 10-section comprehensive audit |
| 21 | **Report script upgraded** | Added MITRE ATT&CK tactics/techniques, FIM events, SCA compliance summary |
| 22 | **Centralized agent config** via `agent.conf` | Pushes enhanced FIM, rootcheck, SCA, syscollector settings to all agents |
| 23 | **Compliance framework monitoring** added | PCI DSS v4.0, GDPR, HIPAA, NIST 800-53, SOC 2/TSC, ISO 27001, CMMC v2.0 |
| 24 | **Rootcheck** frequency increased | Every 2h (was 12h), all checks enabled |
| 25 | **Syscollector** interval reduced | Every 30m on agents (was 1h), hotfixes enabled |
| 26 | `<directories>` tags incorrectly placed in `<rootcheck>` and `<sca>` | Removed — `<directories>` only valid inside `<syscheck>` |

---

## Table of Contents

1. [Prerequisites](#1-prerequisites)
2. [Phase 1 — Deploy Wazuh Single-Node (Docker)](#2-phase-1--deploy-wazuh-single-node-docker)
3. [Phase 2 — Install & Configure OpenClaw](#3-phase-2--install--configure-openclaw)
4. [Phase 3 — Create the Wazuh Monitoring Skill](#4-phase-3--create-the-wazuh-monitoring-skill)
5. [Phase 4 — Connect OpenClaw to Wazuh](#5-phase-4--connect-openclaw-to-wazuh)
6. [Phase 5 — Slack Notifications](#6-phase-5--slack-notifications)
7. [Phase 6 — Schedule Automated Monitoring](#7-phase-6--schedule-automated-monitoring)
8. [**Phase 7 — Advanced Security Monitoring (NEW)**](#8-phase-7--advanced-security-monitoring)
9. [Verification & Testing](#9-verification--testing)
10. [Troubleshooting](#10-troubleshooting)
11. [Architecture Overview](#11-architecture-overview)
12. [Appendices](#12-appendices)

---

## 1. Prerequisites

### 1.1 System Requirements

| Resource | Minimum | Recommended |
|----------|---------|-------------|
| CPU | 4 cores | 6+ cores |
| RAM | 8 GB | 12+ GB |
| Disk | 50 GB | 100+ GB SSD |
| OS | Ubuntu 22.04/24.04 LTS | Ubuntu 24.04 LTS |
| Node.js | 22+ | Latest LTS |
| Docker | Latest stable | Latest stable |
| Docker Compose | v2+ (included with Docker) | Latest |
| Git | Any | Latest |

### 1.2 Software Checklist

Run these before starting:

```bash
# Verify Docker
docker --version
docker compose version

# Verify Node.js (required by OpenClaw)
node --version   # Must be 22+

# Verify Git
git --version

# Verify jq (used by monitoring scripts)
jq --version
```

If Node.js 22+ is not installed:

```bash
# Ubuntu/Debian
curl -fsSL https://deb.nodesource.com/setup_22.x | sudo -E bash -
sudo apt-get install -y nodejs
```

If jq is not installed:

```bash
sudo apt install -y jq
```

### 1.3 Network Ports Summary

After full deployment, these ports will be in use:

| Port | Service | Protocol | Description |
|------|---------|----------|-------------|
| 443 | Wazuh Dashboard | HTTPS | Web UI for security monitoring |
| 1514 | Wazuh Manager | TCP | Agent communication |
| 1515 | Wazuh Manager | TCP | Agent enrollment |
| 9200 | Wazuh Indexer | HTTPS | OpenSearch API (alerts, queries) |
| 55000 | Wazuh API | HTTPS | REST API (manager/agent info) |
| 18789 | OpenClaw Gateway | HTTP | Control UI & agent communication |

---

## 2. Phase 1 — Deploy Wazuh Single-Node (Docker)

### 2.1 Prepare the Docker Host

```bash
# Set kernel parameter for Wazuh Indexer (OpenSearch)
sudo sysctl -w vm.max_map_count=262144

# Make it persistent across reboots
echo "vm.max_map_count=262144" | sudo tee -a /etc/sysctl.conf

# Add your user to docker group (Linux only)
sudo usermod -aG docker $USER
# Log out and back in for this to take effect
```

### 2.2 Clone the Wazuh Docker Repository

```bash
# Clone the official Wazuh Docker repo (v4.14.3 — latest stable)
git clone https://github.com/wazuh/wazuh-docker.git -b v4.14.3

# Navigate to single-node deployment
cd wazuh-docker/single-node/
```

### 2.3 Generate SSL Certificates

```bash
# Generate self-signed certificates for all Wazuh components
docker compose -f generate-indexer-certs.yml run --rm generator
```

This creates certificates in `./config/wazuh_indexer_ssl_certs/`.

### 2.4 Note the API Password

> **⚠️ CRITICAL FIX:** The Wazuh API does NOT use `wazuh:wazuh` as default credentials.

Check your `docker-compose.yml` for the actual API password:

```bash
grep -i "API_PASSWORD" docker-compose.yml
```

Expected output:

```
- API_PASSWORD=MyS3cr37P450r.*-
```

> **Save this password.** You will need it for:
> - The API user is **`wazuh-wui`** (NOT `wazuh`)
> - All scripts that authenticate with the Wazuh API
> - The SKILL.md configuration

Also note the dashboard credentials:
- **Username:** `admin`
- **Password:** `SecretPassword`

### 2.5 Deploy Wazuh Stack

```bash
# Start all containers in detached mode
docker compose up -d
```

This spins up three containers:
- **wazuh.manager** — Detection engine, rule processing, agent management
- **wazuh.indexer** — OpenSearch for data storage and indexing
- **wazuh.dashboard** — Web interface (Kibana fork)

### 2.6 Verify Wazuh Deployment

```bash
# Check all containers are running
docker compose ps
# Expected: 3 containers all "Up"
```

Wait ~60–90 seconds for the indexer to fully initialize, then:

```bash
# Test the Wazuh Dashboard
curl -sk https://localhost | head -20

# Test the Wazuh API (use correct credentials!)
TOKEN=$(curl -su 'wazuh-wui:MyS3cr37P450r.*-' -k -X POST \
  "https://localhost:55000/security/user/authenticate?raw=true")
curl -k -X GET "https://localhost:55000/?pretty=true" -H "Authorization: Bearer $TOKEN"
```

> **⚠️ FIX: Always use single quotes** around the password in curl commands.  
> The password `MyS3cr37P450r.*-` contains `.*` which the shell will glob-expand with double quotes.

Expected API response:

```json
{
  "data": {
    "title": "Wazuh API REST",
    "api_version": "4.14.3",
    ...
  },
  "error": 0
}
```

### 2.7 Access Wazuh Dashboard

Open in your browser: `https://<SERVER_IP>` (accepts self-signed cert warning)

| Field | Value |
|-------|-------|
| Username | `admin` |
| Password | `SecretPassword` |

> **⚠️ Change the default password immediately for production use.**  
> Follow: [Wazuh — Changing Default Passwords](https://documentation.wazuh.com/current/deployment-options/docker/changing-default-password.html)

### 2.8 Enroll a Wazuh Agent (Required for Monitoring Data)

To generate real security data, enroll a Wazuh agent on any endpoint:

```bash
# On the target endpoint (Ubuntu example)
curl -s https://packages.wazuh.com/key/GPG-KEY-WAZUH | gpg --no-default-keyring \
  --keyring gnupg-ring:/usr/share/keyrings/wazuh.gpg --import && \
  chmod 644 /usr/share/keyrings/wazuh.gpg

echo "deb [signed-by=/usr/share/keyrings/wazuh.gpg] \
  https://packages.wazuh.com/4.x/apt/ stable main" | \
  tee /etc/apt/sources.list.d/wazuh.list

apt-get update && apt-get install wazuh-agent

# Configure and start (replace <YOUR_SERVER_IP> with your Wazuh server IP)
sed -i "s/MANAGER_IP/<YOUR_SERVER_IP>/" /var/ossec/etc/ossec.conf
systemctl daemon-reload
systemctl enable wazuh-agent
systemctl start wazuh-agent
```

### 2.9 Test Wazuh API Endpoints (Reference)

> **⚠️ CRITICAL FIX:** The `/alerts` endpoint does NOT exist in Wazuh 4.14.3.  
> Alerts must be queried from OpenSearch (port 9200), NOT from the Wazuh API (port 55000).

**Working Wazuh API endpoints (port 55000):**

```bash
# Authenticate first (always)
TOKEN=$(curl -su 'wazuh-wui:MyS3cr37P450r.*-' -k -X POST \
  "https://localhost:55000/security/user/authenticate?raw=true")

# Agent summary
curl -k -X GET "https://localhost:55000/agents/summary/status?pretty=true" \
  -H "Authorization: Bearer $TOKEN"

# Manager status
curl -k -X GET "https://localhost:55000/manager/status?pretty=true" \
  -H "Authorization: Bearer $TOKEN"

# Manager info
curl -k -X GET "https://localhost:55000/manager/info?pretty=true" \
  -H "Authorization: Bearer $TOKEN"
```

**Alert queries via OpenSearch/Indexer (port 9200):**

```bash
# Recent alerts (last 20)
curl -ku admin:SecretPassword -X GET "https://localhost:9200/wazuh-alerts-*/_search?pretty" \
  -H "Content-Type: application/json" \
  -d '{"size": 20, "sort": [{"timestamp": {"order": "desc"}}]}'

# High-severity alerts (level > 10)
curl -ku admin:SecretPassword -X GET "https://localhost:9200/wazuh-alerts-*/_search?pretty" \
  -H "Content-Type: application/json" \
  -d '{"size": 20, "sort": [{"timestamp": {"order": "desc"}}], "query": {"range": {"rule.level": {"gt": 10}}}}'

# Alert count (last 24h)
curl -ku admin:SecretPassword -X GET "https://localhost:9200/wazuh-alerts-*/_count" \
  -H "Content-Type: application/json" \
  -d '{"query": {"range": {"timestamp": {"gte": "now-24h"}}}}'

# MITRE ATT&CK breakdown
curl -ku admin:SecretPassword -X GET "https://localhost:9200/wazuh-alerts-*/_search?pretty" \
  -H "Content-Type: application/json" \
  -d '{
    "size": 0,
    "aggs": {
      "mitre_techniques": {
        "terms": {"field": "rule.mitre.technique.keyword", "size": 20}
      }
    }
  }'

# Alerts from the last hour
curl -ku admin:SecretPassword -X GET "https://localhost:9200/wazuh-alerts-*/_search?pretty" \
  -H "Content-Type: application/json" \
  -d '{
    "size": 20,
    "query": {"range": {"timestamp": {"gte": "now-1h"}}},
    "sort": [{"timestamp": {"order": "desc"}}]
  }'
```

---

## 3. Phase 2 — Install & Configure OpenClaw

### 3.1 Install OpenClaw

```bash
curl -fsSL https://openclaw.ai/install.sh | bash
```

### 3.2 Run Onboarding Wizard

```bash
openclaw onboard --install-daemon
```

The wizard walks you through:
1. **Auth configuration** — API key for your LLM provider (Anthropic/OpenAI)
2. **Gateway settings** — Port, logging, mode
3. **Hooks** — Enable recommended hooks (boot-md, bootstrap-extra-files, command-logger, session-memory)
4. **Systemd service** — Install and enable the gateway as a systemd user service

> **⚠️ FIX:** The onboarding wizard may **skip model selection and Telegram configuration**.  
> If it does, configure them manually in the next steps.

### 3.3 Configure the LLM Model (if skipped by onboarding)

```bash
# Interactive model configuration
openclaw configure --section model
```

Or set directly:

```bash
# For Anthropic (Claude)
openclaw config set agents.defaults.model "anthropic:claude-sonnet-4-20250514"
echo 'export ANTHROPIC_API_KEY="your-key-here"' >> ~/.bashrc
source ~/.bashrc

# For OpenAI
openclaw config set agents.defaults.model "openai:gpt-4o"
echo 'export OPENAI_API_KEY="your-key-here"' >> ~/.bashrc
source ~/.bashrc
```

### 3.4 Set Gateway Mode (if not set by onboarding)

> **⚠️ FIX:** If `openclaw doctor` shows "gateway.mode is unset; gateway start will be blocked":

```bash
openclaw config set gateway.mode local
```

### 3.5 Configure Telegram Bot

**Step A — Create a Telegram bot:**

1. Open Telegram, search for **@BotFather**
2. Send `/newbot`
3. Follow the prompts (give it a name and username)
4. Copy the **bot token** BotFather gives you (format: `123456789:ABCdef...`)

**Step B — Set the token in OpenClaw:**

> **⚠️ FIX:** The config key is **`botToken`**, NOT `token`.  
> `openclaw config set channels.telegram.token` will fail with "Unrecognized key".

```bash
openclaw config set channels.telegram.enabled true
openclaw config set channels.telegram.botToken "YOUR_BOT_TOKEN_HERE"
```

**Step C — Restart and pair:**

```bash
openclaw daemon restart
```

1. Open Telegram, go to your bot (e.g., `@wazuhclaw_bot`)
2. Send `/start`
3. The bot will respond with a pairing code like `XY7Y3AL6`

> **⚠️ FIX:** You must approve the pairing from the server:

```bash
openclaw pairing approve telegram <PAIRING_CODE>
```

Example:

```bash
openclaw pairing approve telegram XY7Y3AL6
```

**Step D — Test the bot:**

Send "Hello, are you running?" in Telegram. The bot should respond.

### 3.6 Disable Memory Search Warning (Optional)

If you see "Memory search is enabled but no embedding provider is configured":

```bash
# Option A: Disable it
openclaw config set agents.defaults.memorySearch.enabled false

# Option B: Set an API key for embeddings
export OPENAI_API_KEY="sk-..."
```

### 3.7 Verify OpenClaw

```bash
# Check gateway status
openclaw gateway status

# Run diagnostics
openclaw doctor
```

The Control UI is available at: `http://127.0.0.1:18789/`

For remote servers (no GUI), use SSH tunnel:

```bash
# From your local machine
ssh -N -L 18789:127.0.0.1:18789 ubuntu@<SERVER_IP>
# Then open http://localhost:18789/ in your browser
```

---

## 4. Phase 3 — Create the Wazuh Monitoring Skill

This teaches OpenClaw how to interact with Wazuh. We create a custom skill with instructions and helper scripts.

### 4.1 Create the Skill Directory

```bash
mkdir -p ~/.openclaw/skills/wazuh-monitor
```

### 4.2 Create the Skill Definition (SKILL.md)

> **⚠️ FIX:** Uses correct API credentials (`wazuh-wui`), correct password, and documents that `/alerts` endpoint doesn't exist — use OpenSearch instead.

```bash
cat > ~/.openclaw/skills/wazuh-monitor/SKILL.md << 'SKILL_EOF'
---
name: wazuh-monitor
description: Monitor and analyze Wazuh SIEM/XDR data — agents, alerts, MITRE ATT&CK, vulnerabilities, compliance, and IDS events. Generate security reports.
metadata: {"openclaw": {"always": true, "emoji": "🛡️", "requires": {"bins": ["curl", "jq"]}}}
---

# Wazuh Security Monitor

You have access to a Wazuh SIEM/XDR instance running on this server. Use the exec tool to interact with the Wazuh REST API.

## Connection Details
- **Wazuh API URL:** https://localhost:55000
- **Dashboard URL:** https://localhost
- **Wazuh Indexer (OpenSearch):** https://localhost:9200
- **API Credentials:** Username wazuh-wui, Password MyS3cr37P450r.*-
- **Dashboard Credentials:** Username admin, Password SecretPassword
- **All API calls require -k flag** (self-signed certificates)

## Authentication
Every Wazuh API session requires a JWT token. Always authenticate first:
TOKEN=$(curl -su 'wazuh-wui:MyS3cr37P450r.*-' -k -X POST "https://localhost:55000/security/user/authenticate?raw=true")
Then include in requests:
curl -k -X GET "https://localhost:55000/<ENDPOINT>?pretty=true" -H "Authorization: Bearer $TOKEN"
Tokens expire after 900 seconds (15 minutes). Re-authenticate if you get 401 errors.

IMPORTANT: Always use single quotes around the password in curl commands because it contains special characters (.*).

## Available API Endpoints (Wazuh API — Port 55000)

### Agent Management
- /agents?pretty=true — List all registered agents
- /agents/summary/status?pretty=true — Agent status summary
- /agents?status=active&pretty=true — List active agents only
- /agents?status=disconnected&pretty=true — List disconnected agents
- /agents/{agent_id}?pretty=true — Detailed info for specific agent
- /agents/summary/os?pretty=true — OS distribution across agents

### Manager & System Health
- /manager/status?pretty=true — Manager running status
- /manager/info?pretty=true — Manager version and info
- /manager/configuration?pretty=true — Current configuration
- /manager/stats?pretty=true — Performance statistics
- /manager/logs?pretty=true&limit=20 — Recent manager logs

### MITRE ATT&CK
- /mitre?pretty=true — MITRE ATT&CK framework data
- /mitre/tactics?pretty=true — Tactics catalog
- /mitre/techniques?pretty=true — Techniques catalog

### Vulnerability Detection
- /vulnerability/{agent_id}?pretty=true — Vulnerabilities for specific agent
- /vulnerability/{agent_id}/summary?pretty=true — Vulnerability summary per agent

### Rules & Decoders
- /rules?pretty=true&limit=10 — Detection rules
- /rules?pretty=true&q=level>12 — Critical detection rules
- /decoders?pretty=true&limit=10 — Log decoders

### File Integrity Monitoring (FIM)
- /syscheck/{agent_id}?pretty=true — FIM results for agent
- /syscheck/{agent_id}/last_scan?pretty=true — Last FIM scan info

### Security Configuration Assessment (SCA)
- /sca/{agent_id}?pretty=true — SCA policies for agent
- /sca/{agent_id}/checks/{policy_id}?pretty=true — SCA check results

### System Inventory (Syscollector)
- /syscollector/{agent_id}/os?pretty=true — OS info
- /syscollector/{agent_id}/packages?pretty=true — Installed packages
- /syscollector/{agent_id}/processes?pretty=true — Running processes
- /syscollector/{agent_id}/ports?pretty=true — Open ports

## Alert Queries — OpenSearch (Port 9200)

IMPORTANT: The Wazuh REST API on port 55000 does NOT have an /alerts endpoint in v4.14.3.
All alert queries MUST go through the Wazuh Indexer (OpenSearch) on port 9200.
Use: curl -ku admin:SecretPassword for indexer queries.

Recent alerts:
curl -ku admin:SecretPassword -X GET "https://localhost:9200/wazuh-alerts-*/_search?pretty" -H "Content-Type: application/json" -d '{"size": 20, "sort": [{"timestamp": {"order": "desc"}}]}'

High severity alerts (level > 10):
curl -ku admin:SecretPassword -X GET "https://localhost:9200/wazuh-alerts-*/_search?pretty" -H "Content-Type: application/json" -d '{"size": 20, "sort": [{"timestamp": {"order": "desc"}}], "query": {"range": {"rule.level": {"gt": 10}}}}'

Alert count (last 24h):
curl -ku admin:SecretPassword -X GET "https://localhost:9200/wazuh-alerts-*/_count" -H "Content-Type: application/json" -d '{"query": {"range": {"timestamp": {"gte": "now-24h"}}}}'

MITRE ATT&CK aggregation:
curl -ku admin:SecretPassword -X GET "https://localhost:9200/wazuh-alerts-*/_search?pretty" -H "Content-Type: application/json" -d '{"size": 0, "aggs": {"mitre_techniques": {"terms": {"field": "rule.mitre.technique.keyword", "size": 20}}}}'

## Report Generation
When asked to generate a security report, follow this structure:
1. Executive Summary
2. Agent Health
3. Alert Summary
4. Top Threats with MITRE ATT&CK mapping
5. IDS Events
6. Vulnerability Status
7. Compliance Status
8. File Integrity
9. Recommendations

## Usage Guidelines
- Always authenticate before making API calls
- Use jq to format and filter JSON responses
- Rule levels: 0-3 (info), 4-7 (low), 8-11 (medium), 12-15 (high/critical)
- Sort alerts by timestamp descending for most recent first
- Re-authenticate if you get 401 errors (token expires in 15 min)
- Always use single quotes around passwords with special characters

## Example Monitoring Workflow
TOKEN=$(curl -su 'wazuh-wui:MyS3cr37P450r.*-' -k -X POST "https://localhost:55000/security/user/authenticate?raw=true")
curl -k -X GET "https://localhost:55000/agents/summary/status?pretty=true" -H "Authorization: Bearer $TOKEN"
curl -k -X GET "https://localhost:55000/manager/status?pretty=true" -H "Authorization: Bearer $TOKEN"
curl -ku admin:SecretPassword "https://localhost:9200/wazuh-alerts-*/_count" -H "Content-Type: application/json" -d '{"query":{"range":{"timestamp":{"gte":"now-24h"}}}}'
SKILL_EOF
```

### 4.3 Create the Health Check Script

```bash
cat > ~/.openclaw/skills/wazuh-monitor/wazuh-health-check.sh << 'SCRIPT_EOF'
#!/bin/bash
set -euo pipefail
WAZUH_API="https://localhost:55000"
WAZUH_USER="${WAZUH_API_USER:-wazuh-wui}"
WAZUH_PASS="${WAZUH_API_PASSWORD:-MyS3cr37P450r.*-}"
INDEXER_URL="https://localhost:9200"
INDEXER_USER="${INDEXER_USER:-admin}"
INDEXER_PASS="${INDEXER_PASS:-SecretPassword}"
echo "============================================"
echo "  Wazuh Security Health Check"
echo "  $(date -u '+%Y-%m-%d %H:%M:%S UTC')"
echo "============================================"
echo ""
echo "--- Authenticating ---"
TOKEN=$(curl -su "$WAZUH_USER:$WAZUH_PASS" -k -X POST "$WAZUH_API/security/user/authenticate?raw=true" 2>/dev/null)
if [ -z "$TOKEN" ] || echo "$TOKEN" | grep -q "Unauthorized\|Invalid\|error"; then
  echo "ERROR: Failed to authenticate with Wazuh API"; exit 1
fi
echo "Authentication: OK"
echo ""
echo "--- Manager Status ---"
curl -sk -X GET "$WAZUH_API/manager/status?pretty=true" -H "Authorization: Bearer $TOKEN" 2>/dev/null | jq -r '.data.affected_items[0] // "No data"' 2>/dev/null || echo "Could not retrieve"
echo ""
echo "--- Agent Summary ---"
curl -sk -X GET "$WAZUH_API/agents/summary/status?pretty=true" -H "Authorization: Bearer $TOKEN" 2>/dev/null | jq '.data' 2>/dev/null || echo "Could not retrieve"
echo ""
echo "--- High-Severity Alerts (level > 10) ---"
HIGH_COUNT=$(curl -sk -u "$INDEXER_USER:$INDEXER_PASS" "$INDEXER_URL/wazuh-alerts-*/_count" \
  -H "Content-Type: application/json" \
  -d '{"query":{"range":{"rule.level":{"gt":10}}}}' 2>/dev/null | jq '.count // 0' 2>/dev/null)
echo "${HIGH_COUNT:-0} high-severity alerts found"
echo ""
echo "--- Indexer Health ---"
curl -sk -u "$INDEXER_USER:$INDEXER_PASS" "$INDEXER_URL/_cluster/health?pretty" 2>/dev/null | jq '{status, number_of_nodes, active_shards}' 2>/dev/null || echo "Could not reach indexer"
echo ""
echo "--- Alert Count (Last 24h) ---"
ALERT_COUNT=$(curl -sk -u "$INDEXER_USER:$INDEXER_PASS" "$INDEXER_URL/wazuh-alerts-*/_count" \
  -H "Content-Type: application/json" \
  -d '{"query":{"range":{"timestamp":{"gte":"now-24h"}}}}' 2>/dev/null | jq '.count // 0' 2>/dev/null)
echo "Total alerts in last 24 hours: ${ALERT_COUNT:-0}"
echo ""
echo "============================================"
echo "  Health Check Complete"
echo "============================================"
SCRIPT_EOF
chmod +x ~/.openclaw/skills/wazuh-monitor/wazuh-health-check.sh
```

### 4.4 Create the Report Generator Script

> **⚠️ FIX:** This script queries alerts from OpenSearch (port 9200) instead of the non-existent `/alerts` API endpoint. It also uses robust error handling with fallbacks.

```bash
cat > ~/.openclaw/skills/wazuh-monitor/wazuh-report.sh << 'REPORT_EOF'
#!/bin/bash
set -uo pipefail
WAZUH_API="https://localhost:55000"
WAZUH_USER="${WAZUH_API_USER:-wazuh-wui}"
WAZUH_PASS="${WAZUH_API_PASSWORD:-MyS3cr37P450r.*-}"
INDEXER_URL="https://localhost:9200"
INDEXER_USER="${INDEXER_USER:-admin}"
INDEXER_PASS="${INDEXER_PASS:-SecretPassword}"
REPORT_DIR="${REPORT_DIR:-$HOME/.openclaw/reports}"
mkdir -p "$REPORT_DIR"
TIMESTAMP=$(date -u '+%Y-%m-%d_%H-%M-%S')
REPORT_FILE="$REPORT_DIR/wazuh-report-$TIMESTAMP.json"

# Authenticate
TOKEN=$(curl -su "$WAZUH_USER:$WAZUH_PASS" -k -X POST "$WAZUH_API/security/user/authenticate?raw=true" 2>/dev/null)
if [ -z "$TOKEN" ] || echo "$TOKEN" | grep -q "Unauthorized\|Invalid"; then
  echo '{"error": "Authentication failed"}'
  exit 1
fi
AUTH_HEADER="Authorization: Bearer $TOKEN"
echo "Collecting Wazuh security data..."

# Collect API data (with fallbacks for missing endpoints)
MANAGER_INFO=$(curl -sk -X GET "$WAZUH_API/manager/info?pretty=true" -H "$AUTH_HEADER" 2>/dev/null || echo '{}')
MANAGER_STATUS=$(curl -sk -X GET "$WAZUH_API/manager/status?pretty=true" -H "$AUTH_HEADER" 2>/dev/null || echo '{}')
AGENT_SUMMARY=$(curl -sk -X GET "$WAZUH_API/agents/summary/status?pretty=true" -H "$AUTH_HEADER" 2>/dev/null || echo '{}')
AGENTS_LIST=$(curl -sk -X GET "$WAZUH_API/agents?pretty=true&limit=100" -H "$AUTH_HEADER" 2>/dev/null || echo '{}')
MANAGER_LOGS=$(curl -sk -X GET "$WAZUH_API/manager/logs?pretty=true&limit=30" -H "$AUTH_HEADER" 2>/dev/null || echo '{}')

# Get alerts from OpenSearch (NOT the Wazuh API which lacks /alerts in v4.14.3)
RECENT_ALERTS=$(curl -sk -u "$INDEXER_USER:$INDEXER_PASS" "$INDEXER_URL/wazuh-alerts-*/_search" \
  -H "Content-Type: application/json" \
  -d '{"size":20,"sort":[{"timestamp":{"order":"desc"}}]}' 2>/dev/null || echo '{}')

HIGH_ALERTS=$(curl -sk -u "$INDEXER_USER:$INDEXER_PASS" "$INDEXER_URL/wazuh-alerts-*/_search" \
  -H "Content-Type: application/json" \
  -d '{"size":20,"sort":[{"timestamp":{"order":"desc"}}],"query":{"range":{"rule.level":{"gt":10}}}}' 2>/dev/null || echo '{}')

MITRE_BREAKDOWN=$(curl -sk -u "$INDEXER_USER:$INDEXER_PASS" "$INDEXER_URL/wazuh-alerts-*/_search" \
  -H "Content-Type: application/json" \
  -d '{"size":0,"query":{"range":{"timestamp":{"gte":"now-24h"}}},"aggs":{"mitre_tactics":{"terms":{"field":"rule.mitre.tactic.keyword","size":20}},"mitre_techniques":{"terms":{"field":"rule.mitre.technique.keyword","size":20}},"severity_levels":{"terms":{"field":"rule.level","size":16}},"top_rules":{"terms":{"field":"rule.description.keyword","size":10}}}}' 2>/dev/null || echo '{}')

ALERT_24H=$(curl -sk -u "$INDEXER_USER:$INDEXER_PASS" "$INDEXER_URL/wazuh-alerts-*/_count" \
  -H "Content-Type: application/json" \
  -d '{"query":{"range":{"timestamp":{"gte":"now-24h"}}}}' 2>/dev/null | jq '.count // 0' 2>/dev/null || echo "0")

ALERT_1H=$(curl -sk -u "$INDEXER_USER:$INDEXER_PASS" "$INDEXER_URL/wazuh-alerts-*/_count" \
  -H "Content-Type: application/json" \
  -d '{"query":{"range":{"timestamp":{"gte":"now-1h"}}}}' 2>/dev/null | jq '.count // 0' 2>/dev/null || echo "0")

INDEXER_HEALTH=$(curl -sk -u "$INDEXER_USER:$INDEXER_PASS" "$INDEXER_URL/_cluster/health" 2>/dev/null || echo '{}')

# Build report (simple JSON for robustness)
cat > "$REPORT_FILE" << JSONEOF
{
  "report_timestamp": "$TIMESTAMP",
  "alert_counts": {
    "last_1_hour": ${ALERT_1H:-0},
    "last_24_hours": ${ALERT_24H:-0}
  }
}
JSONEOF

echo ""
echo "========== WAZUH SECURITY REPORT =========="
echo "Timestamp: $TIMESTAMP"
echo ""
echo "--- Alert Counts ---"
echo "Last 1 hour:  ${ALERT_1H:-0}"
echo "Last 24 hours: ${ALERT_24H:-0}"
echo ""
echo "--- Manager Info ---"
echo "$MANAGER_INFO" | jq -r '.data.affected_items[0] // .data // "N/A"' 2>/dev/null
echo ""
echo "--- Manager Status ---"
echo "$MANAGER_STATUS" | jq -r '.data.affected_items[0] // "N/A"' 2>/dev/null
echo ""
echo "--- Agent Summary ---"
echo "$AGENT_SUMMARY" | jq '.data' 2>/dev/null
echo ""
echo "--- Agents ---"
echo "$AGENTS_LIST" | jq -r '.data.affected_items[]? | "\(.id) | \(.name) | \(.ip) | \(.status) | \(.os.name // "N/A") \(.os.version // "")"' 2>/dev/null
echo ""
echo "--- High-Severity Alerts (level > 10) ---"
echo "$HIGH_ALERTS" | jq -r '.hits.hits[]?._source | "\(.timestamp) | Level \(.rule.level) | \(.rule.description) | \(.rule.mitre.technique // ["N/A"] | join(","))"' 2>/dev/null || echo "None"
echo ""
echo "--- Top MITRE Tactics (24h) ---"
echo "$MITRE_BREAKDOWN" | jq -r '.aggregations.mitre_tactics.buckets[]? | "\(.key): \(.doc_count)"' 2>/dev/null || echo "None"
echo ""
echo "--- Top MITRE Techniques (24h) ---"
echo "$MITRE_BREAKDOWN" | jq -r '.aggregations.mitre_techniques.buckets[]? | "\(.key): \(.doc_count)"' 2>/dev/null || echo "None"
echo ""
echo "--- Severity Distribution (24h) ---"
echo "$MITRE_BREAKDOWN" | jq -r '.aggregations.severity_levels.buckets[]? | "Level \(.key): \(.doc_count)"' 2>/dev/null || echo "None"
echo ""
echo "--- Top Rules (24h) ---"
echo "$MITRE_BREAKDOWN" | jq -r '.aggregations.top_rules.buckets[]? | "\(.key): \(.doc_count)"' 2>/dev/null || echo "None"
echo ""
echo "--- Indexer Health ---"
echo "$INDEXER_HEALTH" | jq '{status, number_of_nodes, active_shards}' 2>/dev/null
echo ""
echo "Report saved to: $REPORT_FILE"
echo "========== REPORT COMPLETE =========="
REPORT_EOF
chmod +x ~/.openclaw/skills/wazuh-monitor/wazuh-report.sh
```

### 4.5 Verify Skill Installation

```bash
ls -la ~/.openclaw/skills/wazuh-monitor/

# Expected:
# SKILL.md
# wazuh-health-check.sh
# wazuh-report.sh
```

---

## 5. Phase 4 — Connect OpenClaw to Wazuh

### 5.1 Enable Required Tools

> **⚠️ FIX:** `tools.exec.security` valid values are `deny`, `allowlist`, or `full`.  
> The value `none` does NOT exist and will error.  
> Use `full` to allow all exec commands (required for the bot to run curl/scripts).

```bash
openclaw config set tools.exec.host "gateway"
openclaw config set tools.exec.security "full"
openclaw config set tools.web.fetch.enabled true
openclaw config set browser.enabled true
```

### 5.2 Restart Gateway

```bash
openclaw daemon restart
```

### 5.3 Test the Health Check Script

```bash
bash ~/.openclaw/skills/wazuh-monitor/wazuh-health-check.sh
```

Expected output:

```
============================================
  Wazuh Security Health Check
  2026-03-01 02:16:21 UTC
============================================

--- Authenticating ---
Authentication: OK

--- Manager Status ---
{
  "wazuh-analysisd": "running",
  "wazuh-authd": "running",
  "wazuh-remoted": "running",
  "wazuh-syscheckd": "running",
  ...
}

--- Agent Summary ---
{
  "connection": {
    "active": 1,
    "disconnected": 0,
    "total": 1
  },
  ...
}

--- Indexer Health ---
{
  "status": "green",
  "number_of_nodes": 1,
  "active_shards": 26
}

--- Alert Count (Last 24h) ---
Total alerts in last 24 hours: 468

============================================
  Health Check Complete
============================================
```

### 5.4 Test the Report Script

```bash
bash ~/.openclaw/skills/wazuh-monitor/wazuh-report.sh
```

### 5.5 Test via Telegram

In the OpenClaw Telegram bot, send:

```
Check the status of our Wazuh deployment. Authenticate with the Wazuh API 
and show me the agent summary and manager health.
```

### 5.6 Common Monitoring Prompts

Use these in the Telegram bot or Control UI at any time:

| Prompt | Purpose |
|--------|---------|
| Check Wazuh status | Quick health check |
| Show me the last 20 alerts | Recent security alerts |
| Are there any critical alerts? | High-severity alert check |
| Which agents are disconnected? | Agent health check |
| Show MITRE ATT&CK activity | Threat mapping analysis |
| Generate a security report | Full security assessment |
| Run the Wazuh health check script at ~/.openclaw/skills/wazuh-monitor/wazuh-health-check.sh | Run health check via script |
| Run the Wazuh report generator at ~/.openclaw/skills/wazuh-monitor/wazuh-report.sh and give me a comprehensive security report | Full report via script |

---

## 6. Phase 5 — Slack Notifications

### 6.1 Create a Slack App & Webhook

1. Go to [https://api.slack.com/apps](https://api.slack.com/apps)
2. Click **"Create an App"** → **"From scratch"**
3. Name: `WazuhClaw`, Workspace: your workspace
4. In the left sidebar, click **"Incoming Webhooks"**
5. Toggle **"Activate Incoming Webhooks"** to **On**
6. Click **"Add New Webhook to Workspace"**
7. Select your alerts channel (e.g., `#all-wazuhclaw`)
8. Click **Allow**
9. Copy the **Webhook URL** (format: `https://hooks.slack.com/services/T.../B.../...`)

### 6.2 Test the Webhook

```bash
curl -s -X POST "https://hooks.slack.com/services/YOUR/WEBHOOK/URL" \
  -H "Content-Type: application/json" \
  -d '{"text":"🛡️ WazuhClaw connected! Slack notifications are working."}'
```

You should see the message in your Slack channel.

### 6.3 Create the Slack Alert Script

Replace `YOUR/WEBHOOK/URL` with your actual webhook URL in the command below:

```bash
cat > ~/.openclaw/skills/wazuh-monitor/wazuh-alert-check.sh << 'ALERTEOF'
#!/bin/bash
set -uo pipefail

SLACK_WEBHOOK="https://hooks.slack.com/services/YOUR/WEBHOOK/URL"
WAZUH_API="https://localhost:55000"
WAZUH_USER="wazuh-wui"
WAZUH_PASS='MyS3cr37P450r.*-'
INDEXER_URL="https://localhost:9200"
INDEXER_USER="admin"
INDEXER_PASS="SecretPassword"

send_slack() {
  local color="$1"
  local title="$2"
  local message="$3"
  curl -s -X POST "$SLACK_WEBHOOK" \
    -H "Content-Type: application/json" \
    -d "{\"attachments\":[{\"color\":\"$color\",\"title\":\"$title\",\"text\":\"$message\",\"footer\":\"Wazuh Security Monitor | $(date -u '+%Y-%m-%d %H:%M UTC')\"}]}" >/dev/null 2>&1
}

# Authenticate
TOKEN=$(curl -su "$WAZUH_USER:$WAZUH_PASS" -k -X POST "$WAZUH_API/security/user/authenticate?raw=true" 2>/dev/null)
if [ -z "$TOKEN" ] || echo "$TOKEN" | grep -q "Unauthorized\|Invalid"; then
  send_slack "danger" "🚨 AUTH FAILURE" "Failed to authenticate with Wazuh API!"
  exit 1
fi

# Check agents
AGENT_DATA=$(curl -sk -X GET "$WAZUH_API/agents/summary/status?pretty=true" -H "Authorization: Bearer $TOKEN" 2>/dev/null)
ACTIVE=$(echo "$AGENT_DATA" | jq '.data.connection.active // 0' 2>/dev/null)
DISCONNECTED=$(echo "$AGENT_DATA" | jq '.data.connection.disconnected // 0' 2>/dev/null)
TOTAL=$(echo "$AGENT_DATA" | jq '.data.connection.total // 0' 2>/dev/null)

# Check high-severity alerts (last 15 min)
HIGH_COUNT=$(curl -sk -u "$INDEXER_USER:$INDEXER_PASS" "$INDEXER_URL/wazuh-alerts-*/_count" \
  -H "Content-Type: application/json" \
  -d '{"query":{"bool":{"must":[{"range":{"timestamp":{"gte":"now-15m"}}},{"range":{"rule.level":{"gt":10}}}]}}}' 2>/dev/null | jq '.count // 0' 2>/dev/null)

# Alert counts
ALERTS_15M=$(curl -sk -u "$INDEXER_USER:$INDEXER_PASS" "$INDEXER_URL/wazuh-alerts-*/_count" \
  -H "Content-Type: application/json" \
  -d '{"query":{"range":{"timestamp":{"gte":"now-15m"}}}}' 2>/dev/null | jq '.count // 0' 2>/dev/null)

ALERTS_24H=$(curl -sk -u "$INDEXER_USER:$INDEXER_PASS" "$INDEXER_URL/wazuh-alerts-*/_count" \
  -H "Content-Type: application/json" \
  -d '{"query":{"range":{"timestamp":{"gte":"now-24h"}}}}' 2>/dev/null | jq '.count // 0' 2>/dev/null)

# Indexer health
INDEXER_STATUS=$(curl -sk -u "$INDEXER_USER:$INDEXER_PASS" "$INDEXER_URL/_cluster/health" 2>/dev/null | jq -r '.status // "unknown"' 2>/dev/null)

# Send alerts for critical issues
if [ "${DISCONNECTED:-0}" -gt 0 ]; then
  DISC_AGENTS=$(curl -sk -X GET "$WAZUH_API/agents?status=disconnected&pretty=true" -H "Authorization: Bearer $TOKEN" 2>/dev/null \
    | jq -r '.data.affected_items[]? | "• \(.name) (\(.ip))"' 2>/dev/null)
  send_slack "danger" "🚨 AGENTS DISCONNECTED" "$DISCONNECTED agent(s) disconnected!\n$DISC_AGENTS\nActive: $ACTIVE | Total: $TOTAL"
fi

if [ "${HIGH_COUNT:-0}" -gt 0 ]; then
  ALERT_DETAILS=$(curl -sk -u "$INDEXER_USER:$INDEXER_PASS" "$INDEXER_URL/wazuh-alerts-*/_search" \
    -H "Content-Type: application/json" \
    -d '{"size":5,"sort":[{"timestamp":{"order":"desc"}}],"query":{"bool":{"must":[{"range":{"timestamp":{"gte":"now-15m"}}},{"range":{"rule.level":{"gt":10}}}]}}}' 2>/dev/null \
    | jq -r '.hits.hits[]._source | "• Level \(.rule.level): \(.rule.description // "N/A")"' 2>/dev/null)
  send_slack "danger" "🚨 HIGH-SEVERITY ALERTS: $HIGH_COUNT" "$HIGH_COUNT critical alert(s) in last 15 min:\n$ALERT_DETAILS"
fi

if [ "${INDEXER_STATUS}" != "green" ] && [ "${INDEXER_STATUS}" != "yellow" ]; then
  send_slack "danger" "🚨 INDEXER UNHEALTHY" "Indexer status: $INDEXER_STATUS"
fi

# Always send status update
send_slack "good" "✅ Health Check" "Agents: $ACTIVE active / $TOTAL total | Disconnected: ${DISCONNECTED:-0}\nAlerts (15m): ${ALERTS_15M:-0} | Critical: ${HIGH_COUNT:-0} | 24h total: ${ALERTS_24H:-0}\nIndexer: $INDEXER_STATUS"

echo "[$(date -u)] Check complete. Active: $ACTIVE, Disconnected: ${DISCONNECTED:-0}, High: ${HIGH_COUNT:-0}, Alerts(15m): ${ALERTS_15M:-0}"
ALERTEOF
chmod +x ~/.openclaw/skills/wazuh-monitor/wazuh-alert-check.sh
```

### 6.4 Test the Alert Script

```bash
bash ~/.openclaw/skills/wazuh-monitor/wazuh-alert-check.sh
```

Check your Slack channel — you should see the health check message.

### 6.5 Wazuh Native Real-Time Slack Integration

This sends alerts directly from Wazuh Manager to Slack in **real-time** (level 7+, lowered from 10 in Phase 7), without waiting for any cron job.

> **⚠️ CRITICAL FIX:** The `<integration>` block MUST be placed **inside** `</ossec_config>`.  
> Placing it after `</ossec_config>` causes a config parse error and prevents the manager from starting.

**Correct method:**

```bash
docker exec -it single-node-wazuh.manager-1 bash -c "
# Remove the closing tag
sed -i '/<\/ossec_config>/d' /var/ossec/etc/ossec.conf

# Add integration block and re-add closing tag
cat >> /var/ossec/etc/ossec.conf << 'INNEREOF'

  <integration>
    <name>slack</name>
    <hook_url>https://hooks.slack.com/services/YOUR/WEBHOOK/URL</hook_url>
    <level>10</level>
    <alert_format>json</alert_format>
  </integration>

</ossec_config>
INNEREOF
"
```

> **Replace** `YOUR/WEBHOOK/URL` with your actual Slack webhook path.

Verify the config:

```bash
docker exec -it single-node-wazuh.manager-1 tail -10 /var/ossec/etc/ossec.conf
```

Expected output:

```xml
  <integration>
    <name>slack</name>
    <hook_url>https://hooks.slack.com/services/YOUR/WEBHOOK/URL</hook_url>
    <level>10</level>
    <alert_format>json</alert_format>
  </integration>

</ossec_config>
```

Restart the manager:

```bash
docker restart single-node-wazuh.manager-1
```

**Wait ~90 seconds** for the manager to fully initialize, then verify:

```bash
# API should respond
sleep 90
curl -sk "https://localhost:55000/?pretty=true" | head -5

# Check that wazuh-integratord is running (handles Slack integration)
TOKEN=$(curl -su 'wazuh-wui:MyS3cr37P450r.*-' -k -X POST \
  "https://localhost:55000/security/user/authenticate?raw=true" 2>/dev/null)
curl -sk -X GET "https://localhost:55000/manager/status?pretty=true" \
  -H "Authorization: Bearer $TOKEN" | jq '.data.affected_items[0]["wazuh-integratord"]'
# Expected: "running"
```

---

## 7. Phase 6 — Schedule Automated Monitoring

### 7.1 Set Up Cron Jobs

```bash
crontab -l 2>/dev/null > /tmp/mycron || true

# Every 15 minutes: Health check with Slack notification
echo "*/15 * * * * bash ~/.openclaw/skills/wazuh-monitor/wazuh-alert-check.sh >> ~/.openclaw/reports/alert-check.log 2>&1" >> /tmp/mycron

# Every 6 hours: Full security report
echo "0 */6 * * * bash ~/.openclaw/skills/wazuh-monitor/wazuh-report.sh >> ~/.openclaw/reports/report.log 2>&1" >> /tmp/mycron

# Daily at 08:00 UTC: Daily comprehensive report
echo "0 8 * * * bash ~/.openclaw/skills/wazuh-monitor/wazuh-report.sh >> ~/.openclaw/reports/daily-report.log 2>&1" >> /tmp/mycron

crontab /tmp/mycron
```

### 7.2 Verify Cron Jobs

```bash
crontab -l
```

Expected output:

```
*/15 * * * * bash ~/.openclaw/skills/wazuh-monitor/wazuh-alert-check.sh >> ~/.openclaw/reports/alert-check.log 2>&1
0 */6 * * * bash ~/.openclaw/skills/wazuh-monitor/wazuh-report.sh >> ~/.openclaw/reports/report.log 2>&1
0 8 * * * bash ~/.openclaw/skills/wazuh-monitor/wazuh-report.sh >> ~/.openclaw/reports/daily-report.log 2>&1
```

### 7.3 Monitoring Schedule Summary

| Schedule | Script | Notification |
|----------|--------|-------------|
| Every 15 min | `wazuh-alert-check.sh` | Slack: alert-only (fires only when issues detected) |
| Every 6 hours | `wazuh-report.sh` | Log file: full security report |
| Daily 08:00 UTC | `wazuh-report.sh` | Log file: daily comprehensive report |
| Real-time | Wazuh integratord | Slack: any alert level 7+ (lowered from 10) |

---

## 8. Phase 7 — Advanced Security Monitoring

> **Added in v3.0.** This phase transforms the basic monitoring setup into an enterprise-grade security monitoring platform with real-time file integrity monitoring, active response, compliance framework tracking, deep security auditing, and alert-only notifications.

### 8.1 Enhanced Wazuh Manager Configuration (ossec.conf)

These changes are applied inside the Wazuh Manager container at `/var/ossec/etc/ossec.conf`.

#### 8.1.1 Lower Alert Storage Threshold

Store all alert levels (not just level 3+) for comprehensive analysis:

```bash
docker exec -it single-node-wazuh.manager-1 bash -c "
sed -i 's|<log_alert_level>3</log_alert_level>|<log_alert_level>1</log_alert_level>|' /var/ossec/etc/ossec.conf
"
```

Resulting config:

```xml
<alerts>
  <log_alert_level>1</log_alert_level>
  <email_alert_level>12</email_alert_level>
</alerts>
```

#### 8.1.2 Enhanced File Integrity Monitoring (Syscheck)

Reduce scan frequency from 12 hours to 2 hours and add real-time monitoring on critical directories:

```bash
docker exec -it single-node-wazuh.manager-1 bash -c "
# Backup first
cp /var/ossec/etc/ossec.conf /var/ossec/etc/ossec.conf.bak

# Change syscheck frequency from 43200 (12h) to 7200 (2h)
sed -i '/<syscheck>/,/<\/syscheck>/ s|<frequency>43200</frequency>|<frequency>7200</frequency>|' /var/ossec/etc/ossec.conf
"
```

Add additional monitored directories (inside the `<syscheck>` block, before `</syscheck>`):

```xml
<!-- Critical system files - real-time -->
<directories realtime="yes" check_all="yes" report_changes="yes">/root/.ssh,/root/.bashrc,/root/.bash_profile</directories>
<directories realtime="yes" check_all="yes" report_changes="yes">/var/spool/cron,/etc/crontab,/etc/cron.d</directories>
<directories check_all="yes" report_changes="yes">/var/log/auth.log,/var/log/syslog,/var/log/secure</directories>
<directories realtime="yes" check_all="yes">/usr/local/bin,/usr/local/sbin</directories>
<directories check_all="yes">/opt</directories>
```

Final syscheck configuration:

```xml
<syscheck>
  <disabled>no</disabled>
  <frequency>7200</frequency>
  <scan_on_start>yes</scan_on_start>
  <alert_new_files>yes</alert_new_files>
  <auto_ignore frequency="10" timeframe="3600">no</auto_ignore>

  <!-- Core system dirs with real-time monitoring -->
  <directories realtime="yes" check_all="yes" report_changes="yes">/etc,/usr/bin,/usr/sbin</directories>
  <directories realtime="yes" check_all="yes" report_changes="yes">/bin,/sbin,/boot</directories>

  <!-- Ignore noisy files -->
  <ignore>/etc/mtab</ignore>
  <ignore>/etc/hosts.deny</ignore>
  <ignore>/etc/mail/statistics</ignore>
  <ignore>/etc/random-seed</ignore>
  <ignore>/etc/random.seed</ignore>
  <ignore>/etc/adjtime</ignore>
  <ignore>/etc/httpd/logs</ignore>
  <ignore>/etc/utmpx</ignore>
  <ignore>/etc/wtmpx</ignore>
  <ignore>/etc/cups/certs</ignore>
  <ignore>/etc/dumpdates</ignore>
  <ignore>/etc/svc/volatile</ignore>
  <ignore type="sregex">.log$|.swp$</ignore>
  <nodiff>/etc/ssl/private.key</nodiff>
  <skip_nfs>yes</skip_nfs>

  <!-- Critical system files - real-time -->
  <directories realtime="yes" check_all="yes" report_changes="yes">/root/.ssh,/root/.bashrc,/root/.bash_profile</directories>
  <directories realtime="yes" check_all="yes" report_changes="yes">/var/spool/cron,/etc/crontab,/etc/cron.d</directories>
  <directories check_all="yes" report_changes="yes">/var/log/auth.log,/var/log/syslog,/var/log/secure</directories>
  <directories realtime="yes" check_all="yes">/usr/local/bin,/usr/local/sbin</directories>
  <directories check_all="yes">/opt</directories>

  <skip_dev>yes</skip_dev>
  <skip_proc>yes</skip_proc>
  <skip_sys>yes</skip_sys>
  <process_priority>10</process_priority>
  <max_eps>100</max_eps>
  <synchronization>
    <enabled>yes</enabled>
    <interval>5m</interval>
    <max_interval>1h</max_interval>
    <max_eps>10</max_eps>
  </synchronization>
</syscheck>
```

#### 8.1.3 Enhanced Rootcheck

Increase rootcheck frequency from 12h to 2h:

```bash
docker exec -it single-node-wazuh.manager-1 bash -c "
sed -i '/<rootcheck>/,/<\/rootcheck>/ s|<frequency>43200</frequency>|<frequency>7200</frequency>|' /var/ossec/etc/ossec.conf
"
```

#### 8.1.4 Lower Slack Integration Level

Capture medium-severity threats (level 7+) instead of only critical (level 10+):

```bash
docker exec -it single-node-wazuh.manager-1 bash -c "
sed -i '/<integration>/,/<\/integration>/ s|<level>10</level>|<level>7</level>|' /var/ossec/etc/ossec.conf
"
```

#### 8.1.5 Enable Active Response

Add active response rules to auto-block attackers. Place these before `</ossec_config>`:

```bash
docker exec -it single-node-wazuh.manager-1 bash -c "
# Remove closing tag, add active response, re-add closing tag
sed -i '/<\/ossec_config>/d' /var/ossec/etc/ossec.conf
cat >> /var/ossec/etc/ossec.conf << 'AREOF'

  <!-- Active Response: Block IP after SSH brute force -->
  <active-response>
    <command>firewall-drop</command>
    <location>local</location>
    <rules_id>5763</rules_id>
    <timeout>600</timeout>
  </active-response>

  <!-- Active Response: Block IP after multiple auth failures -->
  <active-response>
    <command>firewall-drop</command>
    <location>local</location>
    <rules_id>5720</rules_id>
    <timeout>600</timeout>
  </active-response>

  <!-- Active Response: Disable account after brute force -->
  <active-response>
    <command>host-deny</command>
    <location>local</location>
    <rules_id>5763</rules_id>
    <timeout>1800</timeout>
  </active-response>

</ossec_config>
AREOF
"
```

Active response rules explained:

| Rule | Command | Timeout | Trigger |
|------|---------|---------|---------|
| 5763 | `firewall-drop` | 600s (10 min) | SSH brute force detected |
| 5720 | `firewall-drop` | 600s (10 min) | Multiple authentication failures |
| 5763 | `host-deny` | 1800s (30 min) | SSH brute force (adds to `/etc/hosts.deny`) |

> **⚠️ IMPORTANT:** The `<directories>` tag is ONLY valid inside `<syscheck>`. Do NOT place it inside `<rootcheck>` or `<sca>` — this causes config errors like:
> ```
> wazuh-syscheckd: WARNING: Invalid element in the configuration: 'directories'
> sca: ERROR: No such tag 'directories' at module 'sca'
> ```

#### 8.1.6 Restart the Manager

```bash
docker restart single-node-wazuh.manager-1

# Wait ~120 seconds for full initialization
sleep 120

# Verify no config errors
docker logs single-node-wazuh.manager-1 2>&1 | grep -i "error\|warning\|invalid" | grep -v "gpgv2\|No such file"
# Should return empty (no errors)

# Verify API is responding
TOKEN=$(curl -su 'wazuh-wui:MyS3cr37P450r.*-' -k -X POST \
  "https://localhost:55000/security/user/authenticate?raw=true" 2>/dev/null)
echo "Token length: ${#TOKEN}"
# Expected: 400+ characters
```

### 8.2 Centralized Agent Configuration (agent.conf)

Push enhanced monitoring settings to ALL agents in the default group. This file is deployed to agents automatically — no manual agent-side configuration needed.

```bash
docker exec -it single-node-wazuh.manager-1 bash -c 'cat > /var/ossec/etc/shared/default/agent.conf << '\''AGENTEOF'\''
<agent_config>

  <!-- Enhanced File Integrity Monitoring for agents -->
  <syscheck>
    <frequency>3600</frequency>
    <scan_on_start>yes</scan_on_start>
    <alert_new_files>yes</alert_new_files>
    <auto_ignore frequency="10" timeframe="3600">no</auto_ignore>

    <!-- Core system dirs with real-time monitoring -->
    <directories realtime="yes" check_all="yes" report_changes="yes">/etc</directories>
    <directories realtime="yes" check_all="yes" report_changes="yes">/usr/bin,/usr/sbin,/bin,/sbin</directories>
    <directories realtime="yes" check_all="yes" report_changes="yes">/boot</directories>

    <!-- SSH & authentication - real-time -->
    <directories realtime="yes" check_all="yes" report_changes="yes">/root/.ssh</directories>
    <directories realtime="yes" check_all="yes" report_changes="yes">/home</directories>

    <!-- Cron & scheduled tasks - real-time -->
    <directories realtime="yes" check_all="yes" report_changes="yes">/etc/crontab,/etc/cron.d,/etc/cron.daily,/etc/cron.hourly</directories>
    <directories realtime="yes" check_all="yes" report_changes="yes">/var/spool/cron</directories>

    <!-- Web directories (if applicable) -->
    <directories check_all="yes" report_changes="yes">/var/www</directories>

    <!-- System binaries -->
    <directories realtime="yes" check_all="yes">/usr/local/bin,/usr/local/sbin</directories>
    <directories check_all="yes">/opt</directories>

    <!-- Systemd services -->
    <directories realtime="yes" check_all="yes" report_changes="yes">/etc/systemd/system</directories>
    <directories check_all="yes">/lib/systemd/system</directories>

    <!-- PAM & sudo -->
    <directories realtime="yes" check_all="yes" report_changes="yes">/etc/pam.d</directories>
    <directories realtime="yes" check_all="yes" report_changes="yes">/etc/sudoers,/etc/sudoers.d</directories>

    <!-- Ignore noisy files -->
    <ignore>/etc/mtab</ignore>
    <ignore>/etc/hosts.deny</ignore>
    <ignore>/etc/mail/statistics</ignore>
    <ignore>/etc/random-seed</ignore>
    <ignore>/etc/adjtime</ignore>
    <ignore>/etc/cups/certs</ignore>
    <ignore type="sregex">.log$|.swp$|.tmp$</ignore>
    <ignore type="sregex">^/etc/resolv.conf$</ignore>

    <skip_nfs>yes</skip_nfs>
    <skip_dev>yes</skip_dev>
    <skip_proc>yes</skip_proc>
    <skip_sys>yes</skip_sys>

    <max_eps>200</max_eps>
    <synchronization>
      <enabled>yes</enabled>
      <interval>5m</interval>
      <max_eps>10</max_eps>
    </synchronization>
  </syscheck>

  <!-- Enhanced Rootcheck for malware detection -->
  <rootcheck>
    <frequency>7200</frequency>
    <check_files>yes</check_files>
    <check_trojans>yes</check_trojans>
    <check_dev>yes</check_dev>
    <check_sys>yes</check_sys>
    <check_pids>yes</check_pids>
    <check_ports>yes</check_ports>
    <check_if>yes</check_if>
    <skip_nfs>yes</skip_nfs>
  </rootcheck>

  <!-- SCA - run every 4 hours -->
  <sca>
    <enabled>yes</enabled>
    <scan_on_start>yes</scan_on_start>
    <interval>4h</interval>
    <skip_nfs>yes</skip_nfs>
  </sca>

  <!-- System inventory - collect every 30 min -->
  <wodle name="syscollector">
    <disabled>no</disabled>
    <interval>30m</interval>
    <scan_on_start>yes</scan_on_start>
    <hardware>yes</hardware>
    <os>yes</os>
    <network>yes</network>
    <packages>yes</packages>
    <ports all="yes">yes</ports>
    <processes>yes</processes>
    <hotfixes>yes</hotfixes>
  </wodle>

</agent_config>
AGENTEOF'
```

Verify the agent received the config:

```bash
TOKEN=$(curl -su 'wazuh-wui:MyS3cr37P450r.*-' -k -X POST \
  "https://localhost:55000/security/user/authenticate?raw=true" 2>/dev/null)
curl -sk -X GET "https://localhost:55000/agents?agents_list=001&pretty=true" \
  -H "Authorization: Bearer $TOKEN" | grep group_config_status
# Expected: "group_config_status": "synced"
```

### 8.3 Updated SKILL.md (Advanced Version)

The SKILL.md has been completely rewritten to cover all Wazuh modules. It's now 333 lines with 4 major modules:

- **Module 1: Endpoint Security** — SCA with compliance mappings, Malware/Rootcheck, FIM with real-time dirs
- **Module 2: Threat Intelligence** — Threat Hunting queries, Vulnerability Detection, MITRE ATT&CK
- **Module 3: Security Operations & Compliance** — IT Hygiene (syscollector), PCI DSS, GDPR, HIPAA, NIST 800-53, TSC/SOC 2
- **Module 4: Manager & System Health** — Full manager diagnostics, active response log, indexer health

Deploy with:

```bash
cat > ~/.openclaw/skills/wazuh-monitor/SKILL.md << 'SKILL_EOF'
---
name: wazuh-monitor
description: Advanced Wazuh SIEM/XDR security monitoring — endpoint security, threat intelligence, compliance frameworks (PCI DSS, GDPR, HIPAA, NIST 800-53, TSC), file integrity, vulnerability detection, malware detection, MITRE ATT&CK mapping, and IT hygiene assessment.
metadata: {"openclaw": {"always": true, "emoji": "🛡️", "requires": {"bins": ["curl", "jq"]}}}
---

# Wazuh Advanced Security Monitor

You are a security analyst with access to a Wazuh SIEM/XDR instance. You can detect any change, any vulnerability, any threat across all monitored endpoints. You have deep access to every Wazuh module.

## Connection Details

- Wazuh API URL: https://localhost:55000
- Dashboard URL: https://localhost
- Wazuh Indexer (OpenSearch): https://localhost:9200
- API Credentials: Username wazuh-wui, Password MyS3cr37P450r.*-
- Dashboard/Indexer Credentials: Username admin, Password SecretPassword
- All API calls require -k flag (self-signed certificates)

## Authentication

Every session requires a JWT token. Always authenticate first:

TOKEN=$(curl -su 'wazuh-wui:MyS3cr37P450r.*-' -k -X POST "https://localhost:55000/security/user/authenticate?raw=true")

Then include in requests: -H "Authorization: Bearer $TOKEN"

IMPORTANT: Always use single quotes around the password. Token expires after 900 seconds.

## Monitored Agents

- 000: wazuh.manager (127.0.0.1) - Amazon Linux 2023 - The Wazuh manager itself
- 001: test-server1 (172.31.17.75) - Ubuntu 24.04.3 LTS - Main monitored endpoint

Replace {agent_id} with 001 for test-server1 queries, or 000 for the manager.

---

## MODULE 1: ENDPOINT SECURITY

### 1A. Configuration Assessment (SCA)

SCA scans endpoints against security benchmarks (CIS, PCI DSS, HIPAA, NIST, etc.)

List SCA policies for an agent:
curl -sk "$WAZUH_API/sca/{agent_id}?pretty=true" -H "Authorization: Bearer $TOKEN"

Get FAILED checks only:
curl -sk "$WAZUH_API/sca/{agent_id}/checks/cis_ubuntu24-04?pretty=true&q=result=failed" -H "Authorization: Bearer $TOKEN"

SCA compliance mappings available: cis, pci_dss_v3.2.1, pci_dss_v4.0, hipaa, nist_sp_800-53, soc_2, cmmc_v2.0, iso_27001-2013, mitre_tactics, mitre_techniques, mitre_mitigations

### 1B. Malware Detection (Rootcheck)

Check rootcheck results:
curl -sk "$WAZUH_API/rootcheck/{agent_id}?pretty=true" -H "Authorization: Bearer $TOKEN"

Rootcheck alerts in OpenSearch:
curl -sk -u admin:SecretPassword "https://localhost:9200/wazuh-alerts-*/_search" -H "Content-Type: application/json" -d '{"size":20,"sort":[{"timestamp":{"order":"desc"}}],"query":{"bool":{"should":[{"match":{"rule.groups":"rootcheck"}},{"match":{"rule.groups":"rootkit"}}]}}}'

### 1C. File Integrity Monitoring (FIM / Syscheck)

Real-time monitoring is active on: /etc, /usr/bin, /usr/sbin, /bin, /sbin, /boot, /root/.ssh, /home, /etc/crontab, /etc/cron.d, /var/spool/cron, /etc/systemd/system, /etc/pam.d, /etc/sudoers.

Get FIM results:
curl -sk "$WAZUH_API/syscheck/{agent_id}?pretty=true" -H "Authorization: Bearer $TOKEN"

FIM alerts from OpenSearch:
curl -sk -u admin:SecretPassword "https://localhost:9200/wazuh-alerts-*/_search" -H "Content-Type: application/json" -d '{"size":20,"sort":[{"timestamp":{"order":"desc"}}],"query":{"match":{"rule.groups":"syscheck"}}}'

---

## MODULE 2: THREAT INTELLIGENCE

### 2A. Threat Hunting

Recent alerts:
curl -sk -u admin:SecretPassword "https://localhost:9200/wazuh-alerts-*/_search" -H "Content-Type: application/json" -d '{"size":20,"sort":[{"timestamp":{"order":"desc"}}]}'

Source IP analysis:
curl -sk -u admin:SecretPassword "https://localhost:9200/wazuh-alerts-*/_search" -H "Content-Type: application/json" -d '{"size":0,"query":{"range":{"timestamp":{"gte":"now-24h"}}},"aggs":{"src_ips":{"terms":{"field":"data.srcip","size":20}}}}'

### 2B. Vulnerability Detection

curl -sk "$WAZUH_API/vulnerability/{agent_id}?pretty=true" -H "Authorization: Bearer $TOKEN"

### 2C. MITRE ATT&CK

curl -sk -u admin:SecretPassword "https://localhost:9200/wazuh-alerts-*/_search" -H "Content-Type: application/json" -d '{"size":0,"query":{"range":{"timestamp":{"gte":"now-24h"}}},"aggs":{"mitre_tactics":{"terms":{"field":"rule.mitre.tactic.keyword","size":20}}}}'

---

## MODULE 3: SECURITY OPERATIONS & COMPLIANCE

### 3A. IT Hygiene (Syscollector)

Packages: curl -sk "$WAZUH_API/syscollector/{agent_id}/packages?pretty=true&limit=500" -H "Authorization: Bearer $TOKEN"
Ports: curl -sk "$WAZUH_API/syscollector/{agent_id}/ports?pretty=true" -H "Authorization: Bearer $TOKEN"
Processes: curl -sk "$WAZUH_API/syscollector/{agent_id}/processes?pretty=true" -H "Authorization: Bearer $TOKEN"

### 3B-F. Compliance Frameworks (PCI DSS, GDPR, HIPAA, NIST 800-53, TSC/SOC 2)

Use SCA endpoint with compliance key filtering and OpenSearch alert queries with rule.pci_dss, rule.gdpr, rule.hipaa, rule.nist_800_53, rule.tsc fields.

---

## MODULE 4: MANAGER & SYSTEM HEALTH

Manager status: curl -sk "$WAZUH_API/manager/status?pretty=true" -H "Authorization: Bearer $TOKEN"
Agent summary: curl -sk "$WAZUH_API/agents/summary/status?pretty=true" -H "Authorization: Bearer $TOKEN"
Indexer health: curl -sk -u admin:SecretPassword "https://localhost:9200/_cluster/health?pretty"

## HELPER SCRIPTS

Health check: bash ~/.openclaw/skills/wazuh-monitor/wazuh-health-check.sh
Full report: bash ~/.openclaw/skills/wazuh-monitor/wazuh-report.sh
Deep audit: bash ~/.openclaw/skills/wazuh-monitor/wazuh-deep-audit.sh

## ACTIVE RESPONSE STATUS

Auto-blocks IPs that trigger rules 5763 (SSH brute force) and 5720 (multiple auth failures).
Check: docker exec single-node-wazuh.manager-1 cat /var/ossec/logs/active-responses.log

## SECURITY REPORT STRUCTURE

Include: 1) Executive Summary, 2) Agent Health, 3) Alert Summary, 4) Threat Analysis, 5) MITRE ATT&CK, 6) FIM, 7) Vulnerabilities, 8) Malware/Rootkit, 9) SCA, 10) Compliance (PCI DSS, GDPR, HIPAA, NIST, TSC/SOC2), 11) IT Hygiene, 12) Active Response, 13) Recommendations

## COMPLIANCE QUICK REFERENCE

PCI DSS: Req 1 (network), 2 (config), 5 (malware), 6 (vuln mgmt), 7 (access), 8 (auth), 10 (logging), 11 (testing)
HIPAA: 164.308(a)(1) (activity review), 164.312(a)(1) (access), 164.312(b) (audit), 164.312(c)(1) (integrity)
NIST 800-53: AC (access), AU (audit), CM (config), IA (auth), SC (comms), SI (integrity)
SKILL_EOF
```

> **Note:** The full SKILL.md deployed to the server is 333 lines. The above is a condensed version for documentation. The actual deployed file contains complete API query examples for every endpoint and module.

### 8.4 Updated Alert Check Script (Alert-Only Mode)

The alert script has been completely rewritten. Key changes:

- **Alert-only mode:** No more "all clear" Slack messages every 15 minutes. Only notifies when issues are detected.
- **7 detection categories:** Agent health, high-severity alerts, FIM changes, brute force, rootcheck/malware, indexer health, vulnerability alerts
- **Summary logging:** Always logs a one-line summary to the log file for audit trail

```bash
cat > ~/.openclaw/skills/wazuh-monitor/wazuh-alert-check.sh << 'ALERTEOF'
#!/bin/bash
set -uo pipefail

SLACK_WEBHOOK="https://hooks.slack.com/services/YOUR/WEBHOOK/URL"
WAZUH_API="https://localhost:55000"
WAZUH_USER="wazuh-wui"
WAZUH_PASS='MyS3cr37P450r.*-'
INDEXER_URL="https://localhost:9200"
INDEXER_USER="admin"
INDEXER_PASS="SecretPassword"
ISSUES_FOUND=0

send_slack() {
  local color="$1"
  local title="$2"
  local message="$3"
  curl -s -X POST "$SLACK_WEBHOOK" \
    -H "Content-Type: application/json" \
    -d "{\"attachments\":[{\"color\":\"$color\",\"title\":\"$title\",\"text\":\"$message\",\"footer\":\"WazuhClaw Monitor | $(date -u '+%Y-%m-%d %H:%M UTC')\"}]}" >/dev/null 2>&1
}

# Authenticate
TOKEN=$(curl -su "$WAZUH_USER:$WAZUH_PASS" -k -X POST "$WAZUH_API/security/user/authenticate?raw=true" 2>/dev/null)
if [ -z "$TOKEN" ] || echo "$TOKEN" | grep -q "Unauthorized\|Invalid"; then
  send_slack "danger" "🚨 AUTH FAILURE" "Cannot authenticate with Wazuh API."
  echo "[$(date -u)] AUTH FAILURE"
  exit 1
fi

# ── 1. Agent Health ──
AGENT_DATA=$(curl -sk -X GET "$WAZUH_API/agents/summary/status?pretty=true" -H "Authorization: Bearer $TOKEN" 2>/dev/null)
ACTIVE=$(echo "$AGENT_DATA" | jq '.data.connection.active // 0' 2>/dev/null)
DISCONNECTED=$(echo "$AGENT_DATA" | jq '.data.connection.disconnected // 0' 2>/dev/null)
TOTAL=$(echo "$AGENT_DATA" | jq '.data.connection.total // 0' 2>/dev/null)

if [ "${DISCONNECTED:-0}" -gt 0 ]; then
  DISC_AGENTS=$(curl -sk -X GET "$WAZUH_API/agents?status=disconnected&pretty=true" -H "Authorization: Bearer $TOKEN" 2>/dev/null \
    | jq -r '.data.affected_items[]? | "- \(.name) (\(.ip)) last seen: \(.lastKeepAlive // "unknown")"' 2>/dev/null)
  send_slack "danger" "🚨 AGENTS DISCONNECTED ($DISCONNECTED/$TOTAL)" "Disconnected agents:\n$DISC_AGENTS"
  ISSUES_FOUND=$((ISSUES_FOUND+1))
fi

# ── 2. High-Severity Alerts (last 15 min) ──
HIGH_COUNT=$(curl -sk -u "$INDEXER_USER:$INDEXER_PASS" "$INDEXER_URL/wazuh-alerts-*/_count" \
  -H "Content-Type: application/json" \
  -d '{"query":{"bool":{"must":[{"range":{"timestamp":{"gte":"now-15m"}}},{"range":{"rule.level":{"gt":10}}}]}}}' 2>/dev/null | jq '.count // 0' 2>/dev/null)

if [ "${HIGH_COUNT:-0}" -gt 0 ]; then
  ALERT_DETAILS=$(curl -sk -u "$INDEXER_USER:$INDEXER_PASS" "$INDEXER_URL/wazuh-alerts-*/_search" \
    -H "Content-Type: application/json" \
    -d '{"size":5,"sort":[{"timestamp":{"order":"desc"}}],"query":{"bool":{"must":[{"range":{"timestamp":{"gte":"now-15m"}}},{"range":{"rule.level":{"gt":10}}}]}}}' 2>/dev/null \
    | jq -r '.hits.hits[]._source | "- Level \(.rule.level): \(.rule.description // "N/A") [\(.agent.name // "manager")]"' 2>/dev/null)
  send_slack "danger" "🚨 HIGH-SEVERITY ALERTS: $HIGH_COUNT (last 15m)" "$ALERT_DETAILS"
  ISSUES_FOUND=$((ISSUES_FOUND+1))
fi

# ── 3. File Integrity Changes (last 15 min) ──
FIM_COUNT=$(curl -sk -u "$INDEXER_USER:$INDEXER_PASS" "$INDEXER_URL/wazuh-alerts-*/_count" \
  -H "Content-Type: application/json" \
  -d '{"query":{"bool":{"must":[{"match":{"rule.groups":"syscheck"}},{"range":{"timestamp":{"gte":"now-15m"}}}]}}}' 2>/dev/null | jq '.count // 0' 2>/dev/null)

if [ "${FIM_COUNT:-0}" -gt 0 ]; then
  FIM_DETAILS=$(curl -sk -u "$INDEXER_USER:$INDEXER_PASS" "$INDEXER_URL/wazuh-alerts-*/_search" \
    -H "Content-Type: application/json" \
    -d '{"size":5,"sort":[{"timestamp":{"order":"desc"}}],"query":{"bool":{"must":[{"match":{"rule.groups":"syscheck"}},{"range":{"timestamp":{"gte":"now-15m"}}}]}}}' 2>/dev/null \
    | jq -r '.hits.hits[]._source | "- \(.rule.description // "File change") | \(.syscheck.path // "unknown") [\(.agent.name // "manager")]"' 2>/dev/null)
  send_slack "warning" "📁 FILE INTEGRITY CHANGES: $FIM_COUNT (last 15m)" "File modifications detected:\n$FIM_DETAILS"
  ISSUES_FOUND=$((ISSUES_FOUND+1))
fi

# ── 4. Brute Force / Auth Failures (last 15 min, threshold > 5) ──
AUTH_FAIL_COUNT=$(curl -sk -u "$INDEXER_USER:$INDEXER_PASS" "$INDEXER_URL/wazuh-alerts-*/_count" \
  -H "Content-Type: application/json" \
  -d '{"query":{"bool":{"must":[{"match":{"rule.groups":"authentication_failed"}},{"range":{"timestamp":{"gte":"now-15m"}}}]}}}' 2>/dev/null | jq '.count // 0' 2>/dev/null)

if [ "${AUTH_FAIL_COUNT:-0}" -gt 5 ]; then
  AUTH_DETAILS=$(curl -sk -u "$INDEXER_USER:$INDEXER_PASS" "$INDEXER_URL/wazuh-alerts-*/_search" \
    -H "Content-Type: application/json" \
    -d '{"size":0,"query":{"bool":{"must":[{"match":{"rule.groups":"authentication_failed"}},{"range":{"timestamp":{"gte":"now-15m"}}}]}},"aggs":{"src_ips":{"terms":{"field":"data.srcip","size":10}}}}' 2>/dev/null \
    | jq -r '.aggregations.src_ips.buckets[]? | "- \(.key): \(.doc_count) attempts"' 2>/dev/null)
  send_slack "warning" "🔐 BRUTE FORCE: $AUTH_FAIL_COUNT auth failures (15m)" "Top attacking IPs:\n$AUTH_DETAILS"
  ISSUES_FOUND=$((ISSUES_FOUND+1))
fi

# ── 5. Rootcheck / Malware Detection (last 15 min) ──
ROOTCHECK_COUNT=$(curl -sk -u "$INDEXER_USER:$INDEXER_PASS" "$INDEXER_URL/wazuh-alerts-*/_count" \
  -H "Content-Type: application/json" \
  -d '{"query":{"bool":{"must":[{"bool":{"should":[{"match":{"rule.groups":"rootcheck"}},{"match":{"rule.groups":"rootkit"}}]}},{"range":{"timestamp":{"gte":"now-15m"}}}]}}}' 2>/dev/null | jq '.count // 0' 2>/dev/null)

if [ "${ROOTCHECK_COUNT:-0}" -gt 0 ]; then
  send_slack "danger" "🦠 MALWARE/ROOTKIT DETECTION: $ROOTCHECK_COUNT" "Rootcheck/rootkit alerts detected in last 15 min"
  ISSUES_FOUND=$((ISSUES_FOUND+1))
fi

# ── 6. Indexer Health ──
INDEXER_STATUS=$(curl -sk -u "$INDEXER_USER:$INDEXER_PASS" "$INDEXER_URL/_cluster/health" 2>/dev/null | jq -r '.status // "unknown"' 2>/dev/null)

if [ "${INDEXER_STATUS}" != "green" ] && [ "${INDEXER_STATUS}" != "yellow" ]; then
  send_slack "danger" "🚨 INDEXER UNHEALTHY" "Cluster status: $INDEXER_STATUS"
  ISSUES_FOUND=$((ISSUES_FOUND+1))
fi

# ── 7. Vulnerability Alerts (last 15 min) ──
VULN_ALERTS=$(curl -sk -u "$INDEXER_USER:$INDEXER_PASS" "$INDEXER_URL/wazuh-alerts-*/_count" \
  -H "Content-Type: application/json" \
  -d '{"query":{"bool":{"must":[{"match":{"rule.groups":"vulnerability-detector"}},{"range":{"timestamp":{"gte":"now-15m"}}}]}}}' 2>/dev/null | jq '.count // 0' 2>/dev/null)

if [ "${VULN_ALERTS:-0}" -gt 0 ]; then
  send_slack "warning" "🔓 NEW VULNERABILITIES: $VULN_ALERTS" "Vulnerability alerts in last 15 min"
  ISSUES_FOUND=$((ISSUES_FOUND+1))
fi

# ── Summary (logged only — NO Slack unless issues found) ──
ALERTS_15M=$(curl -sk -u "$INDEXER_USER:$INDEXER_PASS" "$INDEXER_URL/wazuh-alerts-*/_count" \
  -H "Content-Type: application/json" \
  -d '{"query":{"range":{"timestamp":{"gte":"now-15m"}}}}' 2>/dev/null | jq '.count // 0' 2>/dev/null)

echo "[$(date -u)] Check complete. Issues: $ISSUES_FOUND | Active: $ACTIVE | Disconnected: ${DISCONNECTED:-0} | High: ${HIGH_COUNT:-0} | FIM: ${FIM_COUNT:-0} | AuthFail: ${AUTH_FAIL_COUNT:-0} | Rootcheck: ${ROOTCHECK_COUNT:-0} | Vuln: ${VULN_ALERTS:-0} | Alerts(15m): ${ALERTS_15M:-0} | Indexer: $INDEXER_STATUS"
ALERTEOF
chmod +x ~/.openclaw/skills/wazuh-monitor/wazuh-alert-check.sh
```

> **Key difference from v2.0:** The old script had `send_slack "good" "✅ Health Check" "..."` at the bottom that fired every 15 minutes regardless. That line has been removed. The script now only sends Slack messages when `ISSUES_FOUND > 0`.

### 8.5 Updated Report Script (with MITRE, FIM, SCA)

The report script now includes MITRE ATT&CK tactics/techniques, file integrity events, SCA compliance summary, and top triggering rules:

```bash
cat > ~/.openclaw/skills/wazuh-monitor/wazuh-report.sh << 'REPORT_EOF'
#!/bin/bash
set -uo pipefail

WAZUH_API="https://localhost:55000"
WAZUH_USER="wazuh-wui"
WAZUH_PASS='MyS3cr37P450r.*-'
INDEXER_URL="https://localhost:9200"
INDEXER_USER="admin"
INDEXER_PASS="SecretPassword"
REPORT_DIR="$HOME/.openclaw/reports"
mkdir -p "$REPORT_DIR"
TIMESTAMP=$(date -u '+%Y-%m-%d_%H-%M-%S')
REPORT_FILE="$REPORT_DIR/wazuh-report-$TIMESTAMP.txt"

# Authenticate
TOKEN=$(curl -su "$WAZUH_USER:$WAZUH_PASS" -k -X POST "$WAZUH_API/security/user/authenticate?raw=true" 2>/dev/null)
if [ -z "$TOKEN" ] || echo "$TOKEN" | grep -q "Unauthorized\|Invalid"; then
  echo "ERROR: Authentication failed"; exit 1
fi
AUTH="Authorization: Bearer $TOKEN"

{
echo "========== WAZUH SECURITY REPORT =========="
echo "Timestamp: $(date -u '+%Y-%m-%d %H:%M:%S UTC')"
echo ""

# Alert Counts (1h, 6h, 24h)
for PERIOD in "1h" "6h" "24h"; do
  COUNT=$(curl -sk -u "$INDEXER_USER:$INDEXER_PASS" "$INDEXER_URL/wazuh-alerts-*/_count" \
    -H "Content-Type: application/json" \
    -d "{\"query\":{\"range\":{\"timestamp\":{\"gte\":\"now-$PERIOD\"}}}}" 2>/dev/null | jq '.count // 0' 2>/dev/null)
  echo "Alerts ($PERIOD): ${COUNT:-0}"
done

echo ""
echo "--- Manager Info ---"
curl -sk "$WAZUH_API/manager/info?pretty=true" -H "$AUTH" 2>/dev/null \
  | jq -r '.data.affected_items[0] | "Version: \(.version // "?") | Type: \(.type // "?")"' 2>/dev/null

echo ""
echo "--- Manager Status ---"
curl -sk "$WAZUH_API/manager/status?pretty=true" -H "$AUTH" 2>/dev/null \
  | jq -r '.data.affected_items[0] // "N/A"' 2>/dev/null

echo ""
echo "--- Agent Summary ---"
curl -sk "$WAZUH_API/agents/summary/status?pretty=true" -H "$AUTH" 2>/dev/null | jq '.data' 2>/dev/null

echo ""
echo "--- Agents ---"
curl -sk "$WAZUH_API/agents?pretty=true" -H "$AUTH" 2>/dev/null \
  | jq -r '.data.affected_items[] | "\(.id) | \(.name) | \(.ip) | \(.status) | \(.os.name // "N/A") \(.os.version // "")"' 2>/dev/null

echo ""
echo "--- Severity Distribution (24h) ---"
curl -sk -u "$INDEXER_USER:$INDEXER_PASS" "$INDEXER_URL/wazuh-alerts-*/_search" \
  -H "Content-Type: application/json" \
  -d '{"size":0,"query":{"range":{"timestamp":{"gte":"now-24h"}}},"aggs":{"severity":{"terms":{"field":"rule.level","size":16,"order":{"_key":"desc"}}}}}' 2>/dev/null \
  | jq -r '.aggregations.severity.buckets[] | "Level \(.key): \(.doc_count)"' 2>/dev/null

echo ""
echo "--- Top Rules (24h) ---"
curl -sk -u "$INDEXER_USER:$INDEXER_PASS" "$INDEXER_URL/wazuh-alerts-*/_search" \
  -H "Content-Type: application/json" \
  -d '{"size":0,"query":{"range":{"timestamp":{"gte":"now-24h"}}},"aggs":{"rules":{"terms":{"field":"rule.description.keyword","size":15}}}}' 2>/dev/null \
  | jq -r '.aggregations.rules.buckets[] | "\(.doc_count)x \(.key)"' 2>/dev/null

echo ""
echo "--- High-Severity Alerts (level > 10, 24h) ---"
curl -sk -u "$INDEXER_USER:$INDEXER_PASS" "$INDEXER_URL/wazuh-alerts-*/_search" \
  -H "Content-Type: application/json" \
  -d '{"size":10,"sort":[{"timestamp":{"order":"desc"}}],"query":{"bool":{"must":[{"range":{"rule.level":{"gt":10}}},{"range":{"timestamp":{"gte":"now-24h"}}}]}}}' 2>/dev/null \
  | jq -r '.hits.hits[]._source | "\(.timestamp) | Level \(.rule.level) | \(.rule.description) [\(.agent.name)]"' 2>/dev/null || echo "None"

echo ""
echo "--- MITRE ATT&CK (24h) ---"
echo "Tactics:"
curl -sk -u "$INDEXER_USER:$INDEXER_PASS" "$INDEXER_URL/wazuh-alerts-*/_search" \
  -H "Content-Type: application/json" \
  -d '{"size":0,"query":{"range":{"timestamp":{"gte":"now-24h"}}},"aggs":{"tactics":{"terms":{"field":"rule.mitre.tactic.keyword","size":20}}}}' 2>/dev/null \
  | jq -r '.aggregations.tactics.buckets[]? | "  \(.key): \(.doc_count)"' 2>/dev/null || echo "  None"
echo "Techniques:"
curl -sk -u "$INDEXER_USER:$INDEXER_PASS" "$INDEXER_URL/wazuh-alerts-*/_search" \
  -H "Content-Type: application/json" \
  -d '{"size":0,"query":{"range":{"timestamp":{"gte":"now-24h"}}},"aggs":{"techniques":{"terms":{"field":"rule.mitre.technique.keyword","size":20}}}}' 2>/dev/null \
  | jq -r '.aggregations.techniques.buckets[]? | "  \(.key): \(.doc_count)"' 2>/dev/null || echo "  None"

echo ""
echo "--- File Integrity (24h) ---"
FIM_COUNT=$(curl -sk -u "$INDEXER_USER:$INDEXER_PASS" "$INDEXER_URL/wazuh-alerts-*/_count" \
  -H "Content-Type: application/json" \
  -d '{"query":{"bool":{"must":[{"match":{"rule.groups":"syscheck"}},{"range":{"timestamp":{"gte":"now-24h"}}}]}}}' 2>/dev/null | jq '.count // 0' 2>/dev/null)
echo "FIM events: ${FIM_COUNT:-0}"

echo ""
echo "--- SCA Compliance Summary (Agent 001) ---"
curl -sk "$WAZUH_API/sca/001?pretty=true" -H "$AUTH" 2>/dev/null \
  | jq -r '.data.affected_items[] | "  \(.name): Score \(.score)% (Pass:\(.pass) Fail:\(.fail))"' 2>/dev/null

echo ""
echo "--- Indexer Health ---"
curl -sk -u "$INDEXER_USER:$INDEXER_PASS" "$INDEXER_URL/_cluster/health" 2>/dev/null \
  | jq '{status, number_of_nodes, active_shards}' 2>/dev/null

echo ""
echo "Report saved to: $REPORT_FILE"
echo "========== REPORT COMPLETE =========="
} 2>&1 | tee "$REPORT_FILE"
REPORT_EOF
chmod +x ~/.openclaw/skills/wazuh-monitor/wazuh-report.sh
```

### 8.6 Deep Security Audit Script (NEW)

A comprehensive 10-section deep security audit. Run on-demand or via Telegram ("Run the deep security audit"):

```bash
cat > ~/.openclaw/skills/wazuh-monitor/wazuh-deep-audit.sh << 'AUDITEOF'
#!/bin/bash
set -uo pipefail

WAZUH_API="https://localhost:55000"
WAZUH_USER="wazuh-wui"
WAZUH_PASS='MyS3cr37P450r.*-'
INDEXER_URL="https://localhost:9200"
INDEXER_USER="admin"
INDEXER_PASS="SecretPassword"
REPORT_DIR="$HOME/.openclaw/reports"
mkdir -p "$REPORT_DIR"
TIMESTAMP=$(date -u '+%Y-%m-%d_%H-%M-%S')
REPORT_FILE="$REPORT_DIR/deep-audit-$TIMESTAMP.txt"

TOKEN=$(curl -su "$WAZUH_USER:$WAZUH_PASS" -k -X POST "$WAZUH_API/security/user/authenticate?raw=true" 2>/dev/null)
if [ -z "$TOKEN" ] || echo "$TOKEN" | grep -q "Unauthorized\|Invalid"; then
  echo "ERROR: Authentication failed"; exit 1
fi
AUTH="Authorization: Bearer $TOKEN"

{
echo "=================================================================="
echo "  WAZUH DEEP SECURITY AUDIT"
echo "  Generated: $(date -u '+%Y-%m-%d %H:%M:%S UTC')"
echo "=================================================================="

# Section 1: System Health
echo ""
echo "╔══════════════════════════════════════════╗"
echo "║  1. SYSTEM HEALTH                        ║"
echo "╚══════════════════════════════════════════╝"
# ... (Manager status, agent summary, agent details, indexer health)

# Section 2: Alert Overview (1h/6h/24h/7d + severity + groups)
# Section 3: File Integrity Monitoring
# Section 4: Malware & Rootkit Detection
# Section 5: Vulnerability Assessment
# Section 6: Threat Hunting (SSH analysis, source IPs, MITRE)
# Section 7: Configuration Assessment (SCA top 10 failures + remediation)
# Section 8: Compliance Status (PCI DSS, HIPAA, NIST, SOC2, ISO27001, CMMC)
# Section 9: IT Hygiene (open ports, packages, OS, network interfaces)
# Section 10: Active Response Log

echo ""
echo "=================================================================="
echo "  DEEP AUDIT COMPLETE"
echo "  Report saved to: $REPORT_FILE"
echo "=================================================================="
} 2>&1 | tee "$REPORT_FILE"
AUDITEOF
chmod +x ~/.openclaw/skills/wazuh-monitor/wazuh-deep-audit.sh
```

> **Note:** The full script is 280 lines. The above shows the structure. The actual deployed script contains complete curl commands for all 10 sections. It queries every Wazuh module and produces a comprehensive audit report saved to `~/.openclaw/reports/deep-audit-<timestamp>.txt`.

**Deep Audit Sections:**

| # | Section | What It Checks |
|---|---------|---------------|
| 1 | System Health | Manager daemon status, agent list with sync status, indexer cluster health |
| 2 | Alert Overview | Alert counts (1h/6h/24h/7d), severity distribution, top rules, alert groups |
| 3 | File Integrity | FIM event count, last scan times per agent, recent file changes |
| 4 | Malware & Rootkit | Rootcheck scan status, rootcheck findings on each agent |
| 5 | Vulnerability Assessment | CVEs per agent, vulnerability alerts from OpenSearch |
| 6 | Threat Hunting | SSH attack analysis, top attacking IPs, targeted usernames, MITRE ATT&CK mapping |
| 7 | SCA (Config Assessment) | Policy scores, top 10 failed checks with remediation steps |
| 8 | Compliance Status | Failed checks mapped to PCI DSS, HIPAA, NIST 800-53, SOC 2, ISO 27001, CMMC |
| 9 | IT Hygiene | Open ports, installed package count, OS info, network interfaces |
| 10 | Active Response | Recent firewall-drop and host-deny actions |

### 8.7 Restart OpenClaw Gateway

After deploying all new scripts and the updated SKILL.md, restart OpenClaw to reload:

```bash
systemctl --user restart openclaw-gateway
sleep 5
systemctl --user status openclaw-gateway
# Expected: active (running)
```

### 8.8 Verify Everything Works

```bash
# 1. Health check
bash ~/.openclaw/skills/wazuh-monitor/wazuh-health-check.sh

# 2. Alert check (should NOT send Slack if no issues)
bash ~/.openclaw/skills/wazuh-monitor/wazuh-alert-check.sh
# Expected output: [timestamp] Check complete. Issues: 0 | Active: 1 | ...

# 3. Deep audit (full 10-section report)
bash ~/.openclaw/skills/wazuh-monitor/wazuh-deep-audit.sh

# 4. Report
bash ~/.openclaw/skills/wazuh-monitor/wazuh-report.sh

# 5. Agent config sync
TOKEN=$(curl -su 'wazuh-wui:MyS3cr37P450r.*-' -k -X POST \
  "https://localhost:55000/security/user/authenticate?raw=true" 2>/dev/null)
curl -sk -X GET "https://localhost:55000/agents?agents_list=001&pretty=true" \
  -H "Authorization: Bearer $TOKEN" | grep group_config_status
# Expected: "group_config_status": "synced"
```

### 8.9 Test via Telegram

Send these to your Telegram bot to verify the new SKILL.md is being used:

| Prompt | Tests |
|--------|-------|
| "Run the deep security audit" | Deep audit script |
| "Show me HIPAA compliance status" | Module 3D compliance queries |
| "What are the top 10 SCA failures?" | Module 1A SCA queries |
| "Show me all file integrity changes" | Module 1C FIM queries |
| "What IPs are attacking us?" | Module 2A threat hunting |
| "Show PCI DSS compliance" | Module 3B PCI queries |
| "Give me a MITRE ATT&CK breakdown" | Module 2C MITRE queries |

### 8.10 Configuration Summary

#### What Changed in Manager (ossec.conf)

| Setting | Before (v2.0) | After (v3.0) | Why |
|---------|--------------|-------------|-----|
| `log_alert_level` | 3 | **1** | Store all alerts for deep analysis |
| Syscheck `frequency` | 43200 (12h) | **7200 (2h)** | Faster file change detection |
| Syscheck directories | `/etc`, `/usr/bin`, `/usr/sbin`, `/bin`, `/sbin`, `/boot` | + `/root/.ssh`, cron dirs, `/var/log/auth.log`, `/usr/local/bin`, `/opt` | Cover SSH, cron, auth, custom binaries |
| Syscheck `realtime` | No | **Yes** on critical dirs | Instant detection of file changes |
| Syscheck `report_changes` | No | **Yes** | See exact file content diffs |
| Rootcheck `frequency` | 43200 (12h) | **7200 (2h)** | More frequent malware scans |
| Slack `level` | 10 | **7** | Catch medium-severity threats |
| Active Response | None | **3 rules** (firewall-drop + host-deny) | Auto-block attackers |

#### What Changed in Agent Config (agent.conf)

| Setting | Before | After |
|---------|--------|-------|
| Syscheck frequency | Default (12h) | **3600 (1h)** |
| Syscheck directories | Default | **16 directory groups** with real-time |
| Rootcheck frequency | Default | **7200 (2h)** with all checks |
| SCA interval | Default | **4h** with scan_on_start |
| Syscollector interval | Default (1h) | **30m** with hotfixes |

---

## 9. Verification & Testing

### 9.1 Full Stack Verification Checklist

```bash
# ── Wazuh Verification ──

# 1. All Docker containers running
docker compose -f ~/wazuh-docker/single-node/docker-compose.yml ps
# Expected: 3 containers all "Up"

# 2. Wazuh Dashboard accessible
curl -sk -o /dev/null -w "%{http_code}" https://localhost
# Expected: 200 or 302

# 3. Wazuh API responsive
TOKEN=$(curl -su 'wazuh-wui:MyS3cr37P450r.*-' -k -X POST \
  "https://localhost:55000/security/user/authenticate?raw=true" 2>/dev/null)
curl -sk -X GET "https://localhost:55000/?pretty=true" -H "Authorization: Bearer $TOKEN" | jq '.error'
# Expected: 0

# 4. OpenSearch/Indexer healthy
curl -sk -u admin:SecretPassword "https://localhost:9200/_cluster/health" | jq '.status'
# Expected: "green" or "yellow"

# 5. Agents connected
curl -sk -X GET "https://localhost:55000/agents/summary/status?pretty=true" \
  -H "Authorization: Bearer $TOKEN" | jq '.data.connection'
# Expected: active > 0

# ── OpenClaw Verification ──

# 6. OpenClaw gateway running
openclaw gateway status

# 7. Skill files present (5 files in v3.0)
ls -la ~/.openclaw/skills/wazuh-monitor/
# Expected: SKILL.md, wazuh-health-check.sh, wazuh-report.sh, wazuh-alert-check.sh, wazuh-deep-audit.sh

# 8. Health check script works
bash ~/.openclaw/skills/wazuh-monitor/wazuh-health-check.sh

# 9. Report script works (includes MITRE/FIM/SCA sections)
bash ~/.openclaw/skills/wazuh-monitor/wazuh-report.sh

# 10. Alert-only mode works (Slack fires ONLY when issues found)
bash ~/.openclaw/skills/wazuh-monitor/wazuh-alert-check.sh
# Expected: "Issues: 0" = no Slack sent; "Issues: N" (N>0) = Slack sent

# 11. Deep audit script works (10-section comprehensive audit)
bash ~/.openclaw/skills/wazuh-monitor/wazuh-deep-audit.sh

# 12. Cron jobs configured
crontab -l

# 13. Wazuh integratord running (native Slack integration, level 7+)
curl -sk -X GET "https://localhost:55000/manager/status?pretty=true" \
  -H "Authorization: Bearer $TOKEN" | jq '.data.affected_items[0]["wazuh-integratord"]'
# Expected: "running"

# ── Phase 7 Specific Verifications ──

# 14. Active response rules configured
docker compose -f ~/wazuh-docker/single-node/docker-compose.yml \
  exec -T wazuh.manager grep -c 'active-response' /var/ossec/etc/ossec.conf
# Expected: 3 (three active response blocks)

# 15. FIM real-time monitoring enabled
docker compose -f ~/wazuh-docker/single-node/docker-compose.yml \
  exec -T wazuh.manager grep 'realtime' /var/ossec/etc/ossec.conf
# Expected: lines with realtime="yes"

# 16. Agent config deployed (centralized push)
docker compose -f ~/wazuh-docker/single-node/docker-compose.yml \
  exec -T wazuh.manager cat /var/ossec/etc/shared/default/agent.conf | head -5
# Expected: valid XML with <agent_config>

# 17. Agent synced
curl -sk -X GET "https://localhost:55000/agents?pretty=true" \
  -H "Authorization: Bearer $TOKEN" | jq '.data.affected_items[] | {id: .id, name: .name, config_sum: .config_sum, group_config_status: .group_config_status}'
# Expected: group_config_status = "synced"

# 18. Slack level lowered to 7
docker compose -f ~/wazuh-docker/single-node/docker-compose.yml \
  exec -T wazuh.manager grep -A2 'slack' /var/ossec/etc/ossec.conf | grep level
# Expected: <level>7</level>
```

### 9.2 Test Telegram Bot

Send these to your Telegram bot:

1. `Hello, are you running?`
2. `Check the status of our Wazuh deployment.`
3. `Run the Wazuh health check script at ~/.openclaw/skills/wazuh-monitor/wazuh-health-check.sh and analyze the results.`

### 9.3 Test Slack Notifications

```bash
# Manual test
bash ~/.openclaw/skills/wazuh-monitor/wazuh-alert-check.sh
# Check your Slack channel for the health check message
```

---

## 10. Troubleshooting

### 10.1 Wazuh Issues

| Problem | Solution |
|---------|----------|
| Indexer won't start | Check `vm.max_map_count`: `sysctl vm.max_map_count` (must be ≥ 262144) |
| Dashboard shows "not ready" | Wait 60–90 seconds after `docker compose up`. Indexer needs time. |
| API returns "Invalid credentials" | Use `wazuh-wui` user (NOT `wazuh`). Check password in `docker-compose.yml`: `grep API_PASSWORD docker-compose.yml` |
| API returns "Invalid token" | Token expired (15 min TTL). Re-authenticate. Use single quotes around password. |
| API returns 404 on `/alerts` | **This endpoint doesn't exist in v4.14.3.** Query alerts from OpenSearch on port 9200 instead. |
| API returns connection refused | Check Docker containers: `docker compose ps` |
| No data in dashboard | Enroll at least one Wazuh agent to generate data |
| Certificate errors | Regenerate certs: `docker compose -f generate-indexer-certs.yml run --rm generator` |
| Config error: "Invalid element 'integration'" | The `<integration>` block is outside `</ossec_config>`. Fix the XML structure (see Phase 5). |
| Manager API empty response after restart | Wait ~120 seconds for full initialization (longer with advanced config) |
| `curl` password with special chars fails | Always use **single quotes** around passwords containing `.*` or other shell metacharacters |
| Config error: "Invalid element 'directories'" in rootcheck/sca | `<directories>` tags are ONLY valid inside `<syscheck>`. Remove them from `<rootcheck>` and `<sca>` sections. |
| "No such tag 'directories' at module 'sca'" | Same fix — `<directories>` only belongs in `<syscheck>`, not `<sca>` |
| Active response not blocking IPs | Verify `firewall-drop` and `host-deny` commands exist in ossec.conf. Check `<active-response>` is inside `</ossec_config>`. Verify `wazuh-execd` is running. |
| Agent config not syncing | Check `group_config_status` via API. Restart the agent: `systemctl restart wazuh-agent` on the endpoint. |
| FIM not detecting changes | Verify `realtime="yes"` is set on monitored dirs. Check syscheck frequency isn't too long. Run `docker exec single-node-wazuh.manager-1 grep -A5 syscheck /var/ossec/etc/ossec.conf` |

### 10.2 OpenClaw Issues

| Problem | Solution |
|---------|----------|
| `openclaw: command not found` | Re-run install script or add to PATH |
| Gateway won't start | Check Node.js version: `node --version` (must be 22+). Also run `openclaw config set gateway.mode local` |
| "gateway.mode is unset" | Run `openclaw config set gateway.mode local` |
| Skill not loading | Check SKILL.md frontmatter syntax. Restart session / `openclaw daemon restart` |
| Exec tool denied / keeps asking approval | Set `openclaw config set tools.exec.security "full"` and restart |
| `tools.exec.security "none"` error | Valid values are `deny`, `allowlist`, `full`. Use `full`. |
| `tools.exec.allowlist` unrecognized | This key doesn't exist. Use `tools.exec.security "full"` instead. |
| `channels.telegram.token` unrecognized | Correct key is **`channels.telegram.botToken`** |
| Telegram bot says "access not configured" | Run `openclaw pairing approve telegram <PAIRING_CODE>` from the server |
| Memory search warning | Disable: `openclaw config set agents.defaults.memorySearch.enabled false` or set an API key |
| Onboarding skips model/channel setup | Run `openclaw configure --section model` and `openclaw configure --section telegram` manually |

### 10.3 Slack Integration Issues

| Problem | Solution |
|---------|----------|
| Webhook returns error | Verify URL at Slack API → Incoming Webhooks. Test with simple curl POST. |
| No Slack messages from cron | Check cron log: `cat ~/.openclaw/reports/alert-check.log` |
| Wazuh native Slack alerts not firing | Verify `wazuh-integratord` is "running" in manager status. Check manager logs: `docker compose logs wazuh.manager \| tail -30` |
| Integration config crash | Ensure `<integration>` is INSIDE `</ossec_config>`, not after it |

### 10.4 Script Issues

| Problem | Solution |
|---------|----------|
| Health check shows "No data" / "null" | Wrong API credentials. Verify with: `curl -su 'wazuh-wui:MyS3cr37P450r.*-' -k -X POST "https://localhost:55000/security/user/authenticate?raw=true"` |
| Report script exits silently | Likely a `jq` parse error from invalid JSON. Check each curl command manually. |
| Heredoc gets cut off when pasting | Paste each `cat > file << 'EOF'` block separately. Don't paste multiple blocks at once. |
| Scripts won't execute | Run `chmod +x ~/.openclaw/skills/wazuh-monitor/*.sh` |

---

## 11. Architecture Overview

### 11.1 What OpenClaw Actually Is

OpenClaw is a personal AI assistant (by Peter Steinberger), not a purpose-built security monitoring tool. It's an open-source AI agent that:

- Runs on your machine (macOS/Linux/Windows)
- Has persistent memory across sessions
- Can execute shell commands (the `exec` tool)
- Can browse the web (the `browser` tool)
- Can schedule recurring tasks (the `cron` tool)
- Connects to chat apps (WhatsApp, Telegram, Discord, Slack, iMessage)
- Supports custom skills (SKILL.md) that teach the AI how to use tools

### 11.2 How the Integration Works

```
┌────────────────────────────────────────────────────────┐
│                    SINGLE SERVER                        │
│                                                         │
│  ┌─────────────────────┐    ┌───────────────────────┐  │
│  │   OpenClaw Gateway   │    │  Wazuh Docker Stack    │  │
│  │   (Port 18789)       │    │                        │  │
│  │                      │    │  ┌──────────────────┐  │  │
│  │  ┌────────────────┐  │    │  │  Wazuh Manager   │  │  │
│  │  │  AI Agent       │  │    │  │  (1514/1515)     │  │  │
│  │  │  (Claude/GPT)   │──┼────┼─>│  REST API: 55000 │  │  │
│  │  └────────────────┘  │    │  └──────────────────┘  │  │
│  │         │             │    │  ┌──────────────────┐  │  │
│  │  ┌──────┴─────────┐  │    │  │  Wazuh Indexer   │  │  │
│  │  │  Skills         │  │    │  │  (OpenSearch)    │  │  │
│  │  │  - wazuh-monitor│──┼────┼─>│  Port: 9200      │  │  │
│  │  └────────────────┘  │    │  └──────────────────┘  │  │
│  │         │             │    │  ┌──────────────────┐  │  │
│  │  ┌──────┴─────────┐  │    │  │  Wazuh Dashboard │  │  │
│  │  │  Cron Jobs      │  │    │  │  Port: 443       │  │  │
│  │  │  (scheduled     │  │    │  └──────────────────┘  │  │
│  │  │   monitoring)   │  │    │                        │  │
│  │  └────────────────┘  │    └───────────────────────┘  │
│  │         │             │              ▲                 │
│  │  ┌──────┴─────────┐  │              │                 │
│  │  │  Reports        │  │    ┌────────┴──────────┐     │
│  │  │  ~/.openclaw/   │  │    │  Wazuh Agents      │     │
│  │  │    reports/     │  │    │  (remote endpoints) │     │
│  │  └────────────────┘  │    └───────────────────┘     │
│  └─────────────────────┘                                │
│         │                                                │
│  ┌──────┴──────────────────────────────────────────┐    │
│  │  Notification Channels                           │    │
│  │  - Control UI (localhost:18789)                   │    │
│  │  - Telegram Bot                                   │    │
│  │  - Slack (#all-wazuhclaw)                         │    │
│  └──────────────────────────────────────────────────┘    │
└────────────────────────────────────────────────────────┘
```

### 11.3 Data Flow

1. **Wazuh Agents** report to **Wazuh Manager** (port 1514)
2. **Wazuh Manager** processes alerts and stores them in **Wazuh Indexer** (OpenSearch, port 9200)
3. **Wazuh Manager** auto-blocks attackers via **Active Response** (firewall-drop, host-deny)
4. **OpenClaw** queries the **Wazuh API** (port 55000) for agent/manager info, SCA, FIM, vulnerabilities, syscollector
5. **OpenClaw** queries **OpenSearch** (port 9200) for alerts, MITRE ATT&CK, compliance mappings
6. **Cron scripts** run alert checks every 15 min (Slack only on issues), reports every 6h, daily at 8am
7. **Wazuh integratord** sends real-time level 7+ alerts directly to **Slack**
8. **Centralized agent.conf** pushes enhanced FIM/rootcheck/SCA/syscollector config to all agents
9. **Users** interact via **Telegram bot**, **Control UI**, or **Slack**

---

## 12. Appendices

### Appendix A: Useful Commands Reference

#### Wazuh Docker Management

```bash
cd ~/wazuh-docker/single-node/

# Start
docker compose up -d

# Stop
docker compose down

# Restart
docker compose restart

# View logs
docker compose logs -f wazuh.manager
docker compose logs -f wazuh.indexer
docker compose logs -f wazuh.dashboard

# Check resource usage
docker stats
```

#### OpenClaw Management

```bash
# Gateway status
openclaw gateway status

# Start/stop gateway
openclaw gateway start
openclaw gateway stop

# Restart daemon
openclaw daemon restart

# Open dashboard
openclaw dashboard

# View config
openclaw config get

# Health check
openclaw doctor

# List skills (in Control UI)
# Type: /skills
```

#### Quick Security Check

```bash
bash ~/.openclaw/skills/wazuh-monitor/wazuh-health-check.sh
bash ~/.openclaw/skills/wazuh-monitor/wazuh-report.sh
bash ~/.openclaw/skills/wazuh-monitor/wazuh-alert-check.sh
bash ~/.openclaw/skills/wazuh-monitor/wazuh-deep-audit.sh   # Full 10-section audit
```

### Appendix B: File Locations Summary

| Item | Location |
|------|----------|
| Wazuh Docker files | `~/wazuh-docker/single-node/` |
| Wazuh Docker Compose | `~/wazuh-docker/single-node/docker-compose.yml` |
| Wazuh SSL certs | `~/wazuh-docker/single-node/config/wazuh_indexer_ssl_certs/` |
| Wazuh Manager config (inside container) | `/var/ossec/etc/ossec.conf` |
| OpenClaw home | `~/.openclaw/` |
| OpenClaw config | `~/.openclaw/openclaw.json` |
| OpenClaw skills | `~/.openclaw/skills/` |
| Wazuh monitor skill | `~/.openclaw/skills/wazuh-monitor/` |
| SKILL.md (v3.0, 333 lines) | `~/.openclaw/skills/wazuh-monitor/SKILL.md` |
| Health check script | `~/.openclaw/skills/wazuh-monitor/wazuh-health-check.sh` |
| Alert check script (alert-only) | `~/.openclaw/skills/wazuh-monitor/wazuh-alert-check.sh` |
| Report script (MITRE/FIM/SCA) | `~/.openclaw/skills/wazuh-monitor/wazuh-report.sh` |
| Deep audit script (10-section) | `~/.openclaw/skills/wazuh-monitor/wazuh-deep-audit.sh` |
| Wazuh reports output | `~/.openclaw/reports/` |
| Deep audit output | `~/.openclaw/reports/deep-audit-*.txt` |
| Cron logs | `~/.openclaw/reports/alert-check.log` |
| Centralized agent config (container) | `/var/ossec/etc/shared/default/agent.conf` |
| Manager ossec.conf (container) | `/var/ossec/etc/ossec.conf` |
| Systemd service | `~/.config/systemd/user/openclaw-gateway.service` |

### Appendix C: Credentials Reference

> ⚠️ **Change all default passwords for production use!**

| Service | Username | Password | Notes |
|---------|----------|----------|-------|
| Wazuh API | `wazuh-wui` | `MyS3cr37P450r.*-` | From `docker-compose.yml`. Use single quotes in shell. |
| Wazuh Dashboard | `admin` | `SecretPassword` | Change via [Wazuh docs](https://documentation.wazuh.com/current/deployment-options/docker/changing-default-password.html) |
| Wazuh Indexer (OpenSearch) | `admin` | `SecretPassword` | Same as Dashboard |
| OpenClaw Gateway | Token-based | See `openclaw config get gateway.auth.token` | Auto-generated during setup |

### Appendix D: Execution Checklist

- [ ] **Pre-flight:** Docker, Node.js 22+, Git, jq installed
- [ ] **Pre-flight:** `vm.max_map_count=262144` set
- [ ] **Phase 1:** Wazuh repo cloned (v4.14.3)
- [ ] **Phase 1:** SSL certs generated
- [ ] **Phase 1:** `docker compose up -d` successful
- [ ] **Phase 1:** Wazuh Dashboard accessible (`https://localhost`)
- [ ] **Phase 1:** Wazuh API responding (port 55000) — use `wazuh-wui` user
- [ ] **Phase 1:** At least one Wazuh agent enrolled
- [ ] **Phase 2:** OpenClaw installed via script
- [ ] **Phase 2:** Model/LLM provider configured
- [ ] **Phase 2:** Gateway mode set to `local`
- [ ] **Phase 2:** Gateway running, Control UI accessible
- [ ] **Phase 2:** Telegram bot created and paired
- [ ] **Phase 3:** wazuh-monitor skill created (SKILL.md with correct credentials)
- [ ] **Phase 3:** Health check script created & tested
- [ ] **Phase 3:** Report script created & tested
- [ ] **Phase 4:** `tools.exec.security` set to `full`
- [ ] **Phase 4:** Gateway restarted
- [ ] **Phase 4:** Test query to Wazuh via Telegram/Control UI succeeds
- [ ] **Phase 5:** Slack webhook created and tested
- [ ] **Phase 5:** Alert check script created & tested
- [ ] **Phase 5:** Wazuh native Slack integration added (inside `</ossec_config>`)
- [ ] **Phase 5:** `wazuh-integratord` status is "running"
- [ ] **Phase 6:** Cron jobs configured (15min, 6h, daily)
- [ ] **Phase 7:** Manager ossec.conf enhanced (log_alert_level 1, FIM realtime, rootcheck 2h)
- [ ] **Phase 7:** Active response rules added (firewall-drop + host-deny for SSH brute force)
- [ ] **Phase 7:** Slack integration level lowered (10 → 7)
- [ ] **Phase 7:** Centralized agent.conf deployed (FIM 1h, rootcheck 2h, SCA 4h, syscollector 30m)
- [ ] **Phase 7:** Agent(s) synced with new config (`group_config_status: synced`)
- [ ] **Phase 7:** SKILL.md v3.0 deployed (333 lines, 4 modules)
- [ ] **Phase 7:** Alert script rewritten (alert-only mode — Slack only on issues)
- [ ] **Phase 7:** Report script upgraded (MITRE ATT&CK, FIM, SCA sections)
- [ ] **Phase 7:** Deep audit script created (280 lines, 10 sections)
- [ ] **Phase 7:** OpenClaw gateway restarted
- [ ] **Phase 7:** All 4 scripts tested successfully
- [ ] **Post-deploy:** Default passwords changed for production
- [ ] **Post-deploy:** Slack channel receiving alert-only notifications (no noise)

### Appendix E: API Endpoint Reference

#### Wazuh API (Port 55000) — Working Endpoints

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/agents?pretty=true` | GET | List all registered agents |
| `/agents/summary/status?pretty=true` | GET | Agent status summary |
| `/agents/summary/os?pretty=true` | GET | OS distribution across agents |
| `/agents?status=active&pretty=true` | GET | List active agents only |
| `/agents?status=disconnected&pretty=true` | GET | List disconnected agents |
| `/agents/{agent_id}?pretty=true` | GET | Detailed info for specific agent |
| `/manager/status?pretty=true` | GET | Manager running status |
| `/manager/info?pretty=true` | GET | Manager version and info |
| `/manager/configuration?pretty=true` | GET | Current configuration |
| `/manager/stats?pretty=true` | GET | Performance statistics |
| `/manager/logs?pretty=true&limit=20` | GET | Recent manager logs |
| `/mitre?pretty=true` | GET | MITRE ATT&CK framework data |
| `/mitre/tactics?pretty=true` | GET | Tactics catalog |
| `/mitre/techniques?pretty=true` | GET | Techniques catalog |
| `/vulnerability/{agent_id}?pretty=true` | GET | Vulnerabilities for specific agent |
| `/rules?pretty=true&limit=10` | GET | Detection rules |
| `/rules?pretty=true&q=level>12` | GET | Critical detection rules |
| `/syscheck/{agent_id}?pretty=true` | GET | FIM results for agent |
| `/sca/{agent_id}?pretty=true` | GET | SCA policies for agent |
| `/syscollector/{agent_id}/os?pretty=true` | GET | OS info for agent |
| `/syscollector/{agent_id}/packages?pretty=true` | GET | Installed packages |
| `/syscollector/{agent_id}/processes?pretty=true` | GET | Running processes |
| `/syscollector/{agent_id}/ports?pretty=true` | GET | Open ports |

#### ❌ Non-Existent Endpoints (Do NOT Use)

| Endpoint | Status | Alternative |
|----------|--------|-------------|
| `/alerts` | **404 Not Found** | Query `wazuh-alerts-*` index on OpenSearch (port 9200) |

---

> **Document maintained by:** Abdur Razzaq  
> **Repository:** [Wazuh-Security-Platform](https://github.com/AbdurRazzaq2004/Wazuh-Security-Platform)  
> **OpenClaw Docs:** [https://docs.openclaw.ai](https://docs.openclaw.ai)  
> **Wazuh Docs:** [https://documentation.wazuh.com](https://documentation.wazuh.com)

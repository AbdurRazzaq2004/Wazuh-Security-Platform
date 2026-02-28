# OpenClaw + Wazuh — Deployment & Integration Guide

> **Document Type:** Step-by-Step Deployment Guide  
> **Version:** 1.0  
> **Date:** 2026-02-28  
> **Status:** Execution-Ready  

---

## Table of Contents

1. [Prerequisites](#1-prerequisites)
2. [Phase 1 — Deploy Wazuh Single-Node (Docker)](#2-phase-1--deploy-wazuh-single-node-docker)
3. [Phase 2 — Install OpenClaw](#3-phase-2--install-openclaw)
4. [Phase 3 — Create the Wazuh Monitoring Skill](#4-phase-3--create-the-wazuh-monitoring-skill)
5. [Phase 4 — Connect OpenClaw to Wazuh](#5-phase-4--connect-openclaw-to-wazuh)
6. [Phase 5 — Schedule Automated Monitoring](#6-phase-5--schedule-automated-monitoring)
7. [Verification & Testing](#7-verification--testing)
8. [Troubleshooting](#8-troubleshooting)
9. [Architecture Reality Check](#9-architecture-reality-check)

---

## 1. Prerequisites

### 1.1 System Requirements

| Resource | Minimum | Recommended |
|---|---|---|
| **CPU** | 4 cores | 6+ cores |
| **RAM** | 8 GB | 12+ GB |
| **Disk** | 50 GB | 100+ GB SSD |
| **OS** | Ubuntu 22.04 LTS / macOS 14+ | Ubuntu 22.04 LTS |
| **Node.js** | 22+ | Latest LTS |
| **Docker** | Latest stable | Latest stable |
| **Docker Compose** | v2+ (included with Docker) | Latest |
| **Git** | Any | Latest |

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
```

If Node.js 22+ is not installed:

```bash
# macOS (Homebrew)
brew install node@22

# Ubuntu/Debian
curl -fsSL https://deb.nodesource.com/setup_22.x | sudo -E bash -
sudo apt-get install -y nodejs
```

### 1.3 Network Ports Summary

After full deployment, these ports will be in use:

| Port | Service | Protocol | Purpose |
|---|---|---|---|
| **443** | Wazuh Dashboard | HTTPS | Web UI for security monitoring |
| **1514** | Wazuh Manager | TCP | Agent communication |
| **1515** | Wazuh Manager | TCP | Agent enrollment |
| **9200** | Wazuh Indexer | HTTPS | OpenSearch API (internal) |
| **55000** | Wazuh API | HTTPS | REST API (OpenClaw connects here) |
| **18789** | OpenClaw Gateway | HTTP | Control UI & agent communication |

---

## 2. Phase 1 — Deploy Wazuh Single-Node (Docker)

### 2.1 Prepare the Docker Host

```bash
# Set kernel parameter for Wazuh Indexer (OpenSearch)
sudo sysctl -w vm.max_map_count=262144

# Make it persistent across reboots
echo "vm.max_map_count=262144" | sudo tee -a /etc/sysctl.conf

# Add your user to docker group (Linux only, skip on macOS)
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

### 2.4 Deploy Wazuh Stack

```bash
# Start all containers in detached mode
docker compose up -d
```

This spins up three containers:
- **wazuh.manager** — Detection engine, rule processing, agent management
- **wazuh.indexer** — OpenSearch for data storage and indexing
- **wazuh.dashboard** — Web interface (Kibana fork)

### 2.5 Verify Wazuh Deployment

```bash
# Check all containers are running
docker compose ps

# Expected output: 3 containers (manager, indexer, dashboard) all "Up"
```

Wait ~60 seconds for the indexer to fully initialize, then:

```bash
# Test the Wazuh Dashboard
curl -sk https://localhost | head -20

# Test the Wazuh API
TOKEN=$(curl -su wazuh:wazuh -k -X POST "https://localhost:55000/security/user/authenticate?raw=true")
curl -k -X GET "https://localhost:55000/?pretty=true" -H "Authorization: Bearer $TOKEN"
```

Expected API response:

```json
{
  "data": {
    "title": "Wazuh API REST",
    "api_version": "4.14.3",
    "hostname": "wazuh.manager",
    "timestamp": "2026-02-28T..."
  },
  "error": 0
}
```

### 2.6 Access Wazuh Dashboard

Open in your browser: **https://localhost** (or `https://<SERVER_IP>`)

| Field | Value |
|---|---|
| **Username** | `admin` |
| **Password** | `SecretPassword` |

> ⚠️ **Change the default password immediately** for production use.  
> Follow: [Wazuh — Changing Default Passwords](https://documentation.wazuh.com/current/deployment-options/docker/changing-default-password.html)

### 2.7 Test Wazuh API Endpoints (Reference)

These are the key API endpoints OpenClaw will use:

```bash
# Get authentication token
TOKEN=$(curl -su wazuh:wazuh -k -X POST \
  "https://localhost:55000/security/user/authenticate?raw=true")

# Agent summary
curl -k -X GET "https://localhost:55000/agents/summary/status?pretty=true" \
  -H "Authorization: Bearer $TOKEN"

# Recent alerts (last 10)
curl -k -X GET "https://localhost:55000/alerts?pretty=true&limit=10&sort=-timestamp" \
  -H "Authorization: Bearer $TOKEN"

# MITRE ATT&CK data
curl -k -X GET "https://localhost:55000/mitre?pretty=true" \
  -H "Authorization: Bearer $TOKEN"

# Manager status
curl -k -X GET "https://localhost:55000/manager/status?pretty=true" \
  -H "Authorization: Bearer $TOKEN"

# Vulnerability summary
curl -k -X GET "https://localhost:55000/vulnerability?pretty=true" \
  -H "Authorization: Bearer $TOKEN"

# Rules (detection rules)
curl -k -X GET "https://localhost:55000/rules?pretty=true&limit=5" \
  -H "Authorization: Bearer $TOKEN"

# System information
curl -k -X GET "https://localhost:55000/syscollector?pretty=true" \
  -H "Authorization: Bearer $TOKEN"
```

---

## 3. Phase 2 — Install OpenClaw

### 3.1 Install OpenClaw via Official Script

```bash
# Install OpenClaw (macOS / Linux)
curl -fsSL https://openclaw.ai/install.sh | bash
```

The install script downloads the OpenClaw gateway and CLI tools.

### 3.2 Run Onboarding Wizard

```bash
# Configure auth, gateway settings, and optional channels
openclaw onboard --install-daemon
```

The wizard walks you through:
1. **Auth configuration** — API key for your LLM provider (Anthropic/OpenAI)
2. **Gateway settings** — Port, logging
3. **Channel setup** — Optional (WhatsApp, Telegram, Discord, etc.)

### 3.3 Verify OpenClaw Gateway

```bash
# Check gateway status
openclaw gateway status

# Open the Control UI in your browser
openclaw dashboard
```

The Control UI is available at: **http://127.0.0.1:18789/**

### 3.4 Test OpenClaw

In the Control UI (browser), send a test message:

```
Hello, are you running?
```

If you get a response, OpenClaw is operational.

---

## 4. Phase 3 — Create the Wazuh Monitoring Skill

This is where we teach OpenClaw **how** to interact with Wazuh. We create a custom skill that gives the AI agent instructions and helper scripts.

### 4.1 Create the Skill Directory

```bash
# Create the Wazuh monitoring skill in OpenClaw's managed skills folder
mkdir -p ~/.openclaw/skills/wazuh-monitor
```

### 4.2 Create the Skill Definition (SKILL.md)

Create `~/.openclaw/skills/wazuh-monitor/SKILL.md`:

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

- **Wazuh API URL:** `https://localhost:55000`
- **Dashboard URL:** `https://localhost`
- **Wazuh Indexer (OpenSearch):** `https://localhost:9200`
- **API Credentials:** Username `wazuh`, Password `wazuh` (change in production)
- **Dashboard Credentials:** Username `admin`, Password `SecretPassword` (change in production)
- **All API calls require `-k` flag** (self-signed certificates)

## Authentication

Every Wazuh API session requires a JWT token. Always authenticate first:

```bash
TOKEN=$(curl -su wazuh:wazuh -k -X POST "https://localhost:55000/security/user/authenticate?raw=true")
```

Then include the token in subsequent requests:

```bash
curl -k -X GET "https://localhost:55000/<ENDPOINT>?pretty=true" \
  -H "Authorization: Bearer $TOKEN"
```

**Tokens expire after 900 seconds (15 minutes).** Re-authenticate if you get 401 errors.

## Available API Endpoints

Use these endpoints to gather security data:

### Agent Management
| Endpoint | Method | Description |
|---|---|---|
| `/agents?pretty=true` | GET | List all registered agents |
| `/agents/summary/status?pretty=true` | GET | Agent status summary (active/disconnected/never_connected) |
| `/agents/summary/os?pretty=true` | GET | OS distribution across agents |
| `/agents?status=active&pretty=true` | GET | List active agents only |
| `/agents?status=disconnected&pretty=true` | GET | List disconnected agents |
| `/agents/{agent_id}?pretty=true` | GET | Detailed info for specific agent |

### Alerts & IDS
| Endpoint | Method | Description |
|---|---|---|
| `/alerts?pretty=true&limit=20&sort=-timestamp` | GET | Recent alerts sorted by time |
| `/alerts?pretty=true&q=rule.level>10` | GET | High-severity alerts (level > 10) |
| `/alerts?pretty=true&q=rule.groups=intrusion_detection` | GET | IDS-specific alerts |

### MITRE ATT&CK
| Endpoint | Method | Description |
|---|---|---|
| `/mitre?pretty=true` | GET | MITRE ATT&CK framework data |
| `/mitre/tactics?pretty=true` | GET | Tactics catalog |
| `/mitre/techniques?pretty=true` | GET | Techniques catalog |

### Vulnerability Detection
| Endpoint | Method | Description |
|---|---|---|
| `/vulnerability/{agent_id}?pretty=true` | GET | Vulnerabilities for specific agent |
| `/vulnerability/{agent_id}/summary?pretty=true` | GET | Vulnerability summary per agent |

### Manager & System Health
| Endpoint | Method | Description |
|---|---|---|
| `/manager/status?pretty=true` | GET | Wazuh manager running status |
| `/manager/info?pretty=true` | GET | Manager version and info |
| `/manager/configuration?pretty=true` | GET | Current configuration |
| `/manager/stats?pretty=true` | GET | Performance statistics |
| `/manager/logs?pretty=true&limit=20` | GET | Recent manager logs |

### Rules & Decoders
| Endpoint | Method | Description |
|---|---|---|
| `/rules?pretty=true&limit=10` | GET | Detection rules |
| `/rules?pretty=true&q=level>12` | GET | Critical detection rules |
| `/rules/groups?pretty=true` | GET | Rule groups |
| `/decoders?pretty=true&limit=10` | GET | Log decoders |

### File Integrity Monitoring (FIM)
| Endpoint | Method | Description |
|---|---|---|
| `/syscheck/{agent_id}?pretty=true` | GET | FIM results for agent |
| `/syscheck/{agent_id}/last_scan?pretty=true` | GET | Last FIM scan info |

### Security Configuration Assessment (SCA)
| Endpoint | Method | Description |
|---|---|---|
| `/sca/{agent_id}?pretty=true` | GET | SCA policies for agent |
| `/sca/{agent_id}/checks/{policy_id}?pretty=true` | GET | SCA check results |

### System Inventory (Syscollector)
| Endpoint | Method | Description |
|---|---|---|
| `/syscollector/{agent_id}/os?pretty=true` | GET | OS info for agent |
| `/syscollector/{agent_id}/packages?pretty=true` | GET | Installed packages |
| `/syscollector/{agent_id}/processes?pretty=true` | GET | Running processes |
| `/syscollector/{agent_id}/ports?pretty=true` | GET | Open ports |
| `/syscollector/{agent_id}/netaddr?pretty=true` | GET | Network addresses |

### OpenSearch Direct Queries (Wazuh Indexer)

For advanced queries, you can query the Wazuh Indexer (OpenSearch) directly:

```bash
# Search alerts index directly
curl -ku admin:SecretPassword -X GET "https://localhost:9200/wazuh-alerts-*/_search?pretty" \
  -H "Content-Type: application/json" \
  -d '{"size": 10, "sort": [{"timestamp": {"order": "desc"}}]}'

# Count alerts by MITRE technique
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
    "query": {
      "range": {"timestamp": {"gte": "now-1h"}}
    },
    "sort": [{"timestamp": {"order": "desc"}}]
  }'
```

## Report Generation

When asked to generate a security report, follow this structure:

1. **Executive Summary** — Overall security posture in 2-3 sentences
2. **Agent Health** — How many agents active/disconnected/total
3. **Alert Summary** — Total alerts, breakdown by severity (rule.level)
4. **Top Threats** — Highest severity alerts with MITRE ATT&CK mapping
5. **IDS Events** — Intrusion detection alerts
6. **Vulnerability Status** — Critical/high vulnerabilities per agent
7. **Compliance Status** — SCA pass/fail rates
8. **File Integrity** — FIM changes detected
9. **Recommendations** — Actionable steps to improve security posture

## Usage Guidelines

- Always authenticate before making API calls
- Use `jq` to format and filter JSON responses when needed
- For large result sets, use `limit` and `offset` parameters
- Check `error` field in responses (0 = success)
- If an endpoint returns 404, the feature may not have data yet (agents need to be reporting)
- Sort alerts by timestamp descending (`sort=-timestamp`) for most recent first
- Use `q=` parameter for field-level filtering
- Rule levels: 0-3 (info), 4-7 (low), 8-11 (medium), 12-15 (high/critical)

## Example Monitoring Workflow

When performing routine monitoring:

```bash
# Step 1: Authenticate
TOKEN=$(curl -su wazuh:wazuh -k -X POST "https://localhost:55000/security/user/authenticate?raw=true")

# Step 2: Check agent health
curl -k -X GET "https://localhost:55000/agents/summary/status?pretty=true" -H "Authorization: Bearer $TOKEN"

# Step 3: Check for high-severity alerts (last 50)
curl -k -X GET "https://localhost:55000/alerts?pretty=true&limit=50&sort=-timestamp&q=rule.level>10" -H "Authorization: Bearer $TOKEN"

# Step 4: Check manager health
curl -k -X GET "https://localhost:55000/manager/status?pretty=true" -H "Authorization: Bearer $TOKEN"

# Step 5: Check manager stats
curl -k -X GET "https://localhost:55000/manager/stats?pretty=true" -H "Authorization: Bearer $TOKEN"
```

Analyze the results and provide a concise security status update.
SKILL_EOF
```

### 4.3 Create the Helper Script

Create `~/.openclaw/skills/wazuh-monitor/wazuh-health-check.sh`:

```bash
cat > ~/.openclaw/skills/wazuh-monitor/wazuh-health-check.sh << 'SCRIPT_EOF'
#!/bin/bash
# Wazuh Health Check Script — Used by OpenClaw Wazuh Monitor Skill
# This script performs a quick health assessment of the Wazuh deployment

set -euo pipefail

WAZUH_API="https://localhost:55000"
WAZUH_USER="${WAZUH_API_USER:-wazuh}"
WAZUH_PASS="${WAZUH_API_PASSWORD:-wazuh}"
INDEXER_URL="https://localhost:9200"
INDEXER_USER="${INDEXER_USER:-admin}"
INDEXER_PASS="${INDEXER_PASS:-SecretPassword}"

echo "============================================"
echo "  Wazuh Security Health Check"
echo "  $(date -u '+%Y-%m-%d %H:%M:%S UTC')"
echo "============================================"
echo ""

# Step 1: Get API Token
echo "--- Authenticating with Wazuh API ---"
TOKEN=$(curl -su "$WAZUH_USER:$WAZUH_PASS" -k -X POST \
  "$WAZUH_API/security/user/authenticate?raw=true" 2>/dev/null)

if [ -z "$TOKEN" ] || echo "$TOKEN" | grep -q "error"; then
  echo "ERROR: Failed to authenticate with Wazuh API"
  exit 1
fi
echo "Authentication: OK"
echo ""

# Step 2: Manager Status
echo "--- Manager Status ---"
curl -sk -X GET "$WAZUH_API/manager/status?pretty=true" \
  -H "Authorization: Bearer $TOKEN" 2>/dev/null | jq -r '.data.affected_items[0] // "No data"' 2>/dev/null || echo "Could not retrieve manager status"
echo ""

# Step 3: Agent Summary
echo "--- Agent Summary ---"
curl -sk -X GET "$WAZUH_API/agents/summary/status?pretty=true" \
  -H "Authorization: Bearer $TOKEN" 2>/dev/null | jq '.data' 2>/dev/null || echo "Could not retrieve agent summary"
echo ""

# Step 4: Recent High-Severity Alerts (level > 10)
echo "--- High-Severity Alerts (level > 10, last 10) ---"
curl -sk -X GET "$WAZUH_API/alerts?pretty=true&limit=10&sort=-timestamp&q=rule.level>10" \
  -H "Authorization: Bearer $TOKEN" 2>/dev/null | jq '.data.total_affected_items // 0' 2>/dev/null || echo "0"
echo " high-severity alerts found"
echo ""

# Step 5: Manager Stats
echo "--- Manager Performance ---"
curl -sk -X GET "$WAZUH_API/manager/stats?pretty=true" \
  -H "Authorization: Bearer $TOKEN" 2>/dev/null | jq '.' 2>/dev/null || echo "Could not retrieve stats"
echo ""

# Step 6: Indexer Health
echo "--- Indexer (OpenSearch) Health ---"
curl -sk -u "$INDEXER_USER:$INDEXER_PASS" "$INDEXER_URL/_cluster/health?pretty" 2>/dev/null | jq '{status, number_of_nodes, active_shards}' 2>/dev/null || echo "Could not reach indexer"
echo ""

# Step 7: Alert Count (Last 24h via OpenSearch)
echo "--- Alert Count (Last 24h) ---"
ALERT_COUNT=$(curl -sk -u "$INDEXER_USER:$INDEXER_PASS" \
  "$INDEXER_URL/wazuh-alerts-*/_count" \
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

Create `~/.openclaw/skills/wazuh-monitor/wazuh-report.sh`:

```bash
cat > ~/.openclaw/skills/wazuh-monitor/wazuh-report.sh << 'REPORT_EOF'
#!/bin/bash
# Wazuh Security Report Generator — Used by OpenClaw
# Outputs structured data that OpenClaw can analyze and format

set -euo pipefail

WAZUH_API="https://localhost:55000"
WAZUH_USER="${WAZUH_API_USER:-wazuh}"
WAZUH_PASS="${WAZUH_API_PASSWORD:-wazuh}"
INDEXER_URL="https://localhost:9200"
INDEXER_USER="${INDEXER_USER:-admin}"
INDEXER_PASS="${INDEXER_PASS:-SecretPassword}"
REPORT_DIR="${REPORT_DIR:-$HOME/.openclaw/reports}"

mkdir -p "$REPORT_DIR"
TIMESTAMP=$(date -u '+%Y-%m-%d_%H-%M-%S')
REPORT_FILE="$REPORT_DIR/wazuh-report-$TIMESTAMP.json"

# Authenticate
TOKEN=$(curl -su "$WAZUH_USER:$WAZUH_PASS" -k -X POST \
  "$WAZUH_API/security/user/authenticate?raw=true" 2>/dev/null)

if [ -z "$TOKEN" ] || echo "$TOKEN" | grep -q "error"; then
  echo '{"error": "Authentication failed"}' | tee "$REPORT_FILE"
  exit 1
fi

AUTH_HEADER="Authorization: Bearer $TOKEN"

# Collect all data
echo "Collecting Wazuh security data..."

MANAGER_INFO=$(curl -sk -X GET "$WAZUH_API/manager/info?pretty=true" -H "$AUTH_HEADER" 2>/dev/null)
MANAGER_STATUS=$(curl -sk -X GET "$WAZUH_API/manager/status?pretty=true" -H "$AUTH_HEADER" 2>/dev/null)
AGENT_SUMMARY=$(curl -sk -X GET "$WAZUH_API/agents/summary/status?pretty=true" -H "$AUTH_HEADER" 2>/dev/null)
AGENTS_LIST=$(curl -sk -X GET "$WAZUH_API/agents?pretty=true&limit=100" -H "$AUTH_HEADER" 2>/dev/null)
RECENT_ALERTS=$(curl -sk -X GET "$WAZUH_API/alerts?pretty=true&limit=50&sort=-timestamp" -H "$AUTH_HEADER" 2>/dev/null)
HIGH_ALERTS=$(curl -sk -X GET "$WAZUH_API/alerts?pretty=true&limit=20&sort=-timestamp&q=rule.level>10" -H "$AUTH_HEADER" 2>/dev/null)
RULES_SUMMARY=$(curl -sk -X GET "$WAZUH_API/rules/groups?pretty=true" -H "$AUTH_HEADER" 2>/dev/null)
MANAGER_LOGS=$(curl -sk -X GET "$WAZUH_API/manager/logs?pretty=true&limit=30" -H "$AUTH_HEADER" 2>/dev/null)

# OpenSearch: MITRE techniques breakdown
MITRE_BREAKDOWN=$(curl -sk -u "$INDEXER_USER:$INDEXER_PASS" \
  "$INDEXER_URL/wazuh-alerts-*/_search" \
  -H "Content-Type: application/json" \
  -d '{
    "size": 0,
    "query": {"range": {"timestamp": {"gte": "now-24h"}}},
    "aggs": {
      "mitre_tactics": {"terms": {"field": "rule.mitre.tactic.keyword", "size": 20}},
      "mitre_techniques": {"terms": {"field": "rule.mitre.technique.keyword", "size": 20}},
      "severity_levels": {"terms": {"field": "rule.level", "size": 16}},
      "top_rules": {"terms": {"field": "rule.description.keyword", "size": 10}}
    }
  }' 2>/dev/null)

# Alert counts
ALERT_24H=$(curl -sk -u "$INDEXER_USER:$INDEXER_PASS" \
  "$INDEXER_URL/wazuh-alerts-*/_count" \
  -H "Content-Type: application/json" \
  -d '{"query":{"range":{"timestamp":{"gte":"now-24h"}}}}' 2>/dev/null | jq '.count // 0' 2>/dev/null)

ALERT_1H=$(curl -sk -u "$INDEXER_USER:$INDEXER_PASS" \
  "$INDEXER_URL/wazuh-alerts-*/_count" \
  -H "Content-Type: application/json" \
  -d '{"query":{"range":{"timestamp":{"gte":"now-1h"}}}}' 2>/dev/null | jq '.count // 0' 2>/dev/null)

# Indexer health
INDEXER_HEALTH=$(curl -sk -u "$INDEXER_USER:$INDEXER_PASS" "$INDEXER_URL/_cluster/health" 2>/dev/null)

# Build consolidated report
jq -n \
  --arg ts "$TIMESTAMP" \
  --arg alert24h "${ALERT_24H:-0}" \
  --arg alert1h "${ALERT_1H:-0}" \
  --argjson manager_info "$MANAGER_INFO" \
  --argjson manager_status "$MANAGER_STATUS" \
  --argjson agent_summary "$AGENT_SUMMARY" \
  --argjson agents_list "$AGENTS_LIST" \
  --argjson recent_alerts "$RECENT_ALERTS" \
  --argjson high_alerts "$HIGH_ALERTS" \
  --argjson mitre_breakdown "${MITRE_BREAKDOWN:-{}}" \
  --argjson indexer_health "${INDEXER_HEALTH:-{}}" \
  '{
    report_timestamp: $ts,
    alert_counts: {
      last_1_hour: ($alert1h | tonumber),
      last_24_hours: ($alert24h | tonumber)
    },
    manager: {
      info: $manager_info.data,
      status: $manager_status.data
    },
    agents: {
      summary: $agent_summary.data,
      list: $agents_list.data
    },
    alerts: {
      recent: $recent_alerts.data,
      high_severity: $high_alerts.data
    },
    mitre_attack: $mitre_breakdown.aggregations,
    indexer_health: $indexer_health
  }' > "$REPORT_FILE" 2>/dev/null

echo "Report saved to: $REPORT_FILE"
echo ""
cat "$REPORT_FILE" | jq '{
  timestamp: .report_timestamp,
  alert_counts: .alert_counts,
  agents_active: .agents.summary.connection.active,
  agents_total: .agents.summary.connection.total,
  high_severity_alerts: (.alerts.high_severity.total_affected_items // 0),
  indexer_status: .indexer_health.status,
  top_mitre_tactics: [.mitre_attack.mitre_tactics.buckets[]? | {tactic: .key, count: .doc_count}],
  top_mitre_techniques: [.mitre_attack.mitre_techniques.buckets[]? | {technique: .key, count: .doc_count}]
}' 2>/dev/null || cat "$REPORT_FILE"
REPORT_EOF

chmod +x ~/.openclaw/skills/wazuh-monitor/wazuh-report.sh
```

### 4.5 Verify Skill Installation

```bash
# List the skill files
ls -la ~/.openclaw/skills/wazuh-monitor/

# Expected:
# SKILL.md
# wazuh-health-check.sh
# wazuh-report.sh
```

The skill will be automatically loaded on the next OpenClaw session. If OpenClaw is already running, start a new session.

---

## 5. Phase 4 — Connect OpenClaw to Wazuh

### 5.1 Configure OpenClaw for Security Monitoring

Update OpenClaw's configuration to enable the required tools:

```bash
# Enable exec, web_fetch, and cron tools (if not already enabled)
openclaw config set tools.exec.host "gateway"
openclaw config set tools.exec.security "allowlist"
openclaw config set tools.web.fetch.enabled true
openclaw config set browser.enabled true
```

### 5.2 Enable the Wazuh Monitor Skill

The skill is automatically detected from `~/.openclaw/skills/wazuh-monitor/`. Verify it's loaded:

1. Open OpenClaw Control UI: `openclaw dashboard`
2. Type `/skills` to list loaded skills
3. You should see `wazuh-monitor` in the list

### 5.3 Test the Connection

In the OpenClaw Control UI, send these commands:

```
Check the status of our Wazuh deployment. Authenticate with the Wazuh API 
and show me the agent summary and manager health.
```

OpenClaw should use its `exec` tool to run curl commands against the Wazuh API and return the results.

### 5.4 Test the Health Check Script

```
Run the Wazuh health check script at ~/.openclaw/skills/wazuh-monitor/wazuh-health-check.sh 
and analyze the results.
```

### 5.5 Test Report Generation

```
Run the Wazuh report generator at ~/.openclaw/skills/wazuh-monitor/wazuh-report.sh 
and give me a comprehensive security report based on the data.
```

---

## 6. Phase 5 — Schedule Automated Monitoring

### 6.1 Set Up Cron Jobs via OpenClaw

OpenClaw has a built-in `cron` tool. In the Control UI, ask OpenClaw to set up monitoring:

```
Set up the following cron jobs for Wazuh monitoring:

1. Every 15 minutes: Run the health check script and alert me if there are any 
   high-severity alerts or disconnected agents
2. Every 6 hours: Run the full report generator and give me a security summary
3. Daily at 08:00 UTC: Generate a comprehensive daily security report with 
   MITRE ATT&CK analysis, alert trends, and recommendations
```

### 6.2 Manual Cron Setup (Alternative)

If you prefer to set up cron jobs manually via the OpenClaw config:

```bash
# List existing cron jobs
openclaw config get cron
```

Or ask OpenClaw directly:

```
Use the cron tool to:
- Add a job "wazuh-health" that runs every 15 minutes and executes: 
  bash ~/.openclaw/skills/wazuh-monitor/wazuh-health-check.sh
- Add a job "wazuh-report" that runs every 6 hours and executes: 
  bash ~/.openclaw/skills/wazuh-monitor/wazuh-report.sh
```

### 6.3 Common Monitoring Prompts

Use these in the OpenClaw Control UI at any time:

| Prompt | What it does |
|---|---|
| `Check Wazuh status` | Quick health check |
| `Show me the last 20 alerts` | Recent security alerts |
| `Are there any critical alerts?` | High-severity alert check |
| `Which agents are disconnected?` | Agent health check |
| `Show MITRE ATT&CK activity` | Threat mapping analysis |
| `Generate a security report` | Full security assessment |
| `What vulnerabilities were found?` | Vulnerability scan results |
| `Check file integrity changes` | FIM monitoring |
| `Show compliance status` | SCA assessment |
| `Analyze today's security events` | Daily analysis |

---

## 7. Verification & Testing

### 7.1 Full Stack Verification Checklist

```bash
# ── Wazuh Verification ──

# 1. All Docker containers running
docker compose -f ~/wazuh-docker/single-node/docker-compose.yml ps

# 2. Wazuh Dashboard accessible
curl -sk -o /dev/null -w "%{http_code}" https://localhost
# Expected: 200 or 302

# 3. Wazuh API responsive
TOKEN=$(curl -su wazuh:wazuh -k -X POST "https://localhost:55000/security/user/authenticate?raw=true")
curl -sk -X GET "https://localhost:55000/?pretty=true" -H "Authorization: Bearer $TOKEN" | jq '.error'
# Expected: 0

# 4. OpenSearch/Indexer healthy
curl -sk -u admin:SecretPassword "https://localhost:9200/_cluster/health" | jq '.status'
# Expected: "green" or "yellow"

# ── OpenClaw Verification ──

# 5. OpenClaw gateway running
openclaw gateway status

# 6. Skill loaded
# In Control UI, type: /skills
# wazuh-monitor should be listed

# 7. Health check script works
bash ~/.openclaw/skills/wazuh-monitor/wazuh-health-check.sh

# 8. Report script works
bash ~/.openclaw/skills/wazuh-monitor/wazuh-report.sh
```

### 7.2 Enrolling a Wazuh Agent (For Testing)

To generate real security data, enroll a Wazuh agent on any endpoint:

```bash
# On the target endpoint (Ubuntu example)
curl -s https://packages.wazuh.com/key/GPG-KEY-WAZUH | gpg --no-default-keyring \
  --keyring gnupg-ring:/usr/share/keyrings/wazuh.gpg --import && chmod 644 /usr/share/keyrings/wazuh.gpg

echo "deb [signed-by=/usr/share/keyrings/wazuh.gpg] https://packages.wazuh.com/4.x/apt/ stable main" \
  | tee /etc/apt/sources.list.d/wazuh.list

apt-get update && apt-get install wazuh-agent

# Configure and start
sed -i "s/MANAGER_IP/<YOUR_SERVER_IP>/" /var/ossec/etc/ossec.conf
systemctl daemon-reload
systemctl enable wazuh-agent
systemctl start wazuh-agent
```

Or use the Docker agent:

```bash
cd ~/wazuh-docker/wazuh-agent/
# Edit docker-compose.yml: set WAZUH_MANAGER_SERVER=<YOUR_SERVER_IP>
docker compose up -d
```

---

## 8. Troubleshooting

### 8.1 Wazuh Issues

| Problem | Solution |
|---|---|
| Indexer won't start | Check `vm.max_map_count`: `sysctl vm.max_map_count` (must be ≥ 262144) |
| Dashboard shows "not ready" | Wait 60–90 seconds after `docker compose up`. Indexer needs time. |
| API returns 401 | Token expired (15 min TTL). Re-authenticate. |
| API returns connection refused | Check Docker containers: `docker compose ps` |
| No data in dashboard | Enroll at least one Wazuh agent to generate data |
| Certificate errors | Regenerate certs: `docker compose -f generate-indexer-certs.yml run --rm generator` |

### 8.2 OpenClaw Issues

| Problem | Solution |
|---|---|
| `openclaw: command not found` | Re-run install script or add to PATH |
| Gateway won't start | Check Node.js version: `node --version` (must be 22+) |
| Skill not loading | Check SKILL.md frontmatter syntax. Restart session. |
| Exec tool denied | Set `tools.exec.host: "gateway"` and `tools.exec.security: "allowlist"` |
| Can't reach Wazuh API | Verify: `curl -k https://localhost:55000` from the OpenClaw host |

### 8.3 Integration Issues

| Problem | Solution |
|---|---|
| OpenClaw can't query Wazuh | Both must be on same server or network-reachable. Test: `curl -k https://localhost:55000` |
| jq not found | Install: `brew install jq` (macOS) or `apt install jq` (Ubuntu) |
| Reports empty | Ensure agents are enrolled and generating data |
| Cron jobs not firing | Check: ask OpenClaw "list all cron jobs" |

---

## 9. Architecture Reality Check

### 9.1 What OpenClaw Actually Is

**OpenClaw is a personal AI assistant** (by Peter Steinberger), not a purpose-built security monitoring tool. It's an open-source AI agent that:
- Runs on your machine (macOS/Linux/Windows)
- Has persistent memory across sessions
- Can execute shell commands (the `exec` tool)
- Can browse the web (the `browser` tool)
- Can schedule recurring tasks (the `cron` tool)
- Connects to chat apps (WhatsApp, Telegram, Discord, Slack, iMessage)
- Supports custom **skills** (SKILL.md) that teach the AI how to use tools

### 9.2 How the Integration Actually Works

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
│  │  Chat Interfaces (any/all)                       │    │
│  │  - Control UI (localhost:18789)                   │    │
│  │  - WhatsApp / Telegram / Discord / Slack          │    │
│  │  - iMessage                                       │    │
│  └──────────────────────────────────────────────────┘    │
└────────────────────────────────────────────────────────┘
```

### 9.3 The Integration Flow

1. **OpenClaw's AI agent** receives a monitoring request (from you, from a cron job, or from a chat app)
2. The agent reads the **wazuh-monitor skill** instructions
3. It uses the **exec tool** to run `curl` commands against the Wazuh REST API (port 55000) and/or the Wazuh Indexer (port 9200)
4. The AI **analyzes the JSON responses** using its language model
5. It generates a **natural-language security report** or answers your question
6. Results are delivered via the **Control UI** or your connected **chat app**

### 9.4 What This Setup Gives You

| Capability | How it works |
|---|---|
| **See all data** | OpenClaw queries Wazuh API + Indexer via exec/curl |
| **Check logs** | OpenClaw queries Wazuh manager logs and alert indices |
| **MITRE ATT&CK analysis** | OpenClaw queries MITRE endpoints and aggregates OpenSearch data |
| **IDS monitoring** | OpenClaw queries alerts filtered by `rule.groups=intrusion_detection` |
| **Continuous monitoring** | OpenClaw cron jobs run health checks on schedule |
| **Reports** | AI generates natural-language reports from structured Wazuh data |
| **Chat-based access** | Ask security questions via WhatsApp/Telegram/Discord |

### 9.5 Contrast with the Architecture Document (v2)

The original architecture document (Architecture-OpenClaw-Wazuh.md) was designed before understanding what OpenClaw actually is. Key differences:

| Architecture Doc (v2) Assumed | Reality |
|---|---|
| OpenClaw has a PostgreSQL/TimescaleDB backend | OpenClaw stores state in `~/.openclaw/` (file-based + memory) |
| OpenClaw has specific sub-services (Ingestion, MITRE Engine, etc.) | OpenClaw is a single AI agent with tools and skills |
| Inter-process communication via shared network | AI agent uses exec tool to run curl commands |
| Custom REST API between systems | Uses Wazuh's existing REST API directly |
| Complex Docker-to-Docker networking | OpenClaw runs natively; Wazuh runs in Docker |

The architecture document's Wazuh section remains accurate. The integration is simpler but equally functional — OpenClaw's AI capabilities compensate for the lack of specialized sub-services by using AI reasoning to analyze, correlate, and report on Wazuh data.

---

## Appendix A: Useful Commands Reference

### Wazuh Docker Management

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

### OpenClaw Management

```bash
# Gateway status
openclaw gateway status

# Start gateway
openclaw gateway start

# Stop gateway
openclaw gateway stop

# Open dashboard
openclaw dashboard

# View config
openclaw config get

# Health check
openclaw doctor
```

### Quick Security Check (One-Liner)

```bash
bash ~/.openclaw/skills/wazuh-monitor/wazuh-health-check.sh
```

---

## Appendix B: File Locations Summary

| Component | Path |
|---|---|
| Wazuh Docker files | `~/wazuh-docker/single-node/` |
| Wazuh Docker Compose | `~/wazuh-docker/single-node/docker-compose.yml` |
| Wazuh SSL certs | `~/wazuh-docker/single-node/config/wazuh_indexer_ssl_certs/` |
| OpenClaw home | `~/.openclaw/` |
| OpenClaw config | `~/.openclaw/openclaw.json` |
| OpenClaw skills | `~/.openclaw/skills/` |
| Wazuh monitor skill | `~/.openclaw/skills/wazuh-monitor/` |
| Wazuh reports | `~/.openclaw/reports/` |

---

## Appendix C: Execution Checklist

- [ ] **Pre-flight:** Docker, Node.js 22+, Git installed
- [ ] **Pre-flight:** `vm.max_map_count=262144` set
- [ ] **Phase 1:** Wazuh repo cloned (v4.14.3)
- [ ] **Phase 1:** SSL certs generated
- [ ] **Phase 1:** `docker compose up -d` successful
- [ ] **Phase 1:** Wazuh Dashboard accessible (https://localhost)
- [ ] **Phase 1:** Wazuh API responding (port 55000)
- [ ] **Phase 2:** OpenClaw installed via script
- [ ] **Phase 2:** Onboarding wizard completed
- [ ] **Phase 2:** Gateway running, Control UI accessible
- [ ] **Phase 3:** wazuh-monitor skill created
- [ ] **Phase 3:** Health check script created & tested
- [ ] **Phase 3:** Report script created & tested
- [ ] **Phase 4:** OpenClaw exec tool configured
- [ ] **Phase 4:** Skill loaded (verify with /skills)
- [ ] **Phase 4:** Test query to Wazuh via OpenClaw succeeds
- [ ] **Phase 5:** Cron jobs configured
- [ ] **Phase 5:** Auto-monitoring verified
- [ ] **Post-deploy:** Default passwords changed
- [ ] **Post-deploy:** At least one Wazuh agent enrolled

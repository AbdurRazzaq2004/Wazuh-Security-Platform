# ⚙️ Wazuh Agent Configuration — The `ossec.conf` File Explained

---

## 📌 What Is This File?

The most important file in your entire Wazuh deployment is:

```
/var/ossec/etc/ossec.conf
```

> **In simple words:** This is the **master configuration file** for the Wazuh agent (or server). Every feature you want to enable or disable — FIM, vulnerability detection, rootcheck, system inventory, SCA — is controlled from this one XML file. **If `ossec.conf` doesn't say it's enabled, it's NOT running.**

---

## 🔑 Golden Rules Before Editing

| Rule | Why |
|---|---|
| **Always back up first** | `sudo cp /var/ossec/etc/ossec.conf /var/ossec/etc/ossec.conf.bak` |
| **Use `sudo`** | The file is owned by root — you need elevated privileges |
| **Restart after changes** | Changes don't take effect until you restart the manager/agent |
| **Validate XML syntax** | One missing `>` or `</tag>` will break the entire config |
| **Test in lab first** | Never push untested config changes to production |

### How to Edit

```bash
sudo vim /var/ossec/etc/ossec.conf
```

### How to Restart (for changes to take effect)

```bash
# On the Wazuh MANAGER (server):
sudo systemctl restart wazuh-manager

# On a Wazuh AGENT:
sudo systemctl restart wazuh-agent
```

> ⚠️ **CRITICAL:** After editing `ossec.conf`, you MUST restart the service. Without a restart, your changes sit in the file doing absolutely nothing.

---

## 🗂️ File Structure Overview

The `ossec.conf` file is one big XML document wrapped in `<ossec_config>` tags. Inside, each **section** controls a different feature:

```
<ossec_config>
    │
    ├── <global>                    ← Logging & output settings
    │
    ├── <rootcheck>                 ← Rootkit & policy monitoring
    │
    ├── <wodle name="syscollector"> ← System inventory collection
    │
    ├── <sca>                       ← Security Configuration Assessment
    │
    ├── <vulnerability-detection>   ← CVE vulnerability scanning
    │
    ├── <indexer>                   ← Connection to Wazuh Indexer
    │
    ├── <syscheck>                  ← File Integrity Monitoring (FIM)
    │
    └── ... (other sections)
</ossec_config>
```

Now let's break down **every section you enabled** and explain what each line does.

---

---

## 1. 📋 `<global>` — Logging & Output Settings

```xml
<global>
    <jsonout_output>yes</jsonout_output>
    <alerts_log>yes</alerts_log>
    <logall>yes</logall>
    <logall_json>yes</logall_json>
</global>
```

### Line-by-Line Explanation

| Setting | Value | What It Does |
|---|---|---|
| `jsonout_output` | `yes` | Writes alerts in **JSON format** to `/var/ossec/logs/alerts/alerts.json`. This is what Filebeat reads and ships to the Indexer. |
| `alerts_log` | `yes` | Writes alerts in **plain text** to `/var/ossec/logs/alerts/alerts.log`. Human-readable format for quick debugging. |
| `logall` | `yes` | **Archives ALL events** (not just alerts) to `/var/ossec/logs/archives/archives.log`. Even events that don't trigger rules. |
| `logall_json` | `yes` | Same as above but in **JSON format** → `/var/ossec/logs/archives/archives.json` |

### Why This Matters

```
                    All Events from Agents
                            │
                    ┌───────┴───────┐
                    │               │
              Triggers a rule?   No rule match
                    │               │
                    ▼               ▼
              alerts.log       archives.log ← Only if logall=yes
              alerts.json      archives.json ← Only if logall_json=yes
```

> 💡 **SOC Pro Tip:** Enabling `logall` and `logall_json` is **essential for threat hunting**. Without these, you only see events that triggered a rule. But attackers are creative — their activity might not match any existing rule. With archives enabled, you can search through ALL raw logs retroactively when investigating an incident.

> ⚠️ **Storage Warning:** Enabling `logall` generates **significantly more data**. For a busy environment with 100+ agents, archives can grow by **several GB per day**. Plan your disk space accordingly.

### Where the Logs Live

| File | Format | Contains | When Written |
|---|---|---|---|
| `/var/ossec/logs/alerts/alerts.log` | Plain text | Only events matching rules | Always (if `alerts_log=yes`) |
| `/var/ossec/logs/alerts/alerts.json` | JSON | Only events matching rules | If `jsonout_output=yes` |
| `/var/ossec/logs/archives/archives.log` | Plain text | ALL events (alerts + non-alerts) | If `logall=yes` |
| `/var/ossec/logs/archives/archives.json` | JSON | ALL events (alerts + non-alerts) | If `logall_json=yes` |

---

---

## 2. 🔍 `<rootcheck>` — Rootkit & Policy Monitoring

```xml
<rootcheck>
    <disabled>no</disabled>
    <check_files>yes</check_files>
    <check_trojans>yes</check_trojans>
    <check_dev>yes</check_dev>
    <check_sys>yes</check_sys>
    <check_pids>yes</check_pids>
    <check_ports>yes</check_ports>
    <check_if>yes</check_if>
</rootcheck>
```

> **Note:** You mentioned `<disabled>yes</disabled>` in the original — to actually enable rootcheck scanning, this should be set to `no`. Setting it to `yes` means rootcheck is **turned OFF**.

### Line-by-Line Explanation

| Setting | Value | What It Does |
|---|---|---|
| `disabled` | `no` | **Enables** the rootcheck module. Set to `yes` to disable it. |
| `check_files` | `yes` | Scans for **known rootkit files** on the system (e.g., files dropped by t0rn, Adore, Knark rootkits) |
| `check_trojans` | `yes` | Checks if **common system binaries** have been trojaned (replaced with malicious versions). Compares against known signatures. |
| `check_dev` | `yes` | Scans the `/dev` directory for **anomalous files**. Real devices live here — if you find regular files, it's suspicious. Attackers hide files in `/dev`. |
| `check_sys` | `yes` | Performs **system-level checks** — looks for hidden files, directories, and suspicious system configurations. |
| `check_pids` | `yes` | Scans for **hidden processes**. If a process is running but hidden from `ps` — that's a rootkit red flag. |
| `check_ports` | `yes` | Scans for **hidden ports**. If a port is open but hidden from `netstat`/`ss` — something is very wrong. |
| `check_if` | `yes` | Checks **network interfaces** for promiscuous mode. If an interface is in promiscuous mode, someone might be **sniffing network traffic**. |

### What Rootcheck Actually Does

```
Rootcheck Module
      │
      ├── check_files   → Does /usr/bin/.t0rn exist? (known rootkit file)
      ├── check_trojans → Is /bin/ls the REAL ls, or a trojan version?
      ├── check_dev     → Are there regular files hiding in /dev/?
      ├── check_sys     → Are there hidden files/directories?
      ├── check_pids    → Is a process running but invisible to ps?
      ├── check_ports   → Is a port open but invisible to netstat?
      └── check_if      → Is any network interface in promiscuous mode?
```

### Real Alert Example
```
** Alert 1668497750.1838326:
Rule: 510 (level 7) -> 'Host-based anomaly detection event (rootcheck).'
Rootkit 't0rn' detected by the presence of file '/usr/bin/.t0rn'.
```

> 💡 **Pro Tip:** Rootcheck uses known rootkit databases (`rootkit_files.txt`, `rootkit_trojans.txt`) stored in `/var/ossec/etc/shared/`. You can add your own custom entries to these files.

---

---

## 3. 📊 `<wodle name="syscollector">` — System Inventory

```xml
<wodle name="syscollector">
    <disabled>no</disabled>
    <interval>1h</interval>
    <scan_on_start>yes</scan_on_start>
    <hardware>yes</hardware>
    <os>yes</os>
    <network>yes</network>
    <packages>yes</packages>
    <ports all="yes">yes</ports>
    <processes>yes</processes>
    <users>yes</users>
    <groups>yes</groups>
    <services>yes</services>
    <browser_extensions>yes</browser_extensions>
</wodle>
```

> **Note:** Same as rootcheck — `<disabled>no</disabled>` to enable it. If you set `yes`, it's disabled.

### Line-by-Line Explanation

| Setting | Value | What It Does |
|---|---|---|
| `disabled` | `no` | **Enables** system inventory collection |
| `interval` | `1h` | Runs a full inventory scan **every 1 hour** |
| `scan_on_start` | `yes` | Runs a scan **immediately** when the agent starts (don't wait for the interval) |
| `hardware` | `yes` | Collects **hardware info** — CPU, RAM, board serial number |
| `os` | `yes` | Collects **OS info** — name, version, architecture, kernel |
| `network` | `yes` | Collects **network interfaces** — IPs, MACs, gateways, DNS |
| `packages` | `yes` | Collects **installed packages** — name, version, vendor. **This is what Vulnerability Detection uses!** |
| `ports` | `yes` (all="yes") | Collects **all open ports** — including ports from other processes (not just the agent's own). The `all="yes"` attribute means ALL ports, not just listening ones. |
| `processes` | `yes` | Collects **running processes** — PID, name, user, command line |
| `users` | `yes` | Collects **local user accounts** — username, UID, home directory, shell |
| `groups` | `yes` | Collects **local groups** — group name, GID, members |
| `services` | `yes` | Collects **system services** — name, status (running/stopped), startup type |
| `browser_extensions` | `yes` | Collects **browser extensions** installed in Chrome/Firefox/Edge — useful for detecting malicious extensions |

### Why Syscollector Is CRITICAL

```
Syscollector collects packages → Vulnerability Detection matches them against CVE database
                                        │
                                        ▼
                               "openssl 1.1.1k is vulnerable to CVE-2024-XXXX"
```

> ⚠️ **If syscollector is disabled, Vulnerability Detection CANNOT work.** It needs the package inventory to know what's installed on each endpoint.

### What Gets Collected (Example)

```json
{
  "hardware": {
    "cpu_name": "Intel Xeon E5-2680 v4",
    "cpu_cores": 8,
    "ram_total": 16384,
    "ram_free": 8192
  },
  "os": {
    "name": "Ubuntu",
    "version": "22.04.3 LTS",
    "architecture": "x86_64",
    "kernel": "5.15.0-91-generic"
  },
  "packages": [
    {"name": "openssl", "version": "3.0.2-0ubuntu1.12"},
    {"name": "nginx", "version": "1.18.0-6ubuntu14.4"},
    {"name": "openssh-server", "version": "1:8.9p1-3ubuntu0.6"}
  ],
  "ports": [
    {"port": 22, "protocol": "tcp", "state": "listening", "process": "sshd"},
    {"port": 80, "protocol": "tcp", "state": "listening", "process": "nginx"},
    {"port": 443, "protocol": "tcp", "state": "listening", "process": "nginx"}
  ],
  "users": [
    {"name": "root", "uid": 0, "shell": "/bin/bash"},
    {"name": "www-data", "uid": 33, "shell": "/usr/sbin/nologin"}
  ]
}
```

---

---

## 4. ✅ `<sca>` — Security Configuration Assessment

```xml
<sca>
    <enabled>yes</enabled>
    <scan_on_start>yes</scan_on_start>
    <interval>12h</interval>
    <skip_nfs>yes</skip_nfs>
</sca>
```

### Line-by-Line Explanation

| Setting | Value | What It Does |
|---|---|---|
| `enabled` | `yes` | **Enables** SCA scanning |
| `scan_on_start` | `yes` | Runs an SCA scan **immediately** when the agent starts |
| `interval` | `12h` | Runs a full SCA scan **every 12 hours** |
| `skip_nfs` | `yes` | **Skips NFS-mounted filesystems** during scans. NFS scans can be slow and cause network load — so we skip them. |

### What SCA Does

```
SCA Module
    │
    ├── Reads policy files from /var/ossec/ruleset/sca/
    │   ├── cis_ubuntu22-04.yml
    │   ├── cis_win2022.yml
    │   └── ... (one per OS/standard)
    │
    ├── Runs each check against the system
    │   ├── "Is SSH root login disabled?" → Check sshd_config
    │   ├── "Is /tmp a separate partition?" → Check mount points
    │   └── "Is firewall enabled?" → Check ufw/iptables status
    │
    └── Reports results: PASS ✅ / FAIL ❌ / NOT APPLICABLE ⬜
```

> 💡 **Pro Tip:** 12 hours is a good default interval. For critical production servers, consider `6h` or even `4h`. For labs, `12h` is fine to avoid unnecessary CPU load.

---

---

## 5. 🔓 `<vulnerability-detection>` — CVE Vulnerability Scanning

```xml
<vulnerability-detection>
    <enabled>yes</enabled>
    <index-status>yes</index-status>
    <feed-update-interval>60m</feed-update-interval>
</vulnerability-detection>
```

### Line-by-Line Explanation

| Setting | Value | What It Does |
|---|---|---|
| `enabled` | `yes` | **Enables** vulnerability detection on the manager |
| `index-status` | `yes` | Indexes vulnerability **status changes** (new, solved, active) into the Wazuh Indexer. This is what powers the Dashboard's vulnerability views. |
| `feed-update-interval` | `60m` | Downloads updated vulnerability data from **Wazuh CTI** every **60 minutes**. This keeps your CVE database fresh. |

### How Vulnerability Detection Works

```
Step 1: Syscollector (agent) collects package inventory
                │
Step 2: Sends to Wazuh Server
                │
Step 3: Vulnerability Detection module correlates:
        "Agent has openssl 3.0.2" + "CVE DB says 3.0.2 is vulnerable"
                │
Step 4: Alert generated: "CVE-2024-XXXX affects openssl on agent-001"
                │
Step 5: Indexed and displayed on Dashboard
```

> ⚠️ **This section runs on the MANAGER (server), not the agent.** The agent collects inventory; the server does the vulnerability matching.

> ⚠️ **Dependency:** Syscollector MUST be enabled on agents for this to work. No package inventory = no vulnerability detection.

### Feed Update
```
Wazuh CTI (Internet) ──every 60 min──▶ Wazuh Server (vulnerability DB)
                                              │
                                              ▼
                                    Matches against agent inventory
                                              │
                                              ▼
                                    Alerts for new vulnerabilities
```

---

---

## 6. 🗄️ `<indexer>` — Connection to Wazuh Indexer

```xml
<indexer>
    <enabled>yes</enabled>
    <hosts>
        <host>https://127.0.0.1:9200</host>
    </hosts>
</indexer>
```

### Line-by-Line Explanation

| Setting | Value | What It Does |
|---|---|---|
| `enabled` | `yes` | **Enables** the direct Indexer connection from the manager |
| `hosts` → `host` | `https://127.0.0.1:9200` | The **URL of the Wazuh Indexer**. In an all-in-one setup, it's `127.0.0.1` (localhost) since everything runs on the same machine. In multi-node, this would be the Indexer's IP. |

### Why `127.0.0.1`?

```
All-in-One Deployment (your lab):
┌─────────────────────────────────────┐
│         Same Machine                │
│                                     │
│  Wazuh Server ──127.0.0.1:9200──▶ Wazuh Indexer
│                                     │
└─────────────────────────────────────┘

Multi-Node Deployment (production):
┌──────────────┐                ┌──────────────┐
│ Wazuh Server │──10.0.1.50:9200──▶│ Wazuh Indexer│
│ (Machine 1)  │                │ (Machine 2)  │
└──────────────┘                └──────────────┘
```

> 💡 For a **multi-node** cluster with multiple indexer nodes, you'd list all of them:
```xml
<hosts>
    <host>https://10.0.1.50:9200</host>
    <host>https://10.0.1.51:9200</host>
    <host>https://10.0.1.52:9200</host>
</hosts>
```

---

---

## 7. 📁 `<syscheck>` — File Integrity Monitoring (FIM)

```xml
<syscheck>
    <disabled>no</disabled>
    <frequency>43200</frequency>
    <scan_on_start>yes</scan_on_start>
</syscheck>
```

> **Note:** You showed `<disabled>yes</disabled>` — change to `no` to actually **enable** FIM.

### Line-by-Line Explanation

| Setting | Value | What It Does |
|---|---|---|
| `disabled` | `no` | **Enables** File Integrity Monitoring |
| `frequency` | `43200` | Runs a **full scan every 43200 seconds** (= 12 hours). This is the default. |
| `scan_on_start` | `yes` | Runs a FIM scan **immediately** when the agent starts |

### What You Should Add (Directories to Monitor)

The basic config above enables FIM, but you need to tell it **WHAT to monitor**. Add directories:

```xml
<syscheck>
    <disabled>no</disabled>
    <frequency>43200</frequency>
    <scan_on_start>yes</scan_on_start>

    <!-- Critical Linux directories -->
    <directories realtime="yes" check_all="yes">/etc</directories>
    <directories realtime="yes" check_all="yes">/usr/bin</directories>
    <directories realtime="yes" check_all="yes">/usr/sbin</directories>
    <directories realtime="yes" check_all="yes">/bin</directories>
    <directories realtime="yes" check_all="yes">/sbin</directories>
    <directories realtime="yes" check_all="yes">/var/www</directories>

    <!-- Monitor specific critical files -->
    <directories realtime="yes" check_all="yes">/etc/passwd</directories>
    <directories realtime="yes" check_all="yes">/etc/shadow</directories>
    <directories realtime="yes" check_all="yes">/etc/sudoers</directories>
    <directories realtime="yes" check_all="yes">/etc/ssh/sshd_config</directories>

    <!-- Ignore noisy directories -->
    <ignore>/etc/mtab</ignore>
    <ignore>/etc/resolv.conf</ignore>
    <ignore type="sregex">.log$</ignore>
</syscheck>
```

| Attribute | What It Does |
|---|---|
| `realtime="yes"` | Monitor in **real-time** using inotify (Linux) instead of waiting for scheduled scans |
| `check_all="yes"` | Check **everything** — size, permissions, owner, group, MD5, SHA1, SHA256, mtime, inode |
| `<ignore>` | **Skip** specific files or patterns to reduce noise |

---

---

## 📊 Complete `<disabled>` Quick Reference

This is the most common mistake people make — confusion about what `disabled` means:

| Section | `disabled=yes` | `disabled=no` / `enabled=yes` |
|---|---|---|
| `<rootcheck>` | ❌ Rootcheck is OFF | ✅ Rootcheck is ON |
| `<wodle name="syscollector">` | ❌ Inventory is OFF | ✅ Inventory is ON |
| `<sca>` | Uses `<enabled>yes</enabled>` instead | ✅ SCA is ON |
| `<vulnerability-detection>` | Uses `<enabled>yes</enabled>` instead | ✅ Vuln detection is ON |
| `<syscheck>` | ❌ FIM is OFF | ✅ FIM is ON |

> ⚠️ **Watch out!** Some modules use `<disabled>` (where `yes` = OFF) and some use `<enabled>` (where `yes` = ON). It's confusing but that's how Wazuh config works. Double-check your settings!

---

---

## 🔄 The Restart — Making It All Live

After editing `ossec.conf`, **nothing changes until you restart**:

```bash
# Check config syntax first (catches XML errors)
sudo /var/ossec/bin/wazuh-analysisd -t

# Restart the manager
sudo systemctl restart wazuh-manager

# Verify it's running
sudo systemctl status wazuh-manager
```

### What Happens During Restart

```
1. wazuh-manager reads /var/ossec/etc/ossec.conf
2. Validates XML syntax
3. Starts each enabled module:
   ├── rootcheck    → Begins rootkit scanning
   ├── syscollector → Runs first inventory scan (scan_on_start=yes)
   ├── sca          → Runs first SCA scan (scan_on_start=yes)
   ├── vuln-detect  → Downloads latest CVE feeds
   ├── syscheck     → Runs first FIM baseline scan (scan_on_start=yes)
   └── analysis     → Starts listening for agent data
4. Agents reconnect and start sending data
5. You see results on the Dashboard within minutes
```

### If Restart Fails

```bash
# Check the logs for errors
sudo cat /var/ossec/logs/ossec.log | tail -50

# Common issues:
# - XML syntax error (missing closing tag)
# - Invalid value in a setting
# - Permission issue on the config file
```

---

## 🧩 How All These Sections Work Together

Here's the big picture of how every section in `ossec.conf` feeds into the Dashboard:

```
ossec.conf sections              What they produce           Dashboard module
─────────────────               ─────────────────           ────────────────
<global>
  logall=yes          ──▶  Archives (all events)     ──▶  Threat Hunting
  alerts_log=yes      ──▶  Alert logs                ──▶  All alert modules

<rootcheck>           ──▶  Rootkit detection alerts  ──▶  Malware Detection

<syscollector>        ──▶  System inventory data     ──▶  IT Hygiene
                      ──▶  Package list              ──▶  Vulnerability Detection

<sca>                 ──▶  CIS benchmark results     ──▶  Configuration Assessment

<vulnerability-       ──▶  CVE match alerts          ──▶  Vulnerability Detection
  detection>

<indexer>             ──▶  (Stores all the above)    ──▶  (Powers all dashboards)

<syscheck>            ──▶  File change alerts        ──▶  File Integrity Monitoring
```

---

## 💡 Key Takeaways

> 1. **`ossec.conf` is the single source of truth.** If a feature isn't enabled here, it doesn't run.

> 2. **`disabled=yes` means OFF.** This is counterintuitive. For `rootcheck`, `syscollector`, and `syscheck`, set `disabled` to `no` to enable them.

> 3. **Always restart after changes.** `sudo systemctl restart wazuh-manager` (or `wazuh-agent`).

> 4. **Syscollector is a dependency.** Without it, vulnerability detection has no package data to match against.

> 5. **`logall=yes` is essential for threat hunting** but generates a lot of data. Plan your storage.

> 6. **Test config before restarting in production:** `sudo /var/ossec/bin/wazuh-analysisd -t`

---

## 🔗 References
- ossec.conf Reference: https://documentation.wazuh.com/current/user-manual/reference/ossec-conf/index.html
- Rootcheck Config: https://documentation.wazuh.com/current/user-manual/reference/ossec-conf/rootcheck.html
- Syscollector Config: https://documentation.wazuh.com/current/user-manual/reference/ossec-conf/wodle-syscollector.html
- SCA Config: https://documentation.wazuh.com/current/user-manual/capabilities/sec-config-assessment/how-to-configure.html
- Vulnerability Detection Config: https://documentation.wazuh.com/current/user-manual/capabilities/vulnerability-detection/configuring-scans.html
- Syscheck Config: https://documentation.wazuh.com/current/user-manual/reference/ossec-conf/syscheck.html

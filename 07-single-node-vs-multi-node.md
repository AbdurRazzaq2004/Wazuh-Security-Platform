# 🏗️ Wazuh Deployment: Single-Node vs Multi-Node — When to Use What

---

## 📌 Why Does This Matter?

Before you install Wazuh, you need to answer one critical question:

> **"How am I going to deploy this — everything on one machine, or spread across multiple machines?"**

The wrong choice can mean:
- 💸 Wasting money on over-engineering a lab setup
- 💀 Crashing your production SIEM because it's under-resourced
- 😴 3 AM pager alerts because your single server went down and you have **zero visibility**

Let's break it down clearly.

---

## 🧪 Single-Node Deployment (a.k.a. All-in-One)

### What Is It?

All three central components — **Wazuh Server**, **Wazuh Indexer**, and **Wazuh Dashboard** — run on a **single machine**.

```
┌──────────────────────────────────────────────┐
│            ONE Machine / VM                  │
│                                              │
│   ┌──────────────┐                           │
│   │ Wazuh Server │  (analyzes data)          │
│   └──────────────┘                           │
│   ┌──────────────┐                           │
│   │ Wazuh Indexer│  (stores & searches data) │
│   └──────────────┘                           │
│   ┌──────────────┐                           │
│   │ W. Dashboard │  (web UI)                 │
│   └──────────────┘                           │
│                                              │
│   RAM: 8-16 GB  |  CPU: 4-8 cores           │
│   Disk: 50-100 GB                            │
└──────────────────────────────────────────────┘
           ▲          ▲          ▲
           │          │          │
       Agent 1    Agent 2    Agent 3
      (laptop)   (server)   (cloud VM)
```

### ✅ Why We Use Single-Node for Lab / Learning / PoC

| Reason | Explanation |
|---|---|
| **1. Fast to set up** | One machine, one install script, done in 10 minutes. No networking between nodes to configure. |
| **2. Minimal resources** | You only need **one VM** (8 GB RAM is enough). No need for 3-4 separate machines. |
| **3. Cost = almost zero** | Run it on your laptop in a VirtualBox/Vagrant VM, a free-tier cloud instance, or a spare machine. |
| **4. Perfect for learning** | You're learning Wazuh — you need to understand how it works, not how to architect a cluster. Focus on the product, not the infrastructure. |
| **5. Great for PoC/demos** | Showing Wazuh to your team or management? One VM is all you need to demonstrate every feature. |
| **6. Easy to break & rebuild** | Messed up the config? Destroy the VM, re-run the install. 10 minutes and you're back. |
| **7. All logs are local** | Debugging is easier — all logs (`/var/ossec/logs/`, indexer logs, dashboard logs) are on one machine. |
| **8. Sufficient for small environments** | Monitoring **< 25-50 agents**? A single node handles it just fine. |

### ⚠️ Limitations of Single-Node

| Limitation | What This Means in Practice |
|---|---|
| **No high availability** | If this one machine goes down → **ALL visibility is gone**. No alerts, no dashboard, no data ingestion. You're blind. |
| **No fault tolerance** | Disk dies? You lose your indexed data. No replica shards on other nodes. |
| **Resource contention** | Server analysis, indexer search queries, and dashboard rendering all fight for the same CPU/RAM. Under heavy load, everything slows down. |
| **Can't scale horizontally** | Hit the ceiling? You can't just "add another node." You'd need to migrate to a multi-node setup. |
| **Not suitable for compliance** | Auditors for PCI DSS, HIPAA, etc. expect resilient infrastructure. A single point of failure won't pass. |
| **Performance cap** | Typically maxes out at **~50-100 agents** depending on event volume and hardware specs. |

### 🎯 When to Use Single-Node

| Scenario | Use Single-Node? |
|---|---|
| Learning Wazuh for the first time | ✅ Yes |
| Lab / home lab / CTF practice | ✅ Yes |
| Proof of Concept for your team | ✅ Yes |
| Small office (< 25 endpoints) | ✅ Yes |
| Development / testing environment | ✅ Yes |
| Production with 100+ agents | ❌ No |
| Compliance-required environment | ❌ No |
| Business-critical monitoring | ❌ No |

---

## 🏢 Multi-Node Deployment (Production / Enterprise)

### What Is It?

Each component runs on **its own dedicated machine(s)**, and critical components are **clustered** for redundancy and performance.

```
                        ┌────────────────┐
                        │  Load Balancer │
                        │  (HAProxy/NLB) │
                        └───────┬────────┘
                                │
              ┌─────────────────┼─────────────────┐
              ▼                 ▼                  ▼
     ┌────────────────┐ ┌────────────────┐ ┌────────────────┐
     │  Wazuh Server  │ │  Wazuh Server  │ │  Wazuh Server  │
     │   (Master)     │ │   (Worker 1)   │ │   (Worker 2)   │
     │  ~1000 agents  │ │  ~1000 agents  │ │  ~1000 agents  │
     └───────┬────────┘ └───────┬────────┘ └───────┬────────┘
             │                  │                   │
             └──────────┬───────┘───────────────────┘
                        ▼
     ┌─────────────────────────────────────────────────┐
     │            Wazuh Indexer Cluster                 │
     │                                                  │
     │  ┌──────────┐   ┌──────────┐   ┌──────────┐    │
     │  │  Node 1  │   │  Node 2  │   │  Node 3  │    │
     │  │ Primary  │   │ Primary  │   │ Primary  │    │
     │  │ shards   │   │ shards   │   │ shards   │    │
     │  │ +Replica │   │ +Replica │   │ +Replica │    │
     │  │ from N2  │   │ from N3  │   │ from N1  │    │
     │  └──────────┘   └──────────┘   └──────────┘    │
     │                                                  │
     └──────────────────────┬──────────────────────────┘
                            │
                            ▼
                   ┌────────────────┐
                   │ Wazuh Dashboard│
                   │  (Web UI)      │
                   └────────────────┘
```

### ✅ Why We MUST Use Multi-Node for Production

| Reason | Explanation |
|---|---|
| **1. High Availability (HA)** | Wazuh Server node goes down? The other workers keep processing agent data. Indexer node dies? Replica shards on other nodes serve the data. **You never go blind.** |
| **2. Fault Tolerance** | Disk failure on Indexer Node 1? Data is safe because replicas exist on Nodes 2 and 3. No data loss. |
| **3. Horizontal Scalability** | Need to monitor 5,000 more endpoints? Add another Wazuh Server worker. Need more search capacity? Add another Indexer node. **Scale by adding machines, not by buying bigger ones.** |
| **4. Resource Isolation** | The Indexer doesn't compete with the Server for CPU. The Dashboard doesn't slow down because the Server is processing a spike. Each component gets **dedicated resources**. |
| **5. Performance** | Parallel processing across multiple nodes. Indexer queries distributed across shards on multiple machines. Server analysis spread across workers. **Everything is faster.** |
| **6. Compliance Requirements** | PCI DSS, HIPAA, SOC 2 — all expect resilient, redundant infrastructure. Auditors will flag single points of failure. |
| **7. Disaster Recovery** | With indexer snapshots to S3/NFS and multi-node redundancy, you can recover from disasters without losing security data. |
| **8. Maintenance Without Downtime** | Need to patch an Indexer node? Rolling restart — take down one node at a time, others keep serving. **Zero downtime maintenance.** |

### ⚠️ Trade-offs of Multi-Node

| Trade-off | Reality |
|---|---|
| **More complex setup** | More machines to configure, certificates to manage, networking to set up. But Wazuh's install guide walks you through it. |
| **Higher cost** | 5-7+ machines instead of 1. But for production, this is a necessary investment. |
| **More operational overhead** | Cluster health monitoring, certificate rotation, capacity planning. But that's what DevOps engineers do. |
| **Requires networking expertise** | Firewall rules, load balancer config, TLS certificates. But again — standard DevOps work. |

### 🎯 When to Use Multi-Node

| Scenario | Use Multi-Node? |
|---|---|
| Production environment (any size) | ✅ Yes |
| Monitoring 100+ agents | ✅ Yes |
| Compliance-required (PCI, HIPAA, SOC 2) | ✅ Yes |
| Business-critical security monitoring | ✅ Yes |
| High event throughput (>1000 EPS) | ✅ Yes |
| Need 99.9%+ uptime | ✅ Yes |
| Lab or learning | ❌ Overkill |
| Quick PoC or demo | ❌ Overkill |

---

## 📊 Side-by-Side Comparison

| Aspect | Single-Node (Lab) | Multi-Node (Production) |
|---|---|---|
| **Machines needed** | 1 | 5-7+ (minimum) |
| **Setup time** | ~10 minutes | ~1-2 hours |
| **Minimum RAM** | 8 GB | 8 GB per node (40-56 GB total) |
| **Minimum CPU** | 4 cores | 4 cores per node (20-28 total) |
| **Minimum Disk** | 50 GB | 50-500+ GB per indexer node |
| **Agent capacity** | ~25-100 | Thousands to tens of thousands |
| **High availability** | ❌ None | ✅ Full HA |
| **Fault tolerance** | ❌ None | ✅ Replica shards + server cluster |
| **Horizontal scaling** | ❌ Not possible | ✅ Add nodes as needed |
| **Maintenance downtime** | ⚠️ Full outage during updates | ✅ Rolling restarts, zero downtime |
| **Data safety** | ⚠️ Single copy, one disk | ✅ Replicated across nodes |
| **Compliance ready** | ❌ No | ✅ Yes |
| **Cost** | 💚 Low/Free | 💰 Higher (but justified) |
| **Complexity** | 💚 Simple | 🔶 Moderate to Complex |

---

## 🏗️ Recommended Multi-Node Architecture (Minimum Production)

Here's what I recommend as a **minimum production-ready** multi-node setup:

```
Machines:
─────────
1x  Load Balancer          (HAProxy or cloud NLB)
2x  Wazuh Server           (1 Master + 1 Worker)
3x  Wazuh Indexer           (3-node cluster for quorum)
1x  Wazuh Dashboard         (Web UI)
─────────
Total: 7 machines (or 6 if dashboard shares with a server)
```

### Why 3 Indexer Nodes?
The indexer cluster needs **an odd number of nodes** (3, 5, 7...) to form a **quorum** — a majority vote for cluster decisions. With 3 nodes:
- 1 node goes down → 2 remaining nodes agree on the cluster state → **cluster keeps working**
- 2 nodes go down → only 1 left → can't form majority → **cluster goes read-only** (safe mode)

### Why 2 Server Nodes?
- **Master** node manages cluster configuration and state
- **Worker** node(s) process agent data in parallel
- If the master goes down, a worker can be promoted
- Load balancer distributes agent connections evenly

### Resource Recommendations per Node:

| Component | RAM | CPU | Disk | Notes |
|---|---|---|---|---|
| **Wazuh Server** | 8-16 GB | 4-8 cores | 50 GB | More agents = more RAM |
| **Wazuh Indexer** | 16 GB | 8 cores | 200+ GB SSD | SSD is critical for search performance. Disk grows with retention. |
| **Wazuh Dashboard** | 4-8 GB | 2-4 cores | 20 GB | Lightweight — mostly serves the web UI |
| **Load Balancer** | 2-4 GB | 2 cores | 10 GB | Very lightweight |

---

## 🧪 Lab Quick-Start (Single-Node)

If you're setting up a lab, here's the fastest path:

```bash
# On a fresh Ubuntu 22.04 / Amazon Linux 2 VM (min 8 GB RAM):
curl -sO https://packages.wazuh.com/4.10/wazuh-install.sh
sudo bash ./wazuh-install.sh -a
```

That's it. **One command.** You now have:
- ✅ Wazuh Server (analyzing data)
- ✅ Wazuh Indexer (storing data)
- ✅ Wazuh Dashboard (web UI on port 443)
- ✅ Ready to enroll agents



## 💡 The Bottom Line

> 🧪 **For your lab:** Use **single-node**. It's fast, free, and gives you full Wazuh functionality to learn and experiment. Don't waste time on clustering when you should be learning the product.

> 🏢 **For production:** Use **multi-node**. Your security visibility is only as reliable as your infrastructure. A single server failure shouldn't mean you go blind to attacks. Invest in proper deployment — it's the foundation of your entire security monitoring.

> 🎯 **The rule of thumb:** If someone would lose sleep (or their job) when the Wazuh server goes down, it needs to be multi-node.

---

## 🔗 References
- Quickstart (single-node): https://documentation.wazuh.com/current/quickstart.html
- Installation Guide: https://documentation.wazuh.com/current/installation-guide/index.html
- Deployment Options: https://documentation.wazuh.com/current/deployment-options/index.html
- Architecture: https://documentation.wazuh.com/current/getting-started/architecture.html

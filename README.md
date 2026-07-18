# Production-Ready-MongoDB-Replica-Set-Deployment-for-High-Availability

### This project demonstrates how to deploy a production-ready MongoDB Replica Set across three private Linux servers connected through a gateway server.
### The deployment focuses on High Availability, Authentication, Security, and Operational Best Practices.
---

## 🚨 The Challenge (Why this was needed)
Deploying a single database instance on production poses critical risks:
* **Single Point of Failure (SPOF):** If the database server goes down, the entire application goes offline.
* **Security Exposure:** Keeping database instances directly accessible from the internet invites brute-force attacks and security breaches.
* **Data Loss Risk:** No built-in automated replication or failover mechanism to keep data safe during hardware failure.

---

## 💡 The Solution (High-Level Architecture)

To solve these production challenges, we implemented a decoupled, secure, and resilient infrastructure:

1. **Network Isolation:** All MongoDB nodes are hosted inside a private network, strictly inaccessible from the internet.
2. **Access Control:** System administrators can only access the database nodes via SSH tunneling through a hardened public Gateway (Bastion) host.
3. **High Availability:** Configured a 3-Node MongoDB Replica Set to guarantee auto-failover and zero data loss.

## 🚀 Features

* **✔ 3 MongoDB Nodes:** A robust multi-node architecture to prevent single points of failure.
* **✔ Replica Set:** Configured for seamless data redundancy and high availability.
* **✔ Automatic Failover:** Tested and verified instant election of a new primary node.
* **✔ Private Network:** Isolated database tier with zero direct public internet exposure.
* **✔ Keyfile Authentication:** Secure internal cluster membership validation.
* **✔ Internal Communication:** Inter-node traffic fully secured and authenticated.
* **✔ Gateway Server:** Bastion host setup for secure external administrative access.
* **✔ Firewall Configuration:** Strict port filtering to block unauthorized entry points.
* **✔ Backup Strategy:** Automated scheduling and execution of logical database backups.
* **✔ Monitoring Ready:** Outlined integration for comprehensive database health metrics.

---

## 📐 Architecture

The database cluster is isolated inside a secure private network. All administrative access is strictly tunneled through a public-facing Gateway (Bastion) host.
```text
                       Internet
                          │
                   +--------------+
                   | Gateway Host | (Public IP - SSH Only)
                   +--------------+
                          │
       ┌──────────────────┼──────────────────┐
       │ (Private Net)    │ (Private Net)    │ (Private Net)
+-------------+    +-------------+    +-------------+
| Mongo Node1 |    | Mongo Node2 |    | Mongo Node3 |
|  Primary    |    |  Secondary  |    |  Secondary  |
+-------------+    +-------------+    +-------------+
```
![MongoDB Replica Set Architecture](architecture/Architecture1.png)
## 🛠️ Technologies & Tools Used
* OS: Ubuntu 22.04 LTS (On all 4 servers)
* Database: MongoDB Community Edition (v0.0)
* High Availability: MongoDB Replica Set Engine
* Security: OpenSSL (Keyfile generation), SSH (Secure Tunneling)
* Networking: Private Subnets & Host Resolution
* Process Management: systemd
* Scripting: Bash (Automation & Backups)
* Firewall: UFW (Uncomplicated Firewall)
---

## 🛠️ How It Was Implemented (Reference-Based)
### Instead of manual configurations, the deployment was strictly structured according to enterprise production standards and MongoDB official guidelines. For security and compliance, all external references are kept modular:

* Server Provisioning & Hostname Resolution: Hardened 3 Ubuntu VMs and resolved hosts locally (e.g., mongo-node1, mongo-node2, mongo-node3) to allow secure inter-node communication.

* Database Deployment: Installed MongoDB Community Edition by strictly following the official OS installation steps. [https://www.mongodb.com/docs/v7.0/]

* Replica Set Architecture & High Availability: Designed the cluster topology with dedicated Primary and Secondary members, optimizing Elections (using election protocols, priority settings, and heartbeat settings) and monitoring the Oplog size and replication lag for data synchronization. Detailed design was aligned with MongoDB Replication core concepts. [https://www.mongodb.com/docs/v7.0/replication/]

* Production Hardening & Self-Managed OS Tuning: Optimized the underlying infrastructure for self-managed instances. Formatted data drives, modified kernel parameters, disabled THP, and set file descriptors (ulimit) to prevent bottlenecks under production loads. Configured in alignment with official Production Notes. [https://www.mongodb.com/docs/atlas/production-notes/]

* Network & Configuration Hardening: Secured the self-managed deployment by restricting inbound traffic to port 27017 using host-level firewalls (UFW), binding MongoDB to secure private network interfaces only, and preventing unauthorized external access. Built according to Self-Managed Network Hardening practices. [https://www.mongodb.com/docs/atlas/production-notes/]

* Global Access Control & User Security: Implemented Role-Based Access Control (RBAC) and enabled global database authorization (SCRAM-SHA-256) to ensure that only authenticated clients can read or write to the cluster. Guided by the official Security Checklist. [https://www.mongodb.com/docs/v7.0/administration/analyzing-mongodb-performance/]

---

## 🔍 Verification & Screenshots

> 💡 *Below are the real-time verifications from the running environment.*

### 1. Verification of MongoDB Services
Checking that the database service is up, hardened, and active on the nodes:
![mongod Service Status](Screenshots/status.png)

### 2. Cluster Status & Membership
Running `rs.status()` on the primary node to verify all three nodes joined successfully:
![rs.status Output](Screenshots/rs-status1.png)
![rs.status Output](Screenshots/rs-status2.png)

### 3. Automatic Failover Test
Simulating primary node failure (`systemctl stop mongod`) and verifying the seamless automatic election of a new Primary node within seconds:
![Automatic Failover Verification](Screenshots/failover-test1.png)
![Automatic Failover Verification](Screenshots/failover-test2.png)


### 4. Database Security Proof
Verifying that database reads/writes are strictly unauthorized without active credentials:
`![Database Authentication Verification](screenshots/auth-test.png)`

---
## ⚠️ Troubleshooting (Real-World Issues Faced & Solved)
During the deployment of this production environment, several real-world challenges were encountered and successfully resolved:

### 1. Primary Node Failing to Reclaim Its Role After Recovery
* **The Problem:** When the original Primary node went down, a Secondary was successfully elected. However, when the original Primary server recovered and rejoined the cluster, it remained stuck as a SECONDARY even though it possessed higher system resources and was intended to host the primary workload.

* **The Cause:** MongoDB Replica Set elections don't automatically fail back based on hardware specs; they rely strictly on the priority parameter. If all nodes have the same priority (default is 1), the recovered node won't force a new election.

* **The Solution:** Adjusted the cluster configuration members' weights. Configured the preferred Primary node with a higher priority (e.g., priority: 2 or 1) while setting the other nodes to a lower priority or 0 depending on the desired election topology, ensuring the high-resource server always reclaims the PRIMARY status once healthy. Guided by replica set member configuration practices. [https://www.mongodb.com/docs/manual/core/replica-set-elections/]

### 2. Inter-Cluster Communication Blocked by Host Firewall
* **The Problem:** Newly provisioned database nodes failed to ping each other or initialize the replica set, resulting in replication configuration timeouts.

* **The Cause:** Host-level firewall configurations (UFW) were active by default on the OS image, blocking port 27017 traffic between the nodes.

* **The Solution:** Since all database instances are already entirely isolated inside a secure, trusted Private Network Subnet with zero public internet exposure, the redundant local host firewalls were safely disabled/purged. This streamlined inter-node communication and synchronization without compromising security, as network boundaries are fully controlled at the infrastructure/gateway level.

### 3. Complex Secure GUI Access via Double SSH Tunneling (MongoDB Compass)
* **The Problem:** Accessing the isolated private database nodes via a graphical UI (MongoDB Compass) for administrative tasks was highly complex, as the database cluster has no public IPs and cannot be directly reached.

* **The Cause:** Standard single-hop SSH tunneling fails when the destination database nodes are hidden behind a Gateway/Bastion host inside a segregated subnet.

* **The Solution:** Formulated and executed a dynamic multi-hop (double) SSH port forwarding command. This forwards the secure connection from the local machine through the public Gateway host, and from the Gateway directly into the private database node's port 27017. This enabled seamless, secure management via MongoDB Compass over an encrypted tunnel.

### 4. Data Loss Risk & Storage Performance Bottlenecks
* **The Problem:** Storing database files directly on the server's internal/root operating system drive posed a critical risk: if the OS or virtual instance crashed/corrupted, production data could be permanently lost. Additionally, default filesystems underperformed under high database read/write concurrency.

* **The Cause:** Root volumes lack independent persistence and are not optimized for intense database sequential and random I/O workloads.

* **The Solution:** Provisioned an independent, external block storage volume (Detached Network Storage) and attached it to the database instances. To maximize MongoDB's WiredTiger storage engine throughput and prevent corruption, the external drive was formatted using the XFS Filesystem and mounted cleanly, separating the production data lifecycle from the instance lifecycle. Configured following database storage best practices. [https://www.mongodb.com/docs/manual/tutorial/change-replica-set-wiredtiger/]

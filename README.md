# AWS EC2 Infrastructure & Security Group Configuration for Splunk Architecture

##  Project Overview
This repository documents the network security and firewall configuration for a multi-node **Splunk Enterprise** deployment hosted on **AWS EC2**. It defines the exact inbound/outbound rules and port configurations required to secure Splunk web interfaces, forwarders, indexers, cluster communications, and management traffic.

---

##  Security Group Specifications: `Splunk Access`

### Inbound Rules (Ingress)
The following ports are opened in the AWS Security Group to allow necessary traffic into the Splunk architecture:

| Protocol | Port Range | Source | Use Case / Description |
| :--- | :--- | :--- | :--- |
| **TCP** | `8000` | `0.0.0.0/0` | **Splunk Web:** UI access for analysts and admins |
| **TCP** | `9997` | `0.0.0.0/0` | **Splunk Indexing:** Data ingestion from Universal/Heavy Forwarders |
| **TCP** | `8191` | `0.0.0.0/0` | **KV Store:** Search head cluster synchronization & app storage |
| **TCP** | `8080` | `0.0.0.0/0` | **Replication Port:** Indexer clustering replication |
| **TCP** | `8089` | `0.0.0.0/0` | **Management Port:** REST API & Deployment Server management |
| **TCP** | `8088` | `0.0.0.0/0` | **HTTP Event Collector (HEC):** High-volume token-based ingestion |
| **TCP** | `5514` | `0.0.0.0/0` | **Syslog:** Network log ingestion |

---

##  Deployment Steps (AWS Console)

### Step 1: Security Group Creation
1. Navigate to **AWS Management Console** $\rightarrow$ **EC2** $\rightarrow$ **Network & Security** $\rightarrow$ **Security Groups**.
2. Click **Create Security Group**.
3. Set **Group Name**: `Splunk Access` and **Description**: `Inbound rules for Splunk Cluster nodes`.
4. Configure the 7 ingress rules as specified in the table above.

### Step 2: Instance Attachment
1. Navigate to **EC2 Instances**.
2. Select target instances (Deployment Server, Indexers, Search Heads, Forwarders).
3. Select **Actions** $\rightarrow$ **Networking** $\rightarrow$ **Change Security Groups**.
4. Append `Splunk Access` to each instance without removing baseline VPC security groups.

---

##  Production Best Practices
* **Least Privilege Access:** In production environments, replace `0.0.0.0/0` with specific internal CIDR ranges or Bastion host IP addresses to restrict public access.
* **Separation of Roles:** Assign distinct security groups for Indexer Cluster nodes vs. Search Heads to enforce strict network isolation.

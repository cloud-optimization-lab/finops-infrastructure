# 🛡️ FinOps Guard: Enterprise Infrastructure

[![Platform: Azure](https://img.shields.io/badge/Platform-Azure-0089D6?style=flat-square&logo=microsoft-azure)](https://azure.microsoft.com/)
[![IaC: Terraform](https://img.shields.io/badge/IaC-Terraform-623CE4?style=flat-square&logo=terraform)](https://www.terraform.io/)
[![Orchestration: AKS](https://img.shields.io/badge/Orchestration-Kubernetes-326CE5?style=flat-square&logo=kubernetes)](https://kubernetes.io/)
[![Messaging: Kafka](https://img.shields.io/badge/Messaging-Apache_Kafka-231F20?style=flat-square&logo=apache-kafka)](https://kafka.apache.org/)

## 📖 Overview
**FinOps Guard** is an autonomous, enterprise-grade cloud governance platform. This repository manages the **Infrastructure-as-Code (IaC)**, providing a highly secure, Zero-Trust environment. It is designed to host an event-driven microservices architecture that identifies, analyzes, and remediates cloud waste across multiple Azure subscriptions.

---

## 💼 Business Problem & Solution
### The Problem
In large-scale enterprise environments (e.g., Automotive, Finance), high-velocity development often leads to **"Cloud Sprawl."** 
- **Zombie Resources:** Orphaned disks, idle public IPs, and abandoned snapshots.
- **Over-provisioning:** Workloads running on oversized VM instances.
- **Hidden Costs:** These factors typically account for **20-30% of wasted cloud spend.**

### The Solution
FinOps Guard provides an automated **"Financial Security Layer"** by:
1. **Discovering:** Real-time scanning of infrastructure via Go-based collectors.
2. **Analyzing:** Utilizing **LLMs (Gemini Pro)** to interpret usage metrics into human-readable cost-saving advice.
3. **Remediating:** Implementing "One-click" or automated deletion of wasteful resources within a secure network boundary.

---

## 🏗️ System Architecture

```mermaid
graph TD
    subgraph Public_Internet [🌐 Internet]
        User((Admin User))
        VPN[[P2S VPN Client]]
    end

    subgraph Azure_VNet [VNet: 10.0.0.0/16]
        direction TB
        
        subgraph Snet_Front [Subnet: Frontend - 10.0.1.0/24]
            AGW[App Gateway + WAF]
        end

        subgraph Snet_AKS [Subnet: AKS Cluster - 10.0.2.0/24]
            direction LR
            Collector[Go Collector] -->|Produce Events| Kafka[(Kafka)]
            Kafka -->|Consume Events| Analyzer[Python AI]
            Monitoring[Prometheus/Grafana]
        end

        subgraph Snet_Data [Subnet: Data - 10.0.3.0/24]
            DB[(Postgres Flexible)]
            PE[Private Endpoint]
            Redis[(Redis Cache)]
        end

        subgraph Snet_Mgmt [Subnet: Management - 10.0.4.0/24]
            Bastion[Azure Bastion]
            VPNGW[VPN Gateway]
        end
    end

    User -->|Port 443| AGW
    VPN -->|Secure Tunnel| VPNGW
    AGW --> UI[Web Dashboard]
    Analyzer -->|API| Gemini[Gemini Pro AI]
    PE --> DB
    Collector -.->|Managed Identity| Entra[Azure Entra ID]
```
---
### 🛠️ Tech Stack & Responsibility Matrix
Technology	Role & Responsibility:
- **Terraform	IaC**: Automates the provisioning of the entire Azure environment using a modular, reusable design.
- **Azure VNet	Network Isolation**: Segregates the environment into four distinct subnets to enforce "Least Privilege" traffic flow.
- **AKS (Kubernetes) Compute Platform**: Orchestrates microservices and platform tools (Kafka, Prometheus) with high availability.
- **Apache Kafka Event Streaming**: Decouples the Collector from the Analyzer, ensuring data durability and system resilience.
- **App Gateway + WAF L7 Security**: Manages SSL/TLS termination and protects the frontend from SQLi and XSS attacks.
- **VPN Gateway	Secure Access**: Provides a Point-to-Site (P2S) encrypted tunnel for administrative access to the private network.
- **PostgreSQL	Persistence**: Acts as the "Source of Truth" for cost history, waste reports, and audit logs.
- **Private Link	Data Security**: Ensures that traffic between AKS and PostgreSQL never traverses the public internet.
- **Managed	Identity**: Implements passwordless authentication between Azure services using OIDC.
# 🗺️ FinOps Guard: Implementation Roadmap & Checklist

This checklist tracks the progress of the FinOps Guard platform from initial networking to AI-driven automation.

---

## 🏗️ Phase 1: Networking & Zero-Trust Foundation
*Goal: Establish the secure "pipes" and private connectivity.*
- [ ] **Environment Setup:** Configure Azure CLI, Terraform, and `kubectl` locally.
- [ ] **Resource Group:** Provision `rg-finops-dev-ge-001` in Germany West Central.
- [ ] **Virtual Network (VNet):** Define `10.0.0.0/16` address space.
- [ ] **Subnet Segmentation:** 
    - [ ] `snet-frontend` (10.0.1.0/24) - For App Gateway.
    - [ ] `snet-aks` (10.0.2.0/24) - For Compute.
    - [ ] `snet-data` (10.0.3.0/24) - For Persistence.
    - [ ] `snet-mgmt` (10.0.4.0/24) - For VPN/Bastion.
- [ ] **Network Security Groups (NSG):** Implement "Deny All" ingress/egress by default.
- [ ] **Private DNS Zones:** Setup `privatelink.postgres.database.azure.com`.
- [ ] **Connectivity:** Provision **Azure VPN Gateway** (Point-to-Site) for secure remote access.

## 🔐 Phase 2: Identity & Security Layer
*Goal: Implement passwordless authentication and secret management.*
- [ ] **Azure Key Vault:** Provision with RBAC-based access control.
- [ ] **Managed Identities:** 
    - [ ] Create User-Assigned Identity for the Collector (Go).
    - [ ] Create User-Assigned Identity for the Analyzer (Python).
- [ ] **RBAC Assignments:** Grant "Reader" access to the Subscription for cost-discovery.
- [ ] **Workload Identity:** Configure OIDC issuer and Federated Credentials for AKS.

## 🗄️ Phase 3: Persistent Data Tier
*Goal: Provision private database and caching services.*
- [ ] **PostgreSQL Flexible Server:** Deploy into the `snet-data` with VNet Integration.
- [ ] **Private Endpoint:** Ensure the DB is only reachable via internal IP.
- [ ] **Redis Cache:** Setup for high-speed session and API token caching.

## ☸️ Phase 4: The Platform (AKS & ACR)
*Goal: Build the Kubernetes factory.*
- [ ] **Azure Container Registry (ACR):** Provision and enable "AcrPull" for AKS.
- [ ] **AKS Private Cluster:** 
    - [ ] Provision nodes in `snet-aks`.
    - [ ] Disable Public FQDN (API Server access via VPN/Bastion only).
- [ ] **Add-ons:** Install **Cert-Manager** and **External-DNS** via Helm.

## 📩 Phase 5: Event Streaming (Kafka)
*Goal: Implement the asynchronous messaging backbone.*
- [ ] **Strimzi Operator:** Deploy the Kafka Operator to AKS.
- [ ] **Kafka Cluster:** Provision a 3-node cluster for high availability.
- [ ] **Kafka Topics:** Initialize `cost-discovery-events` and `remediation-logs`.

## 🧠 Phase 6: Microservices Development
*Goal: Build and containerize the "Brains" of the system.*
- [ ] **Go Collector:**
    - [ ] Integrate Azure SDK for Cost Management.
    - [ ] Implement Kafka Producer logic.
    - [ ] Containerize with Multi-stage Docker build.
- [ ] **Python Analyzer:**
    - [ ] Implement Kafka Consumer logic.
    - [ ] Integrate **Gemini Pro API** for cost-analysis logic.
    - [ ] Containerize and push to ACR.

## 🖥️ Phase 7: Frontend & Edge Security
*Goal: Expose the system securely to the user.*
- [ ] **Application Gateway:** Configure with **Web Application Firewall (WAF)**.
- [ ] **SSL/TLS:** Bind certificates from Key Vault to the App Gateway.
- [ ] **Dashboard:** Deploy the React/Angular UI to AKS or Azure Static Web Apps.

## 📊 Phase 8: Observability & SRE
*Goal: Monitor system health and fina

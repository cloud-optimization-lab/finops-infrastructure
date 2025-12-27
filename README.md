# 🛡️ FinOps Guard: Enterprise Infrastructure

[![Platform: Azure](https://img.shields.io/badge/Platform-Azure-0089D6?style=flat-square&logo=microsoft-azure)](https://azure.microsoft.com/)
[![Infrastructure: Terraform](https://img.shields.io/badge/IaC-Terraform-623CE4?style=flat-square&logo=terraform)](https://www.terraform.io/)
[![Orchestration: Kubernetes](https://img.shields.io/badge/Orchestration-Kubernetes-326CE5?style=flat-square&logo=kubernetes)](https://kubernetes.io/)

## 📖 Overview
**FinOps Guard** is an automated, AI-powered cloud governance platform designed to solve the problem of "Cloud Sprawl" in enterprise environments. This repository contains the **Infrastructure-as-Code (IaC)** required to deploy a secure, scalable, and observable environment on Microsoft Azure.

### 💼 Business Problem
Large organizations often waste 20-30% of their cloud budget on:
- **Zombie Resources:** Unattached disks and idle public IPs.
- **Over-provisioning:** Large VMs running minimal workloads.
- **Lack of Governance:** Resources without cost-center tagging.

### 🚀 Solution
This infrastructure provisions a **Zero Trust** network environment that hosts a Go-based collector and a Python-based AI analyzer (Gemini Pro) to identify and remediate cloud waste automatically.

---

## 🏗️ System Architecture

```mermaid
graph TB
    subgraph Internet ["🌐 Public Internet"]
        User((Admin User))
        VPN_Client[[VPN Client]]
    end

    subgraph Azure_Cloud ["☁️ Azure Cloud (Germany West Central)"]
        
        subgraph VNet ["VNet: 10.0.0.0/16"]
            
            subgraph Snet_Frontend ["Subnet: Frontend (10.0.1.0/24)"]
                AGW[Azure App Gateway + WAF]
            end

            subgraph Snet_AKS ["Subnet: AKS Cluster (10.0.2.0/24)"]
                direction TB
                subgraph K8s_Pods ["Kubernetes Pods"]
                    UI[Web UI - React/Angular]
                    Collector[Go Collector Service]
                    Analyzer[Python AI Analyzer]
                end
                
                subgraph K8s_Infra ["K8s Platform Services"]
                    Kafka[(Apache Kafka Cluster - Strimzi)]
                    Prometheus[Prometheus / Grafana]
                    CertManager[Cert-Manager / SSL]
                end
            end

            subgraph Snet_Data ["Subnet: Private Data (10.0.3.0/24)"]
                DB[(PostgreSQL Flexible Server)]
                PE_DB[Private Endpoint]
                Redis[(Redis Cache)]
            end

            subgraph Snet_Mgmt ["Subnet: Management (10.0.4.0/24)"]
                Bastion[Azure Bastion]
                VPNGW[Azure VPN Gateway]
            end
        end

        %% Global Services
        Entra[Azure Entra ID / Managed Identity]
        KV[Azure Key Vault]
        ACR[Azure Container Registry]
        Storage[Azure Storage Account]
        AI[Gemini Pro API]
    end

    %% Traffic Flows
    User -->|HTTPS/SSL - Port 443| AGW
    VPN_Client -->|P2S VPN| VPNGW
    VPNGW -->|Secure Tunnel| Snet_AKS
    
    AGW -->|Internal LB| UI
    UI -->|API Requests| Collector
    
    Collector -->|Produce Cost Events| Kafka
    Kafka -->|Consume Events| Analyzer
    Analyzer -->|LLM Prompt| AI
    
    %% Backend Flows
    K8s_Pods -->|Private Link| PE_DB
    PE_DB --> DB
    K8s_Pods -->|Managed Identity| Entra
    K8s_Pods -->|Fetch Secrets| KV
    
    %% Observability
    K8s_Pods -.->|Metrics Scrape| Prometheus
    Bastion -.->|SSH/RDP| Snet_AKS
```
---
## 🛠️ Tech Stack
*   **Infrastructure:** Terraform, Azure CLI
*   **Cloud Platform:** Microsoft Azure (AKS, VNet, PostgreSQL)
*   **Backend:** Go (Collector), Python (Analyzer + Gemini AI)
*   **Frontend:** React / Angular
*   **CI/CD:** GitHub Actions
---
## 📂 Project Structure
```text
infra-repo/
├── terraform/          # Infrastructure-as-Code
│   ├── modules/        # Reusable networking & AKS modules
│   └── environments/   # Dev/Prod configurations
└── docs/               # Architecture diagrams & specs

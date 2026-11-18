# AZ-104 Microsoft Azure Administrator — Complete Project-Based Learning Contents

Goal: Clear the AZ-104 certification, gain hands-on Azure administration skills, build real-world-style projects, and showcase your work on GitHub, LinkedIn, and your resume.

Learning approach: Theory → Architecture → Hands-on Lab → Troubleshooting → Interview Questions → GitHub → LinkedIn

Learning duration

# 12 Weeks

Monthly Azure budget

# ₹1,000

Hands-on projects

# 16

Certification target

# AZ-104

AZ-900 already completed

## Phase 0 — Azure Setup, Cost Management & Governance

Duration: 2–3 days

| Lab | Topics                                           |
| --- | ------------------------------------------------ |
| 0.1 | Azure subscription, tenants, management groups   |
| 0.2 | Azure Portal, Cloud Shell, Azure CLI, PowerShell |
| 0.3 | Resource groups, regions, availability zones     |
| 0.4 | Azure Cost Management and billing                |
| 0.5 | Configure ₹1,000 monthly budget and alerts       |
| 0.6 | Resource naming conventions and tagging          |
| 0.7 | Cost analysis and resource cleanup               |

Project 00: Azure Subscription Cost Governance & Budget Monitoring

GitHub: `phase-00-cost-management`

## Phase 1 — Azure Identity & Governance

Duration: Weeks 1–2

### Module 1 — Microsoft Entra ID

- Tenants, users, groups, and guest accounts
- Microsoft Entra ID vs Active Directory Domain Services
- User and group management
- Microsoft Entra roles vs Azure RBAC roles
- Self-service password reset
- Microsoft Entra licensing basics

### Module 2 — Azure RBAC

- Role assignments and scope
- Owner, Contributor, Reader, User Access Administrator
- Built-in and custom roles
- Least-privilege access
- Role inheritance
- Troubleshooting permission issues

### Module 3 — Azure Governance

- Management groups and subscriptions
- Azure Policy definitions and initiatives
- Policy effects: Deny, Audit, Modify, DeployIfNotExists
- Resource locks
- Resource tags
- Governance and compliance

| Project | Name                                  |
| ------- | ------------------------------------- |
| 01      | Enterprise Identity & RBAC Management |
| 02      | Azure Governance & Compliance         |
| 03      | Azure Cost Management & Budget Alerts |

## Phase 2 — Azure Storage Administration

Duration: Weeks 3–4

### Module 4 — Azure Storage Fundamentals

- Storage accounts and account types
- Standard vs Premium
- LRS, ZRS, GRS, RA-GRS, GZRS
- Blob, File, Queue, and Table Storage
- Hot, Cool, Cold, and Archive access tiers

### Module 5 — Secure Storage

- Azure Storage RBAC
- Shared Access Signatures (SAS)
- Access keys and stored access policies
- Storage firewalls
- Private endpoints
- Encryption and secure transfer

### Module 6 — Data Protection & Lifecycle

- Blob versioning
- Soft delete
- Lifecycle management rules
- Azure Files and snapshots
- AzCopy and Storage Explorer
- Object replication and redundancy

| Project | Name                                 |
| ------- | ------------------------------------ |
| 04      | Secure Enterprise Blob Storage       |
| 05      | Azure Files & Hybrid File Management |
| 06      | Storage Lifecycle & Data Protection  |

## Phase 3 — Azure Compute & Infrastructure Automation

Duration: Weeks 5–6

### Module 7 — Azure Virtual Machines

- VM sizes and pricing
- Windows and Linux VMs
- Managed disks: OS and data disks
- VM images and extensions
- VM networking
- Availability sets and availability zones
- VM resizing, start, stop, and deallocate
- VM troubleshooting

### Module 8 — High Availability & Scaling

- Virtual Machine Scale Sets (VMSS)
- Autoscaling rules
- Azure Load Balancer integration
- Health probes
- Availability and fault tolerance

### Module 9 — App Service & Containers

- Azure App Service plans
- Web App deployment and configuration
- Deployment slots
- Custom domains and TLS
- Azure Container Instances
- Azure Container Apps fundamentals

### Module 10 — Infrastructure as Code

- ARM templates
- Bicep fundamentals
- Parameters and modules
- Azure CLI and PowerShell automation
- Repeatable infrastructure deployments
- Terraform as an additional DevOps project skill

| Project | Name                                           |
| ------- | ---------------------------------------------- |
| 07      | Azure VM Administration                        |
| 08      | Highly Available & Auto-Scaling Infrastructure |
| 09      | Azure Infrastructure Automation with Bicep     |

## Phase 4 — Azure Virtual Networking

Duration: Weeks 7–8

### Module 11 — Networking Fundamentals

- VNet and subnets
- CIDR and IP addressing
- Public and private IPs
- Azure DNS
- Network interfaces
- VNet peering

### Module 12 — Network Security

- Network Security Groups (NSGs)
- Application Security Groups (ASGs)
- Inbound and outbound rules
- Service tags
- Network isolation
- Network troubleshooting

### Module 13 — Routing & Connectivity

- System routes
- User-defined routes (UDRs)
- Route tables
- VPN Gateway concepts
- Site-to-site and point-to-site VPN
- Hybrid connectivity fundamentals

### Module 14 — Load Balancing & Private Access

- Azure Load Balancer
- Public vs internal load balancing
- Health probes
- Azure Application Gateway
- Private Link and private endpoints
- Service endpoints
- Private DNS zones
- Azure Bastion
- Network Watcher

| Project | Name                                        |
| ------- | ------------------------------------------- |
| 10      | Enterprise VNet & Subnet Architecture       |
| 11      | Azure Load Balancing & Traffic Distribution |
| 12      | Secure Private Connectivity                 |

## Phase 5 — Azure Monitoring, Backup & Disaster Recovery

Duration: Weeks 9–10

### Module 15 — Azure Monitoring

- Azure Monitor
- Metrics vs logs
- Log Analytics workspace
- Kusto Query Language (KQL)
- Diagnostic settings
- Azure Monitor Agent
- Data Collection Rules (DCR)
- Alerts and action groups
- Azure Workbooks
- Application Insights fundamentals

### Module 16 — Backup & Recovery

- Azure Backup
- Recovery Services vault
- Backup policies
- VM backup and restore
- Recovery points
- Soft delete
- Backup monitoring

### Module 17 — Disaster Recovery

- Azure Site Recovery
- Replication concepts
- Recovery plans
- Failover and failback
- Recovery Time Objective (RTO)
- Recovery Point Objective (RPO)
- Business continuity planning

| Project | Name                                       |
| ------- | ------------------------------------------ |
| 13      | Azure Infrastructure Monitoring & Alerting |
| 14      | Azure Backup & Recovery                    |
| 15      | Business Continuity & Disaster Recovery    |

## Phase 6 — Enterprise Capstone Project

Duration: Week 11

## Project 16 — Secure Enterprise Azure Infrastructure

[Azure Web Applications - IaaS Versus PaaS - Petri IT Knowledgebase](https://images.openai.com/static-rsc-4/giChPAQwtjpcRhL_u1zmncZjMauYXTUv__0D7p-Ul2l355FfiYsUC4byD9pi6dkBAhh649AsEXAj9L-Ee521HPMnhds7RacxZyeLQB59vD4RhUy82QjW6wXuyI8paY91aFmVaJQss88B_10IYAfkYBO47ojP1d7-WbgQpINHvRU?purpose=inline)

[petri.com](https://petri.com/azure-web-applications-iaas-versus-paas/?utm_source=chatgpt.com)

This project combines the important AZ-104 services into a single production-style lab.

Architecture components:

- Microsoft Entra ID and RBAC
- Azure Resource Groups and Policy
- Virtual Network with isolated subnets
- Azure VMs and Load Balancer
- Azure Storage with private access
- Azure Monitor and Log Analytics
- Azure Backup
- Bicep infrastructure automation

We'll implement the capstone in stages to stay within the ₹1,000 monthly budget. Expensive components may be demonstrated separately rather than kept running together.

## Phase 7 — AZ-104 Exam Preparation

Duration: Week 12

| Activity                          | Target           |
| --------------------------------- | ---------------- |
| Identity and governance questions | 50               |
| Storage questions                 | 50               |
| Compute questions                 | 60               |
| Networking questions              | 60               |
| Monitoring and backup questions   | 40               |
| Scenario-based questions          | 40               |
| Full-length mock exams            | 3                |
| Weak-area revision                | Based on results |

Total: 300 practice questions, plus three mock exams.

We'll focus on understanding why an answer is correct, not memorizing questions.

## Phase 8 — GitHub, LinkedIn & Resume Portfolio

This phase runs throughout the learning program.

For every project, we'll prepare:

1. `README.md` — Project overview, architecture, prerequisites, and implementation.
2. `architecture.png` — Azure architecture diagram.
3. `commands.md` — Azure CLI, PowerShell, and Bicep commands with explanations.
4. `troubleshooting.md` — Problems, root causes, and resolutions.
5. `interview-questions.md` — Scenario-based questions and answers.
6. `cleanup.md` — Resource deletion and cost verification.
7. LinkedIn post — Technical summary and GitHub link.

## 12-week learning schedule

ROADMAP OVERVIEW

Preparation

Phase 0 — Subscription and budget

Weeks 1–2

Identity & Governance

Weeks 3–4

Storage

Weeks 5–6

Compute & Bicep

Weeks 7–8

Networking

Weeks 9–10

Monitoring & Recovery

Week 11

Capstone

Week 12

Exam preparation

## Additional DevOps skills

Since you also want to showcase Cloud and DevOps engineering skills, we can add optional extensions after the core AZ-104 labs:

- Terraform modules and reusable infrastructure
- GitHub Actions and Azure DevOps Pipelines
- Azure Key Vault with managed identities
- Spring Boot application deployment
- Containerization and AKS fundamentals
- Prometheus, Grafana, and Application Insights

These are valuable for interviews but are not all required for AZ-104. We'll keep the certification syllabus as the priority.

Our first milestone: Complete Phase 0 — Azure subscription, Cost Management, budget alerts, naming conventions, and GitHub repository setup. Then begin Project 01 — Enterprise Identity & RBAC Management.

We'll proceed one lab at a time, with detailed explanations, Azure Portal steps, commands, validation, troubleshooting, and GitHub documentation.
# AZ-104 — Phase 00: Azure Foundations

This page is the entry point for the seven foundational labs in my AZ-104 learning journey. It brings the lab objectives and available evidence together; each lab's README contains its detailed walkthrough and commands.

> **Before sharing publicly:** review screenshots for personal information and redact names, email addresses, tenant or subscription identifiers, and billing details. Some existing screenshots still contain account information.

## Learning path

| Lab | Focus | Evidence / outcome |
| --- | --- | --- |
| [Lab 01 — Subscription and Microsoft Entra tenant](<./lab-01-subscription-tenant/README.md>) | Explore the subscription, tenant, users, and current access | Portal and CLI exploration; the credit screenshot is a point-in-time record |
| [Lab 02 — Resource group management](<./Lab-02%20%E2%80%94%20Azure%20Resource%20Group%20Management/README.md>) | Create, tag, inspect, and validate a resource group | `rg-az104-dev-cin-001` in Central India with tags |
| [Lab 03 — Cloud Shell and Azure CLI](<./Lab-03-cloud-shell-cli/README.md>) | Use Azure CLI, output formats, and JMESPath queries | Cloud Shell and resource-group command evidence |
| [Lab 04 — Azure RBAC fundamentals](<./Lab-04-Azure%20RBAC%20Fundamentals/README.md>) | Apply least-privilege, group-based role assignments | Developer group as Contributor; operations and audit groups as Readers |
| [Lab 05 — Cost management and budget alerts](<./Lab-05-Azure%20Cost%20Management-Budget%20Alerts/README.md>) | Review costs, credits, and budget notifications | A monthly USD 160 budget and actual-cost thresholds are shown in the evidence |
| [Lab 06 — Azure Policy and resource locks](<./Lab-06-Azure%20Policy%20%26%20Resource%20Locks/README.md>) | Review policy definitions and tag governance; protect a resource group | Tag-policy assignment and `CanNotDelete` lock evidence |
| [Lab 07 — Resource cleanup and cost verification](<./Lab-07Azure%20Resource%20Cleanup%20%26%20Cost%20Verification/README.md>) | Practice a safe temporary-resource-group cleanup lifecycle | Lab instructions are available; no Lab 07 screenshots are currently present |

## Phase architecture

The diagram below summarizes the Azure foundations covered in this phase.

![AZ-104 Azure Foundations architecture](<./Architecture/AZ-104%20Azure%20Foundations%20Architecture%20Diagram.png>)

## Lab screenshot gallery

Screenshots are embedded from their existing lab folders. The descriptions identify what each image shows; they do not imply that every optional task in a lab was performed.

### Lab 01 — Subscription and Microsoft Entra tenant

| Subscription overview | Tenant overview |
| --- | --- |
| ![Azure subscription overview](<./lab-01-subscription-tenant/Screenshot/01-subscription-overview.png>) | ![Microsoft Entra tenant overview](<./lab-01-subscription-tenant/Screenshot/02-entra-tenant-overview.png>) |

| Users | Current RBAC access |
| --- | --- |
| ![Microsoft Entra users](<./lab-01-subscription-tenant/Screenshot/03-users-overview.png>) | ![Subscription RBAC access](<./lab-01-subscription-tenant/Screenshot/04-rbac-access.png>) |

| Azure CLI validation | Free-trial credit |
| --- | --- |
| ![Azure CLI validation](<./lab-01-subscription-tenant/Screenshot/05-cli-validation.png>) | ![Free-trial credit snapshot](<./lab-01-subscription-tenant/Screenshot/06-free-credit.png>) |

### Lab 02 — Resource group management

| Resource group | Tag validation |
| --- | --- |
| ![Resource group overview](<./Lab-02%20%E2%80%94%20Azure%20Resource%20Group%20Management/screenshots/01-Resource-group.png>) | ![Resource group tag validation](<./Lab-02%20%E2%80%94%20Azure%20Resource%20Group%20Management/screenshots/02-resource-group-tag-validation.png>) |

### Lab 03 — Cloud Shell and Azure CLI

| Cloud Shell | Subscription information |
| --- | --- |
| ![Azure Cloud Shell](<./Lab-03-cloud-shell-cli/screenshots/01-cloud-shell.png>) | ![Subscription information in Azure CLI](<./Lab-03-cloud-shell-cli/screenshots/02-subscription-info.png>) |

| Resource groups | JMESPath query |
| --- | --- |
| ![Azure CLI resource groups](<./Lab-03-cloud-shell-cli/screenshots/03-resource-groups.png>) | ![Azure CLI JMESPath query](<./Lab-03-cloud-shell-cli/screenshots/04-jmespath-query.png>) |

### Lab 04 — Azure RBAC fundamentals

| Current access | Built-in roles |
| --- | --- |
| ![Current RBAC access](<./Lab-04-Azure%20RBAC%20Fundamentals/screenshot/01-current-rbac-access.png>) | ![Built-in Azure roles](<./Lab-04-Azure%20RBAC%20Fundamentals/screenshot/02-built-in-roles.png>) |

| Security groups | Role assignments |
| --- | --- |
| ![Microsoft Entra security groups](<./Lab-04-Azure%20RBAC%20Fundamentals/screenshot/03-security-groups.png>) | ![Azure RBAC role assignments](<./Lab-04-Azure%20RBAC%20Fundamentals/screenshot/04-role-assignments.png>) |

| CLI validation |
| --- |
| ![Azure RBAC CLI validation](<./Lab-04-Azure%20RBAC%20Fundamentals/screenshot/05-cli-validation.png>) |

### Lab 05 — Cost management and budget alerts

| Free-trial credit | Cost analysis |
| --- | --- |
| ![Free-trial credit and billing overview](<./Lab-05-Azure%20Cost%20Management-Budget%20Alerts/screenshots/01-free-trial-credit.png>) | ![Cost analysis scope](<./Lab-05-Azure%20Cost%20Management-Budget%20Alerts/screenshots/02-cost-analysis.png>) |

| Budget and alert thresholds | Azure CLI cost commands |
| --- | --- |
| ![Budget and actual-cost alert thresholds](<./Lab-05-Azure%20Cost%20Management-Budget%20Alerts/screenshots/03-budget-alerts.png>) | ![Azure CLI cost-management commands](<./Lab-05-Azure%20Cost%20Management-Budget%20Alerts/screenshots/04-Cli-commands.png>) |

### Lab 06 — Azure Policy and resource locks

| Allowed locations policy definition | Tag policy assignment |
| --- | --- |
| ![Allowed locations policy definition](<./Lab-06-Azure%20Policy%20%26%20Resource%20Locks/screenshot/01-azure-policy-definitions.png>) | ![Project tag policy assignment](<./Lab-06-Azure%20Policy%20%26%20Resource%20Locks/screenshot/02-tag-policy-assignment.png>) |

| Resource lock |
| --- |
| ![CanNotDelete resource lock](<./Lab-06-Azure%20Policy%20%26%20Resource%20Locks/screenshot/03-resource-lock.png>) |

**Policy note:** The Allowed locations definition screenshot lists `Deny` as its available effect. Review the definition and its effect before using or describing an assignment as audit-only.

### Lab 07 — Resource cleanup and cost verification

There are no Lab 07 screenshots in the repository yet. The [Lab 07 README](<./Lab-07Azure%20Resource%20Cleanup%20%26%20Cost%20Verification/README.md>) includes the cleanup exercise and verification steps; add screenshots after completing those steps and removing sensitive details.

## Key AZ-104 concepts

- **Identity and authorization:** Microsoft Entra ID identifies users and groups; Azure RBAC controls their access to Azure resources at a defined scope.
- **Organization and automation:** Resource groups and tags organize resources, while Azure CLI and JMESPath support repeatable administration.
- **Cost awareness:** Cost Analysis, budgets, and alerts help monitor spending. Budgets generate notifications; they do not automatically stop resources.
- **Governance and protection:** Azure Policy evaluates or enforces resource requirements; resource locks help prevent accidental changes or deletion.
- **Safe cleanup:** Inventory resources and inspect locks and policy assignments before deleting anything. Verify the result and remember cost data may be delayed.

## Notes

- Resource-group and subscription names, credit balances, and cost data shown in screenshots are snapshots and may no longer be current.
- An empty resource inventory or no reported cost can be a valid result for a lab environment.
- Keep detailed instructions in the individual lab READMEs; this page is the phase overview and evidence index.

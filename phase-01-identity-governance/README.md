# AZ-104 Phase 01: Enterprise Identity & Governance

This phase builds on the Azure foundation from Phase 00 and focuses on identity lifecycle management, least-privilege access, Azure Policy, remediation, and administrative auditing.

## Architecture

The diagram summarizes the identity and access flow, governance scope, monitoring, and the six-lab implementation sequence.

![AZ-104 Phase 01 enterprise identity and governance architecture](<architecture/AZ-104 Enterprise Identity & Governance Infographic.png>)

The labs use Microsoft Entra ID for users and groups, Azure RBAC for resource access, Azure Policy for compliance and remediation, and Azure Activity Log for administrative investigation. The architecture includes development and test resource groups and selective use of resource locks.

## Learning objectives

- Manage a test user's joiner, mover, and leaver lifecycle through Entra group membership.
- Create and assign a custom Azure RBAC role, then validate effective access.
- Troubleshoot authorization failures, inherited access, and overlapping role assignments.
- Build and assign an Azure Policy initiative and review compliance.
- Configure tag inheritance and remediate supported non-compliant resources.
- Investigate Azure control-plane operations using Activity Log.

## Lab guide

| Lab | Focus | Documentation |
| --- | --- | --- |
| 01 | Entra ID user lifecycle: joiner, mover, leaver | [Lab 01 README](lab-01-entra-user-lifecycle/README.md) |
| 02 | Custom Azure RBAC roles and least privilege | [Lab 02 README](lab-02-custom-azure-rbac/README.md) |
| 03 | Effective access and RBAC troubleshooting | [Lab 03 README](lab-03-rbac-troubleshooting/README.md) |
| 04 | Azure Policy initiatives and compliance | [Lab 04 README](lab-04-policy-initiatives/README.MD) |
| 05 | Policy remediation and tag inheritance | [Lab 05 README](lab-05-policy-remediation/README.MD) |
| 06 | Activity Logs and access investigation | [Lab 06 README](lab-06-activity-logs/README.md) |

## Screenshot gallery

The screenshots below are embedded from their existing lab folders and have not been modified.

> **Before publishing:** Some screenshots visibly include account, tenant, or subscription identifiers. Review them and confirm you want those details public before pushing this repository to GitHub.

### Lab 01: Entra ID user lifecycle

![Test user created in Microsoft Entra ID](lab-01-entra-user-lifecycle/screenshots/01-test-user-created.png)

![Development group membership and role assignment](lab-01-entra-user-lifecycle/screenshots/02-development-group-members.png)

![User moved to the Operations group](lab-01-entra-user-lifecycle/screenshots/03-user-moved-to-operations.png)

![Entra user account disabled](lab-01-entra-user-lifecycle/screenshots/04-user-disabled.png)

![Test account sign-in blocked during offboarding validation](lab-01-entra-user-lifecycle/screenshots/05-Account-disable.png)

### Lab 02: Custom Azure RBAC

![Custom resource reader and resource group tag manager role](lab-02-custom-azure-rbac/screenshots/01-custom-role-create.png)

![Custom role assignment to the operations group](lab-02-custom-azure-rbac/screenshots/02-custom-role-assignment.png)

![Tag update using the custom role](lab-02-custom-azure-rbac/screenshots/03-allowed-tag-update.png)

![Resource group creation denied for the restricted identity](lab-02-custom-azure-rbac/screenshots/04-authorization-denied.png)

### Lab 03: RBAC troubleshooting

![Effective role assignments at resource group scope](lab-03-rbac-troubleshooting/Screenshots/01-effective-access.png)

![Authorization failure when attempting an unauthorized operation](lab-03-rbac-troubleshooting/Screenshots/02-authorization-failure.png)

### Lab 04: Policy initiatives

![Test resource group in the Azure portal](lab-04-policy-initiatives/Screenshots/01-test-resource-group.png)

![Allowed locations policy definition and effects](lab-04-policy-initiatives/Screenshots/02-policy-definitions.png)

![Custom governance initiative definition](lab-04-policy-initiatives/Screenshots/03-custom-initiative-created.png)

![Initiative assignment in Azure Policy](lab-04-policy-initiatives/Screenshots/04-initiative-assignment.png)

![Initiative compliance results](lab-04-policy-initiatives/Screenshots/05-policy-compliance.png)

### Lab 05: Policy remediation

![Tags on the governance test resource group](lab-05-policy-remediation/screenshots/01-resource-group-tags.png)

![Modify policy assignment for Project tag inheritance](lab-05-policy-remediation/screenshots/02-modify-policy-assignment.png)

### Lab 06: Activity Logs

![Azure Activity Log filtered to the test resource group](lab-06-activity-logs/Screenshots/01-activity-log-overview.png)
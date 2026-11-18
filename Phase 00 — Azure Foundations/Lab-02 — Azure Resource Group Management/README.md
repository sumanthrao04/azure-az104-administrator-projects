# **AZ-104 — Phase 00: Lab 01 Completion & Lab 02**

Great, you've completed the Lab 01 exercises. We'll now document your work for GitHub and move to the next hands-on lab.

Our workflow will be:

1. **Validate Lab 01** — Check the expected Azure configuration.
2. **Prepare GitHub documentation** — Final README and Git commit.
3. **Start Lab 02** — Create and manage Azure resource groups using Azure Portal and Azure CLI.
4. **Practice troubleshooting** — Understand errors you may encounter in real Azure environments.

## **1. Lab 01 — Validation**

Since I haven't seen your final command outputs or screenshots, I can validate the expected configuration, but I cannot independently confirm every task was completed successfully.

| Validation         | Expected result               |
| ------------------ | ----------------------------- |
| Azure subscription | Active                        |
| Subscription offer | Azure Free Trial              |
| Microsoft Entra ID | Default tenant accessible     |
| Users and groups   | Pages accessible              |
| Azure RBAC         | Able to inspect your access   |
| Azure CLI          | Commands execute successfully |
| Resource groups    | Listing works, even if empty  |
| Azure credit       | Remaining balance visible     |
| Trial expiration   | Date verified in Azure Portal |

Your earlier screenshot confirmed that your Azure Free Account was active with **$200 credit remaining at that time**.

### **Run these final validation commands**

Open Azure Cloud Shell (Bash):

```
az account show \
  --query "{Name:name,State:state,IsDefault:isDefault}" \
  -o table
```

```
az group list \
  --query "[].{Name:name,Location:location}" \
  -o table
```

```
az cloud show --query name -o tsv
```

Expected cloud name: `AzureCloud`.

If these commands execute successfully, your subscription and Azure CLI access are working. They do not independently validate the Entra ID, RBAC, or billing exercises.

## **2. Lab 01 — Final GitHub README**

Create the following file in your repository:

`phase-00-azure-foundations/lab-01-subscription-tenant/README.md`

# Lab 01 — Azure Subscription & Microsoft Entra Tenant Exploration

## Project Overview

This lab is part of my **Microsoft Azure Administrator (AZ-104) hands-on learning journey**.

The objective is to understand Azure's resource hierarchy, explore Microsoft Entra ID, verify subscription access, and become familiar with Azure administration tools.

## Lab Environment

| Component                  | Configuration                                            |
| -------------------------- | -------------------------------------------------------- |
| Cloud Platform             | Microsoft Azure                                          |
| Subscription Type          | Azure Free Trial                                         |
| Initial Promotional Credit | USD 200                                                  |
| Azure Environment          | Azure Public Cloud                                       |
| Tools                      | Azure Portal, Azure CLI, Cloud Shell                     |
| Certification              | AZ-104                                                   |
| Lab Status                 | Exercises completed; final evidence verification pending |

## Architecture

```
Microsoft Entra ID Tenant
          |
          | Identity relationship
          v
Azure Free Trial Subscription
          |
          +-- Resource Groups
          |       |
          |       +-- Azure Resources
          |
          +-- Azure RBAC
          |
          +-- Cost Management
```

Management groups are an optional governance layer above subscriptions.

## Concepts Learned

### 1. Microsoft Entra ID

Microsoft Entra ID provides cloud-based identity and access management.

It manages users, groups, applications, and directory roles.

### 2. Azure Subscription

An Azure subscription is a logical boundary for billing, resource management, quotas, and access control.

### 3. Azure Resource Groups

Resource groups organize related Azure resources and provide a scope for access control, policy assignments, and resource lifecycle operations.

### 4. Azure RBAC

Azure Role-Based Access Control determines which identities can perform actions on Azure resources.

Important built-in roles include:

- Owner
- Contributor
- Reader
- User Access Administrator

### 5. Azure Resource Hierarchy

```
Management Group
      |
Subscription
      |
Resource Group
      |
Azure Resource
```

### 6. Azure Cost Management

Azure Cost Management provides tools to monitor resource consumption, analyze spending, and configure budgets and alerts.

## Hands-on Activities

The following exercises were performed during Lab 01:

1. Explored Azure Portal.
2. Inspected the Azure Free Trial subscription.
3. Explored Microsoft Entra ID.
4. Reviewed users, groups, and directory roles.
5. Examined Azure RBAC and access scopes.
6. Explored subscription and resource hierarchy.
7. Reviewed the free-trial credit information.
8. Practiced Azure CLI commands.

## Azure CLI Commands

Check the active subscription:

```
az account show \
  --query "{Name:name,State:state,IsDefault:isDefault}" \
  -o table
```

List subscriptions:

```
az account list \
  --query "[].{Name:name,State:state}" \
  -o table
```

List resource groups:

```
az group list -o table
```

Check the Azure cloud:

```
az cloud show --query name -o tsv
```

## Validation Checklist

- Subscription status verified as Active
- Microsoft Entra tenant inspected
- RBAC permissions inspected
- Azure CLI commands executed successfully
- Remaining trial credit and expiration date checked
- Screenshots captured and sensitive details redacted

## Troubleshooting Knowledge

| Issue                            | Investigation                           |
| -------------------------------- | --------------------------------------- |
| Subscription not visible         | Verify active directory and account     |
| AuthorizationFailed              | Check Azure RBAC permissions            |
| Azure CLI authentication failure | Check login and selected subscription   |
| Cost data unavailable            | Check billing scope and reporting delay |

No specific troubleshooting incident is claimed until documented with actual evidence.

## Key Learnings

This lab introduced the fundamental concepts required to administer Azure resources securely and efficiently.

The main learning areas were Azure identity management, subscription administration, resource hierarchy, access control, and cost awareness.

## Next Lab

**Lab 02 — Azure Resource Group Management**

The next lab covers creating, tagging, inspecting, updating, and safely deleting resource groups using Azure Portal and Azure CLI.

## References

- [Microsoft Learn — AZ-104](https://learn.microsoft.com/en-us/credentials/certifications/resources/study-guides/az-104)
- [Microsoft Learn — Azure RBAC](https://learn.microsoft.com/en-us/azure/role-based-access-control/overview)
- [Microsoft Learn — Resource Groups](https://learn.microsoft.com/en-us/azure/azure-resource-manager/management/overview)

## **3. Git commit for Lab 01**

After adding the README and your redacted screenshots, run these commands from your local Git repository:

```
git add phase-00-azure-foundations/lab-01-subscription-tenant/
```

```
git commit -m "docs(az104): complete phase 00 lab 01 subscription and tenant exploration"
```

```
git push origin main
```

Use your actual branch name if it is not `main`. Ensure the repository has a configured remote before pushing.

# **4. Start Lab 02 — Azure Resource Group Management**

Hands-on Lab

60–90 minutes

## **Lab 02 — Azure Resource Group Management**

Azure service

**Resource Manager**

Resource group cost

**$0**

Region

**Central India**

Subscription

**Free Trial**

## **5. Real-world scenario**

You are working as an Azure Administrator for a company that maintains three environments:

- Development — Used by developers.
- Testing — Used by QA engineers.
- Production — Used for customer-facing applications.

Your organization wants each environment organized separately so that access permissions, governance, costs, and cleanup can be managed independently.

In this lab, we'll create a development resource group first, then practice managing it.

## **6. Architecture — Lab 02**

Only the development resource group will be created in this exercise. Testing and production groups illustrate a common organizational pattern.

## **7. Task 1 — Create a resource group using Azure Portal**

Open [Azure Portal](https://portal.azure.com?utm_source=chatgpt.com).

1. Search for **Resource groups** in the top search bar.
2. Click **+ Create**.
3. Select your Azure Free Trial subscription.
4. Enter the resource group name and region shown below.

| Field          | Value                  |
| -------------- | ---------------------- |
| Subscription   | Azure Free Trial       |
| Resource group | `rg-az104-dev-cin-001` |
| Region         | Central India          |

Next, click **Next: Tags**.

Add these tags:

| Name        | Value        |
| ----------- | ------------ |
| Project     | AZ104        |
| Environment | Dev          |
| Owner       | Personal-Lab |
| ManagedBy   | AzurePortal  |

Click **Review + create**, verify the details, then select **Create**.

**Screenshot 01**

Capture the newly created resource group's Overview page and its tags after hiding sensitive identifiers.

## **8. Task 2 — Validate the resource group using Azure CLI**

Open Azure Cloud Shell in Bash mode.

**Command 1 — List all resource groups**

```
az group list -o table
```

**Command 2 — Verify your resource group**

```
az group show \
  --name rg-az104-dev-cin-001 \
  --query "{Name:name,Location:location,Tags:tags}" \
  -o json
```

Expected output:

```
{
  "Name": "rg-az104-dev-cin-001",
  "Location": "centralindia",
  "Tags": {
    "Project": "AZ104",
    "Environment": "Dev",
    "Owner": "Personal-Lab",
    "ManagedBy": "AzurePortal"
  }
}
```

This is an illustrative expected output; compare it with your actual results.

## **9. Task 3 — Update resource group tags**

In real environments, resource tags may change as projects evolve.

For example, we may want to add a `CostCenter` tag.

Run:

```
az tag update \
  --resource-id "$(az group show --name rg-az104-dev-cin-001 --query id -o tsv)" \
  --operation Merge \
  --tags CostCenter=Learning
```

Now verify:

```
az group show \
  --name rg-az104-dev-cin-001 \
  --query tags \
  -o json
```

You should see the existing tags plus:

```
{
  "CostCenter": "Learning"
}
```

The full output will also contain your other tags.

**Why use&#x20;**`Merge`**?** It adds or updates the specified tag without replacing all existing tags.

## **10. Task 4 — Practice resource group operations**

**Check whether a resource group exists:**

```
az group exists \
  --name rg-az104-dev-cin-001
```

Expected result:

```
true
```

**List resources inside the group:**

```
az resource list \
  --resource-group rg-az104-dev-cin-001 \
  -o table
```

The result should be empty if you haven't deployed any resources.

**Delete the resource group — optional cleanup:**

```
az group delete \
  --name rg-az104-dev-cin-001 \
  --yes
```

This command permanently initiates deletion of the resource group and its contents. **Do not execute it yet** if you want to retain the resource group for Phase 01 or later labs.

## **11. Troubleshooting practice**

| Error or issue             | Likely cause                         | What to check                         |
| -------------------------- | ------------------------------------ | ------------------------------------- |
| `AuthorizationFailed`      | Insufficient permissions             | Subscription IAM                      |
| `ResourceGroupNotFound`    | Incorrect group name or subscription | `az account show` and `az group list` |
| `InvalidResourceGroupName` | Invalid characters or naming         | Resource group name                   |
| Region not available       | Subscription or region restrictions  | Supported locations                   |
| Tags missing               | Tags not applied or replaced         | Resource group Tags page              |

## **12. Lab 02 completion checklist**

**Lab 02 Progress**

8/8

Created rg-az104-dev-cin-001 in Central India

Applied Project, Environment, Owner, and ManagedBy tags

Validated the resource group using Azure CLI

Added CostCenter=Learning using Azure CLI

Verified resource group existence

Listed resources inside the group

Captured redacted screenshots

Recorded any errors and their resolutions



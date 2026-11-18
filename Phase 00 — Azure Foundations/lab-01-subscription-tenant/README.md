# **AZ-104 — Phase 00 | Lab 01: Azure Subscription & Microsoft Entra Tenant Exploration**

Beginner · Hands-on

Duration: 60–90 minutes

## **Understand Your Azure Environment**

Subscription

**Azure Free Trial**

Available credit

**$200**

Estimated lab cost

**$0**

AZ-104 domain

**Identity & Governance**

## **1. Real-world scenario**

Imagine you've joined a company as an **Azure Cloud Administrator**.

Your organization has created a new Azure subscription to host development and production workloads.

Before deploying any applications, your manager asks you to:

- Verify the Azure subscription and its status.
- Identify the Microsoft Entra tenant.
- Understand the relationship between tenants, subscriptions, and resource groups.
- Inspect the current access permissions.
- Verify available credit and trial expiration.
- Validate the environment using Azure CLI.

**Your responsibility:** Understand the existing Azure environment before provisioning resources.

## **2. Architecture — What we're exploring**

Conceptual architecture — no resource groups will be created in Lab 01.

## **3. Concepts to understand before starting**

| Component          | Explanation                                       | Example                         |
| ------------------ | ------------------------------------------------- | ------------------------------- |
| Microsoft Entra ID | Manages users, groups, and identities             | Company identity directory      |
| Tenant             | Dedicated Entra ID instance                       | Organization's directory        |
| Subscription       | Boundary for billing, quotas, and resource access | Azure Free Trial                |
| Management Group   | Groups subscriptions for governance               | Enterprise management hierarchy |
| Resource Group     | Logical container for resources                   | `rg-az104-dev-cin-001`          |
| Azure Resource     | A deployed service                                | VM, Storage, VNet               |
| Azure RBAC         | Controls access to Azure resources                | Reader, Contributor, Owner      |

**Important AZ-104 concept:** A subscription trusts one Microsoft Entra tenant for identity authentication. Azure RBAC permissions can be assigned at management group, subscription, resource group, or individual resource scope.

# **4. Hands-on implementation**

## **Task 1 — Explore your Azure subscription**

Estimated time: 10 minutes

1. Open [Microsoft Azure Portal](https://portal.azure.com?utm_source=chatgpt.com).
2. Search for **Subscriptions** using the top search bar.
3. Click **Subscriptions** and open your Free Trial subscription.
4. On the Overview page, inspect the subscription name, status, subscription ID, directory, and offer.

You should see a subscription with an **Active** status.

Record these details in your private notes:

| Property            | Expected / Action                      |
| ------------------- | -------------------------------------- |
| Subscription name   | Record the displayed name              |
| Subscription status | Active                                 |
| Offer               | Free Trial                             |
| Subscription ID     | Verify, but don't publish              |
| Directory           | Default directory or your Entra tenant |

### **What you learned**

A subscription is where Azure resources are deployed and usage is tracked. A single tenant can have multiple subscriptions.

## **Task 2 — Explore Microsoft Entra ID**

Estimated time: 15 minutes

1. Search for **Microsoft Entra ID** in Azure Portal.
2. Open the **Overview** page.
3. Review the tenant name, tenant ID, primary domain, and license.
4. Navigate to **Users → All users**.
5. Open **Groups → All groups**.
6. Explore **Roles and administrators**.

Do not create additional users or purchase licenses in this lab.

### **Interview concept: Entra ID vs Azure RBAC**

**Microsoft Entra ID** answers: *Who are you?*

**Azure RBAC** answers: *What Azure resources are you allowed to manage?*

For example, a developer may successfully sign in through Entra ID but still receive an authorization error when trying to create a VM without the required Azure RBAC permission.

## **Task 3 — Understand Azure resource hierarchy**

Estimated time: 10 minutes

Explore **Management groups** from Azure Portal if your subscription allows access.

The hierarchy is:

```
Microsoft Entra Tenant
    |
    +-- Root Management Group
          |
          +-- Azure Subscription
                |
                +-- Resource Group
                      |
                      +-- Azure Resources
```

The tenant provides the identity context. Management groups organize subscriptions for governance.

For this lab, you only need to understand the hierarchy. We will create resource groups in Lab 02.

## **Task 4 — Inspect Azure RBAC permissions**

Estimated time: 10 minutes

1. Open **Subscriptions**.
2. Select your Free Trial subscription.
3. Click **Access control (IAM)**.
4. Select **View my access**.
5. Review the roles assigned to your account.

Learn these four important roles:

| Role                      | Permissions                                              |
| ------------------------- | -------------------------------------------------------- |
| Owner                     | Full resource management and role assignment permissions |
| Contributor               | Manage resources but cannot assign Azure RBAC roles      |
| Reader                    | View resources without modifying them                    |
| User Access Administrator | Manage user access to Azure resources                    |

Do not change role assignments yet. We'll implement RBAC practically in Phase 01.

## **Task 5 — Verify your free trial credit**

Estimated time: 10 minutes

Your earlier screenshot showed **$200 in credits remaining**.

Now we need to verify the trial expiration date.

1. Open Azure Portal Home.
2. Click the remaining-credit banner, if available.
3. Alternatively, open **Cost Management + Billing** and look for the Free Trial credit information.
4. Record the remaining credit and exact expiration date.
5. Review any usage already recorded.

The $200 credit is valid for up to 30 days, and remaining credit may be updated with a delay.

## **Task 6 — Validate using Azure CLI**

Estimated time: 15 minutes

Click the **Cloud Shell** icon in the top Azure Portal toolbar.

**Azure Cloud Shell**

Choose **Bash**. If Azure requests a storage account for persistent Cloud Shell storage, review the setup and potential charges first. You can also run these commands from Azure CLI installed on your own computer.

**Command 1 — Check Azure CLI version**

```
az version
```

This confirms the Azure CLI is available.

**Command 2 — Display active subscription**

```
az account show \
  --query "{Name:name,State:state,IsDefault:isDefault}" \
  -o table
```

Expected result resembles:

```
Name              State     IsDefault
----------------  --------  ---------
Free Trial        Enabled   True
```

The subscription name may differ.

**Command 3 — List subscriptions**

```
az account list \
  --query "[].{Name:name,State:state,Default:isDefault}" \
  -o table
```

**Command 4 — List existing resource groups**

```
az group list \
  --query "[].{Name:name,Location:location}" \
  -o table
```

A new account may have no resource groups yet. That's normal.

**Command 5 — Check the current Azure cloud**

```
az cloud show --query name -o tsv
```

For Azure's public cloud, the expected value is:

```
AzureCloud
```

**Command 6 — Display available Azure regions**

```
az account list-locations \
  --query "[?name=='centralindia'].{Name:name,DisplayName:displayName}" \
  -o table
```

This helps verify that Central India is recognized in your Azure environment. Individual services and SKUs can still have regional restrictions.

## **5. Troubleshooting scenarios**

| Scenario                           | Investigation                           | Resolution                                         |
| ---------------------------------- | --------------------------------------- | -------------------------------------------------- |
| Subscription not visible           | Check account and directory             | Switch to the correct tenant                       |
| `AuthorizationFailed`              | Inspect IAM role assignments            | Verify appropriate permissions                     |
| Cloud Shell initialization fails   | Check setup requirements                | Use local Azure CLI if needed                      |
| Azure CLI shows wrong subscription | Check default subscription              | Use `az account set` with the correct subscription |
| Cost data not showing              | Check billing scope and reporting delay | Recheck after data becomes available               |

## **6. GitHub documentation**

For Lab 01, use:

```
phase-00-azure-foundations/
|
|-- lab-01-subscription-tenant/
|   |-- README.md
|   |
|   |-- screenshots/
|       |-- 01-subscription-overview.png
|       |-- 02-entra-tenant-overview.png
|       |-- 03-users-overview.png
|       |-- 04-rbac-access.png
|       |-- 05-cli-validation.png
|       |-- 06-free-credit.png
```

Only add screenshots after performing the corresponding task. Hide subscription IDs, tenant IDs, email addresses, and other sensitive information before uploading.

## **7. AZ-104 interview questions**

Try answering these after completing the lab.

1\. What is the difference between an Azure tenant and a subscription?

2\. Can one Azure tenant have multiple subscriptions?

3\. What is the purpose of a resource group?

4\. What is the difference between Owner and Contributor?

5\. What is the role of management groups?

## **8. Lab 01 completion checklist**

**Lab 01 Progress**

8/8

Verified Azure subscription and status

Explored Microsoft Entra ID tenant

Reviewed users, groups, and roles

Understood Azure resource hierarchy

Inspected subscription RBAC permissions

Verified free credit and expiration date

Executed Azure CLI validation commands

Captured redacted screenshots for GitHub




# **AZ-104 — Phase 00 | Lab 07: Azure Resource Cleanup & Cost Verification**

**Project: Azure Resource Lifecycle Management & Cost Optimization**

This is the final lab in **Phase 00 — Azure Foundations**.

So far, you've worked on Azure subscriptions, resource groups, Cloud Shell, Azure CLI, RBAC, Cost Management, Azure Policy, and resource locks.

Now we'll learn how Azure Administrators safely remove unused infrastructure, verify that resources are no longer consuming money, and prepare the environment for the next project.

Phase 00 · Final Lab

Duration: 1.5–2 hours

Subscription

**Azure Free Trial**

Target resource group

rg-az104-dev-cin-001

New resource cost

## **$0**

AZ-104 domain

**Governance & Administration**

This lab uses existing Azure resources. It does not require deploying any additional billable services.

## **1. Real-world scenario**

Imagine you're an Azure Administrator managing a company's development environment.

The development team has completed testing an application, and your manager has requested that unused Azure infrastructure be removed to reduce costs.

However, the environment contains:

- Resource groups and resource tags.
- Azure RBAC role assignments.
- Azure Policy assignments.
- Resource locks.
- Potentially billable resources such as VMs, disks, and storage accounts.

Your responsibility is to **inspect dependencies, identify what can be deleted, remove protection controls only when authorized, clean up resources, and verify the result**.

In this lab, we'll use a separate temporary resource group for the destructive test, so your existing development resource group remains available for Phase 01.

## **2. Architecture — Before and after cleanup**

We'll retain your main development resource group and delete only the temporary cleanup test group.

## **3. Topics covered**

| Topic                 | What you'll learn                           |
| --------------------- | ------------------------------------------- |
| Resource inventory    | Identify existing Azure resources           |
| Resource dependencies | Understand what may be affected by deletion |
| Resource locks        | Understand deletion protection              |
| Azure CLI             | Automate inspection and cleanup             |
| Azure Policy          | Understand policy assignment scope          |
| RBAC                  | Understand access required for deletion     |
| Cost Management       | Review remaining resource spending          |
| Resource lifecycle    | Create, manage, validate, and delete        |

# **4. Hands-on implementation**

## **Task 1 — Inventory existing Azure resources**

Estimated time: 15 minutes

Before deleting anything, understand what exists in your subscription.

Open [Azure Portal](https://portal.azure.com?utm_source=chatgpt.com) and launch **Cloud Shell (Bash)**.

### **Command 1 — Verify the active subscription**

```
az account show \
  --query "{Name:name,State:state,Default:isDefault}" \
  -o table
```

Confirm you're using your Azure Free Trial subscription.

### **Command 2 — List all resource groups**

```
az group list \
  --query "[].{Name:name,Location:location}" \
  -o table
```

### **Command 3 — List resources across the subscription**

```
az resource list \
  --query "[].{Name:name,Type:type,ResourceGroup:resourceGroup,Location:location}" \
  -o table
```

### **Command 4 — Inspect the development resource group**

```
az resource list \
  --resource-group rg-az104-dev-cin-001 \
  -o table
```

**Expected outcome:** You should be able to identify whether the development resource group contains any resources.

Your Entra security groups are directory objects and won't appear as ordinary resources inside the resource group.

**Screenshot:** `01-resource-inventory.png`

## **Task 2 — Inspect resource locks and policy assignments**

Estimated time: 15 minutes

In Lab 06, we planned to configure a `CanNotDelete` lock.

Let's check whether it exists.

### **List resource locks**

```
az lock list \
  --resource-group rg-az104-dev-cin-001 \
  -o table
```

If the lock was created successfully, you should see something similar to:

```
Name                     Level
-----------------------  -------------
lock-az104-dev-delete     CanNotDelete
```

### **List Azure Policy assignments**

```
RG_SCOPE=$(az group show \
  --name rg-az104-dev-cin-001 \
  --query id \
  -o tsv)

az policy assignment list \
  --scope "$RG_SCOPE" \
  -o table
```

### **Important learning**

Azure resource locks and Azure Policy serve different purposes.

**Resource locks** protect resources from certain management operations.

**Azure Policy** evaluates or enforces organizational configuration requirements.

A policy assignment or lock can affect resource cleanup, so these controls must be inspected before deletion.

## **Task 3 — Create a temporary cleanup resource group**

Estimated time: 15 minutes

We'll create a separate resource group rather than deleting your existing development environment.

### **Step 1 — Create the resource group**

```
az group create \
  --name rg-az104-cleanup-cin-001 \
  --location centralindia \
  --tags Project=AZ104 Environment=Test Purpose=CleanupLab
```

### **Step 2 — Verify creation**

```
az group show \
  --name rg-az104-cleanup-cin-001 \
  --query "{Name:name,Location:location,Tags:tags}" \
  -o json
```

### **Step 3 — Check existence**

```
az group exists \
  --name rg-az104-cleanup-cin-001
```

Expected output:

```
true
```

### **Why are we doing this?**

Creating an empty resource group does not itself incur Azure usage charges.

This allows us to practice the resource lifecycle without deploying VMs, storage accounts, or networking resources.

**Screenshot:** `02-cleanup-resource-group.png`

## **Task 4 — Protect the test resource group with a lock**

Estimated time: 15 minutes

Now we'll simulate a protected environment.

### **Step 1 — Create a CanNotDelete lock**

```
az lock create \
  --name lock-az104-cleanup-delete \
  --lock-type CanNotDelete \
  --resource-group rg-az104-cleanup-cin-001 \
  --notes "Prevent accidental deletion during AZ-104 lab"
```

### **Step 2 — Verify the lock**

```
az lock list \
  --resource-group rg-az104-cleanup-cin-001 \
  -o table
```

Expected result: `CanNotDelete`.

### **Step 3 — Understand the protection**

The lock prevents deletion of the resource group while it remains in place.

Even users with Owner permissions must remove the lock before deleting the protected group.

**Screenshot:** `03-resource-lock.png`

## **Task 5 — Remove the lock safely**

Estimated time: 10 minutes

After verifying the lock, remove it from the temporary group.

```
az lock delete \
  --name lock-az104-cleanup-delete \
  --resource-group rg-az104-cleanup-cin-001
```

Verify:

```
az lock list \
  --resource-group rg-az104-cleanup-cin-001 \
  -o table
```

The temporary group's lock should no longer appear.

**Interview point:** Deleting a locked resource requires resolving the lock, not simply granting more RBAC permissions.

## **Task 6 — Delete the temporary resource group**

Estimated time: 15 minutes

Before deletion, verify that the group contains no resources.

```
az resource list \
  --resource-group rg-az104-cleanup-cin-001 \
  -o table
```

The output should be empty.

### **Delete the group**

This is a destructive operation. Run it only after confirming the exact group name and that it contains nothing important.

```
az group delete \
  --name rg-az104-cleanup-cin-001 \
  --yes
```

Azure may take some time to complete deletion.

### **Verify deletion**

```
az group exists \
  --name rg-az104-cleanup-cin-001
```

Expected:

```
false
```

**Screenshot:** `04-resource-group-deleted.png`

## **Task 7 — Verify Azure spending after cleanup**

Estimated time: 15 minutes

Navigate to:

**Azure Portal → Cost Management → Cost Analysis**

Review the subscription's current spending.

Check:

- Remaining promotional credit.
- Costs by service.
- Costs by resource group.
- Any remaining billable resources.
- Your existing $160 budget and alert settings.

Your previous budget screenshot showed $0.00 evaluated spend at that time, but the current balance may have changed.

### **Important cost concepts**

Deleting a resource group does not erase historical costs.

Cost Analysis may continue to display charges from previously deleted resources.

Also, cost data is not necessarily updated immediately after deletion.

**Screenshot:** `05-cost-analysis-after-cleanup.png`

## **Task 8 — Create an Azure cleanup verification script**

This script checks the active subscription, lists resources, and verifies whether the temporary resource group still exists.

Create a file:

```
nano azure-cleanup-check.sh
```

Paste:

```

#!/usr/bin/env bash
set -euo pipefail

RG_NAME="rg-az104-cleanup-cin-001"

echo "==================================="
echo " AZ-104 Resource Cleanup Report"
echo "==================================="

echo ""
echo "1. Active Subscription"
az account show \
  --query "{Name:name,State:state}" \
  -o table

echo ""
echo "2. Resource Groups"
az group list \
  --query "[].{Name:name,Location:location}" \
  -o table

echo ""
echo "3. Cleanup Resource Group Status"

if [ "$(az group exists --name "$RG_NAME")" = "true" ]; then
    echo "Resource group still exists."

    echo "Resources:"
    az resource list \
      --resource-group "$RG_NAME" \
      -o table

    echo "Locks:"
    az lock list \
      --resource-group "$RG_NAME" \
      -o table
else
    echo "Cleanup resource group does not exist."
fi

echo ""
echo "4. Remaining Azure Resources"
az resource list \
  --query "[].{Name:name,Type:type,RG:resourceGroup}" \
  -o table

echo ""
echo "Cleanup verification completed."

```

Save and exit.

Make it executable:

```
chmod +x azure-cleanup-check.sh
```

Run:

```
./azure-cleanup-check.sh
```

Generate a report:

```
./azure-cleanup-check.sh > cleanup-report.txt
```

Review the report before uploading it to GitHub.

## **5. Troubleshooting scenarios**

| Scenario                       | Root cause                                | Resolution                               |
| ------------------------------ | ----------------------------------------- | ---------------------------------------- |
| `ScopeLocked`                  | CanNotDelete lock exists                  | Inspect and remove the authorized lock   |
| `AuthorizationFailed`          | Insufficient permissions                  | Review Azure RBAC                        |
| `ResourceGroupNotFound`        | Group already deleted or wrong name       | Check active subscription and group name |
| Deletion takes time            | Azure operation is asynchronous           | Wait and verify provisioning state       |
| Resources remain after cleanup | Resources exist outside the deleted group | Review subscription-wide inventory       |
| Cost still appears             | Historical usage or reporting delay       | Review Cost Analysis over time           |

### **Production interview scenario**

A development resource group cannot be deleted even though the engineer has Contributor access.

**How would you troubleshoot?**

1. Confirm the correct subscription and resource group.
2. Inspect resource locks.
3. Check inherited locks from higher scopes.
4. Verify RBAC permissions.
5. Inspect Azure Policy and deployment restrictions.
6. Review the exact Azure error.
7. Remove the lock only after approval.
8. Delete the group and verify the result.
9. Check for remaining resources and costs.

## **6. GitHub project structure**

```
phase-00-azure-foundations/
|
|-- lab-07-resource-cleanup/
|   |-- README.md
```

### **Git commit**

After completing the lab:

```
git add phase-00-azure-foundations/lab-07-resource-cleanup/

git commit -m "feat(az104): implement Azure resource cleanup and cost verification"

git push origin main
```

## **7. AZ-104 interview questions**

1\. What happens when you delete an Azure resource group?

2\. Can a Contributor delete a resource group?

3\. What happens if a CanNotDelete lock exists?

4\. Does stopping a VM stop all Azure charges?

5\. Does deleting a resource group remove historical billing records?

6\. How can you verify resource group deletion?

## **8. Lab 07 completion checklist**

**Lab 07 Progress**

0/11

Verified active Azure subscription

Listed resource groups and resources

Inspected existing locks and policy assignments

Created temporary cleanup resource group

Created and verified CanNotDelete lock

Removed the temporary resource lock

Deleted temporary resource group

Verified resource group no longer exists

Reviewed Azure Cost Analysis and remaining credit

Executed cleanup verification script

Prepared sanitized GitHub documentation




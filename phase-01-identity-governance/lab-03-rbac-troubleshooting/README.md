# **AZ-104 — Phase 01 | Lab 03: Azure RBAC Troubleshooting & Effective Access Validation**

**Project: Enterprise Azure Access Troubleshooting & Security Validation**

In Lab 02, we designed a custom Azure RBAC role to allow resource reading and tag management without granting full Contributor access.

Now we'll investigate how Azure evaluates permissions, why authorization failures occur, and how to troubleshoot access issues in a real enterprise environment.

This lab is especially useful for **AZ-104 certification preparation and Azure Cloud/DevOps Engineer interviews**.

Advanced · Troubleshooting Lab

Duration: 2–3 hours

Subscription

**Azure Free Trial**

Estimated additional cost

**$0**

Target resource group

rg-az104-dev-cin-001

AZ-104 domain

**Identity & Governance**

**Prerequisites:** Your custom role and `az104-custom-ops-group` from Lab 02, plus an authorized test user without inherited Owner or Contributor access.

## **1. Real-world scenario**

You're an Azure Administrator supporting a development and operations team.

The operations engineer reports:

> "I can log in to Azure Portal and view the development resource group, but I cannot perform some operations. Other operations succeed even though I was assigned a restricted custom role."

Your manager asks you to investigate and explain the behavior without unnecessarily increasing the user's permissions.

### **What we'll troubleshoot**

| Scenario | Problem                                                  | Learning objective               |
| -------- | -------------------------------------------------------- | -------------------------------- |
| 01       | User can sign in but cannot access a resource            | Authentication vs authorization  |
| 02       | Custom role permits reading but blocks resource creation | Least privilege                  |
| 03       | User has multiple RBAC assignments                       | Additive permissions             |
| 04       | User receives unexpected access                          | Role inheritance                 |
| 05       | Tag update fails                                         | Missing action vs Policy vs Lock |
| 06       | Permissions don't update immediately                     | Token and RBAC propagation       |

## **2. Architecture — What we're validating**

Conceptual authorization flow. Azure also evaluates deny assignments, applicable policy, locks, and other constraints before completing operations.

## **3. Important AZ-104 concepts**

### **Authentication vs authorization**

| Authentication                      | Authorization                           |
| ----------------------------------- | --------------------------------------- |
| Verifies identity                   | Determines permissions                  |
| Microsoft Entra ID                  | Azure RBAC                              |
| Example: User signs in successfully | Example: User can read a resource group |
| Failure: Invalid credentials        | Failure: `AuthorizationFailed`          |

A successful Azure sign-in does not mean the user has permission to manage Azure resources.

### **How Azure RBAC permissions work**

Azure RBAC considers role assignments at the target scope and inherited parent scopes.

```
Management Group
       |
       v
Subscription
       |
       v
Resource Group
       |
       v
Resource
```

A user can receive permissions through direct assignments or group membership.

**Important:** Multiple RBAC role assignments are generally additive. Assigning a restrictive custom role does not remove broader permissions granted by another role.

### **Common access-related errors**

| Error                        | Typical meaning                                                         |
| ---------------------------- | ----------------------------------------------------------------------- |
| `AuthorizationFailed`        | Identity lacks permission for the requested operation                   |
| `ResourceGroupNotFound`      | Resource group does not exist or is not visible in the selected context |
| `RequestDisallowedByPolicy`  | Azure Policy blocks the request                                         |
| `ScopeLocked`                | Resource lock prevents the operation                                    |
| `InvalidAuthenticationToken` | Authentication token is invalid or unsuitable                           |

# **4. Hands-on implementation**

We'll complete seven troubleshooting exercises. No paid Azure resources need to be deployed.

## **Task 1 — Prepare the test environment**

**Estimated time:** 15 minutes

First, confirm the resources from Lab 02.

| Resource              | Expected configuration                     |
| --------------------- | ------------------------------------------ |
| Development RG        | `rg-az104-dev-cin-001`                     |
| Custom security group | `az104-custom-ops-group`                   |
| Custom role           | `AZ104 Resource Reader and RG Tag Manager` |
| Test user             | Synthetic user from Lab 01                 |
| Role assignment scope | Development resource group                 |

Because your test user was disabled during the Leaver exercise, temporarily enable that synthetic account for this lab if you haven't already.

### **Verify group membership**

Open **Microsoft Entra ID → Groups → az104-custom-ops-group → Members**.

Confirm the test user is a member.

The test user should not belong to `az104-dev-group`, which has Contributor access, or another group granting unnecessary permissions.

### **Check the custom role**

In Azure Cloud Shell (Bash):

```
az role definition list \
  --name "AZ104 Resource Reader and RG Tag Manager" \
  --query "[].{Name:roleName,Type:roleType}" \
  -o table
```

Expected role type: `CustomRole`.

## **Task 2 — Investigate effective access using Azure Portal**

**Estimated time:** 20 minutes

Navigate to:

**Resource groups → rg-az104-dev-cin-001 → Access control (IAM) → Check access**

1. Click **Check access**.
2. Select **User, group, or service principal**.
3. Search for your synthetic test user.
4. Select the user.
5. Review direct and inherited assignments.
6. Expand the relevant custom-role assignment.

Microsoft documents this workflow for inspecting a user's Azure resource access.&#x20;

[image](https://www.google.com/s2/favicons?domain=https://learn.microsoft.com\&sz=32)

Microsoft Learn



### **What to validate**

| Property                 | Expected                 |
| ------------------------ | ------------------------ |
| User                     | Synthetic test user      |
| Group                    | `az104-custom-ops-group` |
| Role                     | Custom RG Tag Manager    |
| Scope                    | `rg-az104-dev-cin-001`   |
| Inherited elevated roles | None for the test user   |

**Screenshot:** `01-effective-access.png`

## **Task 3 — Inspect direct, group, and inherited RBAC assignments using CLI**

**Estimated time:** 20 minutes

### **Step 1 — Define variables**

Run these commands as your administrator identity:

```
RG_NAME="rg-az104-dev-cin-001"

RG_SCOPE=$(az group show \
  --name "$RG_NAME" \
  --query id \
  -o tsv)
```

Retrieve the test user's object ID:

```
TEST_USER_ID=$(az ad user list \
  --filter "displayName eq 'AZ104 Dev User 01'" \
  --query "[0].id" \
  -o tsv)
```

Verify that `TEST_USER_ID` is not empty before proceeding.

### **Step 2 — List assignments for the test user**

```
az role assignment list \
  --assignee "$TEST_USER_ID" \
  --scope "$RG_SCOPE" \
  --include-groups \
  --include-inherited \
  --query "[].{Role:roleDefinitionName,Scope:scope,Principal:principalName}" \
  -o table
```

This includes group-derived and inherited role assignments when they can be resolved.

Microsoft's Azure CLI supports `--include-groups` and `--include-inherited` for this purpose.&#x20;

[image](https://www.google.com/s2/favicons?domain=https://learn.microsoft.com\&sz=32)

Microsoft Learn



### **Step 3 — List direct assignments at resource-group scope**

```
az role assignment list \
  --scope "$RG_SCOPE" \
  --query "[].{Role:roleDefinitionName,Principal:principalName,Type:principalType}" \
  -o table
```

### **Step 4 — Compare inherited assignments**

```
az role assignment list \
  --scope "$RG_SCOPE" \
  --include-inherited \
  --query "[].{Role:roleDefinitionName,Principal:principalName,Scope:scope}" \
  -o table
```

You should observe your administrator Owner role inherited from subscription scope, plus the group assignments configured in previous labs.

**Important:** Your administrator's inherited Owner access is separate from the synthetic test user's permissions.

## **Task 4 — Reproduce and investigate an authorization failure**

**Estimated time:** 25 minutes

This exercise demonstrates how a restricted identity behaves when it attempts an unauthorized operation.

### **Step 1 — Sign in as the test user**

Use a separate browser profile or a local Azure CLI session.

Avoid switching the identity in your primary administrator Cloud Shell session.

### **Step 2 — Confirm the active identity**

```
az account show \
  --query "{Name:name,User:user.name}" \
  -o table
```

Verify that the output identifies the synthetic test user.

### **Step 3 — Test a permitted read operation**

```
az group show \
  --name rg-az104-dev-cin-001 \
  --query "{Name:name,Location:location}" \
  -o table
```

Expected: The operation succeeds.

### **Step 4 — Attempt an unauthorized operation**

Use a temporary resource group name:

```
az group create \
  --name rg-az104-rbac-denied-cin-001 \
  --location centralindia
```

Expected: `AuthorizationFailed`, provided the test user has only the restricted custom role and no additional permissions.

### **Step 5 — Analyze the error**

Look for details similar to:

```
AuthorizationFailed:
The client does not have authorization
to perform action
Microsoft.Resources/subscriptions/resourceGroups/write
over the requested scope.
```

The exact error text may differ.

### **What this teaches**

Your custom role does not include:

```
Microsoft.Resources/subscriptions/resourceGroups/write
```

Therefore, the test identity should not be able to create a new resource group.

**Screenshot:** `02-authorization-failure.png`

If the operation unexpectedly succeeds, inspect other role assignments before concluding the custom role is misconfigured. Delete the temporary group only after verifying it is empty and you are using an authorized administrator account.

## **Task 5 — Troubleshoot tag-update permissions**

**Estimated time:** 20 minutes

Now we'll test the Tags API permission granted in Lab 02.

### **Step 1 — Set the target resource group ID**

As the test user:

```
RG_SCOPE=$(az group show \
  --name rg-az104-dev-cin-001 \
  --query id \
  -o tsv)
```

### **Step 2 — Update a tag**

```
az tag update \
  --resource-id "$RG_SCOPE" \
  --operation Merge \
  --tags RBACValidation=Successful
```

### **Step 3 — Verify**

```
az group show \
  --name rg-az104-dev-cin-001 \
  --query tags.RBACValidation \
  -o tsv
```

Expected:

```
Successful
```

### **If the update fails**

Investigate in this order:

1. Verify the test user is signed in.
2. Check group membership and the custom role assignment.
3. Inspect the custom role's `Actions`.
4. Check for Azure Policy restrictions.
5. Check for ReadOnly locks.
6. Review the exact error and denied operation.

A `CanNotDelete` lock generally does not prevent tag updates, but a ReadOnly lock can block management-plane write operations.

**Screenshot:** `03-tag-update-validation.png`

## **Task 6 — Investigate unexpected permissions**

**Estimated time:** 20 minutes

In real organizations, users sometimes have broader permissions than expected because they belong to multiple security groups.

We'll demonstrate this without assigning Owner.

### **Step 1 — Review the current state**

Your synthetic test user should currently receive the custom role through:

`az104-custom-ops-group`

### **Step 2 — Temporarily add Reader access**

As an administrator, add the test user to the existing `az104-ops-group`.

That group already has Reader access to the development resource group.

### **Step 3 — Inspect effective access**

Run the CLI role-assignment query from Task 3 again.

You should see both:

- Custom RG Tag Manager
- Reader

### **Step 4 — Understand the result**

The user receives the combined permissions of the applicable roles.

Reader adds broad resource-reading permissions. The custom role adds the specified Tags API operations.

### **Step 5 — Restore the original configuration**

Remove the test user from `az104-ops-group`.

Keep the custom group membership for the next troubleshooting exercise.

**Screenshot:** `04-multiple-role-assignments.png`

## **Task 7 — Investigate RBAC propagation delays**

**Estimated time:** 15 minutes

Azure RBAC changes are not always immediately effective.

Microsoft notes that role-assignment changes can take up to approximately 10 minutes to become effective, and signing out and signing back in can help refresh credentials. Some identity scenarios take longer.&#x20;

[image](https://www.google.com/s2/favicons?domain=https://learn.microsoft.com\&sz=32)

Microsoft Learn



### **Exercise**

1. Remove the test user from the temporary Reader group, if not already done.
2. Refresh the Azure Portal session.
3. Sign out and sign back in as the test user.
4. Inspect effective access again.
5. Verify that the expected custom role remains.
6. Record whether the permission change was immediate or delayed.

### **Interview takeaway**

When an RBAC change appears ineffective, investigate propagation and credential caching before adding more permissions.

## **5. Troubleshooting decision flow**

How to investigate an Azure access issue

**User receives access error**

**Check identity, tenant, and subscription**

**Check group membership and role assignments**

**Check scope, inheritance, and required action**

**Check Policy, locks, and deny assignments**

**Correct the cause → Retest → Document**

## **6. Cleanup**

After validating the exercises:

- Remove temporary membership in `az104-ops-group`.
- Remove the `RBACValidation` test tag.
- Verify that `rg-az104-rbac-denied-cin-001` was not created; if it exists, inspect it before deleting.
- Keep the custom role and `az104-custom-ops-group` until Lab 03 documentation is finished.
- Disable the synthetic test user again when testing is complete.

To remove the test tag using an authorized identity:

```
az tag update \
  --resource-id "$RG_SCOPE" \
  --operation Delete \
  --tags RBACValidation=Successful
```

Do not remove your subscription Owner role or modify unrelated security groups.

## **7. GitHub documentation**

Use the following structure:

```
phase-01-identity-governance/
|
|-- lab-03-rbac-troubleshooting/
|   |-- README.md
|   |
|   |-- screenshots/
|       |-- 01-effective-access.png
|       |-- 02-authorization-failure.png
```

### **Git commit**

After completing and documenting the exercises:

```
git add phase-01-identity-governance/lab-03-rbac-troubleshooting/

git commit -m "feat(az104): validate effective RBAC access and troubleshoot authorization failures"

git push origin main
```

Redact user identifiers, tenant IDs, subscription IDs, and sensitive error details before uploading screenshots or CLI output.

## **8. AZ-104 interview questions**

1\. Why can a user sign in but receive AuthorizationFailed?

2\. How does Azure RBAC inheritance work?

3\. Are Azure RBAC permissions additive?

4\. What does --include-groups do?

5\. Why might a custom-role user unexpectedly have Contributor access?

6\. How do you distinguish an RBAC error from an Azure Policy error?

7\. How do you troubleshoot delayed RBAC changes?

## **9. Lab 03 completion checklist**

**Lab 03 Progress**



Verified the custom role and test security group

Inspected effective access in Azure Portal

Listed direct, group-based, and inherited role assignments

Reproduced an authorization failure

Validated the permitted tag-update operation

Investigated multiple role assignments

Tested permission propagation and refreshed credentials

Restored temporary group memberships

Removed test tags and checked cleanup

Captured redacted screenshots

Documented troubleshooting findings in GitHub

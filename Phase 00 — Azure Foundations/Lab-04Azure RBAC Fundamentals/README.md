# **AZ-104 — Phase 00 | Lab 04: Azure RBAC Fundamentals**

**Project: Enterprise Azure Role-Based Access Control (RBAC)**

In this lab, we'll learn how Azure administrators manage permissions for developers, operations engineers, and auditors using **Azure Role-Based Access Control (Azure RBAC)**.

We'll use your existing Azure Free Trial subscription and the resource group created in Lab 02.

Hands-on Lab

Duration: 2–3 hours

Subscription

**Azure Free Trial**

Target resource group

rg-az104-dev-cin-001

Estimated Azure resource cost

## **$0**

AZ-104 exam domain

**Identity & Governance**

Basic Azure RBAC role assignments do not have a separate usage charge. This lab avoids deploying VMs, databases, or other billable resources.

## **1. Real-world scenario**

Imagine you're an Azure Administrator for a company that has three teams.

| Team        | Requirement                                                   | Azure RBAC role |
| ----------- | ------------------------------------------------------------- | --------------- |
| Development | Create and manage resources in the development resource group | Contributor     |
| Operations  | Monitor and inspect resources without modifying them          | Reader          |
| Audit       | View resources for compliance verification                    | Reader          |

Your organization follows the **principle of least privilege**, meaning users receive only the permissions necessary to perform their responsibilities.

Your manager asks you to implement group-based access control rather than assigning permissions individually to every employee.

### **What we'll implement**

- Inspect your current Azure RBAC permissions.
- Understand role assignments and scope inheritance.
- Create Microsoft Entra security groups, if your directory permissions allow.
- Assign Azure RBAC roles to groups.
- Validate role assignments using Azure Portal and CLI.
- Troubleshoot common authorization failures.
- Document everything in GitHub.

We'll avoid creating unnecessary test users. Testing access with a second identity is optional and requires appropriate directory permissions.

## **2. Architecture — What we're building**

Conceptual access flow. Role assignments are made directly on the development resource group; they do not need to be assigned at subscription scope.

## **3. Azure RBAC concepts for AZ-104**

### **What is Azure RBAC?**

Azure Role-Based Access Control is an authorization system built on Azure Resource Manager that determines **who can perform which actions on which Azure resources**.

A role assignment combines three components:

Principal

Who?

Role

What actions?

Scope

Where?

Security Group + Contributor + Resource Group

An example Azure RBAC role assignment

### **Important built-in roles**

| Role                                    | What it can do                                                       |
| --------------------------------------- | -------------------------------------------------------------------- |
| Owner                                   | Manage resources and assign Azure RBAC roles                         |
| Contributor                             | Manage resources, but cannot assign Azure RBAC roles                 |
| Reader                                  | View resources without changing them                                 |
| User Access Administrator               | Manage access to Azure resources                                     |
| Role Based Access Control Administrator | Manage Azure RBAC role assignments, subject to applicable conditions |

### **Role assignment scopes**

Azure RBAC supports four scopes:

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
Individual Resource
```

Permissions assigned at a parent scope are inherited by its child scopes.

**Example:** If someone has Reader access at the subscription level, they inherit Reader access to the resource groups and resources within that subscription.

An additional Contributor role at one resource group grants broader capabilities there. Azure RBAC role assignments are generally additive, although deny assignments and other controls can restrict effective access.

### **Azure RBAC vs Microsoft Entra roles**

| Azure RBAC                              | Microsoft Entra roles                                |
| --------------------------------------- | ---------------------------------------------------- |
| Manages Azure resource access           | Manages directory administration                     |
| Example: Contributor                    | Example: User Administrator                          |
| Scope: Subscription or resource group   | Scope: Directory or administrative unit              |
| Used to manage VMs, storage, networking | Used to manage users, groups, and directory settings |

A Global Administrator does not automatically have unrestricted Azure resource permissions.

# **4. Hands-on implementation**

## **Task 1 — Inspect your existing RBAC permissions**

**Estimated time:** 15 minutes

1. Open [Azure Portal](https://portal.azure.com?utm_source=chatgpt.com).
2. Search for **Resource groups**.
3. Open `rg-az104-dev-cin-001`.
4. From the left menu, select **Access control (IAM)**.
5. Click **View my access**.
6. Review your assigned roles and their scopes.

Validation

Record which role you have on the resource group and whether it is inherited from the subscription.



## **Task 2 — Explore built-in Azure RBAC roles**

**Estimated time:** 15 minutes

Inside the resource group:

1. Open **Access control (IAM)**.
2. Select the **Roles** tab.
3. Search for `Reader`.
4. Open the role and inspect its permissions.
5. Repeat for `Contributor` and `Owner`.

Pay attention to the following concepts:

- `Actions` — Allowed management-plane operations.
- `NotActions` — Management-plane operations excluded from the allowed actions.
- `DataActions` — Allowed data-plane operations.
- `NotDataActions` — Data-plane operations excluded from allowed data actions.
- `AssignableScopes` — Where a custom role can be assigned.

**Interview point:** Contributor permissions on a storage account do not automatically provide access to read all blob contents. Data-plane access may require a separate Storage Blob Data role.

## **Task 3 — Create Microsoft Entra security groups**

**Estimated time:** 20 minutes

We'll create three groups, provided your account has permission to create security groups.

Navigate to:

**Microsoft Entra ID → Groups → All groups → New group**

Use these configurations:

| Field           | Group 1            | Group 2           | Group 3             |
| --------------- | ------------------ | ----------------- | ------------------- |
| Group type      | Security           | Security          | Security            |
| Group name      | `az104-dev-group`  | `az104-ops-group` | `az104-audit-group` |
| Membership type | Assigned           | Assigned          | Assigned            |
| Purpose         | Development access | Operations access | Audit access        |

For this lab, leave the groups without additional members unless you have an authorized test identity.

If the portal blocks group creation, record the permission or licensing error and continue with the role-inspection exercises.

### **Why security groups?**

In an enterprise environment, assigning permissions to groups is easier to maintain than assigning roles separately to every employee.

When a new developer joins, an authorized administrator can add them to the appropriate group rather than creating new Azure RBAC assignments.

## **Task 4 — Assign Azure RBAC roles**

**Estimated time:** 20 minutes

Before starting, verify that you have permission to create role assignments, such as Owner, User Access Administrator, or an appropriate RBAC administrator role at the target scope.

### **Assign Contributor to the development group**

1. Open **Resource groups → rg-az104-dev-cin-001**.
2. Click **Access control (IAM)**.
3. Click **+ Add → Add role assignment**.
4. Search for **Contributor**.
5. Select the role.
6. Click **Next**.
7. Choose **User, group, or service principal**.
8. Click **Select members**.
9. Select `az104-dev-group`.
10. Review and assign.

Repeat for the other groups:

| Group               | Role        | Scope                  |
| ------------------- | ----------- | ---------------------- |
| `az104-dev-group`   | Contributor | `rg-az104-dev-cin-001` |
| `az104-ops-group`   | Reader      | `rg-az104-dev-cin-001` |
| `az104-audit-group` | Reader      | `rg-az104-dev-cin-001` |

Do not assign these groups roles at the entire subscription level.

### **Validate in Azure Portal**

Open **Access control (IAM) → Role assignments** and verify that all three assignments are present.

Role assignments may take several minutes to propagate.

## **Task 5 — Validate RBAC using Azure CLI**

Open Azure Cloud Shell in Bash mode.

### **Step 1 — Set the resource group variable**

```
RG_NAME="rg-az104-dev-cin-001"
```

### **Step 2 — Retrieve the resource group scope**

```
RG_SCOPE=$(az group show \
  --name "$RG_NAME" \
  --query id \
  -o tsv)
```

### **Step 3 — Confirm the scope**

```
echo "$RG_SCOPE"
```

The scope will have this structure:

```
/subscriptions/<subscription-id>/resourceGroups/rg-az104-dev-cin-001
```

Do not publish your real subscription ID.

### **Step 4 — List role assignments at the resource group scope**

```
az role assignment list \
  --scope "$RG_SCOPE" \
  --query "[].{Role:roleDefinitionName,Principal:principalName,Type:principalType}" \
  -o table
```

### **Step 5 — Include inherited assignments**

```
az role assignment list \
  --scope "$RG_SCOPE" \
  --include-inherited \
  --query "[].{Role:roleDefinitionName,Principal:principalName,Type:principalType}" \
  -o table
```

**What to observe:** Compare direct assignments with inherited permissions from the subscription or management-group hierarchy.

### **Step 6 — Inspect a built-in role definition**

```
az role definition list \
  --name "Reader" \
  --query "[].{Role:roleName,Description:description}" \
  -o table
```

### **Expected outcome**

You should be able to inspect role assignments and understand which identities have access at the resource-group scope.

## **Task 6 — Test least-privilege access**

**Estimated time:** 20 minutes

This is an optional exercise because meaningful access testing requires another authorized identity.

If you have a permitted test user, you can add that user to one of the security groups and sign in separately to test its permissions.

| Test                           | Expected behavior                                 |
| ------------------------------ | ------------------------------------------------- |
| Reader views resource group    | Allowed                                           |
| Reader creates a resource      | Denied                                            |
| Contributor manages resources  | Allowed, subject to policy and other restrictions |
| Contributor assigns RBAC roles | Denied                                            |
| Owner manages role assignments | Allowed, subject to applicable conditions         |

An empty resource group may limit what you can test without creating billable resources.

You can test a Reader's inability to create a resource group without deploying a chargeable VM or storage account, but only within an authorized test subscription and scope.

**Do not change or remove your own administrative access for testing.**

## **Task 7 — Troubleshooting scenarios**

These are common enterprise Azure authorization issues.

| Scenario                                  | Possible cause                             | Resolution                                           |
| ----------------------------------------- | ------------------------------------------ | ---------------------------------------------------- |
| `AuthorizationFailed`                     | Missing role or incorrect scope            | Inspect role assignments                             |
| Add role assignment disabled              | Insufficient role-assignment permissions   | Check your effective access                          |
| Group not visible                         | Wrong tenant or directory permissions      | Verify Entra tenant and group                        |
| Reader cannot modify resource             | Expected read-only behavior                | Use an authorized identity with the required role    |
| Role assignment not effective immediately | Propagation delay                          | Wait and refresh credentials if necessary            |
| Contributor cannot read blob data         | Management-plane vs data-plane permissions | Assign an appropriate storage data role if justified |

### **Real-world scenario**

A developer reports:

> "I can log in to Azure Portal, but I cannot create a virtual machine in the development resource group."

**Troubleshooting approach:**

1. Confirm the correct subscription and resource group.
2. Check the developer's group membership.
3. Inspect role assignments and inherited permissions.
4. Check Azure Policy restrictions.
5. Review the detailed authorization error.
6. Grant only the required permission if authorized.
7. Retest and document the resolution.

## **5. GitHub project structure**

After completing the exercises, document the lab under:

```
phase-00-azure-foundations/
|
|-- lab-04-azure-rbac/
|   |-- README.md
|   |
|   |-- screenshots/
|       |-- 01-current-rbac-access.png
|       |-- 02-built-in-roles.png
|       |-- 03-security-groups.png
|       |-- 04-role-assignments.png
|       |-- 05-cli-validation.png
```

Your `README.md` should explain the business scenario, role assignments, scope inheritance, validation results, and any actual authorization errors you encountered.

### **Git commit**

```
git add phase-00-azure-foundations/lab-04-azure-rbac/

git commit -m "feat(az104): implement group-based Azure RBAC and least-privilege access"

git push origin main
```

Only use this completion commit once the role assignments have been implemented and verified. Otherwise, use a documentation-only commit message.

## **6. AZ-104 interview preparation**

Practice these questions after the lab:

1. What are the three components of an Azure RBAC role assignment?
2. What is the difference between Owner and Contributor?
3. How does Azure RBAC scope inheritance work?
4. What is the difference between Microsoft Entra roles and Azure RBAC roles?
5. Why should permissions be assigned to security groups?
6. What are `Actions`, `NotActions`, and `DataActions`?
7. Why might a Contributor be unable to access blob contents?
8. How would you troubleshoot `AuthorizationFailed`?
9. What is the principle of least privilege?
10. How can you verify role assignments using Azure CLI?

## **7. Lab 04 completion checklist**

**Lab 04 Progress**

10/10

Inspected my existing Azure RBAC permissions

Explored Reader, Contributor, and Owner roles

Created the three Entra security groups, or documented a permission restriction

Assigned roles at resource-group scope, or documented why assignment was unavailable

Validated role assignments using Azure CLI

Compared direct and inherited role assignments

Practiced or documented least-privilege testing

Recorded troubleshooting findings

Captured redacted screenshots

Prepared GitHub documentation

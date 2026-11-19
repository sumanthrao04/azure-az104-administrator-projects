# **AZ-104 — Phase 01 | Lab 02: Custom Azure RBAC Roles**

**Project: Enterprise Least-Privilege Access Using Custom Azure Roles**

You've completed Lab 01 — Microsoft Entra ID User Lifecycle Management, where you practiced the Joiner, Mover, and Leaver processes.

Now we'll move beyond built-in Azure RBAC roles and learn how to create **custom roles with specific permissions**, assign them to security groups, and troubleshoot authorization issues.

Advanced Hands-on Lab

Duration: 2–3 hours

Subscription

**Azure Free Trial**

Estimated additional cost

**$0**

Target resource group

rg-az104-dev-cin-001

AZ-104 domain

**Identity & Governance**

**Prerequisites:** Your existing Azure Free Trial subscription, development resource group, and three security groups from Phase 00.

## **1. Real-world scenario**

Imagine you're working as an Azure Administrator for a company hosting applications in Azure.

The operations team needs to monitor infrastructure and manage resource group tags, but it should not be able to create or delete virtual machines, modify network security groups, or assign permissions.

Your manager asks:

> "Can we give the operations team permission to view Azure resources and manage resource group tags without giving them full Contributor access?"

Yes. We can create a **custom Azure RBAC role**.

### **Business requirements**

| Operation                           | Required permission |
| ----------------------------------- | ------------------- |
| View Azure resources                | Allow               |
| Read resource group information     | Allow               |
| Read and update resource group tags | Allow               |
| Create virtual machines             | Deny by omission    |
| Delete resources                    | Deny by omission    |
| Assign Azure RBAC roles             | Deny by omission    |
| Modify network security groups      | Deny by omission    |

**Important:** Custom roles define allowed operations. They do not create explicit deny rules. If a user has another role granting additional permissions, the effective access can be broader.

## **2. Architecture — What we're building**

A dedicated security group receives a custom role at resource-group scope. Effective permissions still depend on all other applicable role assignments.

## **3. What you'll learn**

| Topic                    | Description                                                         |
| ------------------------ | ------------------------------------------------------------------- |
| Built-in vs custom roles | Why organizations create custom RBAC roles                          |
| Role definitions         | Actions, NotActions, DataActions, AssignableScopes                  |
| Role assignments         | Principal + role + scope                                            |
| Least privilege          | Grant only necessary operations                                     |
| Custom role JSON         | Define permissions using Infrastructure as Code-style configuration |
| Effective permissions    | Understand additive permissions and inheritance                     |
| Troubleshooting          | Investigate `AuthorizationFailed`                                   |
| Azure CLI                | Create, inspect, and assign custom roles                            |

### **Built-in roles vs custom roles**

| Built-in role                           | Custom role                                        |
| --------------------------------------- | -------------------------------------------------- |
| Maintained by Microsoft                 | Created and maintained by your organization        |
| General-purpose permissions             | Specific permissions for business requirements     |
| Example: Contributor                    | Example: RG Tag Manager                            |
| Available within supported Azure scopes | Available only within configured assignable scopes |

### **Important RBAC concepts**

**Actions** define allowed Azure Resource Manager management-plane operations.

**NotActions** subtract operations from a broad set of allowed actions. They are not explicit deny rules.

**DataActions** control supported data-plane operations, such as reading blob contents.

**AssignableScopes** define where a custom role can be assigned. They do not automatically assign the role.

For this lab, we'll avoid wildcards and explicitly grant the permissions needed for resource-group tag administration.

# **4. Hands-on implementation**

## **Task 1 — Inspect existing roles and permissions**

**Estimated time:** 15 minutes

Before creating a custom role, inspect the built-in roles.

### **Azure Portal**

1. Open [Azure Portal](https://portal.azure.com?utm_source=chatgpt.com).
2. Navigate to **Resource groups**.
3. Open `rg-az104-dev-cin-001`.
4. Select **Access control (IAM)**.
5. Open the **Roles** tab.
6. Search for **Reader**.
7. Inspect the permissions.
8. Repeat for **Contributor**.

### **Azure CLI**

```
az role definition list \
  --name "Reader" \
  --query "[].{Name:roleName,Description:description}" \
  -o table
```

```
az role definition list \
  --name "Contributor" \
  --query "[].{Name:roleName,Description:description}" \
  -o table
```

To inspect Contributor's permission definitions:

```
az role definition list \
  --name "Contributor" \
  --query "[].permissions" \
  -o json
```

**Expected outcome:** Understand why Contributor is too permissive for a user who only needs to inspect resources and update tags.

## **Task 2 — Create a dedicated security group**

**Estimated time:** 15 minutes

We already have:

- `az104-dev-group` — Contributor
- `az104-ops-group` — Reader
- `az104-audit-group` — Reader

For this lab, we'll create a separate security group to isolate our custom-role testing.

### **Azure Portal**

Navigate to:

**Microsoft Entra ID → Groups → All groups → New group**

| Field           | Value                       |
| --------------- | --------------------------- |
| Group type      | Security                    |
| Group name      | `az104-custom-ops-group`    |
| Description     | Custom Azure RBAC lab group |
| Membership type | Assigned                    |

Create the group.

**Why not reuse&#x20;**`az104-dev-group`**?** It already has Contributor permissions. Those permissions would invalidate our least-privilege test.

You may reuse your synthetic test user from Lab 01 if it remains available. If the user is disabled, enable it only temporarily for testing and disable it again afterward.

## **Task 3 — Create the custom role JSON**

**Estimated time:** 20 minutes

We'll create a custom role named:

`AZ104 Resource Reader and RG Tag Manager`

### **Permissions**

The role will permit:

- Reading Azure Resource Manager resources.
- Reading resource group metadata.
- Updating resource group tags.

It will not grant VM creation, resource deletion, or RBAC assignment permissions.

### **Step 1 — Open Azure Cloud Shell**

Choose **Bash**.

Create a file:

```
nano custom-rg-tag-manager.json
```

### **Step 2 — Add the custom role definition**

We'll use the current subscription ID dynamically to construct the assignable scope, so you don't need to hard-code private identifiers into GitHub.

```

SUBSCRIPTION_ID=$(az account show --query id -o tsv)

cat > custom-rg-tag-manager.json <<EOF
{
  "Name": "AZ104 Resource Reader and RG Tag Manager",
  "IsCustom": true,
  "Description": "Read Azure resources and manage tags on resource groups.",
  "Actions": [
    "Microsoft.Resources/subscriptions/resourceGroups/read",
    "Microsoft.Resources/subscriptions/resourceGroups/write",
    "Microsoft.Resources/tags/read",
    "Microsoft.Resources/tags/write",
    "Microsoft.Resources/subscriptions/resources/read"
  ],
  "NotActions": [],
  "DataActions": [],
  "NotDataActions": [],
  "AssignableScopes": [
    "/subscriptions/$SUBSCRIPTION_ID"
  ]
}
EOF

```

### **Understand the JSON**

| Field              | Meaning                               |
| ------------------ | ------------------------------------- |
| `Name`             | Custom role name                      |
| `IsCustom`         | Identifies a custom role              |
| `Description`      | Purpose of the role                   |
| `Actions`          | Allowed management-plane operations   |
| `NotActions`       | Excluded operations                   |
| `DataActions`      | Allowed data-plane operations         |
| `AssignableScopes` | Scopes where the role may be assigned |

**Security detail:** `Microsoft.Resources/subscriptions/resourceGroups/write` is broader than tag-only access because it also permits resource-group write operations. For stricter tag-only administration, we should use the narrower tag operations and validate the exact behavior. We'll use the narrower version below for our least-privilege implementation.

### **Step 3 — Replace with a narrower role**

```

SUBSCRIPTION_ID=$(az account show --query id -o tsv)

cat > custom-rg-tag-manager.json <<EOF
{
  "Name": "AZ104 Resource Reader and RG Tag Manager",
  "IsCustom": true,
  "Description": "Read Azure resources and update tags using the Azure Tags API.",
  "Actions": [
    "Microsoft.Resources/subscriptions/resourceGroups/read",
    "Microsoft.Resources/subscriptions/resources/read",
    "Microsoft.Resources/tags/read",
    "Microsoft.Resources/tags/write"
  ],
  "NotActions": [],
  "DataActions": [],
  "NotDataActions": [],
  "AssignableScopes": [
    "/subscriptions/$SUBSCRIPTION_ID"
  ]
}
EOF

```

This second definition is the one we'll deploy.

**Important:** The role is intended for read operations and tag updates through the supported Tags API. It is not a general-purpose Reader role for every Azure resource type. We will verify the exact permitted operations during testing.

## **Task 4 — Create the custom Azure RBAC role**

**Estimated time:** 15 minutes

### **Step 1 — Validate the JSON**

```
python3 -m json.tool custom-rg-tag-manager.json
```

If the JSON is valid, the formatted content will be displayed.

### **Step 2 — Create the role**

```
az role definition create \
  --role-definition custom-rg-tag-manager.json
```

You need permission to create role definitions at the specified assignable scope. Subscription Owner generally has this permission.

### **Step 3 — Verify the custom role**

```
az role definition list \
  --name "AZ104 Resource Reader and RG Tag Manager" \
  -o json
```

### **Step 4 — Review the role in Azure Portal**

Navigate to:

**Subscriptions → Your subscription → Access control (IAM) → Roles**

Search for:

`AZ104 Resource Reader and RG Tag Manager`

Confirm that the role is marked **CustomRole**.

**Screenshot:** `01-custom-role-created.png`

## **Task 5 — Assign the custom role to the security group**

**Estimated time:** 15 minutes

### **Step 1 — Retrieve the resource group scope**

```
RG_NAME="rg-az104-dev-cin-001"

RG_SCOPE=$(az group show \
  --name "$RG_NAME" \
  --query id \
  -o tsv)
```

### **Step 2 — Retrieve the security group ID**

```
GROUP_ID=$(az ad group show \
  --group az104-custom-ops-group \
  --query id \
  -o tsv)
```

### **Step 3 — Assign the custom role**

```
az role assignment create \
  --assignee-object-id "$GROUP_ID" \
  --assignee-principal-type Group \
  --role "AZ104 Resource Reader and RG Tag Manager" \
  --scope "$RG_SCOPE"
```

### **Step 4 — Verify the assignment**

```
az role assignment list \
  --scope "$RG_SCOPE" \
  --role "AZ104 Resource Reader and RG Tag Manager" \
  --query "[].{Role:roleDefinitionName,Type:principalType,Principal:principalName}" \
  -o table
```

**Expected outcome:** The custom role is assigned to `az104-custom-ops-group` at resource-group scope.

**Screenshot:** `02-custom-role-assignment.png`

## **Task 6 — Test custom-role permissions**

**Estimated time:** 25 minutes

This is the most important part of the lab.

We'll validate whether the test user can perform the intended operations without receiving excessive permissions.

### **Step 1 — Prepare the test identity**

Use the synthetic test user from Lab 01.

Before testing:

1. Remove the user from `az104-dev-group`, which grants Contributor.
2. Remove the user from other groups granting overlapping permissions, where appropriate.
3. Add the user to `az104-custom-ops-group`.
4. Confirm the user has no direct or inherited Owner/Contributor assignment.
5. Sign in using the test identity in a separate browser profile or CLI session.

Do not remove your own administrator account's permissions.

### **Step 2 — Verify resource group read access**

As the test user, run:

```
az group show \
  --name rg-az104-dev-cin-001 \
  --query "{Name:name,Location:location}" \
  -o table
```

Expected: The resource group information is displayed.

### **Step 3 — Test tag updates**

Use the Tags API through Azure CLI:

```
RG_SCOPE=$(az group show \
  --name rg-az104-dev-cin-001 \
  --query id \
  -o tsv)

az tag update \
  --resource-id "$RG_SCOPE" \
  --operation Merge \
  --tags AccessTest=CustomRole
```

Expected: The tag update succeeds if the custom role contains the required permissions and no other controls block the operation.

### **Step 4 — Verify the new tag**

```
az group show \
  --name rg-az104-dev-cin-001 \
  --query tags.AccessTest \
  -o tsv
```

Expected:

```
CustomRole
```

### **Step 5 — Test an unauthorized operation**

Attempt to create a temporary empty resource group using the test identity:

```
az group create \
  --name rg-az104-rbac-denied-cin-001 \
  --location centralindia
```

Expected: An authorization failure, provided the user has no other role granting resource-group creation permissions.

If the operation unexpectedly succeeds, investigate the user's other role assignments and delete the temporary group after confirming it is empty.

### **Expected results**

| Operation                    | Expected |
| ---------------------------- | -------- |
| Read resource group          | Allowed  |
| Update tags through Tags API | Allowed  |
| Create resource group        | Denied   |
| Create virtual machine       | Denied   |
| Delete resource group        | Denied   |
| Assign Azure RBAC role       | Denied   |

You do not need to deploy a VM to test the custom role.

**Important:** An operation failing does not automatically prove that RBAC caused the failure. Check the error details to distinguish RBAC restrictions from Azure Policy, resource locks, and subscription restrictions.

## **Task 7 — Troubleshooting scenarios**

| Issue                                    | Possible cause                                    | Investigation                              |
| ---------------------------------------- | ------------------------------------------------- | ------------------------------------------ |
| `AuthorizationFailed` when creating role | Missing role-definition permissions               | Inspect subscription IAM                   |
| Custom role not visible                  | Role propagation delay or wrong scope             | Check assignable scopes                    |
| User cannot read resource group          | Missing required action or assignment             | Inspect custom role and effective access   |
| Tag update denied                        | Missing Tags API permission or policy restriction | Inspect error and role definition          |
| User can create resources unexpectedly   | Another inherited or direct role                  | Review all effective role assignments      |
| Role assignment fails                    | Invalid principal or insufficient authorization   | Verify group ID and assignment permissions |

### **Real-world interview scenario**

An operations engineer has been assigned a custom role to manage resource tags but receives:

`AuthorizationFailed`

**Investigation:**

1. Verify the user is in the correct Entra security group.
2. Confirm the custom role assignment exists.
3. Check the assignment scope.
4. Inspect the custom role's `Actions`.
5. Determine which Azure API operation was denied.
6. Check inherited permissions, Azure Policy, and resource locks.
7. Update the role only if the additional permission is justified.
8. Retest using the same test identity.

This is an important Azure Cloud Engineer troubleshooting scenario.

## **5. Cleanup after the lab**

Remove the temporary test tag:

```
az tag update \
  --resource-id "$RG_SCOPE" \
  --operation Delete \
  --tags AccessTest=CustomRole
```

Perform this with an authorized account if the test identity lacks the required permission.

Keep the custom role and security group until Lab 03, where we'll use them for RBAC troubleshooting.

Do not remove your own subscription Owner assignment.

## **6. GitHub documentation**

```
phase-01-identity-governance/
|
|-- lab-02-custom-azure-rbac/
|   |-- README.md
|   |
|   |
|   |-- screenshots/
|       |-- 01-custom-role-created.png
|       |-- 02-custom-role-assignment.png
|       |-- 03-allowed-tag-update.png
|       |-- 04-authorization-denied.png
```

When committing `custom-rg-tag-manager.json`, replace the actual subscription ID in `AssignableScopes` with a documented placeholder. Keep the generated file with real identifiers out of the public repository.

### **Git commit**

```
git add phase-01-identity-governance/lab-02-custom-azure-rbac/

git commit -m "feat(az104): implement custom RBAC role with least-privilege permissions"

git push origin main
```

## **7. AZ-104 interview questions**

1\. Why create a custom Azure RBAC role?

2\. What are Actions and NotActions?

3\. What is AssignableScopes?

4\. What is the difference between role definition and role assignment?

5\. Can a custom role override an inherited Owner role?

6\. How would you troubleshoot an unexpected authorization failure?

## **8. Lab 02 completion checklist**

**Lab 02 Progress**

12/12

Inspected Reader and Contributor role definitions

Created az104-custom-ops-group

Created the custom role JSON

Deployed and verified the custom role

Assigned the custom role at resource-group scope

Prepared a test user without overlapping elevated permissions

Validated resource group read access

Validated tag update permission

Verified unauthorized resource-group creation is denied

Captured screenshots and documented results

Removed the temporary test tag


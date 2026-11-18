# **AZ-104 — Phase 00 | Lab 06: Azure Policy & Resource Locks**

**Project: Enterprise Azure Governance & Compliance**

You've progressed through Azure subscriptions, resource groups, Cloud Shell, RBAC, and Cost Management. Now we'll implement **Azure governance controls** to enforce organizational standards and protect resources from accidental deletion.

Your Lab 05 screenshot confirms that you've created the following budget:

| Setting           | Verified value            |
| ----------------- | ------------------------- |
| Budget name       | `az104-free-trial-budget` |
| Budget amount     | $160 USD                  |
| Period            | Monthly                   |
| Evaluated spend   | $0.00 at screenshot time  |
| Scope             | Billing account           |
| Budget expiration | December 31, 2026         |

The screenshot does not show the alert recipients or thresholds, so those remain to be verified. Also, the budget expiration is not your Azure Free Trial credit expiration.

Now let's begin Lab 06.

Hands-on Lab

Duration: 2–3 hours

Subscription

**Azure Free Trial**

Resource group

rg-az104-dev-cin-001

New resource cost

## **$0**

AZ-104 domain

**Identity & Governance**

We'll use built-in Azure Policy definitions, resource tags, and resource locks. No VM or storage deployment is required.

## **1. Real-world scenario**

Imagine you're an Azure Administrator managing a company's Azure development environment.

The company has introduced these governance requirements:

- Developers should deploy resources only in approved Azure regions.
- Every resource group must have standard tags.
- Important resource groups must be protected from accidental deletion.
- Administrators must be able to check compliance and investigate policy violations.

Your manager asks you to implement these controls centrally using Azure Policy and resource locks.

**Your responsibility:** Enforce governance without unnecessarily restricting other environments.

## **2. Architecture — What we're building**

Target architecture: Azure Policy evaluates resource configurations, while a resource lock protects the resource group from deletion.

## **3. Topics you'll learn**

| Topic             | What you'll understand                                |
| ----------------- | ----------------------------------------------------- |
| Azure Policy      | Governance and compliance enforcement                 |
| Policy definition | Rule that evaluates Azure resources                   |
| Policy assignment | Applies a definition at a selected scope              |
| Policy initiative | Collection of policy definitions                      |
| Policy effects    | Audit, Deny, Modify, DeployIfNotExists                |
| Compliance        | Compliant, non-compliant, and other evaluation states |
| Resource locks    | CanNotDelete and ReadOnly                             |
| Tags              | Resource classification and governance                |
| RBAC vs Policy    | Permissions vs resource configuration rules           |

### **Azure RBAC vs Azure Policy vs Resource Locks**

| Feature       | Purpose                                                           | Example                         |
| ------------- | ----------------------------------------------------------------- | ------------------------------- |
| Azure RBAC    | Controls who can perform actions                                  | Developer gets Contributor      |
| Azure Policy  | Controls which resource configurations are permitted or evaluated | Audit unapproved locations      |
| Resource lock | Protects resources against certain changes or deletion            | Prevent resource group deletion |

A user with Owner access can generally manage locks, so locks are safeguards against accidental operations rather than absolute protection against a determined administrator.

# **4. Hands-on implementation**

We'll complete five tasks in this lab.

## **Task 1 — Explore Azure Policy**

Estimated time: 15 minutes

1. Open [Azure Portal](https://portal.azure.com?utm_source=chatgpt.com).
2. Search for **Policy**.
3. Open **Azure Policy**.
4. Select **Definitions** from the left menu.
5. Search for `Allowed locations`.
6. Open the built-in policy definition.

Review the following properties:

| Property     | Meaning                                       |
| ------------ | --------------------------------------------- |
| Display name | Human-readable policy name                    |
| Description  | Purpose of the policy                         |
| Policy rule  | Conditions evaluated against resources        |
| Effect       | Action when conditions match                  |
| Parameters   | Configurable policy inputs                    |
| Mode         | Determines which resource types are evaluated |

### **Important policy effects**

| Effect            | Behavior                                                                |
| ----------------- | ----------------------------------------------------------------------- |
| Audit             | Reports non-compliant resources without blocking them                   |
| Deny              | Blocks non-compliant create/update requests                             |
| Modify            | Can add or change supported resource properties                         |
| DeployIfNotExists | Can trigger a deployment when required related configuration is missing |
| Disabled          | Disables policy evaluation for that assignment                          |

Azure Policy includes built-in definitions for allowed regions, tags, and other governance requirements.&#x20;

[image](https://www.google.com/s2/favicons?domain=https://learn.microsoft.com\&sz=32)

Microsoft Learn



**Screenshot:** `01-azure-policy-definitions.png`

## **Task 2 — Assign an Allowed Locations policy**

Estimated time: 25 minutes

We'll restrict the lab's permitted resource deployment regions, starting with **Audit** rather than Deny.

### **Step 1 — Open policy assignments**

Navigate to:

**Azure Policy → Assignments → Assign policy**

### **Step 2 — Configure the assignment**

| Field              | Value                     |
| ------------------ | ------------------------- |
| Scope              | `rg-az104-dev-cin-001`    |
| Policy definition  | Allowed locations         |
| Assignment name    | `az104-allowed-locations` |
| Policy enforcement | Enabled                   |
| Allowed locations  | Central India             |
| Effect             | Audit, if selectable      |
| Exclusions         | None                      |

If the policy's effect is configured through a parameter, select **Audit** under Parameters. Otherwise, review the definition's supported effects before proceeding.

For this lab, **do not select Deny initially**.

### **Step 3 — Create the assignment**

1. Select the resource group scope.
2. Choose the built-in **Allowed locations** definition.
3. Configure the permitted location as Central India.
4. Choose Audit.
5. Review the configuration.
6. Select **Review + create** or **Create**, depending on the portal experience.

### **Step 4 — Validate**

Navigate to:

**Azure Policy → Compliance**

Select your policy assignment and inspect the compliance result.

Policy evaluation is asynchronous, so the status may not appear immediately.

**Important:** The built-in `Allowed locations` policy excludes resource groups. It evaluates applicable resources within the assigned scope. If your resource group is empty, there may be no resources to evaluate.&#x20;

[image](https://www.google.com/s2/favicons?domain=https://learn.microsoft.com\&sz=32)

Microsoft Learn



### **Optional advanced exercise — Deny**

After understanding Audit, you can change the effect to Deny to study enforcement.

With Deny, Azure rejects applicable resource deployments outside the allowed regions.

For your $200 trial, don't deploy a VM merely to test this. We'll encounter Azure Policy enforcement naturally in later compute and storage projects.

## **Task 3 — Implement mandatory tagging governance**

Estimated time: 25 minutes

Your resource group already contains tags such as:

```
Project      = AZ104
Environment  = Dev
CostCenter   = Learning
ManagedBy    = AzurePortal
```

Now we'll use Azure Policy to evaluate tagging requirements.

### **Step 1 — Find a built-in tag policy**

Navigate to:

**Azure Policy → Definitions**

Search for:

`Require a tag on resources`

This policy is useful for checking that deployed resources contain required tags.

### **Step 2 — Create a policy assignment**

| Setting           | Value                       |
| ----------------- | --------------------------- |
| Scope             | `rg-az104-dev-cin-001`      |
| Policy definition | Require a tag on resources  |
| Assignment name   | `az104-require-project-tag` |
| Tag name          | Project                     |
| Effect            | Audit, if supported         |
| Enforcement       | Enabled                     |

If the selected built-in definition does not support Audit, do not apply a Deny effect without reviewing its impact on subsequent labs.

### **Step 3 — Understand the scope**

This policy evaluates applicable **resources inside the resource group**, not necessarily the resource group's own tags.

For a policy specifically evaluating resource group tags, use a built-in definition such as **Require a tag on resource groups**, assigned at subscription scope. We won't use subscription-wide enforcement in this lab.

### **Step 4 — Validate**

Open **Azure Policy → Compliance** and review the assignment.

If the resource group contains no applicable resources, you may see no evaluated resources.

**Screenshot:** `02-tag-policy-assignment.png`

## **Task 4 — Create a CanNotDelete resource lock**

Estimated time: 20 minutes

This is our main resource protection exercise.

### **Step 1 — Open the resource group**

Navigate to:

**Resource groups → rg-az104-dev-cin-001 → Locks**

### **Step 2 — Add a lock**

Click **+ Add**.

| Setting   | Value                                                       |
| --------- | ----------------------------------------------------------- |
| Lock name | `lock-az104-dev-delete`                                     |
| Lock type | Delete / CanNotDelete                                       |
| Notes     | Prevent accidental deletion of AZ-104 development resources |

Select **OK** or **Save**.

### **Step 3 — Validate the lock**

Open Azure Cloud Shell and run:

```
az lock list \
  --resource-group rg-az104-dev-cin-001 \
  -o table
```

Expected lock type: `CanNotDelete`.

### **Step 4 — Understand the lock behavior**

| Operation                     | With CanNotDelete         |
| ----------------------------- | ------------------------- |
| Read resource group           | Allowed                   |
| Update tags                   | Generally allowed         |
| Create supported resources    | Generally allowed         |
| Delete resource group         | Blocked                   |
| Delete resources inside group | Blocked by inherited lock |

Resource locks apply to Azure management-plane operations and are inherited by resources at lower scopes.&#x20;

[image](https://www.google.com/s2/favicons?domain=https://learn.microsoft.com\&sz=32)

Microsoft Learn



**Safe validation:** Verify the lock in Azure Portal and CLI. Do not attempt destructive deletion of your main lab resource group just to demonstrate the lock.

**Screenshot:** `03-resource-lock.png`

## **Task 5 — Compare CanNotDelete and ReadOnly locks**

Estimated time: 15 minutes

Azure supports two management lock levels.

| Feature                       | CanNotDelete                   | ReadOnly                      |
| ----------------------------- | ------------------------------ | ----------------------------- |
| Read resources                | Yes                            | Yes                           |
| Modify resource configuration | Generally yes                  | No                            |
| Delete resources              | No                             | No                            |
| Suitable for                  | Accidental deletion protection | Preventing management changes |

For this lab, retain the **CanNotDelete** lock.

Avoid placing a ReadOnly lock on your main development resource group because it could interfere with later exercises.

### **Azure CLI commands**

**List locks:**

```
az lock list \
  --resource-group rg-az104-dev-cin-001 \
  -o table
```

**Inspect a specific lock:**

```
az lock show \
  --name lock-az104-dev-delete \
  --resource-group rg-az104-dev-cin-001 \
  -o json
```

**Remove the lock when cleanup is required:**

```
az lock delete \
  --name lock-az104-dev-delete \
  --resource-group rg-az104-dev-cin-001
```

Only run the deletion command when you intentionally want to remove the protection.

# **5. Azure CLI policy validation**

Use these commands to inspect your configuration.

### **List policy assignments at the resource group**

```
RG_SCOPE=$(az group show \
  --name rg-az104-dev-cin-001 \
  --query id \
  -o tsv)

az policy assignment list \
  --scope "$RG_SCOPE" \
  -o table
```

### **List built-in policy definitions**

```
az policy definition list \
  --query "[?policyType=='BuiltIn'].{Name:displayName}" \
  -o table
```

### **List resource locks**

```
az lock list \
  --resource-group rg-az104-dev-cin-001 \
  -o table
```

### **Verify resource group tags**

```
az group show \
  --name rg-az104-dev-cin-001 \
  --query tags \
  -o json
```

These commands help verify that the policies, locks, and tags are configured.

## **6. Real-world troubleshooting scenarios**

| Scenario                        | Possible cause                                | Resolution                                   |
| ------------------------------- | --------------------------------------------- | -------------------------------------------- |
| `RequestDisallowedByPolicy`     | Deny policy blocked a deployment              | Inspect policy assignment and allowed values |
| Policy not showing compliance   | Evaluation pending or no applicable resources | Check scope and evaluation status            |
| Cannot delete resource group    | CanNotDelete lock                             | Inspect and remove lock only if authorized   |
| Cannot update resource          | ReadOnly lock                                 | Inspect inherited locks                      |
| Tags not compliant              | Missing required tag                          | Add tags or configure approved remediation   |
| Cannot create policy assignment | Missing authorization                         | Check Azure RBAC permissions                 |

### **Interview scenario**

A developer has Contributor access but cannot delete a VM.

**How would you troubleshoot?**

1. Check the developer's RBAC assignment.
2. Inspect locks on the VM, resource group, and subscription.
3. Review Azure Policy and relevant deny assignments.
4. Check the detailed error message.
5. Identify whether the operation is blocked by authorization, policy, or a resource lock.
6. Remove or modify the restriction only through an approved process.

## **7. GitHub documentation**

```
phase-00-azure-foundations/
|
|-- lab-06-azure-governance/
|   |-- README.md
|   |-- commands.md
|   |-- troubleshooting.md
|   |-- policy-assignments.md
|   |
|   |-- screenshots/
|       |-- 01-policy-definitions.png
|       |-- 02-tag-policy.png
|       |-- 03-resource-lock.png

```

### **Git commit**

After implementing and verifying the controls:

```
git add phase-00-azure-foundations/lab-06-azure-governance/

git commit -m "feat(az104): implement Azure Policy governance and resource locks"

git push origin main
```

## **8. AZ-104 interview questions**

1\. What is Azure Policy?

2\. What is the difference between Audit and Deny?

3\. What is a policy initiative?

4\. What is the difference between Azure RBAC and Azure Policy?

5\. What is the difference between CanNotDelete and ReadOnly?

6\. Can an Owner delete a locked resource?

7\. Do resource group tags automatically propagate to resources?

## **9. Lab 06 completion checklist**

**Lab 06 Progress**

10/10

Explored Azure Policy definitions

Assigned Allowed locations policy with Audit effect

Assigned a tag governance policy

Reviewed policy compliance

Created CanNotDelete lock on development resource group

Validated resource lock using Azure CLI

Inspected policy assignments using Azure CLI

Documented governance troubleshooting scenarios

Captured redacted screenshots

Prepared GitHub documentation




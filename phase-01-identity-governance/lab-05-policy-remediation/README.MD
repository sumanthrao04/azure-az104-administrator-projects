# **AZ-104 — Phase 01 | Lab 05: Azure Policy Remediation & Automatic Tag Inheritance**

**Project: Enterprise Azure Governance Automation**

In Lab 04, we created an Azure Policy initiative to audit resource locations and missing tags.

In Lab 05, we'll go one step further: instead of only identifying non-compliant resources, we'll learn how Azure Policy can **automatically correct supported configuration issues**.

Our real-world scenario is automatic tag inheritance from resource groups to Azure resources.

Advanced Hands-on Lab

Duration: 2–3 hours

Subscription

**Azure Free Trial**

Target resource group

rg-az104-test-cin-001

New resource budget

**$0–$2 target**

AZ-104 domain

**Identity & Governance**

The budget is a planning target, not a quoted Azure price. We'll use a temporary storage account with no uploaded data if supported, and verify pricing before deployment.

## **1. Real-world scenario**

Imagine your organization has 200 Azure resources deployed by multiple development teams.

The Cloud Governance team requires every resource to contain:

| Tag           | Example value | Purpose                    |
| ------------- | ------------- | -------------------------- |
| `Project`     | AZ104         | Project identification     |
| `Environment` | Test          | Environment identification |
| `CostCenter`  | Learning      | Cost allocation            |

However, developers frequently forget to apply tags when deploying resources.

Without consistent tags, the organization struggles to track costs, ownership, and compliance.

Your manager asks you to implement an automated solution that:

1. Detects resources missing mandatory tags.
2. Inherits approved tags from the parent resource group.
3. Updates supported resources automatically.
4. Remediates existing non-compliant resources.
5. Records compliance and remediation results.

## **2. Architecture — What we're building**

A Modify policy can update supported new or changed resources, while a remediation task applies changes to existing resources through an authorized managed identity.

## **3. Topics we'll learn**

| Topic                      | What you'll understand                               |
| -------------------------- | ---------------------------------------------------- |
| Azure Policy Modify effect | How Azure changes supported resource properties      |
| Tag inheritance            | How resources inherit resource-group tags            |
| Managed identity           | How Azure Policy authenticates for remediation       |
| Role assignments           | Permissions required for remediation                 |
| Remediation task           | Correcting existing non-compliant resources          |
| Compliance evaluation      | Confirming whether policy requirements are satisfied |
| Audit vs Modify            | Detection compared with automated correction         |
| Troubleshooting            | Investigating failed remediation                     |

### **Audit vs Deny vs Modify**

| Effect | Behavior                                                                                             |
| ------ | ---------------------------------------------------------------------------------------------------- |
| Audit  | Reports non-compliance without changing resources                                                    |
| Deny   | Blocks requests that violate policy                                                                  |
| Modify | Changes supported properties during eligible requests and supports remediation of existing resources |

**Important:** A Modify policy doesn't automatically fix every existing resource immediately. Existing resources generally require a remediation task.

# **4. Hands-on implementation**

We'll complete eight tasks. Unlike Lab 04, this lab will include an actual remediation test.

## **Task 1 — Verify the existing resource group and tags**

**Estimated time:** 10 minutes

We'll reuse the resource group from Lab 04:

`rg-az104-test-cin-001`

Open **Azure Portal → Resource groups → rg-az104-test-cin-001 → Tags**.

Verify these tags:

| Tag name    | Expected value |
| ----------- | -------------- |
| Project     | AZ104          |
| Environment | Test           |
| CostCenter  | Learning       |

Alternatively, run:

```
az group show \
  --name rg-az104-test-cin-001 \
  --query "{Name:name,Location:location,Tags:tags}" \
  -o json
```

If any tags are missing, add them before proceeding.

**Screenshot:** `01-resource-group-tags.png`

## **Task 2 — Understand the built-in Modify policy**

**Estimated time:** 15 minutes

Navigate to:

**Azure Portal → Policy → Definitions**

Search for the exact built-in policy:

`Inherit a tag from the resource group if missing`

This is an actual Microsoft Azure built-in policy with the `modify` effect. It copies a tag value from the parent resource group to supported resources when the tag is missing.&#x20;

[image](https://www.google.com/s2/favicons?domain=https://learn.microsoft.com\&sz=32)

Microsoft Learn+1



Open the definition and inspect:

- Policy mode: `Indexed`
- Parameter: `tagName`
- Effect: `Modify`
- Operations: Add a tag when missing
- Role definition requirements for remediation

### **Two similar policies**

| Policy                                           | Behavior                                           |
| ------------------------------------------------ | -------------------------------------------------- |
| Inherit a tag from the resource group            | Adds or replaces a tag to match the resource group |
| Inherit a tag from the resource group if missing | Adds a missing tag but preserves an existing value |

For this lab, use the **if missing** version.

## **Task 3 — Create the Modify policy assignment**

**Estimated time:** 20 minutes

Navigate to:

**Azure Policy → Assignments → Assign policy**

### **Basics**

| Setting            | Value                                                 |
| ------------------ | ----------------------------------------------------- |
| Scope              | `rg-az104-test-cin-001`                               |
| Policy definition  | Inherit a tag from the resource group if missing      |
| Assignment name    | `az104-inherit-project-tag`                           |
| Description        | Automatically inherit Project tag from resource group |
| Policy enforcement | Enabled                                               |
| Exclusions         | None                                                  |

### **Parameters**

| Parameter | Value   |
| --------- | ------- |
| Tag Name  | Project |

### **Managed identity and remediation**

Because this is a Modify policy, configure a managed identity on the assignment.

For a simple lab, select **System assigned managed identity** if offered, with location **Central India**.

In the Remediation tab, do not start an immediate remediation task yet. We'll create a non-compliant resource first.

The policy assignment's managed identity must have sufficient permissions for remediation. When configured through Azure Portal, Azure can grant the roles defined by the built-in policy if your administrator account has the required permissions. Verify those role assignments rather than assuming they were created.&#x20;

[image](https://www.google.com/s2/favicons?domain=https://learn.microsoft.com\&sz=32)

Microsoft Learn



Select **Review + create → Create**.

### **Validate**

Open:

**Azure Policy → Assignments → az104-inherit-project-tag**

Confirm:

- Assignment scope is correct.
- Tag Name is `Project`.
- Enforcement is enabled.
- A managed identity is configured.

**Screenshot:** `02-modify-policy-assignment.png`

## **Task 4 — Create a resource that inherits the tag automatically**

**Estimated time:** 20 minutes

We need an actual Azure resource to demonstrate the Modify effect.

We'll use a temporary Azure Storage account, which will also introduce a service you'll explore in Phase 02.

### **Step 1 — Check cost and availability**

Open the [Azure Pricing Calculator](https://azure.microsoft.com/en-in/pricing/calculator/?utm_source=chatgpt.com) and verify the estimated cost for a Standard general-purpose v2 storage account in Central India.

Use minimal storage, do not upload files, and avoid premium or geo-redundant configurations.

### **Step 2 — Create the storage account**

Navigate to:

**Storage accounts → Create**

| Setting              | Value                           |
| -------------------- | ------------------------------- |
| Subscription         | Azure Free Trial                |
| Resource group       | `rg-az104-test-cin-001`         |
| Storage account name | `staz104tag` + unique suffix    |
| Region               | Central India                   |
| Performance          | Standard                        |
| Redundancy           | Locally-redundant storage (LRS) |
| Account type         | StorageV2 / General-purpose v2  |

For security, retain secure transfer and public blob access restrictions.

### **Step 3 — Configure tags**

Under the Tags tab:

- Do not manually add `Project`.
- You may add `Environment=Test` if desired.

Select **Review + create → Create**.

### **Step 4 — Verify automatic inheritance**

After deployment, open the storage account's **Tags** page.

Expected:

```
Project = AZ104
```

The Modify policy should have added the missing Project tag from the resource group.

### **If the tag does not appear**

Check:

1. Policy assignment scope.
2. Policy enforcement.
3. Policy parameter `Project`.
4. Whether the resource type supports tags.
5. Policy evaluation details and Activity Log.
6. Whether an existing Deny policy blocked or altered the request.

**Screenshot:** `03-automatic-tag-inheritance.png`

## **Task 5 — Prepare an existing resource for remediation**

**Estimated time:** 15 minutes

Now we'll demonstrate how remediation differs from automatic modification during resource creation.

We need the storage account to become non-compliant.

### **Step 1 — Temporarily disable policy enforcement**

Open:

**Azure Policy → Assignments → az104-inherit-project-tag → Edit assignment**

Change **Policy enforcement** to Disabled and save.

This is only for the isolated test assignment.

### **Step 2 — Remove the Project tag**

Open your temporary storage account's **Tags** page.

Remove:

```
Project = AZ104
```

Save the change.

### **Step 3 — Re-enable enforcement**

Return to the policy assignment and enable enforcement again.

The existing storage account should now be eligible for compliance evaluation because it is missing the required tag.

### **Step 4 — Trigger a compliance scan**

```
az policy state trigger-scan \
  --resource-group rg-az104-test-cin-001
```

Wait for the scan to finish and refresh the Compliance page.

**Expected:** The storage account is reported as non-compliant with the tag inheritance policy.

If the portal continues showing the old state, allow time for evaluation and check the assignment scope.

**Screenshot:** `04-noncompliant-storage-account.png`

## **Task 6 — Create an Azure Policy remediation task**

**Estimated time:** 25 minutes

Navigate to:

**Azure Portal → Policy → Remediation**

Under **Policies to remediate**, locate:

`az104-inherit-project-tag`

Select the assignment.

### **Configure remediation**

| Setting              | Value                         |
| -------------------- | ----------------------------- |
| Policy assignment    | `az104-inherit-project-tag`   |
| Scope                | `rg-az104-test-cin-001`       |
| Resource selection   | Non-compliant resources       |
| Remediation identity | Assignment's managed identity |

Select **Remediate**.

Azure Policy will use the assignment's managed identity to update the supported non-compliant resource.

### **What happens internally?**

**Policy detects missing Project tag**

**Remediation task is created**

**Managed identity performs authorized tag update**

**Storage account receives Project=AZ104**

### **Verify the remediation result**

Open **Azure Policy → Remediation → Remediation tasks**.

Check whether the task completed successfully.

Then open the storage account and confirm that the tag was restored.

Microsoft documents this process for `modify` and `deployIfNotExists` policy assignments.&#x20;

[image](https://www.google.com/s2/favicons?domain=https://learn.microsoft.com\&sz=32)

Microsoft Learn



**Screenshot:** `05-remediation-success.png`

## **Task 7 — Azure CLI validation**

**Estimated time:** 20 minutes

### **Verify the policy assignment**

```
az policy assignment list \
  --resource-group rg-az104-test-cin-001 \
  --query "[].{Name:name,DisplayName:displayName}" \
  -o table
```

### **Inspect the policy assignment**

```
az policy assignment show \
  --name az104-inherit-project-tag \
  --resource-group rg-az104-test-cin-001 \
  -o json
```

### **List remediation tasks**

```
az policy remediation list \
  --resource-group rg-az104-test-cin-001 \
  -o table
```

### **Inspect storage account tags**

Replace the placeholder with your actual storage account name:

```
az storage account show \
  --name <storage-account-name> \
  --resource-group rg-az104-test-cin-001 \
  --query "{Name:name,Tags:tags}" \
  -o json
```

Expected tag:

```
{
  "Project": "AZ104"
}
```

The actual output may contain additional tags.

## **Task 8 — Optional: Inherit multiple tags**

**Estimated time:** 15 minutes

After successfully implementing the Project tag, you can repeat the same process for:

- `Environment`
- `CostCenter`

Create separate assignments using the same built-in policy definition, each with a different `Tag Name` parameter.

For example:

| Assignment                      | Tag Name    |
| ------------------------------- | ----------- |
| `az104-inherit-project-tag`     | Project     |
| `az104-inherit-environment-tag` | Environment |
| `az104-inherit-costcenter-tag`  | CostCenter  |

This demonstrates how organizations enforce standardized resource metadata across environments.

# **5. Troubleshooting scenarios**

| Problem                                       | Possible cause                                          | Resolution                                    |
| --------------------------------------------- | ------------------------------------------------------- | --------------------------------------------- |
| Policy assignment fails                       | Missing permissions                                     | Check Azure RBAC                              |
| Managed identity missing                      | Identity not configured                                 | Edit assignment and configure identity        |
| Remediation fails                             | Missing managed identity permissions                    | Check role assignments                        |
| Storage tag not inherited                     | Policy scope or parameter issue                         | Inspect assignment                            |
| Compliance shows Not started                  | Evaluation pending                                      | Trigger scan and wait                         |
| Existing resource remains non-compliant       | Remediation not executed                                | Create remediation task                       |
| Remediation task completes but tag is missing | Wrong target, skipped resource, or unexpected operation | Inspect remediation details and resource tags |

### **Real-world interview scenario**

A company has 500 Azure resources, and 150 are missing mandatory cost-center tags.

**How would you automate correction?**

A strong answer:

1. Configure a built-in Modify policy to inherit the CostCenter tag.
2. Assign it at the required scope.
3. Configure a managed identity.
4. Ensure the identity has the required permissions.
5. Evaluate compliance.
6. Create a remediation task for existing non-compliant resources.
7. Validate the updated tags.
8. Monitor future compliance.

## **6. GitHub documentation**

```
phase-01-identity-governance/
|
|-- lab-05-policy-remediation/
|   |-- README.md
|   |-- commands.md
|   |-- troubleshooting.md
|   |-- interview-questions.md
|   |-- cleanup.md
|   |
|   |-- screenshots/
|       |-- 01-resource-group-tags.png
|       |-- 02-modify-policy-assignment.png
|       |-- 03-automatic-tag-inheritance.png
|       |-- 04-noncompliant-resource.png
|       |-- 05-remediation-success.png
```

### **Git commit**

After completing and validating the lab:

```
git add phase-01-identity-governance/lab-05-policy-remediation/

git commit -m "feat(az104): implement Azure Policy remediation and automatic tag inheritance"

git push origin main
```

## **7. Cleanup and cost control**

Because you're using the $200 Azure Free Trial credit, cleanup is important.

After the lab:

1. Capture all screenshots.
2. Record the policy assignment and remediation results.
3. Delete the temporary storage account.
4. Remove the temporary Modify policy assignments if they are no longer needed.
5. Remove any policy-assignment managed identities and role assignments that remain after cleanup.
6. Keep `rg-az104-test-cin-001` if required for Lab 06.
7. Review Azure Cost Analysis and remaining credit.

Do not delete your existing development resource group or the security groups used in previous labs.

## **8. AZ-104 interview questions**

1\. What is the Modify effect in Azure Policy?

2\. What is an Azure Policy remediation task?

3\. Why does remediation require a managed identity?

4\. Does a Modify policy automatically remediate every existing resource?

5\. What is the difference between Audit and Modify?

6\. What is the difference between tag inheritance and tag replacement?

7\. How would you troubleshoot failed remediation?

## **9. Lab 05 completion checklist**

**Lab 05 Progress**

0/13

Verified resource group tags

Explored built-in Modify policy

Assigned Inherit a tag from the resource group if missing

Verified managed identity and permissions

Created temporary storage account

Validated automatic Project tag inheritance

Created an existing non-compliant resource scenario

Triggered policy compliance scan

Created and completed remediation task

Verified Project tag restoration

Executed Azure CLI validation commands

Deleted temporary storage account and reviewed costs

Prepared GitHub documentation


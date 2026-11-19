# **AZ-104 — Phase 01 | Lab 04: Azure Policy Initiatives & Compliance Management**

**Project: Enterprise Azure Governance Using Policy Initiatives**

In Phase 00, you learned how to configure individual Azure Policies for allowed locations and required tags.

In this lab, we'll move to enterprise-level governance by combining multiple Azure Policies into a single **Policy Initiative**, assigning it to a test environment, and monitoring compliance.

This is an important topic for both the **AZ-104 certification and Azure Cloud Engineer interviews**.

Advanced Hands-on Lab

Duration: 2–3 hours

Subscription

**Azure Free Trial**

Estimated additional cost

**$0**

Region

**Central India**

AZ-104 domain

**Identity & Governance**

**Prerequisites:** Your existing Azure subscription, resource groups, Azure Policy access, and permission to create policy definitions and assignments.

## **1. Real-world scenario**

Imagine your company manages Azure infrastructure across Development, Testing, and Production environments.

The Cloud Governance team introduces three mandatory standards:

- **Region compliance:** Azure resources must be deployed in approved regions.
- **Tag compliance:** Resources must contain a `Project` tag.
- **Environment compliance:** Resources must contain an `Environment` tag.

Previously, you assigned policies individually. As the number of subscriptions and resources increases, managing each policy separately becomes difficult.

Your manager asks you to combine the policies into a reusable governance package.

### **Business requirements**

| Requirement                 | Policy                     | Expected behavior                     |
| --------------------------- | -------------------------- | ------------------------------------- |
| Approved deployment regions | Allowed locations          | Audit resources outside Central India |
| Project ownership           | Require a tag on resources | Audit resources missing `Project`     |
| Environment identification  | Require a tag on resources | Audit resources missing `Environment` |

We'll initially use **Audit** effects to avoid blocking deployments during your remaining AZ-104 labs.

## **2. Architecture — What we're building**

A custom initiative combines three built-in policy definitions and evaluates resources inside a dedicated test resource group.

## **3. Concepts you need to understand**

### **Azure Policy definition vs initiative vs assignment**

| Component             | Purpose                                   | Example                        |
| --------------------- | ----------------------------------------- | ------------------------------ |
| Policy definition     | Individual governance rule                | Allowed locations              |
| Policy initiative     | Collection of related policies            | AZ104 Governance Baseline      |
| Policy assignment     | Applies a policy or initiative to a scope | Test resource group            |
| Policy parameter      | Makes a definition reusable               | Allowed region = Central India |
| Compliance evaluation | Checks applicable resource configurations | Compliant / Non-compliant      |
| Remediation task      | Corrects supported non-compliance         | Add missing tags               |

### **Why use initiatives?**

Suppose a company manages 20 Azure subscriptions, each requiring 10 governance policies.

Instead of managing each policy independently, administrators can group related policies into an initiative and assign the initiative at the appropriate management group or subscription scope.

For our lab, we'll use a **resource-group scope** to avoid affecting unrelated projects.

### **Audit vs Deny**

| Audit                                | Deny                              |
| ------------------------------------ | --------------------------------- |
| Reports non-compliant resources      | Blocks non-compliant requests     |
| Suitable for learning and assessment | Suitable for approved enforcement |
| Does not block deployments           | Can prevent deployments           |
| Our initial configuration            | Optional future exercise          |

**Important:** Some built-in tag policies have a fixed Deny effect. To build an audit-only initiative, we must choose definitions that support Audit or use custom audit policy definitions.

# **4. Hands-on implementation**

## **Task 1 — Create an isolated governance test resource group**

**Estimated time:** 15 minutes

We will not modify your main development resource group.

### **Step 1 — Open Azure Portal**

Navigate to:

**Resource groups → Create**

Use these settings:

| Field           | Value                   |
| --------------- | ----------------------- |
| Subscription    | Azure Free Trial        |
| Resource group  | `rg-az104-test-cin-001` |
| Region          | Central India           |
| Project tag     | AZ104                   |
| Environment tag | Test                    |
| CostCenter tag  | Learning                |

### **Step 2 — Create using Azure CLI**

Alternatively, run:

```
az group create \
  --name rg-az104-test-cin-001 \
  --location centralindia \
  --tags Project=AZ104 Environment=Test CostCenter=Learning
```

### **Step 3 — Verify**

```
az group show \
  --name rg-az104-test-cin-001 \
  --query "{Name:name,Location:location,Tags:tags}" \
  -o json
```

**Expected outcome:** A tagged, empty test resource group exists.

Creating an empty resource group does not itself incur usage charges.

**Screenshot:** `01-test-resource-group.png`

## **Task 2 — Explore built-in Azure Policy definitions**

**Estimated time:** 15 minutes

Open:

**Azure Portal → Policy → Definitions**

Search for these policies:

1. `Allowed locations`
2. `Require a tag on resources`
3. `Audit resources with a given tag`

Inspect each definition's parameters and supported effects.

For this lab, select built-in policies that support Audit. Depending on the available definitions, you may need to use an audit-oriented tag definition rather than the Deny-only `Require a tag on resources` policy.

### **What to verify**

- Policy definition display name.
- Policy mode.
- Parameters.
- Policy rule.
- Supported effects.

**Screenshot:** `02-policy-definitions.png`

## **Task 3 — Create a custom Policy Initiative**

**Estimated time:** 25 minutes

### **Step 1 — Open Azure Policy**

Navigate to:

**Azure Portal → Policy → Definitions → + Initiative Definition**

Microsoft supports creating an initiative by grouping existing policy definitions, then assigning that initiative to an Azure scope.&#x20;

[image](https://www.google.com/s2/favicons?domain=https://learn.microsoft.com\&sz=32)

Microsoft Learn



### **Step 2 — Configure initiative basics**

| Field               | Value                                              |
| ------------------- | -------------------------------------------------- |
| Initiative location | Your Azure Free Trial subscription                 |
| Name                | `az104-governance-baseline`                        |
| Display name        | AZ104 Enterprise Governance Baseline               |
| Description         | Audit approved regions and mandatory resource tags |
| Category            | Custom / General, if available                     |

The **initiative location** determines where the definition is stored. It must be a subscription or management group, not a resource group.&#x20;

[image](https://www.google.com/s2/favicons?domain=https://learn.microsoft.com\&sz=32)

Microsoft Learn



### **Step 3 — Add policy definitions**

In the initiative wizard, open the **Policies** tab and add the following definitions.

| Policy                                       | Parameter         | Desired value |
| -------------------------------------------- | ----------------- | ------------- |
| Allowed locations                            | Allowed locations | Central India |
| Audit resources with missing Project tag     | Tag name          | Project       |
| Audit resources with missing Environment tag | Tag name          | Environment   |

For the two tag requirements, select an audit-capable built-in definition if available. Otherwise, create a small custom Audit policy rather than choosing a Deny-only definition.

**Important:** The policy definition name and parameter options may differ in the portal. Inspect the selected definition's JSON before adding it. Do not assume a built-in definition supports Audit merely because its name contains "Require".

### **Step 4 — Configure parameters**

Where supported, configure:

```
AllowedLocations = centralindia
ProjectTag        = Project
EnvironmentTag    = Environment
Effect            = Audit
```

Use separate references for the two tag policies so each can receive its own tag-name parameter.

### **Step 5 — Create the initiative**

Select **Review + create → Create**.

### **Step 6 — Verify**

Navigate to:

**Azure Policy → Definitions**

Filter by:

- Definition type: Initiative
- Category: Custom, if configured

Search for:

`AZ104 Enterprise Governance Baseline`

**Expected outcome:** The custom initiative appears and contains three policy references.

**Screenshot:** `03-custom-initiative-created.png`

## **Task 4 — Assign the initiative to the test resource group**

**Estimated time:** 20 minutes

### **Step 1 — Open the initiative**

Navigate to:

**Azure Policy → Definitions → AZ104 Enterprise Governance Baseline**

Select **Assign initiative**.

### **Step 2 — Configure assignment basics**

| Setting            | Value                                                      |
| ------------------ | ---------------------------------------------------------- |
| Scope              | `rg-az104-test-cin-001`                                    |
| Assignment name    | `az104-governance-test-assignment`                         |
| Description        | Audit governance compliance in the AZ-104 test environment |
| Policy enforcement | Enabled                                                    |
| Exclusions         | None                                                       |

### **Step 3 — Configure parameters**

Verify that:

- Allowed location is Central India.
- Required tag names are Project and Environment.
- All included policy effects are Audit.

### **Step 4 — Remediation tab**

Because we're using Audit policies, we do not need a managed identity or remediation task.

Leave remediation settings disabled.

### **Step 5 — Create the assignment**

Select **Review + create → Create**.

**Expected outcome:** The initiative is assigned to the test resource group only, without affecting your development resource group.

**Screenshot:** `04-initiative-assignment.png`

## **Task 5 — Validate compliance**

**Estimated time:** 20 minutes

Navigate to:

**Azure Policy → Compliance**

Search for:

`AZ104 Enterprise Governance Baseline`

Open the initiative assignment.

### **What you should see**

The initiative displays compliance information for each included policy.

| Policy                 | Expected evaluation                     |
| ---------------------- | --------------------------------------- |
| Allowed locations      | Evaluates applicable resource locations |
| Project tag policy     | Evaluates applicable resource tags      |
| Environment tag policy | Evaluates applicable resource tags      |

### **Important limitation**

Because our test resource group is empty, Azure may show **Not started, zero evaluated resources, or no applicable resources**.

That does not mean the policy is broken. Azure Policy evaluates applicable resources within the assignment scope.&#x20;

[image](https://www.google.com/s2/favicons?domain=https://learn.microsoft.com\&sz=32)

Microsoft Learn



### **Trigger a compliance scan**

From Azure Cloud Shell:

```
RG_SCOPE=$(az group show \
  --name rg-az104-test-cin-001 \
  --query id \
  -o tsv)

az policy state trigger-scan \
  --resource-group rg-az104-test-cin-001
```

The scan runs asynchronously. Wait for the evaluation to complete, then refresh the Compliance page.

**Screenshot:** `05-policy-compliance.png`

## **Task 6 — Create a non-compliance scenario without paid resources**

**Estimated time:** 20 minutes

To observe a real non-compliant resource, you would normally deploy a resource missing required tags or using an unapproved location.

However, we're keeping this lab at approximately $0.

### **Recommended approach**

Use the existing empty resource group to understand policy assignment and evaluation, then document a non-compliance scenario for later execution.

**Scenario:** A developer deploys a storage account without the required `Project` tag.

Expected result with an Audit policy:

- The deployment is permitted.
- The storage account is evaluated as non-compliant.
- The compliance dashboard identifies the policy violation.
- An administrator can investigate and apply the missing tag.

We'll perform a real resource-level compliance test during Phase 02, when we create Azure Storage accounts.

This avoids creating and deleting extra resources solely for this governance lab.

## **Task 7 — Validate initiative configuration using Azure CLI**

**Estimated time:** 20 minutes

### **Command 1 — List initiative definitions**

```
az policy set-definition list \
  --query "[].{Name:displayName,Type:policyType}" \
  -o table
```

### **Command 2 — Inspect your initiative**

```
az policy set-definition show \
  --name az104-governance-baseline \
  -o json
```

### **Command 3 — List initiative assignments**

```
az policy assignment list \
  --scope "$RG_SCOPE" \
  --query "[].{Name:name,DisplayName:displayName,Scope:scope}" \
  -o table
```

### **Command 4 — Inspect compliance states**

```
az policy state list \
  --resource-group rg-az104-test-cin-001 \
  --query "[].{Resource:resourceId,State:complianceState,Policy:policyDefinitionName}" \
  -o table
```

An empty result is possible if no applicable resources have been evaluated.

### **Command 5 — Check the test resource group**

```
az group show \
  --name rg-az104-test-cin-001 \
  --query "{Name:name,Location:location,Tags:tags}" \
  -o json
```

## **5. Troubleshooting scenarios**

| Issue                         | Possible cause                         | Investigation                         |
| ----------------------------- | -------------------------------------- | ------------------------------------- |
| Cannot create initiative      | Missing permissions                    | Check subscription IAM                |
| Policy definition unavailable | Wrong scope or definition type         | Verify definition location            |
| Audit effect unavailable      | Selected definition supports only Deny | Choose an audit-capable definition    |
| Initiative assignment fails   | Invalid parameters                     | Inspect initiative parameter mappings |
| Compliance shows Not started  | Evaluation pending                     | Trigger scan and wait                 |
| No evaluated resources        | Empty test resource group              | Validate scope and applicability      |
| Unexpected non-compliance     | Missing tags or wrong region           | Inspect policy evaluation details     |

### **Real-world interview scenario**

Your company has deployed hundreds of resources across multiple subscriptions.

Some resources do not contain mandatory tags, and others are deployed outside approved regions.

**How would you implement centralized governance?**

A strong interview answer:

1. Define organizational governance requirements.
2. Select or create appropriate Azure Policy definitions.
3. Group related policies into a reusable initiative.
4. Store the initiative at a suitable management group or subscription scope.
5. Assign it to the required environments.
6. Start with Audit to assess the impact.
7. Review compliance results and exceptions.
8. Introduce enforcement after testing and approval.
9. Use remediation tasks for supported policies.
10. Monitor compliance continuously.

## **6. GitHub repository structure**

```
phase-01-identity-governance/
|
|-- lab-04-policy-initiatives/
|   |-- README.md
|   |
|   |-- screenshots/
|       |-- 01-test-resource-group.png
|       |-- 02-policy-definitions.png
|       |-- 03-custom-initiative-created.png
|       |-- 04-initiative-assignment.png
|       |-- 05-policy-compliance.png
```

### **Git commit**

After completing the lab:

```
git add phase-01-identity-governance/lab-04-policy-initiatives/

git commit -m "feat(az104): implement Azure Policy initiative and compliance monitoring"

git push origin main
```

Do not upload subscription IDs or other sensitive identifiers.

## **7. AZ-104 interview questions**

1\. What is an Azure Policy initiative?

2\. What is the difference between a policy definition and an assignment?

3\. Why use Audit before Deny?

4\. Where can a custom initiative definition be stored?

5\. Can an initiative contain multiple policies?

6\. Why might compliance show zero evaluated resources?

7\. What is the difference between compliance and remediation?

## **8. Lab 04 completion checklist**

**Lab 04 Progress**

0/11

Created rg-az104-test-cin-001

Explored built-in Azure Policy definitions

Selected audit-capable policies

Created AZ104 Governance Initiative

Assigned initiative at test resource-group scope

Verified assignment parameters

Reviewed compliance results

Triggered a compliance scan

Executed Azure CLI validation commands

Documented the non-compliance scenario

Prepared GitHub documentation


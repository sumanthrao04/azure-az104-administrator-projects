# **AZ-104 — Phase 00 | Lab 05: Azure Cost Management & Budget Alerts**

**Project: Azure Cloud Cost Governance & Budget Monitoring**

In this lab, you'll learn how an Azure Administrator monitors cloud spending, configures budgets, creates cost alerts, identifies expensive resources, and prevents unnecessary costs.

We'll use your existing **Azure Free Trial subscription with $200 promotional credit** and the resource group you created in Lab 02.

Hands-on Lab

Duration: 2–3 hours

Subscription

**Azure Free Trial**

Promotional credit

# **$200**

Initial credit, not verified current balance

Future monthly budget

# **₹1,000**

Lab deployment cost

# **$0**

No new billable resources required

## **1. Real-world scenario**

Imagine you're working as an Azure Cloud Administrator for an organization with Development, Testing, and Production environments.

Your manager reports that Azure infrastructure spending is increasing, but the company cannot easily determine which resources or teams are responsible.

You are asked to:

- Monitor subscription spending and identify cost drivers.
- Configure monthly budgets and email alerts.
- Track resource costs using tags.
- Review spending by service, resource group, and region.
- Recommend cost optimization measures.
- Establish a repeatable cloud cost monitoring process.

This is a common **FinOps (Cloud Financial Operations)** responsibility for Azure Administrators and Cloud Engineers.

## **2. Architecture — What we're implementing**

Cost Management monitors subscription usage. Budgets and alerts are monitoring controls, not automatic spending limits.

## **3. Concepts to learn for AZ-104**

| Concept                  | Explanation                                           |
| ------------------------ | ----------------------------------------------------- |
| Azure Cost Management    | Tools for analyzing and monitoring Azure costs        |
| Cost Analysis            | Displays cost trends and breakdowns                   |
| Budget                   | Defines a spending threshold for a scope and period   |
| Budget Alert             | Notifies recipients when a threshold is reached       |
| Actual Cost              | Cost incurred according to reported usage             |
| Forecasted Cost          | Estimated spending based on usage trends              |
| Cost Allocation Tags     | Metadata used to classify resource costs              |
| Azure Advisor            | Provides recommendations, including cost optimization |
| Azure Pricing Calculator | Estimates the cost of planned deployments             |

### **Important: Azure Free Trial vs Pay-As-You-Go**

| Feature                        | Azure Free Trial                          | Pay-As-You-Go                       |
| ------------------------------ | ----------------------------------------- | ----------------------------------- |
| Promotional credit             | $200 for up to 30 days                    | No new trial credit                 |
| Ordinary usage charges to card | Protected by free-trial spending limit    | Charged for billable usage          |
| Budget alerts                  | Subject to scope and availability         | Supported with appropriate access   |
| Spending limit                 | Free-trial credit protection              | No automatic hard cap               |
| After trial                    | Upgrade required for continued paid usage | Continues until canceled or changed |

For this lab, we'll prioritize checking your remaining promotional credit. If budget creation is unavailable on your Free Trial subscription, we'll document the limitation and continue with the available cost-monitoring features.

# **4. Hands-on implementation**

## **Task 1 — Verify Azure Free Trial credit**

**Estimated time:** 15 minutes

1. Sign in to [Azure Portal](https://portal.azure.com?utm_source=chatgpt.com).
2. Open the Azure Free Trial credit information from the Home page, if available.
3. Alternatively, search for **Cost Management + Billing**.
4. Open your billing or subscription overview.
5. Locate the free-trial credit details.

Record the following information:

| Property                   | Value to record        |
| -------------------------- | ---------------------- |
| Subscription               | Azure Free Trial       |
| Initial promotional credit | $200                   |
| Remaining credit           | Read from Azure Portal |
| Credit expiration date     | Read from Azure Portal |
| Credit consumed            | Read from Azure Portal |
| Subscription status        | Active, if shown       |

**Screenshot:** `01-free-trial-credit.png`

Do not publish billing identifiers, payment details, or other sensitive information.

## **Task 2 — Explore Azure Cost Analysis**

**Estimated time:** 20 minutes

Navigate to:

**Subscriptions → Your Free Trial subscription → Cost Management → Cost analysis**

[Customize views in Cost Analysis - Microsoft Cost Management | Microsoft Learn](https://images.openai.com/static-rsc-4/U1IqGinbl7iuQSyKatAkKOLUNWQoKmdXHAMWe1myg9y_y_CfqUag225dLszqzzqqWolmnrZTn7p_7wnpEVgWCrl4NO2f5VADWocL6KnmOZGL0t_YP7XwJaf_7bPsV4a_sN4dhGQnxTVVtLqe7bbz9WK4jUMJmp5brbyn6pYUtrU?purpose=inline)

[learn.microsoft.com](https://learn.microsoft.com/en-us/azure/cost-management-billing/costs/customize-cost-analysis-views?utm_source=chatgpt.com)

Configure the following:

| Setting     | Value                         |
| ----------- | ----------------------------- |
| Scope       | Azure Free Trial subscription |
| Date range  | Current month                 |
| View        | Accumulated costs             |
| Group by    | Service name                  |
| Granularity | Daily, where available        |

Observe:

- Total reported spending.
- Services contributing to costs.
- Daily usage patterns.
- Cost changes over time.

Now change **Group by** to **Resource group** and look for `rg-az104-dev-cin-001`.

Your resource group may not appear in the cost chart if it contains no billable resources.

### **What you learned**

Cost Analysis helps administrators identify which services and resources are responsible for spending.

**Screenshot:** `02-cost-analysis.png`

## **Task 3 — Configure an Azure budget**

**Estimated time:** 20–30 minutes

Navigate to:

**Cost Management → Budgets → + Add**

Choose the subscription-level scope.

### **Budget configuration**

For your current Free Trial, I recommend a **$160 monthly monitoring budget** if your billing currency is USD and budget creation is supported. This leaves a $40 planning reserve against the initial $200 promotional credit.

| Setting       | Value                         |
| ------------- | ----------------------------- |
| Budget name   | `az104-free-trial-budget`     |
| Scope         | Azure Free Trial subscription |
| Reset period  | Monthly                       |
| Budget amount | $160, if currency is USD      |
| Start date    | Current month                 |
| Expiration    | At least the current month    |

If Azure displays INR or another billing currency, use the appropriate equivalent based on your billing currency. Do not enter `160` without checking the currency.

**Important distinction:** A monthly budget resets according to the calendar period. Your $200 promotional credit is a one-time balance that expires after up to 30 days. These are not the same thing.

If the portal does not permit a $160 budget or does not support budgets for your subscription, do not upgrade your account just to complete this exercise.

## **Task 4 — Configure budget alerts**

**Estimated time:** 15 minutes

For the proposed $160 budget, configure the following thresholds:

| Alert          | Threshold | Equivalent USD amount | Action                     |
| -------------- | --------- | --------------------- | -------------------------- |
| Warning        | 50%       | $80                   | Review spending            |
| High usage     | 80%       | $128                  | Stop unnecessary resources |
| Budget reached | 100%      | $160                  | Pause new deployments      |

Use **Actual** as the alert type and add your email address as a recipient.

If available, configure a separate **Forecasted** alert at 80%.

Save the budget.

**Important limitation**

Budget alerts do not automatically stop Azure resources, and cost reporting is delayed. A $160 budget does not guarantee that $40 of promotional credit will remain.

**Screenshot:** `03-budget-alerts.png`

## **Task 5 — Review cost allocation tags**

You already configured tags on `rg-az104-dev-cin-001` during Lab 02.

Verify them using Azure CLI:

```
az group show \
  --name rg-az104-dev-cin-001 \
  --query tags \
  -o json
```

Your previous lab included tags such as:

```
{
  "Project": "AZ104",
  "Environment": "Dev",
  "ManagedBy": "AzurePortal",
  "CostCenter": "Learning"
}
```

The Owner tag can remain configured with your chosen value.

### **Why do tags matter?**

In enterprise environments, tags help classify resources by project, environment, department, owner, and cost center.

**Important:** Resource group tags do not automatically propagate to all resources inside the group. For accurate resource-level cost analysis, tag individual billable resources or configure appropriate tag inheritance mechanisms where supported.

## **Task 6 — Explore Azure Advisor cost recommendations**

**Estimated time:** 15 minutes

1. Search for **Azure Advisor**.
2. Open **Recommendations**.
3. Select the **Cost** category.
4. Review any recommendations.

Examples include underutilized VMs, idle resources, and opportunities to reduce resource spending.

A new Free Trial subscription with no running infrastructure may have no cost recommendations. That is a valid result.

**Screenshot:** `04-azure-advisor.png`

## **Task 7 — Use Azure CLI for cost governance**

Azure CLI can inspect resource tags and budget resources, subject to the installed command extensions and your subscription permissions.

### **Command 1 — Verify your resource group**

```
az group show \
  --name rg-az104-dev-cin-001 \
  --query "{Name:name,Location:location,Tags:tags}" \
  -o json
```

### **Command 2 — List deployed resources**

```
az resource list \
  --resource-group rg-az104-dev-cin-001 \
  -o table
```

### **Command 3 — Inspect available budget commands**

```
az consumption budget --help
```

If the command group is unavailable, do not install extensions blindly; first check your Azure CLI version and the relevant documentation.

### **Command 4 — List subscription budgets, if supported**

```
az consumption budget list -o table
```

This may not work for all subscription offers or billing scopes. Record any error and continue using Azure Portal.

## **5. Troubleshooting scenarios**

These are useful for both AZ-104 and Cloud Engineer interviews.

| Problem                                       | Possible reason                                     | Troubleshooting                               |
| --------------------------------------------- | --------------------------------------------------- | --------------------------------------------- |
| Budget creation disabled                      | Free Trial restrictions or insufficient permissions | Verify subscription offer and billing scope   |
| Cost Analysis shows no data                   | No billable usage or reporting delay                | Check usage period and credit dashboard       |
| Budget email not received                     | Threshold not reached or evaluation pending         | Verify recipient and alert configuration      |
| Resource costs not grouped by tag             | Missing resource-level tags                         | Inspect resource tags and cost data           |
| Budget reached but resources continue running | Budgets are notification-based                      | Manually stop or delete unnecessary resources |
| Azure Advisor shows no recommendations        | Insufficient usage history                          | Recheck after deploying and using resources   |

### **Production-style troubleshooting scenario**

**Incident:** The development team's Azure spending increases unexpectedly.

As the Azure Administrator, explain how you would investigate:

1. Open Cost Analysis at subscription scope.
2. Compare spending with the previous period.
3. Group costs by service and resource group.
4. Identify resources responsible for the increase.
5. Check whether VMs, disks, public IPs, or other services were left running.
6. Review resource tags and deployment activity.
7. Apply approved cost optimizations.
8. Configure appropriate budget notifications to reduce the likelihood of recurrence.

## **6. GitHub project structure**

```
phase-00-azure-foundations/
|
|-- lab-05-cost-management/
|   |-- README.md
|   |
|   |-- screenshots/
|       |-- 01-free-trial-credit.png
|       |-- 02-cost-analysis.png
|       |-- 03-budget-alerts.png
```

Your README should document the subscription type, budget settings, alert thresholds, cost analysis findings, and limitations encountered.

### **Git commit**

Once the lab is completed:

```
git add phase-00-azure-foundations/lab-05-cost-management/

git commit -m "feat(az104): configure Azure cost monitoring and budget alerts"

git push origin main
```

If budget creation was unavailable, use a commit message that accurately describes the work completed rather than claiming alerts were configured.

## **7. AZ-104 interview questions**

1\. What is Azure Cost Management?

2\. What is the difference between actual and forecasted cost?

3\. Will Azure automatically stop resources when a budget reaches 100%?

4\. How do resource tags help cost management?

5\. What is Azure Advisor?

6\. What happens when Azure Free Trial credit expires?

## **8. Lab 05 completion checklist**

**Lab 05 Progress**

10/10

Verified remaining promotional credit and trial expiration

Explored subscription Cost Analysis

Reviewed costs by service and resource group

Created a monthly budget, or documented Free Trial restrictions

Configured actual-cost email alerts, if supported

Verified resource group tags

Explored Azure Advisor cost recommendations

Practiced Azure CLI cost governance commands

Documented troubleshooting and cost optimization findings

Prepared redacted screenshots and GitHub documentation


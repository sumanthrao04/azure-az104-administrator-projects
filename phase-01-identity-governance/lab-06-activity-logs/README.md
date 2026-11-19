# **AZ-104 — Phase 01 | Lab 06: Azure Activity Logs & Security Access Investigation**

**Project: Enterprise Azure Audit Logging & Administrative Activity Monitoring**

This is the final lab in our revised **Phase 01 — Enterprise Identity & Governance**.

In previous labs, you've worked on Entra ID user lifecycle management, custom RBAC, access troubleshooting, policy initiatives, and policy remediation.

Now we'll investigate **who performed an action, what changed, when it happened, and whether the operation succeeded or failed**.

Advanced · Troubleshooting Lab

Duration: 2–3 hours

Subscription

**Azure Free Trial**

Additional lab cost

**$0 target**

Resource group

rg-az104-test-cin-001

AZ-104 domain

**Identity & Governance / Monitoring**

We'll use Azure Activity Log and existing lab resources. Optional export to Log Analytics can incur ingestion and retention charges, so it is excluded from the core lab.

## **1. Real-world scenario**

Imagine you're working as an Azure Administrator.

Your security team raises the following incidents:

- A developer changed the tags on an Azure resource group without informing the operations team.
- A user attempted to create a resource but received `AuthorizationFailed`.
- A Contributor role was assigned to a security group.
- An Azure Policy assignment was modified.
- A temporary storage account was deleted.

Your manager asks you to investigate these events and produce an audit report.

### **Your responsibilities**

1. Identify the operation performed.
2. Determine which identity initiated it.
3. Identify the affected resource.
4. Check the operation timestamp.
5. Determine whether it succeeded or failed.
6. Correlate related events using a correlation ID.
7. Document findings and preventive controls.

## **2. Architecture — What we're investigating**

Azure Resource Manager administrative events are recorded in Azure Activity Log and can be investigated using Azure Portal and Azure CLI.

## **3. Important AZ-104 concepts**

### **What is Azure Activity Log?**

Azure Activity Log is a subscription-level platform log that records Azure control-plane events.

Examples include creating resource groups, updating resources, changing role assignments, and modifying Azure Policy assignments.

Activity Log events are retained for **90 days by default**. Longer retention requires routing them to another supported destination.

### **Activity Log vs Microsoft Entra logs vs Resource logs**

| Log type           | What it records                      | Example                         |
| ------------------ | ------------------------------------ | ------------------------------- |
| Azure Activity Log | Azure resource management operations | Resource group updated          |
| Entra sign-in logs | Authentication activity              | User sign-in success or failure |
| Entra audit logs   | Directory changes                    | User added to a group           |
| Resource logs      | Service-specific activity            | Storage account data operations |

**Important:** Creating a Microsoft Entra user or changing group membership is a directory operation, so it is investigated using Entra audit logs, not Azure Activity Log.

### **Important Activity Log fields**

| Field           | Purpose                                                      |
| --------------- | ------------------------------------------------------------ |
| Event timestamp | When the event occurred                                      |
| Caller          | Identity associated with the operation                       |
| Operation name  | Action performed                                             |
| Resource ID     | Target Azure resource                                        |
| Resource group  | Resource group involved                                      |
| Status          | Succeeded, Failed, Started, etc.                             |
| Correlation ID  | Identifier used to correlate related events                  |
| Category        | Administrative, Policy, Security, and other event categories |

# **4. Hands-on implementation**

We'll complete seven exercises.

## **Task 1 — Explore Azure Activity Log**

**Estimated time:** 15 minutes

1. Sign in to [Azure Portal](https://portal.azure.com?utm_source=chatgpt.com).
2. Search for **Monitor**.
3. Select **Activity log**.
4. Set the subscription to your Azure Free Trial.
5. Set the time range to **Last 24 hours**.
6. Review the available events.

[Azure Monitor-Aktivitätsprotokoll - Azure Monitor | Microsoft Learn](https://images.openai.com/static-rsc-4/S0zV3Xdzjb8l7DwQi1dm5-Jy19XAhoc9e6CDqOX8yeT-y_9Z1vG6-o-AsDZQoLfn87pK8ZsMAMQ2nH2KD_DdXUSvQV5uBbTzbmpBc18olyvfoe96W_9WeO-xor0koyBpCntAfoIyJki5eNWyUH-h_zTscD5VarnIcfYVZuvXVsg?purpose=inline)

[learn.microsoft.com](https://learn.microsoft.com/de-de/azure/azure-monitor/platform/activity-log?utm_source=chatgpt.com)

### **Practice filters**

| Filter           | Value                   |
| ---------------- | ----------------------- |
| Timespan         | Last 24 hours           |
| Event severity   | All                     |
| Resource group   | `rg-az104-test-cin-001` |
| Event category   | Administrative          |
| Operation status | All                     |

If no events appear, increase the time range to seven days and remove the resource-group filter.

### **What to observe**

Select an event and inspect:

- Operation name
- Timestamp
- Caller
- Status
- Resource ID
- Correlation ID

**Screenshot:** `01-activity-log-overview.png`

## **Task 2 — Generate a resource group update event**

**Estimated time:** 20 minutes

We'll create a harmless administrative change that should generate an Activity Log event.

### **Step 1 — Open Azure Cloud Shell**

Choose Bash.

### **Step 2 — Verify the resource group**

```
az group show \
  --name rg-az104-test-cin-001 \
  --query "{Name:name,Location:location,Tags:tags}" \
  -o json
```

### **Step 3 — Add a temporary tag**

Use the resource-group Tags API:

```
RG_SCOPE=$(az group show \
  --name rg-az104-test-cin-001 \
  --query id \
  -o tsv)

az tag update \
  --resource-id "$RG_SCOPE" \
  --operation Merge \
  --tags AuditLab=Phase01Lab06
```

### **Step 4 — Verify the tag**

```
az group show \
  --name rg-az104-test-cin-001 \
  --query tags.AuditLab \
  -o tsv
```

Expected:

```
Phase01Lab06
```

### **Step 5 — Find the Activity Log event**

Return to **Monitor → Activity log**.

Filter by the test resource group and look for an operation related to updating tags or resource-group metadata.

Depending on the API used, the event may appear under an operation such as `Microsoft.Resources/tags/write`.

**Screenshot:** `02-resource-tag-update-event.png`

Activity Log ingestion can take several minutes. If the event does not appear immediately, wait and refresh.

## **Task 3 — Investigate Azure RBAC changes**

**Estimated time:** 20 minutes

In Phase 00, you created:

- `az104-dev-group` — Contributor
- `az104-ops-group` — Reader
- `az104-audit-group` — Reader

In Lab 02, you also created a custom role assignment.

We'll investigate those changes using Activity Log.

### **Step 1 — Filter RBAC operations**

Navigate to:

**Monitor → Activity log**

Set:

- Time range: Last 7 days, or the period covering your role assignments.
- Category: Administrative.
- Operation: Search for role assignment write operations.

Look for:

```
Microsoft.Authorization/roleAssignments/write
```

### **Step 2 — Inspect the event**

Open the event and record:

| Property       | What to record                     |
| -------------- | ---------------------------------- |
| Operation      | Role assignment write              |
| Status         | Succeeded or Failed                |
| Caller         | Identity that initiated the change |
| Scope          | Resource group or subscription     |
| Timestamp      | Event time                         |
| Correlation ID | Related operation identifier       |

### **Important distinction**

An Activity Log event shows that a role assignment operation occurred.

To determine the **current permissions**, you must also inspect Azure RBAC role assignments. Historical events alone do not prove that an assignment still exists.

### **Step 3 — Validate current RBAC**

```
RG_SCOPE=$(az group show \
  --name rg-az104-dev-cin-001 \
  --query id \
  -o tsv)

az role assignment list \
  --scope "$RG_SCOPE" \
  --query "[].{Role:roleDefinitionName,Principal:principalName}" \
  -o table
```

**Screenshot:** `03-rbac-activity-log.png`

## **Task 4 — Investigate Azure Policy activity**

**Estimated time:** 20 minutes

In Lab 04, you created the governance initiative `az104-governance-baseline`. In Lab 05, we configured a Modify policy for tag inheritance.

Now we'll investigate policy-related administrative events.

### **Step 1 — Open Activity Log**

Navigate to **Monitor → Activity log**.

Set the time range to include your previous policy labs.

Search for operations such as:

```
Microsoft.Authorization/policyAssignments/write
Microsoft.Authorization/policyAssignments/delete
Microsoft.Authorization/policySetDefinitions/write
```

The first two represent policy assignment changes. The third represents changes to initiative definitions.

### **Step 2 — Inspect an event**

Open a relevant event and record its operation, caller, timestamp, status, and correlation ID.

### **Step 3 — Compare historical activity with current configuration**

Run:

```
az policy assignment list \
  --resource-group rg-az104-test-cin-001 \
  --query "[].{Name:name,DisplayName:displayName}" \
  -o table
```

**What you'll learn:** Activity Log helps establish that a management operation occurred, while the current policy assignment tells you what is configured now.

**Screenshot:** `04-policy-assignment-activity.png`

## **Task 5 — Investigate a failed authorization operation**

**Estimated time:** 20 minutes

In Lab 03, we tested a restricted custom RBAC role using a synthetic identity.

If you captured an `AuthorizationFailed` error during that exercise, we'll investigate it.

### **Step 1 — Review the previous error**

Look for details such as:

```
AuthorizationFailed

Action:
Microsoft.Resources/subscriptions/resourceGroups/write

Target:
rg-az104-rbac-denied-cin-001
```

### **Step 2 — Search Activity Log**

Open **Monitor → Activity log** and filter for the approximate time of the failed request.

Set **Status = Failed** and inspect relevant events.

Not every authorization failure is guaranteed to produce an easily discoverable Activity Log event. If the operation was rejected before an Azure Resource Manager event was recorded, the CLI error itself may be your primary evidence.

### **Step 3 — Correlate the evidence**

| Evidence                       | Purpose                                      |
| ------------------------------ | -------------------------------------------- |
| CLI error                      | Identifies denied action                     |
| Activity Log event, if present | Shows platform-recorded operation            |
| RBAC assignment                | Shows assigned permissions                   |
| Custom role definition         | Shows allowed actions                        |
| Azure Policy                   | Identifies policy restrictions               |
| Resource locks                 | Identifies management-operation restrictions |

### **Expected outcome**

Explain why the test identity could read or tag the resource group but could not create a new resource group.

**Screenshot:** `05-authorization-investigation.png`

## **Task 6 — Investigate events using Azure CLI**

**Estimated time:** 25 minutes

Azure CLI provides `az monitor activity-log` commands to retrieve and filter Azure Activity Log events.

### **Command 1 — List recent events**

```
az monitor activity-log list \
  --offset 1d \
  --max-events 20 \
  --query "[].{Time:eventTimestamp,Operation:operationName.localizedValue,Status:status.value}" \
  -o table
```

This displays up to 20 recent events from the current subscription.

### **Command 2 — Filter by resource group**

```
az monitor activity-log list \
  --resource-group rg-az104-test-cin-001 \
  --offset 7d \
  --max-events 50 \
  --query "[].{Time:eventTimestamp,Operation:operationName.localizedValue,Status:status.value}" \
  -o table
```

### **Command 3 — Find failed events**

```
az monitor activity-log list \
  --offset 7d \
  --status Failed \
  --max-events 50 \
  --query "[].{Time:eventTimestamp,Operation:operationName.localizedValue,Status:status.value}" \
  -o table
```

### **Command 4 — Filter RBAC role assignment changes**

```
az monitor activity-log list \
  --offset 7d \
  --max-events 100 \
  --query "[?operationName.value=='Microsoft.Authorization/roleAssignments/write'].{Time:eventTimestamp,Status:status.value,Resource:resourceId}" \
  -o table
```

### **Command 5 — Retrieve a specific event**

First, list event IDs:

```
az monitor activity-log list \
  --offset 1d \
  --max-events 10 \
  --query "[].{EventId:eventDataId,Operation:operationName.localizedValue}" \
  -o table
```

Then use an actual event ID:

```
az monitor activity-log show \
  --event-data-id "<event-data-id>" \
  -o json
```

Replace the placeholder with an ID returned by your subscription.

**Important:** Event details may include caller identifiers, subscription IDs, resource IDs, and other sensitive information. Don't upload raw JSON to a public GitHub repository.

## **Task 7 — Generate an Azure audit report using Bash**

**Estimated time:** 25 minutes

Now we'll automate Activity Log retrieval.

### **Step 1 — Create the script**

In Cloud Shell Bash:

```
nano azure-activity-audit.sh
```

Paste:

```

#!/usr/bin/env bash
set -euo pipefail

RG_NAME="rg-az104-test-cin-001"
OUTPUT_DIR="audit-output"

mkdir -p "$OUTPUT_DIR"

echo "======================================"
echo " AZ-104 Azure Activity Audit Report"
echo "======================================"

echo ""
echo "1. Active subscription"

az account show \
  --query "{Name:name,State:state}" \
  -o table

echo ""
echo "2. Recent administrative events"

az monitor activity-log list \
  --resource-group "$RG_NAME" \
  --offset 7d \
  --max-events 100 \
  --query "[].{Time:eventTimestamp,Operation:operationName.localizedValue,Status:status.value}" \
  -o table

echo ""
echo "3. Exporting audit data"

az monitor activity-log list \
  --resource-group "$RG_NAME" \
  --offset 7d \
  --max-events 100 \
  -o json > "$OUTPUT_DIR/activity-log.json"

echo ""
echo "4. Generating CSV report"

az monitor activity-log list \
  --resource-group "$RG_NAME" \
  --offset 7d \
  --max-events 100 \
  --query "[].{Time:eventTimestamp,Operation:operationName.value,Status:status.value,Category:category.value}" \
  -o json > "$OUTPUT_DIR/audit-summary.json"

python3 - "$OUTPUT_DIR/audit-summary.json" \
  "$OUTPUT_DIR/activity-summary.csv" <<'PY'
import csv
import json
import sys

source, destination = sys.argv[1:3]

with open(source, encoding="utf-8") as file:
    events = json.load(file)

fields = ["Time", "Operation", "Status", "Category"]

with open(destination, "w", newline="", encoding="utf-8") as file:
    writer = csv.DictWriter(file, fieldnames=fields)
    writer.writeheader()
    writer.writerows(events)

print(f"Exported {len(events)} events")
PY

echo ""
echo "5. Audit report files"
ls -lh "$OUTPUT_DIR"

echo ""
echo "Audit report generation completed."
echo "Review and redact files before sharing."

```

### **Step 2 — Make it executable**

```
chmod +x azure-activity-audit.sh
```

### **Step 3 — Run the script**

```
./azure-activity-audit.sh
```

### **Step 4 — Inspect the generated files**

```
ls -lh audit-output/
```

```
head -n 10 audit-output/activity-summary.csv
```

### **Expected output files**

```
audit-output/
├── activity-log.json
├── audit-summary.json
└── activity-summary.csv
```

The CSV is a useful starting point for your GitHub project. However, the full JSON exports may contain sensitive information and should remain private.

The script retrieves a maximum of 100 events in its seven-day query. For a large enterprise audit, you would need appropriate pagination or centralized logging to avoid incomplete reporting.

# **5. Optional advanced exercise — Activity Log export**

In a production environment, Azure administrators often forward Activity Logs to:

- Log Analytics workspaces
- Azure Storage accounts
- Azure Event Hubs

This is configured through **Azure Monitor diagnostic settings**.

For your $200 Free Trial, we'll keep this as a design exercise because Log Analytics ingestion and retention can incur charges.

In a future monitoring phase, we'll implement Log Analytics and KQL using a controlled data volume.

## **6. Real-world troubleshooting scenarios**

| Scenario                      | Investigation                              | Resolution or conclusion                                |
| ----------------------------- | ------------------------------------------ | ------------------------------------------------------- |
| Unknown resource modification | Filter Activity Log by resource and time   | Identify operation and caller                           |
| Unexpected RBAC assignment    | Search role assignment write events        | Review current assignments and approval                 |
| Policy changed unexpectedly   | Inspect policy assignment events           | Compare current configuration                           |
| Failed resource deployment    | Review failed operations and error details | Identify RBAC, Policy, quota, or configuration issue    |
| No Activity Log event found   | Check time, scope, category, and retention | Use deployment errors or other appropriate logs         |
| User sign-in failure          | Check Entra sign-in logs                   | Investigate authentication, not Azure resource activity |
| Group membership changed      | Check Entra audit logs                     | Identify directory modification                         |

## **7. GitHub repository structure**

```
phase-01-identity-governance/
|
|-- lab-06-activity-logs/
|   |-- README.md
|   |-- screenshots/
|       |-- 01-activity-log-overview.png
```

Add this to your repository `.gitignore`:

```
audit-output/
activity-log.json
audit-summary.json
```

Only commit a sanitized report after reviewing the data.

### **Git commit**

```
git add phase-01-identity-governance/lab-06-activity-logs/

git commit -m "feat(az104): implement Azure Activity Log investigation and audit reporting"

git push origin main
```

## **8. AZ-104 interview questions**

1\. What is Azure Activity Log?

2\. What is the default Azure Activity Log retention period?

3\. What is the difference between Activity Log and Entra audit logs?

4\. What is a correlation ID?

5\. Does Activity Log record every application request?

6\. How would you investigate an unauthorized RBAC assignment?

7\. How can Activity Logs be retained beyond 90 days?

8\. Why might an authorization failure not appear in Activity Log?

## **9. Lab 06 completion checklist**

**Lab 06 Progress**



Explored Azure Activity Log in the Portal

Generated a temporary resource-group tag update

Located and inspected the corresponding event

Investigated historical RBAC changes

Investigated Azure Policy assignment activity

Reviewed a previous authorization failure

Executed Activity Log CLI queries

Created and executed azure-activity-audit.sh

Generated and reviewed CSV/JSON audit reports

Removed the temporary AuditLab tag

Prepared sanitized GitHub documentation





### **What you've built in Phase 01**

| Lab | Enterprise skill                                       |
| --- | ------------------------------------------------------ |
| 01  | Entra ID user lifecycle management                     |
| 02  | Custom RBAC and least privilege                        |
| 03  | RBAC troubleshooting and effective access              |
| 04  | Azure Policy initiatives and compliance                |
| 05  | Automated tag inheritance and remediation              |
| 06  | Administrative auditing and Activity Log investigation |

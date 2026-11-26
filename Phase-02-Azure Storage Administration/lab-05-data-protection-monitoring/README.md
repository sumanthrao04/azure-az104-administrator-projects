# **AZ-104 — Phase 02 | Lab 05: Azure Storage Data Protection, Lifecycle Management & Monitoring**

**Project: Enterprise Storage Recovery, Cost Optimization & Observability**

This is the final lab in **Phase 02 — Azure Storage Administration**.

In previous labs, we covered storage account security, Blob Storage, Azure Files, Queue Storage, and Table Storage.

Now we'll focus on protecting enterprise data, automating storage cost optimization, recovering accidentally deleted files, transferring data using AzCopy, and monitoring storage performance.

Duration

**3–4 hours**

Additional cost target

**Under $2**

Storage account

**appsharedfiles**

Region

**Central India**

**Resource group:** `rg-az104-storage-cin-001`

**AZ-104 exam priority:** High

We'll reuse your existing storage account. Costs depend on transactions, retained versions, monitoring configuration and data usage. Check your trial expiration before starting.

## **1. Real-world scenario**

Imagine you're working as an Azure Cloud Administrator for a financial services company.

The company stores application documents, customer reports, configuration files, and historical logs in Azure Storage.

The organization faces several challenges:

- Developers accidentally delete important files.
- Application configuration files are overwritten incorrectly.
- Old reports remain in expensive storage tiers.
- Large amounts of historical data increase monthly costs.
- The operations team has limited visibility into storage performance.
- Administrators need to transfer files between local servers and Azure Storage.

Your manager asks you to implement the following solutions:

| Business requirement                 | Azure solution        |
| ------------------------------------ | --------------------- |
| Recover deleted blobs                | Blob soft delete      |
| Recover overwritten files            | Blob versioning       |
| Recover deleted containers           | Container soft delete |
| Automatically optimize storage costs | Lifecycle management  |
| Transfer files efficiently           | AzCopy                |
| Monitor storage performance          | Azure Monitor metrics |
| Detect storage failures              | Azure Monitor alerts  |
| Investigate administrative changes   | Azure Activity Log    |

## **2. Architecture — What we're building**

Target architecture. We'll implement recovery and lifecycle controls on isolated test data without affecting existing application files.

## **3. Topics covered in Lab 05**

| Module | Topic                                      |
| ------ | ------------------------------------------ |
| 01     | Blob soft delete                           |
| 02     | Blob versioning and restoration            |
| 03     | Container soft delete                      |
| 04     | Lifecycle management policies              |
| 05     | Storage access tiers and cost optimization |
| 06     | AzCopy data transfer                       |
| 07     | Azure Storage metrics                      |
| 08     | Azure Monitor alerts                       |
| 09     | Diagnostic settings                        |
| 10     | Resource cleanup and cost verification     |

# **4. Hands-on implementation**

## **Task 1 — Verify your existing storage account**

Open Azure Cloud Shell in Bash mode or use your authorized local Azure CLI environment.

```
RG_NAME="rg-az104-storage-cin-001"
STORAGE_NAME="appsharedfiles"
CONTAINER_NAME="recovery-lab"
```

Verify the storage account:

```
az storage account show \
  --name "$STORAGE_NAME" \
  --resource-group "$RG_NAME" \
  --query "{Name:name,Kind:kind,SKU:sku.name,Location:location}" \
  -o table
```

### **Verify your permissions**

Because you encountered data-plane authorization errors in Lab 04, make sure your current login identity has the following permissions:

| Role                          | Purpose                                                     |
| ----------------------------- | ----------------------------------------------------------- |
| Storage Blob Data Contributor | Upload, modify, delete and recover supported blob data      |
| Contributor or Owner          | Configure storage account protection and lifecycle settings |
| Monitoring Reader             | Read Azure Monitor data, if not already permitted           |

Also verify that the storage firewall permits your client network.

### **Create a dedicated recovery container**

```
az storage container create \
  --account-name "$STORAGE_NAME" \
  --name "$CONTAINER_NAME" \
  --auth-mode login
```

We'll use this container exclusively for recovery testing.

## **Task 2 — Configure Blob soft delete and versioning**

**Estimated time:** 15 minutes

Navigate to:

**Azure Portal → appsharedfiles → Data management → Data protection**

Configure the following settings:

| Setting                           | Value                 |
| --------------------------------- | --------------------- |
| Enable soft delete for blobs      | Enabled               |
| Blob retention period             | 7 days                |
| Enable soft delete for containers | Enabled               |
| Container retention period        | 7 days                |
| Enable versioning for blobs       | Enabled               |
| Point-in-time restore             | Disabled for this lab |

You may already have some of these settings enabled from Lab 02. Verify them rather than configuring them again unnecessarily.

### **Configure using Azure CLI**

```
az storage account blob-service-properties update \
  --account-name "$STORAGE_NAME" \
  --resource-group "$RG_NAME" \
  --enable-delete-retention true \
  --delete-retention-days 7 \
  --enable-container-delete-retention true \
  --container-delete-retention-days 7 \
  --enable-versioning true
```

### **Validate**

```
az storage account blob-service-properties show \
  --account-name "$STORAGE_NAME" \
  --resource-group "$RG_NAME" \
  --query "{Versioning:isVersioningEnabled,BlobSoftDelete:deleteRetentionPolicy,ContainerSoftDelete:containerDeleteRetentionPolicy}" \
  -o json
```

**Screenshot:** `01-storage-data-protection.png`

## **Task 3 — Test Blob versioning and recovery**

**Estimated time:** 25 minutes

### **Real-world incident**

A developer accidentally overwrites an application's configuration file with incorrect values.

We need to recover the previous working configuration.

### **Step 1 — Create the original file**

```
cat > application.properties <<'EOF'
server.port=8080
spring.application.name=az104-demo
logging.level.root=INFO
EOF
```

### **Step 2 — Upload the original file**

```
az storage blob upload \
  --account-name "$STORAGE_NAME" \
  --container-name "$CONTAINER_NAME" \
  --name application.properties \
  --file application.properties \
  --auth-mode login
```

### **Step 3 — Simulate an incorrect configuration change**

```
cat > application.properties <<'EOF'
server.port=9090
spring.application.name=az104-demo
logging.level.root=DEBUG
EOF
```

Upload the modified file:

```
az storage blob upload \
  --account-name "$STORAGE_NAME" \
  --container-name "$CONTAINER_NAME" \
  --name application.properties \
  --file application.properties \
  --overwrite true \
  --auth-mode login
```

### **Step 4 — View the version history**

Navigate to:

**appsharedfiles → Containers → recovery-lab → application.properties → Versions**

**application.properties — Expected version history**

**Version 2 — Current**

server.port=9090 · DEBUG

**Version 1 — Previous**

server.port=8080 · INFO

### **Azure CLI validation**

```
az storage blob list \
  --account-name "$STORAGE_NAME" \
  --container-name "$CONTAINER_NAME" \
  --include v \
  --auth-mode login \
  --query "[].{Name:name,Version:versionId,Current:isCurrentVersion}" \
  -o table
```

### **Step 5 — Restore the previous version**

In Azure Portal, select the original version and use **Make current version**, if available.

Alternatively, download the original version using its version ID and upload it again as the current blob.

```
VERSION_ID="<previous-version-id>"
```

```
az storage blob download \
  --account-name "$STORAGE_NAME" \
  --container-name "$CONTAINER_NAME" \
  --name application.properties \
  --version-id "$VERSION_ID" \
  --file restored.properties \
  --auth-mode login
```

Verify the content:

```
cat restored.properties
```

Upload the recovered configuration:

```
az storage blob upload \
  --account-name "$STORAGE_NAME" \
  --container-name "$CONTAINER_NAME" \
  --name application.properties \
  --file restored.properties \
  --overwrite true \
  --auth-mode login
```

**Expected outcome:** The original configuration with port `8080` is restored.

**Screenshot:** `02-blob-version-recovery.png`

## **Task 4 — Recover an accidentally deleted blob**

**Estimated time:** 20 minutes

### **Step 1 — Create a test file**

```
echo "AZ104 Important Application Report" > recovery-test.txt
```

### **Step 2 — Upload it**

```
az storage blob upload \
  --account-name "$STORAGE_NAME" \
  --container-name "$CONTAINER_NAME" \
  --name recovery-test.txt \
  --file recovery-test.txt \
  --auth-mode login
```

### **Step 3 — Simulate accidental deletion**

```
az storage blob delete \
  --account-name "$STORAGE_NAME" \
  --container-name "$CONTAINER_NAME" \
  --name recovery-test.txt \
  --auth-mode login
```

### **Step 4 — Locate the deleted blob**

Open:

**Storage account → Containers → recovery-lab**

Enable the option to show deleted blobs and inspect the version history.

### **Step 5 — Recover the blob**

Because we enabled both versioning and soft delete, deleting the current blob can leave its previous version available without a current version.

Select the appropriate previous version and promote it to the current version, or download and re-upload it.

For a genuinely soft-deleted blob or version, use the supported undelete operation first where applicable.

### **Recovery workflow**

Original blob

Deleted

Retained data

Recovered

### **Step 6 — Validate recovery**

```
az storage blob exists \
  --account-name "$STORAGE_NAME" \
  --container-name "$CONTAINER_NAME" \
  --name recovery-test.txt \
  --auth-mode login
```

Expected:

```
{
  "exists": true
}
```

**Screenshot:** `03-deleted-blob-recovery.png`

### **Important AZ-104 concept**

| Feature       | Purpose                                                        |
| ------------- | -------------------------------------------------------------- |
| Soft delete   | Retains supported deleted data for a configured period         |
| Versioning    | Retains previous versions of blobs                             |
| Snapshot      | Read-only point-in-time copy                                   |
| Resource lock | Protects management-plane resources against certain operations |
| Redundancy    | Protects against specified infrastructure failures             |

Redundancy is not a replacement for backup or recovery controls. A logical deletion can also be replicated.

## **Task 5 — Test container soft delete**

**Estimated time:** 15 minutes

We'll create a separate temporary container so your existing data remains safe.

### **Step 1 — Create a test container**

```
az storage container create \
  --account-name "$STORAGE_NAME" \
  --name container-recovery-test \
  --auth-mode login
```

### **Step 2 — Upload a sample file**

```
az storage blob upload \
  --account-name "$STORAGE_NAME" \
  --container-name container-recovery-test \
  --name recovery-test.txt \
  --file recovery-test.txt \
  --auth-mode login
```

### **Step 3 — Delete the test container**

```
az storage container delete \
  --account-name "$STORAGE_NAME" \
  --name container-recovery-test \
  --auth-mode login
```

### **Step 4 — Restore the container**

Navigate to:

**appsharedfiles → Containers → Show deleted containers**

Select `container-recovery-test` and click **Undelete**.

### **Step 5 — Verify the recovered file**

```
az storage blob list \
  --account-name "$STORAGE_NAME" \
  --container-name container-recovery-test \
  --auth-mode login \
  -o table
```

**Expected outcome:** The deleted container and its sample file are available again.

**Screenshot:** `04-container-recovery.png`

# **5. Azure Storage Lifecycle Management**

## **Task 6 — Configure automated lifecycle policies**

**Estimated time:** 25 minutes

### **Real-world scenario**

Your organization stores application logs and reports for long periods.

Recently generated files are accessed frequently, but older files are rarely accessed.

Keeping everything in the Hot tier increases storage costs.

Azure Lifecycle Management can automatically transition eligible blobs between storage tiers or delete them according to configured rules.

### **Storage access tiers**

| Feature                         | Hot          | Cool              | Cold        | Archive            |
| ------------------------------- | ------------ | ----------------- | ----------- | ------------------ |
| Storage cost                    | Higher       | Lower             | Lower       | Lowest             |
| Access cost                     | Lower        | Higher            | Higher      | Higher             |
| Availability                    | Online       | Online            | Online      | Offline            |
| Minimum retention consideration | None         | 30 days           | 90 days     | 180 days           |
| Typical use                     | Active files | Occasional access | Rare access | Long-term archives |

### **Enterprise data lifecycle**

**Hot**

Day 0 — Frequently accessed data

**Cool**

Day 30 — Infrequently accessed data

**Cold**

Day 90 — Rarely accessed data

**Archive**

Day 180 — Long-term retention

Illustrative enterprise lifecycle. Actual transition dates depend on access patterns, retention requirements and pricing.

### **Step 1 — Open lifecycle management**

Navigate to:

**Storage account → Data management → Lifecycle management → + Add a rule**

### **Step 2 — Configure rule details**

| Field         | Value                     |
| ------------- | ------------------------- |
| Rule name     | `az104-blob-lifecycle`    |
| Rule scope    | Limit blobs with filters  |
| Blob type     | Block blobs               |
| Blob subtype  | Base blobs                |
| Prefix filter | `recovery-lab/lifecycle/` |

### **Step 3 — Configure lifecycle actions**

| Condition                            | Action          |
| ------------------------------------ | --------------- |
| Last modified more than 30 days ago  | Move to Cool    |
| Last modified more than 90 days ago  | Move to Cold    |
| Last modified more than 180 days ago | Move to Archive |
| Last modified more than 365 days ago | Delete          |

These are demonstration settings, not a recommended retention policy for actual financial records.

### **Step 4 — Create using JSON (alternative)**

Create a file named `lifecycle-policy.json`.

```

{
  "rules": [
    {
      "enabled": true,
      "name": "az104-blob-lifecycle",
      "type": "Lifecycle",
      "definition": {
        "filters": {
          "blobTypes": [
            "blockBlob"
          ],
          "prefixMatch": [
            "recovery-lab/lifecycle/"
          ]
        },
        "actions": {
          "baseBlob": {
            "tierToCool": {
              "daysAfterModificationGreaterThan": 30
            },
            "tierToCold": {
              "daysAfterModificationGreaterThan": 90
            },
            "tierToArchive": {
              "daysAfterModificationGreaterThan": 180
            },
            "delete": {
              "daysAfterModificationGreaterThan": 365
            }
          }
        }
      }
    }
  ]
}

```

Before applying it, check whether your account already has a lifecycle policy:

```
az storage account management-policy show \
  --account-name "$STORAGE_NAME" \
  --resource-group "$RG_NAME"
```

If a policy already exists, preserve its existing rules and add the new rule rather than replacing the entire policy.

If no policy exists, create it:

```
az storage account management-policy create \
  --account-name "$STORAGE_NAME" \
  --resource-group "$RG_NAME" \
  --policy @lifecycle-policy.json
```

### **Step 5 — Validate the policy**

```
az storage account management-policy show \
  --account-name "$STORAGE_NAME" \
  --resource-group "$RG_NAME" \
  -o json
```

**Important:** Lifecycle management runs asynchronously. You will not see a newly uploaded blob transition immediately under these rules. We'll validate the policy configuration rather than wait 30 days.

Also, these actions target base blobs only. Previous versions and snapshots require separate lifecycle actions if you want to manage their retention automatically.

**Screenshot:** `05-lifecycle-management-policy.png`

# **6. AzCopy — Azure Storage Data Transfer**

## **Task 7 — Transfer files using AzCopy**

**Estimated time:** 25 minutes

AzCopy is a command-line utility designed to transfer data to, from, and between supported Azure Storage services.

This is an important practical skill for AZ-104.

### **Step 1 — Verify AzCopy**

```
azcopy --version
```

If AzCopy is unavailable, install it using the [official Microsoft AzCopy guide](https://learn.microsoft.com/en-us/azure/storage/common/storage-use-azcopy-v10?utm_source=chatgpt.com).

### **Step 2 — Authenticate using Azure CLI credentials**

From Bash:

```
export AZCOPY_AUTO_LOGIN_TYPE=AZCLI
```

Make sure your Azure CLI session is authenticated and has the required Blob data permissions.

### **Step 3 — Create sample files**

```
mkdir -p azcopy-demo
```

```
echo "Application log 01" > azcopy-demo/app01.log
echo "Application log 02" > azcopy-demo/app02.log
echo "Application log 03" > azcopy-demo/app03.log
```

### **Step 4 — Upload files**

```
azcopy copy \
  "./azcopy-demo/*" \
  "https://appsharedfiles.blob.core.windows.net/recovery-lab/lifecycle/" \
  --recursive=true
```

### **Step 5 — List uploaded blobs**

```
azcopy list \
  "https://appsharedfiles.blob.core.windows.net/recovery-lab/lifecycle/"
```

### **Step 6 — Download the files**

```
mkdir -p azcopy-download
```

```
azcopy copy \
  "https://appsharedfiles.blob.core.windows.net/recovery-lab/lifecycle/*" \
  "./azcopy-download/" \
  --recursive=true
```

### **Step 7 — Verify files**

```
ls -lh azcopy-download/
```

```
diff -r azcopy-demo azcopy-download
```

If the files and directory contents match, `diff` produces no output.

### **Common AzCopy commands**

| Command              | Purpose                            |
| -------------------- | ---------------------------------- |
| `azcopy copy`        | Upload, download or copy data      |
| `azcopy sync`        | Synchronize source and destination |
| `azcopy list`        | List supported storage contents    |
| `azcopy jobs list`   | List transfer jobs                 |
| `azcopy jobs show`   | Inspect transfer job details       |
| `azcopy jobs resume` | Resume an eligible failed transfer |

**Interview takeaway:** AzCopy is particularly useful for bulk data transfers, migration activities, backups, and storage automation.

**Screenshot:** `06-azcopy-transfer-success.png`

# **7. Azure Storage Monitoring**

## **Task 8 — Configure Azure Monitor metrics**

**Estimated time:** 20 minutes

Navigate to:

**appsharedfiles → Monitoring → Metrics**

Explore the following metrics.

| Metric                 | Purpose                                   |
| ---------------------- | ----------------------------------------- |
| Transactions           | Number of storage requests                |
| Availability           | Percentage of available requests          |
| Success E2E Latency    | End-to-end latency of successful requests |
| Success Server Latency | Server-side processing time               |
| Ingress                | Incoming data volume                      |
| Egress                 | Outgoing data volume                      |
| Used Capacity          | Storage capacity consumed                 |
| Blob Capacity          | Capacity used by Blob Storage             |
| Blob Count             | Number of blobs                           |

### **Create your first metric chart**

Configure:

| Field            | Value                           |
| ---------------- | ------------------------------- |
| Scope            | appsharedfiles                  |
| Metric namespace | Account or Blob service metrics |
| Metric           | Transactions                    |
| Aggregation      | Sum                             |
| Time range       | Last 1 hour                     |

### **Generate storage activity**

```
for i in {1..10}; do
  az storage blob list \
    --account-name "$STORAGE_NAME" \
    --container-name "$CONTAINER_NAME" \
    --auth-mode login \
    -o none
done
```

Refresh the metrics after allowing time for data collection.

### **Storage Transactions**

Illustrative monitoring chart — not live Azure data

0510152010:0010:0510:1010:1510:2010:2510:3010:35

### **Azure CLI validation**

```
STORAGE_SCOPE=$(az storage account show \
  --name "$STORAGE_NAME" \
  --resource-group "$RG_NAME" \
  --query id -o tsv)
```

List supported metrics:

```
az monitor metrics list-definitions \
  --resource "$STORAGE_SCOPE" \
  --query "[].{Metric:name.value,Unit:unit}" \
  -o table
```

Retrieve transaction metrics:

```
az monitor metrics list \
  --resource "$STORAGE_SCOPE" \
  --metric Transactions \
  --aggregation Total \
  --interval PT5M \
  -o json
```

**Screenshot:** `07-storage-metrics.png`

## **Task 9 — Configure Azure Monitor alerts**

**Estimated time:** 20 minutes

### **Real-world scenario**

The operations team needs notifications when storage availability drops or storage request failures increase.

Navigate to:

**Storage account → Monitoring → Alerts → Create → Alert rule**

### **Configure a test alert**

| Setting              | Value                              |
| -------------------- | ---------------------------------- |
| Alert name           | `alert-az104-storage-transactions` |
| Scope                | appsharedfiles                     |
| Signal               | Transactions                       |
| Aggregation          | Total                              |
| Operator             | Greater than                       |
| Threshold            | 5                                  |
| Evaluation frequency | 5 minutes                          |
| Lookback period      | 5 minutes                          |
| Severity             | Sev 3 — Informational              |

This low threshold is only for testing alert functionality, not a production recommendation.

### **Create an Action Group**

| Field             | Value                     |
| ----------------- | ------------------------- |
| Action group name | `ag-az104-storage-alerts` |
| Display name      | `az104store`              |
| Notification type | Email                     |
| Recipient         | Your email address        |

### **Alert architecture**

**Azure Storage**

Generates transaction metrics

**Azure Monitor**

Evaluates metric condition

**Alert Rule**

Threshold exceeded

**Action Group**

Send email notification

**Operations Engineer**

Generate another small batch of storage transactions and observe the alert evaluation.

**Cost warning:** Azure Monitor alert rules can incur charges. Review the estimated price before creation and delete the test rule after validation. If you choose not to incur the cost, document the configuration as a design exercise.

**Screenshot:** `08-storage-alert-rule.png`

## **Task 10 — Understand diagnostic settings**

Azure Monitor metrics provide numerical measurements, while resource logs provide more detailed information about service operations.

### **Metrics vs Activity Log vs Resource Logs**

| Component           | Purpose                                                                |
| ------------------- | ---------------------------------------------------------------------- |
| Metrics             | Performance and availability measurements                              |
| Activity Log        | Azure management-plane operations                                      |
| Resource logs       | Service-specific operations, including supported storage data requests |
| Diagnostic settings | Route logs and metrics to supported destinations                       |
| Log Analytics       | Centralized log storage and KQL queries                                |

### **Production monitoring architecture**

Conceptual production architecture. Diagnostic settings for storage data operations are configured at the relevant storage service scope.

### **Example KQL query for Blob Storage logs**

```
StorageBlobLogs
| where TimeGenerated > ago(1h)
| where StatusCode >= 400
| summarize FailedRequests = count()
    by OperationName, StatusCode
| order by FailedRequests desc
```

This query requires Blob resource logs to be routed to a Log Analytics workspace using the appropriate resource-specific table configuration.

We'll implement Log Analytics in **Phase 05 — Monitoring & Recovery** rather than creating unnecessary billable monitoring infrastructure now.

# **8. Real-world troubleshooting scenarios**

| Problem                           | Possible cause                              | Resolution                                 |
| --------------------------------- | ------------------------------------------- | ------------------------------------------ |
| Deleted blob cannot be restored   | Retention expired or protection disabled    | Check soft delete and versions             |
| Previous version unavailable      | Versioning was not enabled before overwrite | Review protection settings                 |
| Lifecycle policy not working      | Incorrect prefix or evaluation pending      | Inspect filters and policy configuration   |
| Archive blob cannot be downloaded | Blob is offline                             | Rehydrate to an online tier                |
| AzCopy returns 403                | RBAC or firewall restriction                | Verify data permissions and client network |
| Storage costs increasing          | Retained versions, snapshots or unused data | Review capacity and lifecycle rules        |
| High E2E latency                  | Client network or service latency           | Compare server and E2E latency             |
| Alert not triggered               | Threshold or evaluation window incorrect    | Inspect alert conditions                   |
| No storage logs in Log Analytics  | Diagnostic settings not configured          | Verify service-level log routing           |

## **Production interview scenario**

**Incident:** An application team reports that Azure Blob Storage requests are taking significantly longer than usual.

How would you troubleshoot?

I would first check Azure Monitor metrics for availability, transaction volume, Success E2E Latency and Success Server Latency.

| Observation                             | Area to investigate                                       |
| --------------------------------------- | --------------------------------------------------------- |
| High E2E latency, normal server latency | Client, network or application behavior                   |
| High E2E and server latency             | Storage service processing or workload pressure           |
| Increasing failed transactions          | Authentication, authorization, throttling or other errors |
| Sudden increase in transactions         | Application traffic or retry behavior                     |
| Increasing capacity                     | Data growth and retention configuration                   |

Next, I would review resource logs if enabled, check storage firewall configuration, inspect application retry behavior, and investigate Azure Service Health.

After identifying the root cause, I would apply the appropriate resolution and monitor the metrics to confirm recovery.

# **9. GitHub repository structure**

```
phase-02-storage/
│
└── lab-05-data-protection-monitoring/
    ├── README.md
    ├── commands.md
    ├── troubleshooting.md
    ├── interview-questions.md
    ├── cleanup.md
    │
    ├── architecture/
    │   └── storage-protection-architecture.png
    │
    ├── scripts/
    │   └── lifecycle-policy.json
    │
    ├── sample-data/
    │   ├── application.properties
    │   └── recovery-test.txt
    │
    └── screenshots/
        ├── 01-storage-data-protection.png
        ├── 02-blob-version-recovery.png
        ├── 03-deleted-blob-recovery.png
        ├── 04-container-recovery.png
        ├── 05-lifecycle-management-policy.png
        ├── 06-azcopy-transfer-success.png
        ├── 07-storage-metrics.png
        └── 08-storage-alert-rule.png
```

### **Git commit**

After completing and validating the exercises:

```
git add phase-02-storage/lab-05-data-protection-monitoring/

git commit -m "feat(az104): implement storage recovery lifecycle management AzCopy and monitoring"

git push origin main
```

# **10. AZ-104 & DevOps interview questions**

1\. What is Blob soft delete?

2\. What is Blob versioning?

3\. What is the difference between soft delete and versioning?

4\. What is Azure Storage Lifecycle Management?

5\. What is the difference between Hot, Cool, Cold and Archive tiers?

6\. Can you directly download an archived blob?

7\. What is AzCopy?

8\. What is the difference between AzCopy copy and sync?

9\. What is the difference between E2E latency and server latency?

10\. What is an Azure Monitor Action Group?

11\. What is the difference between Activity Log and resource logs?

12\. Does Azure Storage redundancy replace backups?

13\. Why can Blob versioning increase storage costs?

14\. How would you troubleshoot an AzCopy 403 error?

15\. How would you reduce Azure Storage costs?

# **11. Cleanup and cost management**

Since this is the final storage lab, review all resources created during Phase 02.

| Resource               | Recommended action                                                   |
| ---------------------- | -------------------------------------------------------------------- |
| Test Blob containers   | Delete after capturing evidence                                      |
| Previous blob versions | Review and remove unnecessary versions                               |
| Azure Files snapshots  | Remove unnecessary snapshots                                         |
| Azure file shares      | Delete if no longer needed                                           |
| Queue Storage          | Delete temporary queues                                              |
| Table Storage          | Delete synthetic tables                                              |
| Metric alert rule      | Delete after testing                                                 |
| Action Group           | Delete if no longer required                                         |
| Lifecycle policy       | Retain only if keeping the storage account and rules are appropriate |
| Storage account        | Delete if no longer required                                         |

Remember that soft-deleted data can continue consuming storage during its retention period.

Before deleting the storage account, ensure that all required evidence and sample data have been saved locally and that no other labs depend on it.

Review your remaining Azure credit and actual trial expiration date.

# **12. Lab 05 completion checklist**

**Lab 05 Progress**


Verified existing appsharedfiles storage account

Verified Blob data-plane permissions

Configured Blob and container soft delete

Verified Blob versioning

Recovered an overwritten configuration file

Recovered an accidentally deleted blob

Recovered a deleted test container

Created lifecycle management policy

Validated lifecycle policy configuration

Authenticated AzCopy using Entra credentials

Uploaded and downloaded files using AzCopy

Explored Azure Storage metrics

Configured and tested an alert or documented cost limitation

Understood diagnostic settings and Log Analytics

Completed cleanup and cost verification




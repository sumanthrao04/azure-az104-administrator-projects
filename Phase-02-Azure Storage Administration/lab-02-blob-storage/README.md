# **AZ-104 — Phase 02 | Lab 02: Azure Blob Storage Management & Access Control**

**Project: Secure Enterprise Blob Storage with Microsoft Entra ID, RBAC & SAS**

In Lab 01, we deployed a secure Azure StorageV2 account and configured storage security, networking, and RBAC.

Now we'll use that storage account to manage application files, secure access to Blob containers, test different authentication methods, and troubleshoot storage authorization failures.

Hands-on Lab

Duration: 3–4 hours

Subscription

**Azure Free Trial**

Region

**Central India**

Storage type

**Standard StorageV2 · LRS**

Additional cost target

**Under $1**

Estimate, not guaranteed

**Prerequisite:** Your storage account in `rg-az104-storage-cin-001` from Lab 01.

## **1. Real-world scenario**

Imagine you're working as an Azure Cloud Administrator for a financial services company.

The company has three teams that interact with application documents:

| Team            | Business requirement                     | Access              |
| --------------- | ---------------------------------------- | ------------------- |
| Development     | Upload and manage application files      | Read, write, delete |
| Operations      | Download files for troubleshooting       | Read only           |
| External vendor | Download one approved report temporarily | Time-limited access |

Your manager asks you to design a secure storage solution that:

- Stores files in private Blob containers.
- Uses Microsoft Entra ID and Azure RBAC for internal users.
- Provides temporary, restricted access to an external vendor.
- Prevents anonymous public access.
- Supports secure file uploads and downloads.
- Protects data from accidental deletion.

We'll implement all these requirements using your existing storage account.

## **2. Architecture — What we're building**

Internal users access private containers using Entra ID and Blob data roles. An external recipient receives temporary access to a single approved blob through SAS.

## **3. Topics we'll learn**

| Topic               | What you'll understand                       |
| ------------------- | -------------------------------------------- |
| Azure Blob Storage  | Object storage for unstructured data         |
| Containers          | Logical grouping of blobs                    |
| Block blobs         | Files, images, logs, documents               |
| Blob access tiers   | Hot, Cool, Cold, Archive                     |
| Azure RBAC          | Identity-based Blob data permissions         |
| SAS tokens          | Delegated, time-limited access               |
| User delegation SAS | SAS signed using Microsoft Entra credentials |
| Storage firewall    | Network access restrictions                  |
| Blob versioning     | Preserve previous blob versions              |
| Soft delete         | Recover deleted blobs                        |

### **Blob Storage hierarchy**

```
Azure Subscription
       |
       v
Resource Group
       |
       v
Storage Account
       |
       +-- Container: app-documents
       |       |
       |       +-- application-config.json
       |       +-- sample-report.txt
       |
       +-- Container: vendor-reports
               |
               +-- vendor-report.txt
```

**Important:** Containers are private by default when anonymous access is disabled. A private container can still be accessed by authorized users or clients with valid SAS credentials, subject to network restrictions.

# **4. Hands-on implementation**

## **Task 1 — Verify your existing storage account**

**Estimated time:** 10 minutes

Open [Azure Portal](https://portal.azure.com?utm_source=chatgpt.com).

Navigate to:

**Storage accounts → Your storage account → Overview**

Verify:

| Setting               | Expected value             |
| --------------------- | -------------------------- |
| Resource group        | `rg-az104-storage-cin-001` |
| Region                | Central India              |
| Performance           | Standard                   |
| Redundancy            | LRS                        |
| Account kind          | StorageV2                  |
| Secure transfer       | Enabled                    |
| Anonymous Blob access | Disabled                   |

### **Azure CLI validation**

Open Cloud Shell Bash:

```
RG_NAME="rg-az104-storage-cin-001"
STORAGE_NAME="<your-storage-account-name>"
```

Replace the placeholder with your actual storage account name.

```
az storage account show \
  --name "$STORAGE_NAME" \
  --resource-group "$RG_NAME" \
  --query "{Name:name,Kind:kind,SKU:sku.name,HTTPS:enableHttpsTrafficOnly,PublicBlob:allowBlobPublicAccess}" \
  -o json
```

### **Important networking check**

In Lab 01, you configured selected-network restrictions.

If you use Cloud Shell for Blob operations, its outbound IP might not be allowed by the storage firewall.

For this lab, prefer running data-plane commands from your local computer or another authorized client whose network is permitted. Do not weaken storage firewall settings just to make Cloud Shell work.

## **Task 2 — Create private Blob containers**

**Estimated time:** 15 minutes

### **Azure Portal steps**

1. Open your Storage account.
2. Under **Data storage**, select **Containers**.
3. Click **+ Container**.
4. Enter the name `app-documents`.
5. Set anonymous access level to **Private (no anonymous access)**.
6. Create the container.

Repeat for:

`vendor-reports`

### **Expected containers**

| Container        | Access  |
| ---------------- | ------- |
| `app-documents`  | Private |
| `vendor-reports` | Private |

### **Azure CLI**

Use Microsoft Entra authentication:

```
az storage container create \
  --account-name "$STORAGE_NAME" \
  --name app-documents \
  --auth-mode login
```

```
az storage container create \
  --account-name "$STORAGE_NAME" \
  --name vendor-reports \
  --auth-mode login
```

You need an appropriate Blob data role, such as Storage Blob Data Contributor, for these commands.

### **Validate**

```
az storage container list \
  --account-name "$STORAGE_NAME" \
  --auth-mode login \
  --query "[].{Name:name,PublicAccess:properties.publicAccess}" \
  -o table
```

Expected: Both containers appear without anonymous public access.

**Screenshot:** `01-private-containers.png`

## **Task 3 — Upload application files**

**Estimated time:** 20 minutes

We'll create sample application files and upload them as block blobs.

### **Step 1 — Create test files**

In Bash:

```
mkdir -p az104-blob-lab
cd az104-blob-lab
```

Create a sample report:

```
cat > sample-report.txt <<'EOF'
AZ-104 Azure Blob Storage Lab
Environment: Development
Project: Azure Administrator Learning
Classification: Sample Data
EOF
```

Create a sample application configuration:

```
cat > application-config.json <<'EOF'
{
  "application": "az104-storage-demo",
  "environment": "development",
  "version": "1.0",
  "logging": {
    "level": "INFO"
  }
}
EOF
```

These files contain no secrets or real customer data.

### **Step 2 — Upload files**

```
az storage blob upload \
  --account-name "$STORAGE_NAME" \
  --container-name app-documents \
  --name sample-report.txt \
  --file sample-report.txt \
  --auth-mode login
```

```
az storage blob upload \
  --account-name "$STORAGE_NAME" \
  --container-name app-documents \
  --name application-config.json \
  --file application-config.json \
  --auth-mode login
```

### **Step 3 — List blobs**

```
az storage blob list \
  --account-name "$STORAGE_NAME" \
  --container-name app-documents \
  --auth-mode login \
  --query "[].{Name:name,Size:properties.contentLength}" \
  -o table
```

### **Expected outcome**

Both files should appear inside `app-documents`.

**Screenshot:** `02-uploaded-blobs.png`

## **Task 4 — Download and validate Blob data**

**Estimated time:** 15 minutes

Download the sample report:

```
az storage blob download \
  --account-name "$STORAGE_NAME" \
  --container-name app-documents \
  --name sample-report.txt \
  --file downloaded-report.txt \
  --auth-mode login
```

Compare the original and downloaded files:

```
diff sample-report.txt downloaded-report.txt
```

If the files are identical, `diff` produces no output.

### **Validate using SHA-256**

```
sha256sum sample-report.txt downloaded-report.txt
```

Both files should have identical hashes.

### **What you learned**

This demonstrates data integrity validation after transferring files to and from Azure Blob Storage.

## **Task 5 — Configure and test Azure RBAC**

**Estimated time:** 25 minutes

In Lab 01, we created:

- `az104-storage-admins` — Storage Blob Data Contributor
- `az104-storage-readers` — Storage Blob Data Reader

Now we'll test the difference.

### **Step 1 — Verify role assignments**

Open:

**Storage account → Access control (IAM) → Role assignments**

Confirm the two group assignments.

### **Step 2 — Prepare test identities**

Use your synthetic Entra test user from Phase 01, if available.

To test Reader access:

1. Remove the user from groups granting Blob Data Contributor or broader data access.
2. Add the user to `az104-storage-readers`.
3. Verify no other role or inherited permission grants write access.
4. Sign in using a separate browser profile or local Azure CLI session.

### **Step 3 — Test Reader permissions**

| Operation     | Expected |
| ------------- | -------- |
| List blobs    | Allowed  |
| Download blob | Allowed  |
| Upload blob   | Denied   |
| Delete blob   | Denied   |

### **Step 4 — Test Contributor permissions**

Remove the user from the Reader group and add them to `az104-storage-admins`.

After permissions propagate and the session is refreshed:

| Operation     | Expected |
| ------------- | -------- |
| List blobs    | Allowed  |
| Download blob | Allowed  |
| Upload blob   | Allowed  |
| Delete blob   | Allowed  |

Use only synthetic test data for deletion testing.

### **Important AZ-104 concept**

**Storage account Contributor** and **Storage Blob Data Contributor** are not the same.

The first is primarily a management-plane role, while the second grants Blob data-plane permissions.

**Screenshot:** `03-blob-rbac-validation.png`

## **Task 6 — Explore Blob access tiers**

**Estimated time:** 20 minutes

Azure Blob Storage supports multiple access tiers for optimizing cost.

| Access tier | Typical usage              | Access pattern           |
| ----------- | -------------------------- | ------------------------ |
| Hot         | Frequently accessed data   | Frequent                 |
| Cool        | Infrequently accessed data | Occasional               |
| Cold        | Rarely accessed data       | Rare                     |
| Archive     | Long-term archival data    | Offline until rehydrated |

### **Practical exercise**

We'll change a sample blob between Hot and Cool.

Open:

**Storage account → Containers → app-documents → sample-report.txt**

Select **Change tier**, if available.

Choose **Cool**, then save.

### **Azure CLI**

```
az storage blob set-tier \
  --account-name "$STORAGE_NAME" \
  --container-name app-documents \
  --name sample-report.txt \
  --tier Cool \
  --auth-mode login
```

### **Verify**

```
az storage blob show \
  --account-name "$STORAGE_NAME" \
  --container-name app-documents \
  --name sample-report.txt \
  --auth-mode login \
  --query "{Name:name,Tier:properties.blobTier}" \
  -o json
```

### **Return the blob to Hot**

```
az storage blob set-tier \
  --account-name "$STORAGE_NAME" \
  --container-name app-documents \
  --name sample-report.txt \
  --tier Hot \
  --auth-mode login
```

**Cost warning:** Cool, Cold, and Archive tiers have minimum retention periods and may incur early deletion charges. Keep test files very small. Do not move your lab files to Archive because rehydration takes time and can generate additional charges.

**Screenshot:** `04-blob-access-tier.png`

## **Task 7 — Generate a time-limited SAS token**

**Estimated time:** 25 minutes

This exercise demonstrates how to provide temporary access to a specific file without making its container public.

### **Business requirement**

An external vendor needs to download one report for 15 minutes.

The vendor must not be able to upload, modify, or delete files.

### **Step 1 — Create a sample vendor report**

```
echo "AZ104 Sample Vendor Report" > vendor-report.txt
```

Upload it:

```
az storage blob upload \
  --account-name "$STORAGE_NAME" \
  --container-name vendor-reports \
  --name vendor-report.txt \
  --file vendor-report.txt \
  --auth-mode login
```

### **Step 2 — Understand SAS types**

| SAS type            | Signed using                | Scope                     |
| ------------------- | --------------------------- | ------------------------- |
| User delegation SAS | Microsoft Entra credentials | Blob Storage              |
| Service SAS         | Storage account key         | Specific storage service  |
| Account SAS         | Storage account key         | Multiple storage services |

For our lab, we'll use a **user delegation SAS** because it avoids signing the token with the storage account key.

### **Step 3 — Verify SAS generation permissions**

The identity generating a user delegation SAS needs permission for:

`Microsoft.Storage/storageAccounts/blobServices/generateUserDelegationKey/action`

A role such as Storage Blob Data Contributor does not necessarily provide that action. Storage Blob Delegator at an appropriate scope, or another role containing the action, may be required.

### **Step 4 — Generate a short-lived user delegation SAS**

From an authorized Azure CLI session:

```
EXPIRY=$(date -u -d '+15 minutes' '+%Y-%m-%dT%H:%MZ')

SAS_TOKEN=$(az storage blob generate-sas \
  --account-name "$STORAGE_NAME" \
  --container-name vendor-reports \
  --name vendor-report.txt \
  --permissions r \
  --expiry "$EXPIRY" \
  --auth-mode login \
  --as-user \
  -o tsv)
```

**Important:** The SAS token is a bearer credential. Anyone who possesses it may be able to access the specified blob until it expires, subject to network restrictions.

Do not print the token, commit it to GitHub, or include it in screenshots.

### **Step 5 — Test the SAS**

Construct the blob URL locally:

```
BLOB_URL="https://${STORAGE_NAME}.blob.core.windows.net/vendor-reports/vendor-report.txt"
```

Then test:

```
curl -fsS "${BLOB_URL}?${SAS_TOKEN}" \
  -o vendor-download.txt
```

Verify:

```
cat vendor-download.txt
```

Expected:

```
AZ104 Sample Vendor Report
```

### **Step 6 — Test expiry**

After the SAS expires, try downloading the blob again using the same URL.

Expected: The request is rejected, typically with HTTP 403.

### **Step 7 — Clear the token**

```
unset SAS_TOKEN
```

**Security note:** Avoid sharing SAS URLs through email or chat for real confidential data without approved controls. A SAS is a delegated credential, not a user identity.

**Screenshot:** `05-sas-validation.png` — capture only a sanitized result, never the token.

## **Task 8 — Enable Blob versioning and soft delete**

**Estimated time:** 20 minutes

We'll introduce data protection now and perform more advanced recovery scenarios in Lab 05.

Navigate to:

**Storage account → Data management → Data protection**

Enable:

| Setting               | Recommended lab value |
| --------------------- | --------------------- |
| Blob soft delete      | Enabled               |
| Retention period      | 7 days                |
| Blob versioning       | Enabled               |
| Container soft delete | Enabled, 7 days       |

### **Versioning test**

1. Open `sample-report.txt`.
2. Upload a new version of the same blob.
3. Navigate to the blob's version history.
4. Verify that the previous version is retained.

### **Soft delete test**

1. Upload a separate test blob, such as `delete-test.txt`.
2. Delete it.
3. Enable the portal option to show deleted blobs.
4. Restore the deleted blob.

**Important:** Versioning and soft delete can retain additional data and generate storage costs. We'll remove unnecessary versions and test data during Lab 05 cleanup.

**Screenshot:** `06-versioning-soft-delete.png`

## **5. Troubleshooting scenarios**

| Problem                               | Possible cause                                        | Investigation                                  |
| ------------------------------------- | ----------------------------------------------------- | ---------------------------------------------- |
| `AuthorizationPermissionMismatch`     | Missing Blob data permission                          | Check RBAC                                     |
| `AuthorizationFailure`                | Storage firewall or authorization issue               | Inspect network rules and identity             |
| `AuthenticationFailed`                | Invalid credential or SAS                             | Inspect authentication method                  |
| `PublicAccessNotPermitted`            | Anonymous access disabled                             | Use Entra authentication or valid SAS          |
| SAS download returns 403              | Expired SAS, incorrect scope, or network restrictions | Check expiry, permissions, and firewall        |
| Upload denied for Reader              | Expected least-privilege behavior                     | Verify data role                               |
| Blob not accessible after tier change | Archived/offline blob                                 | Check access tier and rehydration requirements |
| Deleted blob not visible              | Soft delete disabled or filter settings               | Check data protection configuration            |

### **Production-style interview scenario**

A developer reports:

> "I can view the storage account in Azure Portal, but I cannot upload files to the Blob container."

A structured troubleshooting answer:

1. Confirm the correct subscription, storage account, and container.
2. Verify the identity and tenant.
3. Check Blob data-plane role assignments.
4. Check group membership and role propagation.
5. Verify storage firewall and network access.
6. Inspect the exact HTTP 403 error code.
7. Determine whether Shared Key, SAS, or Entra authentication is being used.
8. Apply the minimum necessary permission.
9. Retest and document the result.

## **6. GitHub repository structure**

```
phase-02-storage/
|
|-- lab-02-blob-storage/
|   |-- README.md
|   |
|   |-- screenshots/
|       |-- 01-private-containers.png
|       |-- 02-uploaded-blobs.png
|       |-- 03-blob-rbac-validation.png
|       |-- 04-blob-access-tier.png
|       |-- 05-sas-validation.png
|       |-- 06-versioning-soft-delete.png
```

### **Git commit**

After completing the lab:

```
git add phase-02-storage/lab-02-blob-storage/

git commit -m "feat(az104): implement secure Azure Blob Storage with RBAC SAS and data protection"

git push origin main
```

Only upload synthetic sample data. Never commit SAS tokens, account keys, connection strings, or real customer files.

## **7. AZ-104 interview questions**

1\. What is the difference between a storage account and a Blob container?

2\. What is the difference between Blob Data Reader and Blob Data Contributor?

3\. What is a user delegation SAS?

4\. Why is SAS more secure than making a container public?

5\. What is the difference between Hot and Cool tiers?

6\. What does Blob versioning do?

7\. What is Blob soft delete?

8\. Why can a user view a storage account but not download a blob?

## **8. Lab 02 completion checklist**

**Lab 02 Progress**


Verified existing StorageV2 account

Created app-documents and vendor-reports private containers

Uploaded sample application files

Downloaded files and validated SHA-256 hashes

Tested Blob Data Reader permissions

Tested Blob Data Contributor permissions

Changed Blob access tier and restored Hot

Generated a 15-minute user delegation SAS

Tested SAS access and expiration

Enabled Blob versioning and soft delete

Tested blob version history and recovery

Reviewed storage costs and sanitized GitHub evidence


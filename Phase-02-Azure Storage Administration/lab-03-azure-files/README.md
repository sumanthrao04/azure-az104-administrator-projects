# **AZ-104 — Phase 02 | Lab 03: Azure Files & SMB Administration**

**Project: Enterprise Centralized File Storage & Recovery**

In Lab 02, we covered Azure Blob Storage, private containers, RBAC, SAS tokens, and access tiers.

Now we'll learn how organizations use **Azure Files as a centralized file-sharing solution**, allowing multiple servers and users to access shared files using the SMB protocol.

Hands-on Lab

Duration: 2–3 hours

Subscription

**Azure Free Trial**

Region

**Central India**

Additional cost target

**Under $1**

Protocol

**SMB 3.x**

We'll reuse the existing StorageV2 account in rg-az104-storage-cin-001. No additional VM is required for the core lab. Actual costs depend on storage usage, transactions, and snapshots.

## **1. Real-world scenario**

Imagine you're working as an Azure Cloud Administrator for a financial services company.

The organization has multiple Windows application servers. Each server currently stores application documents and reports locally.

This creates several problems:

- Files are duplicated across servers.
- Application servers cannot easily share documents.
- Accidental file deletion causes data loss.
- Managing storage independently on every server is difficult.
- Storage capacity is difficult to manage centrally.

Your manager asks you to implement a centralized file-sharing solution using Azure Files.

### **Business requirements**

| Requirement                 | Azure solution                                                 |
| --------------------------- | -------------------------------------------------------------- |
| Centralized file storage    | Azure Files                                                    |
| Windows server connectivity | SMB protocol                                                   |
| Network security            | Storage firewall                                               |
| Authentication              | Storage account key or supported identity-based authentication |
| Point-in-time recovery      | File share snapshots                                           |
| Deleted share recovery      | File share soft delete                                         |
| Cost optimization           | Standard LRS storage                                           |

## **2. Architecture — What we're building**

Target architecture. We'll use your local Windows computer for SMB connectivity if port 445 is available. Application VM integration is reserved for later phases.

## **3. Topics we'll learn**

| Topic                         | Description                                            |
| ----------------------------- | ------------------------------------------------------ |
| Azure Files                   | Fully managed cloud file shares                        |
| SMB                           | Server Message Block protocol                          |
| NFS                           | Network File System protocol                           |
| File share                    | Centralized storage accessible by multiple clients     |
| UNC path                      | Windows network file share path                        |
| Share quota                   | Maximum configured share capacity                      |
| Share snapshot                | Read-only point-in-time copy of a file share           |
| Soft delete                   | Recovery of accidentally deleted file shares           |
| Azure File Sync               | Synchronization between Windows Server and Azure Files |
| Identity-based authentication | Supported directory-based SMB access                   |

### **Azure Blob Storage vs Azure Files**

| Feature               | Blob Storage                         | Azure Files                               |
| --------------------- | ------------------------------------ | ----------------------------------------- |
| Storage type          | Object storage                       | File storage                              |
| Organization          | Containers and blobs                 | Shares, directories and files             |
| Common access         | REST API, SDK, HTTPS                 | SMB, NFS, REST                            |
| Windows network drive | Not native SMB                       | Supported                                 |
| Typical use           | Images, backups, application objects | Shared documents, application file shares |
| Recovery              | Blob versions and soft delete        | Share snapshots and share soft delete     |

**Interview takeaway:** Use Azure Files when applications require traditional shared file-system access. Use Blob Storage when applications require scalable object storage.

# **4. Hands-on implementation**

## **Task 1 — Verify your existing storage account**

Open [Azure Portal](https://portal.azure.com?utm_source=chatgpt.com) and navigate to your storage account from Lab 01.

Open Azure Cloud Shell in Bash mode.

```
RG_NAME="rg-az104-storage-cin-001"

STORAGE_NAME="<your-storage-account-name>"

SHARE_NAME="app-shared-files"
```

Replace the storage account placeholder with your actual account name.

Verify the account:

```
az storage account show \
  --name "$STORAGE_NAME" \
  --resource-group "$RG_NAME" \
  --query "{Name:name,Kind:kind,SKU:sku.name,Location:location}" \
  -o table
```

Expected configuration:

| Property | Value        |
| -------- | ------------ |
| Kind     | StorageV2    |
| SKU      | Standard_LRS |
| Location | centralindia |

Also check your remaining trial credit before continuing.

## **Task 2 — Create an Azure file share**

**Estimated time:** 15 minutes

Navigate to:

**Storage account → Data storage → File shares → + File share**

[Create Azure File Share Using Azure Portal](https://images.openai.com/static-rsc-4/JVXaXKiGAa4voo8KmFntp3stJMEBLsnnTSHfiaJjesVdY0cWX0nacGhWCSRezViBFA3MWNQD4L4tx33zNMBjYeGEar0eqwppC875U1g36jE_yRk5aUh4dmZ0Yo7Ob-SanQETp88tGLWoQNkkdRtvpOnMnWYbbP_mDbbW_eBPjaU?purpose=inline)

[c-sharpcorner.com](https://www.c-sharpcorner.com/article/create-azure-file-share-using-azure-portal/?utm_source=chatgpt.com)

Configure:

| Setting         | Value                               |
| --------------- | ----------------------------------- |
| File share name | `app-shared-files`                  |
| Access tier     | Transaction optimized, if available |
| Protocol        | SMB                                 |
| Quota           | 5 GiB, if configurable              |
| Backup          | Disabled for this lab               |

We will configure snapshots manually rather than enabling a separate paid backup solution.

### **Azure Files access tiers**

| Tier                  | Suitable workload                          |
| --------------------- | ------------------------------------------ |
| Transaction optimized | Transaction-heavy workloads                |
| Hot                   | Frequently accessed files                  |
| Cool                  | Infrequently accessed files                |
| Premium               | High-performance and low-latency workloads |

### **Create using Azure CLI — alternative**

```
az storage share-rm create \
  --resource-group "$RG_NAME" \
  --storage-account "$STORAGE_NAME" \
  --name "$SHARE_NAME" \
  --quota 5 \
  --access-tier TransactionOptimized
```

### **Verify the file share**

```
az storage share-rm list \
  --resource-group "$RG_NAME" \
  --storage-account "$STORAGE_NAME" \
  -o table
```

**Screenshot:** `01-file-share-created.png`

## **Task 3 — Create directories and upload files**

Open:

**Storage account → File shares → app-shared-files → Browse**

Create three directories:

```
app-shared-files/
├── configs/
├── reports/
└── logs/
```

Create the following sample files on your computer.

**application.properties**

```
server.port=8080
spring.application.name=az104-demo
logging.level.root=INFO
```

**daily-report.txt**

```
AZ-104 Daily Application Report
Environment: Development
Application Status: Healthy
```

**application.log**

```
INFO Application started successfully
INFO Database connectivity verified
INFO Health check completed
```

Upload the files into their respective directories.

**app-shared-files**

&#x20;configs

&#x20;application.properties

&#x20;reports

&#x20;daily-report.txt

&#x20;logs

&#x20;application.log

The portal's file browser requires an appropriate supported authentication method and network connectivity.

**Screenshot:** `02-file-share-directories.png`

## **Task 4 — Connect Azure Files to your Windows computer**

**Estimated time:** 25 minutes

This is the most important practical exercise in this lab.

We'll mount the Azure file share as a Windows network drive.

### **Step 1 — Understand the SMB endpoint**

The UNC path follows this format:

```
\\<storage-account-name>.file.core.windows.net\app-shared-files
```

SMB connectivity uses **TCP port 445**.

### **Step 2 — Test network connectivity**

Open PowerShell on your local Windows computer.

```
$StorageName = "<your-storage-account-name>"

Test-NetConnection `
  -ComputerName "$StorageName.file.core.windows.net" `
  -Port 445
```

Expected result:

```
RemotePort       : 445
TcpTestSucceeded : True
```

If `TcpTestSucceeded` is `False`, your ISP, corporate firewall, VPN, or local network may be blocking SMB traffic.

Do not bypass your company's security restrictions.

### **Step 3 — Retrieve the connection script**

Navigate to:

**Azure Portal → Storage account → File shares → app-shared-files → Connect**

[How to Create, Configure, and Mount Persistent Azure File Share - Cloudspress](https://images.openai.com/static-rsc-4/JR8XMdUIJzOlLbgrFyy2suuStyEj1BvgCrvv7Nq2e4UowF61SHcphdWXRcUz9dbTg4d1YawrPnl-a7-NnMOJID6XLjdkS1TmllYqFh7cDuKI24FzWN8pIh0Tkk-bhvIkSkyqCpDyxpnAxvKNZV6n6ATYEn9I3SC4ouQZdLkOE5I?purpose=inline)

[cloudspress.com](https://www.cloudspress.com/how-to-create-configure-and-mount-persistent-azure-file-share/?utm_source=chatgpt.com)

Select:

| Setting               | Value                                       |
| --------------------- | ------------------------------------------- |
| Operating system      | Windows                                     |
| Drive letter          | Z                                           |
| Authentication method | Storage account key, for this temporary lab |

The portal generates a PowerShell connection script.

Review and execute it locally.

**Security warning:** The generated script may contain your storage account key. Never upload that script, key, or connection string to GitHub.

### **Step 4 — Mount the drive manually (alternative)**

```
$StorageName = "<your-storage-account-name>"

net use Z: "\\$StorageName.file.core.windows.net\app-shared-files" "/user:localhost\$StorageName" * /persistent:no
```

The command prompts for the storage account key as the password.

### **Step 5 — Verify the mounted drive**

```
Get-PSDrive -Name Z
```

```
Get-ChildItem Z:\
```

Open Windows File Explorer and verify the network drive.

[Mount Azure File Share in Windows](https://images.openai.com/static-rsc-4/CFwwx4ytFDYFBwslGvJi0HDhIfFR0qPoi5uVzWGeJvdl-5mg3wp9p0NGlqYBuqIZpDF6tTSA3QWtFr_5udzIhjEZK_KnJQeAomcgIdtL_NlNN4SkEIenh7ZUz0lc5w_ZU4pgFnmBOuy7iaDUyW6mwHavIaOuBpSUPYm2BOmdydQ?purpose=inline)

[c-sharpcorner.com](https://www.c-sharpcorner.com/article/mount-azure-file-share-in-windows/?utm_source=chatgpt.com)

You should be able to access the `configs`, `reports`, and `logs` directories.

**Screenshot:** `03-windows-smb-mounted-drive.png`

### **What if port 445 is blocked?**

Continue with the Azure Portal file browser and supported REST-based tools such as Azure Storage Explorer.

You can complete file management and snapshot recovery without SMB connectivity. Document the SMB limitation rather than claiming the drive was mounted successfully.

## **Task 5 — Test centralized file sharing**

After successfully mounting the drive, run these PowerShell commands.

### **Create a file from Windows**

```
Set-Content `
  -Path "Z:\reports\windows-test.txt" `
  -Value "File created from Windows SMB client"
```

### **Read the file**

```
Get-Content "Z:\reports\windows-test.txt"
```

### **List all report files**

```
Get-ChildItem "Z:\reports"
```

Now return to Azure Portal and open the file share.

Verify that `windows-test.txt` appears inside the `reports` directory.

### **What you've demonstrated**

Windows client

Azure Files

Portal file browser

Both clients access the same centralized files. This is shared storage, not a separate file synchronization process.

This is how multiple application servers can access common application configurations, documents, and reports.

**Screenshot:** `04-smb-file-validation.png`

## **Task 6 — Configure Azure Files access control**

Azure Files supports multiple authentication approaches.

| Authentication method            | Purpose                                                              |
| -------------------------------- | -------------------------------------------------------------------- |
| Storage account key              | Broad shared-secret access                                           |
| Active Directory Domain Services | Domain-based SMB authentication                                      |
| Microsoft Entra Domain Services  | Managed domain-based authentication                                  |
| Microsoft Entra Kerberos         | Supported identity-based SMB authentication scenarios                |
| SAS                              | Delegated access through supported REST operations, not SMB mounting |

### **Azure Files RBAC roles**

| Role                                             | Permission                                                |
| ------------------------------------------------ | --------------------------------------------------------- |
| Storage File Data SMB Share Reader               | Read files and directories                                |
| Storage File Data SMB Share Contributor          | Read, write and delete files                              |
| Storage File Data SMB Share Elevated Contributor | Read, write, delete and modify supported file permissions |

### **Understand the permission architecture**

**User authentication**

Supported directory identity

**Share-level permissions**

Azure RBAC — SMB Share Reader / Contributor

**Directory and file permissions**

Windows ACL / NTFS-style permissions

**Effective file access**

### **Practical exercise**

Navigate to:

**File share → Access control (IAM) → Add role assignment**

Assign the following roles to your existing storage security groups:

| Group                   | Role                                    | Scope      |
| ----------------------- | --------------------------------------- | ---------- |
| `az104-storage-admins`  | Storage File Data SMB Share Contributor | File share |
| `az104-storage-readers` | Storage File Data SMB Share Reader      | File share |

**Important:** Assigning these roles alone does not enable identity-based SMB authentication. The storage account must also have a supported identity source configured, and file-level permissions must be appropriate.

Since we're using a personal Azure Free Trial, we'll study the identity-based architecture and configure the RBAC assignments without deploying domain infrastructure.

Our actual SMB connection in Task 4 uses the storage account key, which does not demonstrate Reader-versus-Contributor enforcement.

**Screenshot:** `05-file-share-rbac.png`

## **Task 7 — Create a file share snapshot**

**Estimated time:** 20 minutes

A file share snapshot captures a read-only, point-in-time view of the share.

### **Step 1 — Verify the original file**

Open:

`reports/daily-report.txt`

Confirm the original content:

```
Application Status: Healthy
```

### **Step 2 — Create a snapshot**

Navigate to:

**File shares → app-shared-files → Snapshots → + Add snapshot**

[Use Azure Files Share Snapshots | Microsoft Learn](https://images.openai.com/static-rsc-4/uR3gcDg3P1otTyS8wzR5pyOWrUTz6u0QzOSY_nIPkcQs1Mbu2pCwMp7wfeqZzBkH643YquRAdPUGBZ3ShmL3fvAa3XBFPX_AS4Fd76CkIiLto56FlgsXwBNzNqUSao6jxYkSVdMGTzjC1o5qIQju9_lMk1k6ptyhVmmEGo9e8Vk?purpose=inline)

[learn.microsoft.com](https://learn.microsoft.com/th-th/azure/storage/files/storage-snapshots-files?utm_source=chatgpt.com)

Create the snapshot and record its timestamp.

### **Step 3 — Modify the original file**

From the Windows mounted drive:

```
Set-Content `
  -Path "Z:\reports\daily-report.txt" `
  -Value "Application Status: Failed"
```

Alternatively, replace the file through Azure Portal.

### **Step 4 — Verify the difference**

| File location      | Expected content            |
| ------------------ | --------------------------- |
| Current file share | Application Status: Failed  |
| Share snapshot     | Application Status: Healthy |

**Screenshot:** `06-file-share-snapshot.png`

## **Task 8 — Recover an accidentally deleted file**

**Estimated time:** 20 minutes

Now we'll simulate a real production incident.

### **Incident**

A developer accidentally deletes the application's daily report.

### **Step 1 — Delete the sample file**

Only delete the synthetic file created for this exercise.

```
Remove-Item "Z:\reports\daily-report.txt"
```

### **Step 2 — Verify deletion**

```
Test-Path "Z:\reports\daily-report.txt"
```

Expected:

```
False
```

### **Step 3 — Recover from the snapshot**

Navigate to:

**Azure Portal → File shares → app-shared-files → Snapshots**

Open the previously created snapshot.

Browse to:

`reports/daily-report.txt`

Select the file and use the available restore option. If direct restoration is unavailable, download the snapshot copy and upload it back to the active share.

### **Step 4 — Validate recovery**

```
Get-Content "Z:\reports\daily-report.txt"
```

Expected original content:

```
AZ-104 Daily Application Report
Environment: Development
Application Status: Healthy
```

#### **File recovery workflow**

Original file

Snapshot

Deleted file

Restored file

**Screenshot:** `07-file-restoration.png`

## **Task 9 — Configure file share soft delete**

Navigate to:

**Storage account → Data management → Data protection**

Enable soft delete for Azure file shares.

| Setting                | Value   |
| ---------------------- | ------- |
| File share soft delete | Enabled |
| Retention period       | 7 days  |

Save the configuration.

### **Azure CLI**

```
az storage account file-service-properties update \
  --resource-group "$RG_NAME" \
  --account-name "$STORAGE_NAME" \
  --enable-delete-retention true \
  --delete-retention-days 7
```

Verify:

```
az storage account file-service-properties show \
  --resource-group "$RG_NAME" \
  --account-name "$STORAGE_NAME" \
  --query shareDeleteRetentionPolicy \
  -o json
```

Expected:

```
{
  "enabled": true,
  "days": 7
}
```

### **Snapshot vs soft delete**

| Feature                          | Share snapshot                                   | File share soft delete     |
| -------------------------------- | ------------------------------------------------ | -------------------------- |
| Recover individual deleted files | Yes, from a snapshot                             | No                         |
| Recover an entire deleted share  | Not independently after permanent share deletion | Yes, within retention      |
| Point-in-time recovery           | Yes                                              | Recovers the deleted share |
| Additional storage charges       | Possible                                         | Possible                   |

Do not delete your main file share to test this. If you want to test share recovery, create a separate temporary share containing only synthetic data.

**Screenshot:** `08-file-share-soft-delete.png`

## **Task 10 — Understand Azure File Sync**

Azure File Sync allows organizations to synchronize supported Windows Server file shares with Azure Files while retaining local file-server access.

Azure File Sync hybrid architecture — conceptual exercise for this lab.

### **Important components**

| Component             | Purpose                                                                               |
| --------------------- | ------------------------------------------------------------------------------------- |
| Storage Sync Service  | Central management of synchronization                                                 |
| Sync group            | Defines synchronization topology                                                      |
| Cloud endpoint        | Azure file share                                                                      |
| Server endpoint       | Directory on a registered Windows Server                                              |
| Azure File Sync agent | Software installed on Windows Server                                                  |
| Cloud tiering         | Keeps frequently accessed files locally while tiering eligible file contents to Azure |

We'll cover the architecture for AZ-104 now without deploying additional Windows Servers and consuming unnecessary trial credit.

# **5. Real-world troubleshooting scenarios**

| Problem                          | Possible cause                            | Troubleshooting                      |
| -------------------------------- | ----------------------------------------- | ------------------------------------ |
| Cannot mount Azure file share    | TCP 445 blocked                           | `Test-NetConnection`                 |
| System error 53                  | Network path, DNS or connectivity problem | Verify endpoint and network          |
| System error 5                   | Access denied                             | Check authentication and permissions |
| System error 1219                | Conflicting SMB credentials               | Inspect existing SMB connections     |
| File share access denied         | Incorrect key or identity configuration   | Verify authentication method         |
| Storage endpoint not reachable   | Storage firewall restriction              | Check allowed networks               |
| Deleted file cannot be recovered | No suitable snapshot                      | Verify snapshot availability         |
| Share-level RBAC not working     | Identity-based SMB not configured         | Check identity source and file ACLs  |

## **Production-style interview scenario**

An application deployed on a Windows Server cannot access an Azure file share.

**How would you troubleshoot?**

First, I would verify DNS resolution and SMB connectivity to the storage account endpoint.

```
Resolve-DnsName "$StorageName.file.core.windows.net"
```

```
Test-NetConnection `
  -ComputerName "$StorageName.file.core.windows.net" `
  -Port 445
```

Then I would check the storage firewall, UNC path, authentication method, share-level permissions, and directory-level ACLs.

I would also verify whether the application runs under a Windows service account different from the currently logged-in user. A drive mapped in an interactive user session might not be available to a Windows service.

For production applications, I would prefer an appropriately secured UNC path and supported service identity rather than relying on a manually mapped drive.



### **Git commit**

After completing and validating the lab:

```
git add phase-02-storage/lab-03-azure-files/

git commit -m "feat(az104): implement Azure Files SMB sharing snapshots and file recovery"

git push origin main
```

Never commit storage account keys or generated connection scripts containing credentials.

# **7. AZ-104 interview questions**

1\. What is Azure Files?

2\. What is the difference between Azure Blob Storage and Azure Files?

3\. Which port is required for Azure Files SMB connectivity?

4\. What is Azure File Sync?

5\. What is a file share snapshot?

6\. Can file share soft delete recover an individual deleted file?

7\. What is the difference between share-level RBAC and NTFS permissions?

8\. Does assigning SMB Share Contributor automatically enable identity-based authentication?

9\. How would you troubleshoot an Azure Files mounting failure?

10\. What is cloud tiering in Azure File Sync?

# **8. Cleanup and cost management**

Disconnect the Windows network drive after testing:

```
net use Z: /delete
```

If you used the portal-generated connection script, check Windows Credential Manager for any saved storage credentials and remove the lab credentials when no longer required.

Delete unnecessary snapshots after validating recovery. Keep only the small sample files needed for subsequent exercises.

Review Azure Cost Management and remaining promotional credit.

**Do not delete the storage account.** We'll reuse it in Lab 04 and Lab 05.

# **9. Lab 03 completion checklist**

**Lab 03 Progress**



Verified existing StorageV2 account

Created app-shared-files

Created configs, reports and logs directories

Uploaded sample application files

Tested SMB port 445 connectivity

Mounted Windows network drive or documented connectivity restriction

Validated shared file access

Configured file share RBAC roles

Understood identity-based authentication prerequisites

Created a file share snapshot

Recovered an accidentally deleted file

Configured file share soft delete

Understood Azure File Sync architecture

Reviewed costs and prepared GitHub documentation



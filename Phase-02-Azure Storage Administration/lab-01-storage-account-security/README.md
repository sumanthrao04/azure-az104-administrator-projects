# **AZ-104 — Phase 02 | Lab 01: Azure Storage Account Deployment & Security**

**Project: Secure Enterprise Azure Storage Infrastructure**

Welcome to Phase 02! In this lab, we'll deploy a secure Azure Storage account and configure the foundational settings used in enterprise environments.

This storage account will be reused for **Lab 02 (Blob Storage), Lab 03 (Azure Files), Lab 04 (Queues & Tables), and Lab 05 (Data Protection)**.

Hands-on Project

Duration: 2–3 hours

Subscription

**Azure Free Trial**

Region

**Central India**

Resource group

rg-az104-storage-cin-001

Storage configuration

**Standard · LRS · StorageV2**

Cost target: under $1 for this small initial exercise, not a price guarantee. Storage capacity, transactions, and networking can generate charges. Verify your free allowance and remaining trial credit before deployment.

## **1. Real-world scenario**

Imagine you're an Azure Administrator for a financial services organization.

The development team needs centralized cloud storage for application files, reports, and internal documents.

The security team requires:

- Data must be encrypted in transit and at rest.
- Anonymous public Blob access must be disabled.
- Only authorized users should access storage data.
- Resource access must be controlled using Azure RBAC.
- Storage must be organized with consistent tags.
- Network access should be restricted after administrative connectivity is validated.
- Data must be recoverable if accidentally deleted.

**Your responsibility:** Deploy a secure, cost-efficient Storage account that can support multiple application teams.

## **2. Architecture — What we're building**

We'll create the resource group and StorageV2 account now. Blob containers, file shares, queues, and tables will be configured in later labs.

## **3. Topics you'll learn**

| Topic                 | What it means                                                |
| --------------------- | ------------------------------------------------------------ |
| Storage account       | Azure resource that contains storage services                |
| StorageV2             | General-purpose v2 account for Blob, Files, Queue, and Table |
| Standard vs Premium   | Different performance and service options                    |
| LRS, ZRS, GRS, GZRS   | Storage redundancy choices                                   |
| Secure transfer       | Require HTTPS for supported storage access                   |
| Public network access | Controls network connectivity to storage                     |
| Anonymous Blob access | Controls whether blobs can be accessed anonymously           |
| Azure RBAC            | Controls management and supported data access                |
| Access keys           | Shared secrets granting broad account-level access           |
| Tags                  | Metadata for organization and cost tracking                  |

### **Storage redundancy — AZ-104 exam topic**

| Type   | Replication                                                   | Suitable scenario          |
| ------ | ------------------------------------------------------------- | -------------------------- |
| LRS    | Three copies in one physical datacenter in the primary region | Low-cost development       |
| ZRS    | Copies across availability zones in a region                  | Zone resilience            |
| GRS    | LRS in primary and secondary regions                          | Regional disaster recovery |
| RA-GRS | GRS with secondary read access                                | Read access to secondary   |
| GZRS   | ZRS in primary region plus geo-replication                    | Higher resilience          |

For our lab, choose **LRS** because it's typically the lowest-cost redundancy option and sufficient for temporary learning workloads.

# **4. Hands-on implementation**

## **Task 1 — Verify subscription, credit, and policies**

**Estimated time:** 10 minutes

Before creating the storage account, check the remaining Azure Free Trial credit and trial expiration date.

Open [Azure Portal](https://portal.azure.com?utm_source=chatgpt.com).

Verify:

1. Your subscription is active.
2. Your remaining trial credit is sufficient.
3. Your intended region is supported.
4. No Azure Policy blocks the planned deployment.

Because you configured Azure Policy in Phase 01, an Allowed Locations or required-tags policy could affect this lab.

If a policy blocks deployment, inspect the policy assignment and adjust the deployment to comply. Do not disable organizational governance merely to bypass it.

## **Task 2 — Create the storage resource group**

**Estimated time:** 10 minutes

Navigate to:

**Resource groups → + Create**

Use:

| Field           | Value                      |
| --------------- | -------------------------- |
| Subscription    | Azure Free Trial           |
| Resource group  | `rg-az104-storage-cin-001` |
| Region          | Central India              |
| Project tag     | AZ104                      |
| Environment tag | Dev                        |
| CostCenter tag  | Learning                   |
| Phase tag       | Phase02                    |

Select **Review + create → Create**.

### **Validate using Azure CLI**

```
az group show \
  --name rg-az104-storage-cin-001 \
  --query "{Name:name,Location:location,Tags:tags}" \
  -o json
```

**Expected:** The resource group exists in `centralindia` and contains the configured tags.

**Screenshot:** `01-storage-resource-group.png`

## **Task 3 — Create an Azure Storage account**

**Estimated time:** 20–30 minutes

Navigate to:

**Azure Portal → Storage accounts → + Create**

[How to deploy to Azure Blob Storage | Uniform for Sitecore Docs](https://images.openai.com/static-rsc-4/sTG3S5wtp8xr5dw5OqkzaC_eJlvEEeGankoU1okRK_rD0sj68cL1iP3cszffdxZZAyoF6F_jxhTDelGH70S9PUebjYITQYQp2SRh4a6l8DsbBc6j9kmvatXgoNwl_jiIPWrc1aGsnE-O6w2JFKSXrgn41ryaz-SAaEm-tXff8Ds?purpose=inline)

[docs.uniform.dev](https://docs.uniform.dev/sitecore/deploy/how-tos/how-to-deploy-to-azure-blob-storage/?utm_source=chatgpt.com)

### **Basics tab**

| Setting              | Value                           |
| -------------------- | ------------------------------- |
| Subscription         | Azure Free Trial                |
| Resource group       | `rg-az104-storage-cin-001`      |
| Storage account name | `staz104cin001` if available    |
| Region               | Central India                   |
| Performance          | Standard                        |
| Redundancy           | Locally-redundant storage (LRS) |
| Account kind         | StorageV2 / General-purpose v2  |

Storage account names must be globally unique, contain 3–24 characters, and use lowercase letters and numbers only.

If `staz104cin001` is unavailable, append a short unique suffix.

### **Advanced tab**

Configure:

| Setting                                         | Value                                                               |
| ----------------------------------------------- | ------------------------------------------------------------------- |
| Require secure transfer                         | Enabled                                                             |
| Allow anonymous access on individual containers | Disabled                                                            |
| Minimum TLS version                             | TLS 1.2 or higher, where supported                                  |
| Allow storage account key access                | Enabled initially for comparison; we'll evaluate disabling it later |
| Hierarchical namespace                          | Disabled                                                            |
| SFTP                                            | Disabled                                                            |
| NFS v3                                          | Disabled                                                            |
| Infrastructure encryption                       | Leave default unless explicitly required                            |

**Why these settings?** They establish a secure baseline without enabling features that are unnecessary for our AZ-104 lab.

### **Networking tab**

For the initial deployment:

| Setting                     | Value                  |
| --------------------------- | ---------------------- |
| Public network access       | Enabled                |
| Public network access scope | All networks initially |
| Private endpoint            | None                   |

We'll restrict network access after validating that the storage account is reachable.

**Important:** Public network access being enabled does not mean anonymous Blob access is enabled. Authentication and network controls are separate.

### **Data protection tab**

Use the portal's supported defaults for now. We'll configure and test soft delete, versioning, and lifecycle policies in Lab 05.

### **Encryption tab**

Use Microsoft-managed keys.

Azure Storage encrypts data at rest by default.

### **Tags tab**

Apply:

```
Project     = AZ104
Environment = Dev
CostCenter  = Learning
Phase       = Phase02
```

Select **Review + create → Create**.

### **Validation**

Open the newly created storage account and verify:

- Deployment succeeded.
- Region is Central India.
- Performance is Standard.
- Redundancy is LRS.
- Secure transfer is required.
- Anonymous Blob access is disabled.

**Screenshot:** `02-storage-account-overview.png`

## **Task 4 — Verify storage security settings**

**Estimated time:** 15 minutes

Open your Storage account → **Settings → Configuration** or **Security + networking**, depending on the portal layout.

Check:

| Property                                           | Expected              |
| -------------------------------------------------- | --------------------- |
| Secure transfer required                           | Enabled               |
| Minimum TLS version                                | 1.2 or higher         |
| Anonymous Blob access                              | Disabled              |
| Storage account key access                         | Enabled temporarily   |
| Hierarchical namespace                             | Disabled              |
| Default to Microsoft Entra authorization in portal | Enabled, if available |

Microsoft recommends Microsoft Entra authorization over Shared Key for supported storage data operations. We'll keep Shared Key enabled temporarily to explore authentication methods in later labs, then evaluate disabling it.&#x20;

[image](https://www.google.com/s2/favicons?domain=https://learn.microsoft.com\&sz=32)

Microsoft Learn+1



### **Validate with Azure CLI**

First, set variables:

```
RG_NAME="rg-az104-storage-cin-001"
STORAGE_NAME="<your-actual-storage-account-name>"
```

Replace the placeholder with the name you created.

Check the storage account:

```
az storage account show \
  --name "$STORAGE_NAME" \
  --resource-group "$RG_NAME" \
  --query "{Name:name,Location:location,Kind:kind,Sku:sku.name,HttpsOnly:enableHttpsTrafficOnly,TLS:minimumTlsVersion,PublicBlob:allowBlobPublicAccess}" \
  -o json
```

Expected values include:

```
{
  "Kind": "StorageV2",
  "Sku": "Standard_LRS",
  "HttpsOnly": true,
  "TLS": "TLS1_2",
  "PublicBlob": false
}
```

The actual output also contains your chosen storage account name and location.

**Screenshot:** `03-storage-security-settings.png`

## **Task 5 — Configure Azure RBAC for Storage**

**Estimated time:** 20 minutes

In Phase 01, you learned that Azure RBAC controls permissions at different scopes.

Now we'll apply that knowledge to Azure Storage.

### **Management plane vs data plane**

| Permission       | Example role                  | Purpose                            |
| ---------------- | ----------------------------- | ---------------------------------- |
| Management plane | Reader                        | View storage account configuration |
| Management plane | Contributor                   | Manage storage account resources   |
| Data plane       | Storage Blob Data Reader      | Read blob contents                 |
| Data plane       | Storage Blob Data Contributor | Read, write, and delete blobs      |

A management-plane Contributor role may also permit retrieving storage account keys, so it should not be treated as a strict data-access boundary while Shared Key access remains enabled.

### **Create storage security groups**

For a realistic enterprise scenario, create two security groups in Microsoft Entra ID:

| Group                   | Role                          |
| ----------------------- | ----------------------------- |
| `az104-storage-admins`  | Storage Blob Data Contributor |
| `az104-storage-readers` | Storage Blob Data Reader      |

These groups are separate from your Phase 01 identity-governance groups.

### **Assign the roles**

Navigate to:

**Storage account → Access control (IAM) → Add role assignment**

For `az104-storage-admins`:

1. Select **Storage Blob Data Contributor**.
2. Choose **User, group, or service principal**.
3. Select `az104-storage-admins`.
4. Assign the role at storage-account scope.

Repeat for `az104-storage-readers`, assigning **Storage Blob Data Reader**.

These roles apply to Blob data access, not Azure Files SMB permissions.

### **Validate using CLI**

```
STORAGE_SCOPE=$(az storage account show \
  --name "$STORAGE_NAME" \
  --resource-group "$RG_NAME" \
  --query id \
  -o tsv)

az role assignment list \
  --scope "$STORAGE_SCOPE" \
  --query "[].{Role:roleDefinitionName,Principal:principalName,Type:principalType}" \
  -o table
```

Expected: The two group assignments appear.

**Screenshot:** `04-storage-rbac-assignments.png`

## **Task 6 — Configure storage networking**

**Estimated time:** 20 minutes

Azure Storage supports network restrictions using public IP rules, virtual network rules, and private endpoints.

We'll first understand the available options.

| Network option                 | Purpose                                                            | Lab decision                                  |
| ------------------------------ | ------------------------------------------------------------------ | --------------------------------------------- |
| All networks                   | Allow connectivity from public networks, subject to authentication | Initial validation only                       |
| Selected networks              | Restrict supported public endpoint access                          | Hands-on                                      |
| Private endpoint               | Access through private IP connectivity                             | Later networking lab                          |
| Public network access disabled | Block public endpoint access                                       | Later, when private connectivity is available |

### **Configure selected networks**

1. Open your storage account.
2. Select **Security + networking → Networking**.
3. Choose **Manage** or the available network access settings.
4. Select **Enabled from selected virtual networks and IP addresses**.
5. Add your current public IPv4 address, if supported.
6. Save the changes.

The public IP should be your actual client internet egress IP, not a private address such as `192.168.x.x`.

**Important:** Azure Cloud Shell may use a different outbound IP from your laptop. Restricting storage to your laptop's public IP can cause Cloud Shell data operations to fail, even while management-plane commands still work.

Do not enable a private endpoint in this lab because it may incur additional charges.

Microsoft recommends restricting storage networking where possible.&#x20;

[image](https://www.google.com/s2/favicons?domain=https://learn.microsoft.com\&sz=32)

Microsoft Learn+1



### **Validate network configuration**

```
az storage account show \
  --name "$STORAGE_NAME" \
  --resource-group "$RG_NAME" \
  --query "{PublicNetwork:publicNetworkAccess,DefaultAction:networkRuleSet.defaultAction,IPRules:networkRuleSet.ipRules}" \
  -o json
```

Expected:

- Public network access enabled.
- Default network action set to Deny after selected-network restrictions are applied.
- Your allowed public IP present in the network rules.

**Screenshot:** `05-storage-network-rules.png`

## **Task 7 — Test Azure Storage endpoints**

**Estimated time:** 15 minutes

A general-purpose v2 storage account can expose multiple service endpoints.

| Service | Endpoint pattern                           |
| ------- | ------------------------------------------ |
| Blob    | `https://<account>.blob.core.windows.net`  |
| Files   | `https://<account>.file.core.windows.net`  |
| Queue   | `https://<account>.queue.core.windows.net` |
| Table   | `https://<account>.table.core.windows.net` |

Retrieve the endpoints:

```
az storage account show \
  --name "$STORAGE_NAME" \
  --resource-group "$RG_NAME" \
  --query primaryEndpoints \
  -o json
```

These are service endpoints, not proof that data is publicly accessible.

### **Test HTTPS connectivity**

From a terminal with network access:

```
curl -I "https://${STORAGE_NAME}.blob.core.windows.net/"
```

An HTTP 400 or 403 response can be normal for an unauthenticated request to the service root. The purpose is to inspect HTTPS connectivity, not to prove authorized data access.

If networking restrictions block your client, the request may fail differently.

## **Task 8 — Troubleshooting scenarios**

| Problem                              | Possible cause                                        | Investigation                      |
| ------------------------------------ | ----------------------------------------------------- | ---------------------------------- |
| Storage account name unavailable     | Globally unique name already used                     | Choose another name                |
| Deployment blocked                   | Azure Policy or quota restriction                     | Inspect deployment error           |
| `AuthorizationFailure`               | Missing data-plane permission or network restriction  | Check RBAC and networking          |
| `AuthenticationFailed`               | Invalid credential or authorization method            | Inspect authentication details     |
| Storage Explorer cannot access blobs | Missing data role or network access                   | Check identity, role, firewall     |
| Blob access returns 403              | Authentication, authorization, or network restriction | Review exact error code            |
| Shared Key disabled                  | Client relies on account keys                         | Use supported Entra authentication |

### **Real-world interview scenario**

A developer has **Contributor** access to the storage account but cannot access Blob data using Microsoft Entra authentication.

How would you investigate?

1. Confirm the developer is authenticated to the correct tenant.
2. Check the storage account and container scope.
3. Verify a suitable Blob data-plane role is assigned.
4. Check group membership and role propagation.
5. Inspect storage firewall and private endpoint restrictions.
6. Review the exact storage error.
7. Grant only the required data access role.
8. Retest using Microsoft Entra authorization.

## **5. GitHub repository structure**

```
phase-02-storage/
|
|-- lab-01-storage-account-security/
|   |-- README.md
|   |
|   |-- architecture/
|   |   |-- storage-account-architecture.png
|   |
|   |-- screenshots/
|       |-- 01-storage-resource-group.png
|       |-- 02-storage-account-overview.png
|       |-- 03-storage-security-settings.png
|       |-- 04-storage-rbac-assignments.png
|       |-- 05-storage-network-rules.png
```

### **Git commit**

After completing and validating the lab:

```
git add phase-02-storage/lab-01-storage-account-security/

git commit -m "feat(az104): deploy secure Azure StorageV2 account with RBAC and network controls"

git push origin main
```

Only upload redacted screenshots. Never commit account keys, SAS tokens, passwords, or private connection strings.

## **6. AZ-104 interview questions**

1\. What is a StorageV2 account?

2\. What is the difference between LRS and ZRS?

3\. Does disabling anonymous Blob access block authorized users?

4\. Why use Storage Blob Data Contributor instead of Contributor?

5\. What does secure transfer required do?

6\. What is the difference between a storage firewall and Azure RBAC?

7\. Why might Cloud Shell fail after enabling selected networks?

8\. What is the purpose of a private endpoint?

## **7. Lab 01 completion checklist**

**Lab 01 Progress**



Verified subscription credit and deployment permissions

Created rg-az104-storage-cin-001

Deployed Standard StorageV2 account with LRS

Verified HTTPS, TLS, and anonymous access settings

Created storage administration and reader groups

Assigned Blob data-plane RBAC roles

Configured selected-network storage firewall rules

Validated account settings using Azure CLI

Inspected service endpoints

Documented troubleshooting and screenshots

Reviewed Azure cost and remaining credit


# **AZ-104 — Phase 00 | Lab 03: Azure Cloud Shell & Azure CLI**

Hands-on Lab

Duration: 2–3 hours

## **Azure Cloud Administration Using CLI**

Subscription

**Azure Free Trial**

Primary region

**Central India**

Estimated cost

**$0 for CLI/resource-group operations**

AZ-104 domain

**Azure Administration**

**Prerequisite:** Lab 02 completed, with the existing resource group `rg-az104-dev-cin-001`.

## **1. Real-world scenario**

You are working as an Azure Cloud Administrator.

Your organization has multiple Azure resource groups across development, testing, and production environments.

Your manager asks you to:

- Inspect Azure subscriptions and resource groups using commands.
- Retrieve specific resource information without manually opening Azure Portal.
- Update resource tags.
- Automate repetitive Azure administration activities.
- Generate infrastructure inventory reports.
- Troubleshoot Azure CLI failures.

**Goal:** Learn to manage Azure efficiently using Azure Cloud Shell, Azure CLI, and basic Bash scripting.

## **2. Architecture — What we're building**

Azure Cloud Shell provides a command environment; Azure CLI communicates with Azure Resource Manager to manage subscription resources.

## **3. Topics covered in Lab 03**

| Module | Topics                             |
| ------ | ---------------------------------- |
| 1      | Azure Cloud Shell architecture     |
| 2      | Bash vs PowerShell                 |
| 3      | Azure CLI command structure        |
| 4      | Subscription management            |
| 5      | Resource group operations          |
| 6      | Azure CLI output formats           |
| 7      | JMESPath queries                   |
| 8      | Variables and Bash scripting       |
| 9      | Azure resource inventory reporting |
| 10     | Troubleshooting and cleanup        |

# **4. Hands-on implementation**

## **Task 1 — Explore Azure Cloud Shell**

You already used Cloud Shell in Lab 02. Now we'll understand how it works.

[Need a storage account to use cloud shell in exercise for the VM creation - Microsoft Q&A](https://images.openai.com/static-rsc-4/G3zCm7NYjUyYJQoC_mYy6tx6XgdtfF006RhT1Ng3_rLCk0Nu2Sq3R4igS2pojwDptf9W2qk0DReRGmD6e_lki8P6ZLzlTC1Lat_VfW2wq8j-40I7EWGRDgNQEb7jGaSm0-nE33N3MvDuR1IYgNhc5HcmGagJDeWd4WdhcWw1GK0?purpose=inline)

[learn.microsoft.com](https://learn.microsoft.com/en-us/answers/questions/1461657/need-a-storage-account-to-use-cloud-shell-in-exerc?utm_source=chatgpt.com)

Open [Azure Portal](https://portal.azure.com?utm_source=chatgpt.com) and click the Cloud Shell icon (`>_`) in the top toolbar.

Choose **Bash**.

### **Cloud Shell vs Azure CLI vs Azure PowerShell**

| Tool              | Description                       | Example                 |
| ----------------- | --------------------------------- | ----------------------- |
| Azure Cloud Shell | Browser-based command environment | Runs Bash or PowerShell |
| Azure CLI         | Cross-platform command-line tool  | `az group list`         |
| Azure PowerShell  | PowerShell modules for Azure      | `Get-AzResourceGroup`   |
| Bash              | Shell and scripting language      | `echo`, `grep`, `for`   |

Cloud Shell may use temporary or persistent storage depending on its configuration. Check whether any storage resource is being created before accepting setup prompts.

### **Practice commands**

**1. Check your current directory**

```
pwd
```

Shows your current working directory.

**2. List files**

```
ls -la
```

Shows files, directories, and permissions.

**3. Check the current user**

```
whoami
```

Shows the current Linux user.

**4. Check Azure CLI version**

```
az version
```

**5. Display Azure CLI help**

```
az --help
```

**6. Explore resource group commands**

```
az group --help
```

**7. Explore resource group creation options**

```
az group create --help
```

### **Expected outcome**

You should understand that Cloud Shell is the execution environment, while Azure CLI is the tool used to communicate with Azure.

## **Task 2 — Understand Azure CLI command structure**

Most Azure CLI commands follow this pattern:

```
az <command-group> <subcommand> [parameters]
```

For example:

```
az group show --name rg-az104-dev-cin-001
```

| Part                   | Meaning                                       |
| ---------------------- | --------------------------------------------- |
| `az`                   | Azure CLI executable                          |
| `group`                | Resource group command category               |
| `show`                 | Retrieve information about one resource group |
| `--name`               | Specifies the resource group name             |
| `rg-az104-dev-cin-001` | Target resource group                         |

### **Common Azure CLI commands**

| Command            | Purpose                           |
| ------------------ | --------------------------------- |
| `az login`         | Authenticate to Azure             |
| `az account show`  | Show active subscription          |
| `az account list`  | List accessible subscriptions     |
| `az group list`    | List resource groups              |
| `az group show`    | Show one resource group           |
| `az group create`  | Create or update a resource group |
| `az group exists`  | Check group existence             |
| `az resource list` | List Azure resources              |
| `az tag update`    | Update tags on a resource         |
| `az group delete`  | Delete a resource group           |

## **Task 3 — Subscription administration**

### **Command 1 — Display the active subscription**

```
az account show -o table
```

This may include subscription identifiers. Avoid publishing unredacted output.

### **Command 2 — Display only the subscription name and status**

```
az account show \
  --query "{Name:name,State:state,Default:isDefault}" \
  -o table
```

**Expected output example:**

```
Name          State     Default
------------  --------  -------
Free Trial    Enabled   True
```

### **Command 3 — List accessible subscriptions**

```
az account list \
  --query "[].{Name:name,State:state}" \
  -o table
```

### **Command 4 — Check the active Azure cloud**

```
az cloud show --query name -o tsv
```

Expected:

```
AzureCloud
```

### **Command 5 — List available regions**

```
az account list-locations \
  --query "[].{Name:name,DisplayName:displayName}" \
  -o table
```

This lists Azure locations available in subscription metadata. It does not guarantee that every service or VM size is available in each region.

## **Task 4 — Resource group administration**

We will use the existing resource group from Lab 02.

**Target resource group**

`rg-az104-dev-cin-001`

Do not delete this group during Lab 03.

### **Command 1 — List all resource groups**

```
az group list -o table
```

### **Command 2 — Retrieve a specific resource group**

```
az group show \
  --name rg-az104-dev-cin-001 \
  -o json
```

### **Command 3 — Retrieve only its name and region**

```
az group show \
  --name rg-az104-dev-cin-001 \
  --query "{Name:name,Location:location}" \
  -o table
```

### **Command 4 — Check provisioning status**

```
az group show \
  --name rg-az104-dev-cin-001 \
  --query properties.provisioningState \
  -o tsv
```

Expected:

```
Succeeded
```

### **Command 5 — Check whether the group exists**

```
az group exists \
  --name rg-az104-dev-cin-001
```

Expected:

```
true
```

### **Command 6 — List resources inside the group**

```
az resource list \
  --resource-group rg-az104-dev-cin-001 \
  -o table
```

If the resource group is empty, the command may return no rows.

## **Task 5 — Azure CLI output formats**

Azure CLI supports several output formats.

| **Format** | **Usage**                       |
| ---------- | ------------------------------- |
| `json`     | Structured data and automation  |
| `table`    | Human-readable terminal output  |
| `tsv`      | Single values and shell scripts |
| `yaml`     | Readable structured output      |

Run the same command with different formats:

```
az group list -o json
```

```
az group list -o table
```

```
az group list -o yaml
```

```
az group list --query "[].name" -o tsv
```

### **Why this matters**

In real DevOps pipelines, JSON is useful for structured processing, while TSV is useful for assigning individual values to shell variables.

## **Task 6 — Learn JMESPath queries**

Azure CLI uses **JMESPath** to filter and transform JSON output with `--query`.

This is especially useful when working with large Azure environments.

### **Example 1 — Display resource group names**

```
az group list \
  --query "[].name" \
  -o tsv
```

### **Example 2 — Filter resource groups in Central India**

```
az group list \
  --query "[?location=='centralindia'].name" \
  -o table
```

### **Example 3 — Display resource groups and locations**

```
az group list \
  --query "[].{Name:name,Region:location}" \
  -o table
```

### **Example 4 — Display a resource group's tags**

```
az group show \
  --name rg-az104-dev-cin-001 \
  --query tags \
  -o json
```

### **Example 5 — Get only the Environment tag**

```
az group show \
  --name rg-az104-dev-cin-001 \
  --query "tags.Environment" \
  -o tsv
```

Expected:

```
Dev
```

### **Example 6 — Count resource groups**

```
az group list \
  --query "length(@)" \
  -o tsv
```

This returns the number of resource groups in your current subscription.

## **Task 7 — Bash variables with Azure CLI**

Instead of repeating resource names, use Bash variables.

### **Step 1 — Define variables**

```
RG_NAME="rg-az104-dev-cin-001"
LOCATION="centralindia"
```

### **Step 2 — Print variables**

```
echo "$RG_NAME"
echo "$LOCATION"
```

### **Step 3 — Use variables in Azure CLI**

```
az group show \
  --name "$RG_NAME" \
  -o table
```

### **Step 4 — Store Azure CLI output in a variable**

```
RG_LOCATION=$(az group show \
  --name "$RG_NAME" \
  --query location \
  -o tsv)
```

### **Step 5 — Display the result**

```
echo "Resource group location: $RG_LOCATION"
```

Expected:

```
Resource group location: centralindia
```

**Interview takeaway:** Variables make scripts easier to maintain and reuse across environments.

## **Task 8 — Create your first Azure administration script**

Now we'll combine the commands into one Bash script.

### **Step 1 — Create a script**

In Cloud Shell Bash, run:

```
nano azure-inventory.sh
```

Paste the following:

```

#!/usr/bin/env bash
set -euo pipefail

RG_NAME="rg-az104-dev-cin-001"

echo "===================================="
echo " AZ-104 Azure Infrastructure Report "
echo "===================================="

echo ""
echo "1. Active Azure Subscription"
az account show \
  --query "{Name:name,State:state}" \
  -o table

echo ""
echo "2. Resource Groups"
az group list \
  --query "[].{Name:name,Location:location}" \
  -o table

echo ""
echo "3. Target Resource Group"
az group show \
  --name "$RG_NAME" \
  --query "{Name:name,Location:location,Tags:tags}" \
  -o json

echo ""
echo "4. Resources in Target Group"
az resource list \
  --resource-group "$RG_NAME" \
  --query "[].{Name:name,Type:type,Location:location}" \
  -o table

echo ""
echo "5. Total Resource Groups"
az group list \
  --query "length(@)" \
  -o tsv

echo ""
echo "Azure inventory completed successfully."

```

Save the file:

- Press `Ctrl + O`, then Enter.
- Press `Ctrl + X`.

### **Step 2 — Make the script executable**

```
chmod +x azure-inventory.sh
```

### **Step 3 — Run the script**

```
./azure-inventory.sh
```

### **Step 4 — Save the report**

```
./azure-inventory.sh > azure-inventory-report.txt
```

### **Step 5 — View the report**

```
cat azure-inventory-report.txt
```

**Expected deliverable**

`azure-inventory-report.txt`

A report containing subscription status, resource group inventory, tags, deployed resources, and resource group count. Review it for sensitive details before uploading.

## **Task 9 — Troubleshooting exercises**

Practice these safe scenarios.

| Scenario                      | Command / investigation                        | Learning                              |
| ----------------------------- | ---------------------------------------------- | ------------------------------------- |
| Incorrect resource group name | `az group show --name rg-does-not-exist`       | ResourceGroupNotFound                 |
| Wrong subscription selected   | `az account show`                              | Subscription context                  |
| Missing tags                  | `az group show --name "$RG_NAME" --query tags` | Governance validation                 |
| CLI syntax issue              | `az group show --help`                         | Command troubleshooting               |
| Empty resource inventory      | `az resource list -g "$RG_NAME"`               | Distinguish empty results from errors |

Do not deliberately remove your own access or change subscriptions to create failures.

## **5. GitHub deliverables**

After completing the lab, document the results under:

```
phase-00-azure-foundations/
|
|-- lab-03-cloud-shell-cli/
|   |-- README.md
|   |
|   |-- screenshots/
|       |-- 01-cloud-shell.png
|       |-- 02-subscription-info.png
|       |-- 03-resource-groups.png
|       |-- 04-jmespath-query.png
|      
```

### **Git commit**

```
git add phase-00-azure-foundations/lab-03-cloud-shell-cli/

git commit -m "feat(az104): automate Azure resource inventory using Cloud Shell and CLI"

git push origin main
```

Only commit sanitized reports and screenshots.

## **6. AZ-104 and DevOps interview questions**

1\. What is Azure Cloud Shell?

2\. What is the difference between Azure CLI and Azure PowerShell?

3\. What is JMESPath in Azure CLI?

4\. What is the difference between -o json and -o tsv?

5\. Why use Bash variables in Azure administration?

6\. How can you check the currently selected subscription?

## **7. Lab 03 completion checklist**

**Lab 03 Progress**

10/10

Explored Azure Cloud Shell and Azure CLI help

Verified active subscription and Azure cloud

Executed resource group administration commands

Practiced JSON, table, TSV, and YAML output

Completed JMESPath filtering exercises

Used Bash variables with Azure CLI

Created azure-inventory.sh

Executed the script and generated an inventory report

Practiced troubleshooting scenarios


# **AZ-104 — Phase 01 | Lab 01: Microsoft Entra ID User Lifecycle Management**

**Project: Enterprise Identity Management — Joiner, Mover & Leaver (JML)**

In Phase 00, you already created three Microsoft Entra security groups and configured Azure RBAC permissions.

Now we'll build on that foundation by simulating how an organization manages employees from onboarding through role changes and offboarding.

Hands-on Lab

Duration: 2–3 hours

Subscription

**Azure Free Trial**

Estimated lab cost

**$0**

AZ-104 domain

**Identity & Governance**

Resource group

rg-az104-dev-cin-001

We'll use basic Microsoft Entra ID features without purchasing premium licenses. User creation requires appropriate directory permissions.

## **1. Real-world scenario**

Imagine you're working as an Azure Administrator for a company with three departments.

| Department  | Responsibilities                              | Existing security group |
| ----------- | --------------------------------------------- | ----------------------- |
| Development | Deploy and manage Azure development resources | `az104-dev-group`       |
| Operations  | Monitor and inspect Azure resources           | `az104-ops-group`       |
| Audit       | Review resources for compliance               | `az104-audit-group`     |

The HR team sends three requests:

1. **Joiner:** A new developer joins the company and needs development access.
2. **Mover:** The developer transfers to the operations team and should no longer have development permissions.
3. **Leaver:** The employee leaves the organization, and access must be revoked.

Your responsibility is to manage the employee's identity, group memberships, and Azure permissions without granting unnecessary access.

## **2. Architecture — What we're implementing**

The same test user moves between groups. Azure RBAC assignments remain on the groups, while effective permissions change with membership.

## **3. Concepts you need to understand**

### **What is identity lifecycle management?**

Identity lifecycle management controls an employee's access throughout their employment.

| Stage  | Description                 | Administrator action                              |
| ------ | --------------------------- | ------------------------------------------------- |
| Joiner | New employee joins          | Create identity and grant required access         |
| Mover  | Employee changes department | Remove old permissions and assign new permissions |
| Leaver | Employee leaves             | Block sign-in and revoke access                   |

### **Direct role assignments vs group-based access**

**Direct assignment:**

```
User → Azure RBAC Role → Resource Group
```

**Group-based assignment:**

```
User → Security Group → Azure RBAC Role → Resource Group
```

Group-based access is easier to maintain when employees join, move between departments, or leave.

For example, when a developer moves to operations, we can remove them from `az104-dev-group` and add them to `az104-ops-group` without modifying the existing Azure RBAC assignments.

# **4. Hands-on implementation**

## **Task 1 — Verify existing security groups**

**Estimated time:** 10 minutes

Open [Microsoft Entra admin center](https://entra.microsoft.com?utm_source=chatgpt.com) or use Azure Portal.

Navigate to:

**Microsoft Entra ID → Groups → All groups**

Verify these three groups exist:

| Security group      | Azure RBAC role |
| ------------------- | --------------- |
| `az104-dev-group`   | Contributor     |
| `az104-ops-group`   | Reader          |
| `az104-audit-group` | Reader          |

Your Phase 00 screenshot already confirmed these role assignments at `rg-az104-dev-cin-001`.

We will reuse them instead of creating duplicate groups.

## **Task 2 — Create a test employee (Joiner)**

**Estimated time:** 20 minutes

### **Step 1 — Navigate to Users**

Open:

**Microsoft Entra ID → Users → All users → New user → Create new user**

### **Step 2 — Enter test user details**

Use the following values:

| Field                   | Value                            |
| ----------------------- | -------------------------------- |
| User principal name     | `devuser01@<your-tenant-domain>` |
| Display name            | `AZ104 Dev User 01`              |
| First name              | `AZ104`                          |
| Last name               | `DevUser01`                      |
| Account enabled         | Yes                              |
| Password                | Auto-generated                   |
| Require password change | Yes, if available                |

Replace `<your-tenant-domain>` with the verified domain shown in Microsoft Entra ID.

Do not use a real employee's identity for this lab.

### **Step 3 — Create the user**

Select **Review + create**, then **Create**.

Store the temporary password securely. Never add it to GitHub, screenshots, or documentation.

### **Step 4 — Verify**

Search for `AZ104 Dev User 01` under **All users**.

**Expected outcome:** The test user exists and is enabled.

**Screenshot 01**

`01-test-user-created.png`

Capture the test user's overview page, hiding identifiers and any personal details.

## **Task 3 — Add the user to the development group**

**Estimated time:** 15 minutes

Navigate to:

**Microsoft Entra ID → Groups → az104-dev-group → Members → Add members**

1. Search for `AZ104 Dev User 01`.
2. Select the user.
3. Click **Select**.
4. Verify the user appears in the group membership list.

### **What happens?**

Your existing Azure RBAC configuration is:

```
az104-dev-group
      |
      | Contributor
      v
rg-az104-dev-cin-001
```

Because the test user belongs to `az104-dev-group`, they inherit the group's Contributor permissions at that resource-group scope.

### **Validation**

Open:

**Resource groups → rg-az104-dev-cin-001 → Access control (IAM) → Check access**

Search for the test user and review their effective access.

Group membership and RBAC changes can take time to propagate.

**Screenshot:** `02-development-group-membership.png`

## **Task 4 — Validate developer access**

**Estimated time:** 20 minutes

Use a separate browser profile or private window to sign in as the test user, if you have completed its password setup.

### **Expected permissions**

| Action                           | Expected result                      |
| -------------------------------- | ------------------------------------ |
| View development resource group  | Allowed                              |
| View resource group tags         | Allowed                              |
| Update resource group tags       | Generally allowed                    |
| Assign Azure RBAC roles          | Denied                               |
| Manage supported Azure resources | Allowed, subject to policy and locks |

### **Safe test**

You can inspect the resource group and attempt a permitted tag update.

Avoid deploying chargeable VMs or storage accounts for this exercise.

**Important:** If your own administrator account is used for testing, its inherited Owner permissions will invalidate a Reader/Contributor comparison. Test using the separate identity.

## **Task 5 — Simulate employee transfer (Mover)**

**Estimated time:** 20 minutes

Now assume the employee transfers from Development to Operations.

The employee should no longer have Contributor permissions.

### **Step 1 — Remove development membership**

Navigate to:

**Microsoft Entra ID → Groups → az104-dev-group → Members**

Select `AZ104 Dev User 01` and remove the membership.

### **Step 2 — Add operations membership**

Navigate to:

**Microsoft Entra ID → Groups → az104-ops-group → Members → Add members**

Select the same test user.

### **Step 3 — Verify membership**

The intended state is:

| Group               | Membership   |
| ------------------- | ------------ |
| `az104-dev-group`   | Removed      |
| `az104-ops-group`   | Added        |
| `az104-audit-group` | Not assigned |

### **Step 4 — Validate permissions**

After membership and token changes propagate, sign in again as the test user.

Expected behavior:

| Action           | Before transfer | After transfer |
| ---------------- | --------------- | -------------- |
| View resources   | Allowed         | Allowed        |
| Modify resources | Allowed         | Denied         |
| Assign roles     | Denied          | Denied         |

This demonstrates how **group membership controls Azure resource access**.

If Contributor permissions remain, investigate inherited roles, direct role assignments, stale credentials, and propagation delays.

**Screenshot:** `03-user-moved-to-operations.png`

## **Task 6 — Simulate employee offboarding (Leaver)**

**Estimated time:** 20 minutes

Now assume the employee leaves the organization.

### **Step 1 — Remove group memberships**

Remove the test user from `az104-ops-group` and any other lab groups.

### **Step 2 — Block sign-in**

Navigate to:

**Microsoft Entra ID → Users → AZ104 Dev User 01 → Properties**

Find the account enabled setting and disable the account.

Save the changes.

### **Step 3 — Revoke sessions**

Where available, use **Revoke sessions** to invalidate refresh tokens.

Blocking sign-in and revoking sessions do not necessarily invalidate already-issued access tokens immediately; token expiration and service behavior affect enforcement timing.

### **Step 4 — Verify**

Confirm:

- The test user is disabled.
- The user no longer belongs to the development or operations groups.
- No direct Azure RBAC assignments remain for the user.
- New sign-in attempts are blocked.

**Screenshot:** `04-user-disabled.png`

### **Optional cleanup**

Once you've documented the lab, you can delete the synthetic test user if no further validation is needed.

Deleting a user may affect recovery and retention behavior, so perform it only for your test identity.

# **5. Azure CLI and Microsoft Graph validation**

Azure CLI can help inspect Microsoft Entra users and group membership.

Some directory operations require Microsoft Graph permissions and appropriate Entra roles.

## **Command 1 — List security groups**

```
az ad group list \
  --query "[?starts_with(displayName, 'az104-')].{Name:displayName}" \
  -o table
```

## **Command 2 — Find the test user**

```
az ad user list \
  --filter "displayName eq 'AZ104 Dev User 01'" \
  --query "[].{Name:displayName,UPN:userPrincipalName}" \
  -o table
```

## **Command 3 — Retrieve the development group object ID**

```
DEV_GROUP_ID=$(az ad group show \
  --group az104-dev-group \
  --query id \
  -o tsv)
```

## **Command 4 — List group members**

```
az ad group member list \
  --group "$DEV_GROUP_ID" \
  --query "[].{Name:displayName,UPN:userPrincipalName}" \
  -o table
```

## **Command 5 — Check the operations group**

```
az ad group member list \
  --group az104-ops-group \
  --query "[].{Name:displayName,UPN:userPrincipalName}" \
  -o table
```

### **What these commands demonstrate**

You can query Microsoft Entra identities and security groups without navigating through the portal.

These commands are useful for identity audits, access reviews, and administrative automation.

# **6. Real-world troubleshooting scenarios**

| Scenario                                     | Possible cause                                 | Investigation                              |
| -------------------------------------------- | ---------------------------------------------- | ------------------------------------------ |
| Cannot create a user                         | Missing directory permissions                  | Check Entra role and tenant settings       |
| User cannot access resource group            | Membership or RBAC not effective               | Inspect group and role assignments         |
| User still has Contributor after transfer    | Direct/inherited role or cached credentials    | Check effective access and reauthenticate  |
| Disabled user still has an active session    | Previously issued tokens                       | Revoke sessions and allow token expiration |
| Group not found in CLI                       | Incorrect name or tenant                       | Verify directory and group object ID       |
| User can sign in but cannot manage resources | Authentication succeeded, authorization denied | Inspect Azure RBAC and Policy              |

## **7. GitHub documentation**

Create this structure:

```
phase-01-identity-governance/
|
|-- lab-01-entra-user-lifecycle/
|   |-- README.md
|   |
|   |-- screenshots/
|       |-- 01-test-user-created.png
|       |-- 02-development-group-membership.png
|       |-- 03-user-moved-to-operations.png
|       |-- 04-user-disabled.png

```

### **Git commit**

After completing the lab:

```
git add phase-01-identity-governance/lab-01-entra-user-lifecycle/

git commit -m "feat(az104): implement Entra ID joiner mover leaver lifecycle lab"

git push origin main
```

Do not upload temporary passwords, authentication tokens, or unredacted identity information.

## **8. AZ-104 interview questions**

1\. What is Joiner-Mover-Leaver (JML) in identity management?

2\. Why use security groups for Azure RBAC?

3\. What happens when a user is removed from a group with Contributor access?

4\. What is the difference between disabling and deleting a user?

5\. Why might permissions remain after a group membership change?

6\. How do you troubleshoot a user who can sign in but cannot access Azure resources?

## **9. Lab 01 completion checklist**

**Lab 01 Progress**

11/11

Verified the three existing security groups

Created a synthetic test user

Added the user to az104-dev-group

Validated Contributor access

Removed the user from development group

Added the user to operations group

Validated Reader-only access

Removed remaining lab group memberships

Disabled the user and reviewed session revocation

Executed Azure CLI identity queries

Prepared redacted screenshots and GitHub documentation


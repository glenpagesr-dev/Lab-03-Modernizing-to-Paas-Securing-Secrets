# Lab 03: Modernizing to PaaS & Securing Secrets
In this lab, i will modernize an application to Azure Paas services and implement secure secret management using Azure key Vault, RBAC, and managed identities to eliminate hard-coded credentials and enforce least-privilege access. 

**Author:** Glen Page  
**Estimated Time:** 60 Minutes  
**Difficulty:** Intermediate  

---

## Objective

This lab demonstrates how to modernize an Infrastructure-as-a-Service (IaaS) architecture by replacing a Database Virtual Machine with a fully managed Azure SQL Database (PaaS).  

You will also deploy Azure Key Vault to securely store database credentials and enable a Managed Identity on the Web Server so it can access secrets without storing passwords in configuration files.

---

## Architecture Overview

In this refactored design:

- `vm-web-01` retrieves a secret from **Azure Key Vault**
- The web server connects to **Azure SQL Database (PaaS)**
- `vm-db-01` no longer exists

Flow:


vm-web-01 → Azure Key Vault → Azure SQL Database


---

## Prerequisites

- [ ] Active Azure Subscription  
- [ ] Completed Lab 02 (vm-web-01 must be running)  
- [ ] Completed Week 3 Video Modules  

---

## Lab Variables (Naming Convention)

- **Resource Group:** `rg-lab03-[yourname]`
- **Region:** East US
- **SQL Server Name:** `sql-server-[yourname]` (Must be globally unique)
- **Database Name:** `sqldb-app`
- **Key Vault Name:** `kv-lab03-[yourname]` (Must be globally unique)

---

## Steps Performed

---

### Phase 1: Decommission the Old Database

1. Navigate to:

Resource Groups → rg-lab02-[yourname]


2. Locate:
- `vm-db-01`
- Associated disk
- Network Interface

3. Delete only these resources.

> ⚠️ Do NOT delete the Resource Group or `vm-web-01`.

---

### Phase 2: Deploy Azure SQL Database (PaaS)

1. Search for **SQL databases** → Click **Create**

#### Basics Tab

- Resource Group → Create New → `rg-lab03-[yourname]`
- Database name → `sqldb-app`
- Server → Click **Create new**

**Server Configuration:**
- Server name → `sql-server-[yourname]` (lowercase)
- Location → East US
- Authentication → SQL authentication
- Server admin login → `sqladmin`
- Password → Create and store securely

Click **OK**

---

#### Compute + Storage (CRITICAL STEP)

1. Click **Configure database**
2. Service tier → **Basic (DTU-based)**
3. Confirm slider is set to **Basic**
4. Click **Apply**

- Backup storage redundancy → Locally-redundant (LRS)

---

#### Networking Tab

- Connectivity method → Public endpoint
- Allow Azure services → Yes
- Add current client IP address → Yes

Click:


Review + Create → Create


---

### Phase 3: Deploy Azure Key Vault

1. Search **Key Vaults** → Click **Create**

#### Basics Tab

- Resource Group → `rg-lab03-[yourname]`
- Key Vault Name → `kv-lab03-[yourname]`
- Region → East US
- Pricing tier → Standard

#### Access Configuration

- Permission model → **Vault access policy**

Click:


Review + Create → Create


---

### Phase 4: Secure the Secret

1. Navigate to your Key Vault.
2. Select:

Objects → Secrets


3. Click **+ Generate/Import**

**Configuration:**
- Upload options → Manual
- Name → `SqlAdminPassword`
- Secret value → Enter SQL password from Phase 2

Click **Create**

---

### Phase 5: Managed Identity & Access

#### Enable Identity on VM

1. Navigate to `vm-web-01`
2. Click:

Identity → System Assigned

3. Switch **Status** to ON
4. Click **Save**

---

#### Grant Key Vault Access

1. Navigate to `kv-lab03-[yourname]`
2. Click **Access policies**
3. Click **+ Create**

**Permissions:**
- Secret permissions → Check:
- Get
- List

**Principal:**
- Click Select
- Search `vm-web-01`
- Select it

Click:


Next → Create


✅ Result: `vm-web-01` can now retrieve secrets from Key Vault.

---

### Phase 6: Validation (Azure Monitor)

1. Navigate to:

sqldb-app


2. Click:

Monitoring → Metrics


3. Metric → DTU percentage  
4. Aggregation → Max  

A metrics chart should appear.  

This confirms the database is operational and observable.

---

## Troubleshooting

**Issue:** Cannot create SQL Database  
**Fix:**  
- Ensure server name is globally unique  
- Confirm Service Tier is set to Basic  

---

**Issue:** Cannot find vm-web-01 when adding Access Policy  
**Fix:**  
- Confirm System Assigned Identity was enabled  
- Click Save and wait 30–60 seconds  

---

## Clean Up Resources

To avoid unnecessary charges:

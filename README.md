
# Terraform Fabric Starter Kit

This guide walks you through everything from installing Terraform and Azure CLI to provisioning Azure resources using CSV-based configurations.

---

## Table of Contents

| #  | Section                          |
|----|----------------------------------|
| 1  | Install Terraform                |
| 2  | Install Azure CLI                |
| 3  | Login to Azure Account           |
| 4  | Creation of Resource group for handling Statefiles           |
| 5  | Terraform Configuration          |
| 6  | Execution Guide                  |
| 7  | Common Issues and Tips           |


---

## 1. Install Terraform

Terraform is used to **automate Azure infrastructure deployment** using simple configuration files.

### 🔧 Installation Steps

1. Visit the official site: [Terraform Downloads](https://developer.hashicorp.com/terraform/install)
2. Download the Windows version.
3. Extract the zip file.
4. Copy `terraform.exe` to a folder like `C:\Terraform`.
5. Add `C:\Terraform` to your system `PATH`:

   * Open Windows Search → Type "Environment Variables"
   * Edit the `Path` variable in system settings
   * Add `C:\Terraform`

### ✅ Verify Installation

```bash
terraform -version
```

Expected Output:

```
Terraform v1.6.x
```

---

## 2. Install Azure CLI

Azure CLI is a command-line tool to **manage and configure Azure resources**.

### 🔧 Installation via PowerShell

```powershell
winget install --exact --id Microsoft.AzureCLI
```

If you don’t have `winget`, download manually:
👉 [Azure CLI Manual Install](https://learn.microsoft.com/en-us/cli/azure/install-azure-cli-windows?view=azure-cli-latest)

### ✅ Verify Installation

```bash
az version
```

---

## 🔐 3. Login to Azure Account

After installing the Azure CLI, **you must log in to access your Azure resources**.

### 👤 Login to Azure

Run this command in Command Prompt or PowerShell:

```bash
az login
```

* This opens a browser and prompts you to sign in with your **Azure AD (Active Directory)** credentials.
* After successful login, it shows your subscriptions and tenant details.

### Tenant and Subscription Selection

| No  | Default | Subscription Name | Subscription ID                             | Tenant           |
|-----|---------|-------------------|---------------------------------------------|------------------|
| [1] | *       | development       | v1979g48-807e-43a4-8f19-c1056b6d5dc4         | JMAN Group Ltd   |

> The default is marked with an `*`; the default tenant is **JMAN Group Ltd** and subscription ID is `v1979g48-807e-43a4-8f19-c1056b6d5dc4`.

Select a subscription and tenant (Type a number or Enter for no changes): 1


### 📍 Purpose

* Required for Terraform to access Azure.
* Your credentials are stored locally for CLI and Terraform usage.

### 💡 Useful Azure AD CLI Commands

* View logged-in user:

  ```bash
  az account show
  ```

---

## 4. Creation of Resource group for handling Statefiles:

* If your project is under the JMAN Tenant ID, the product is already linked to the required resource group; you only need to update the container name in terraform/provider.tf.
* If your project is under a different Tenant ID, you must create a resource group and a storage container, then update these details in terraform/provider.tf.

---

## 5. Terraform Configuration:

* Navigate to the `terraform-configs` folder in fabric starter kit repository.
* Each module in the `terraform-configs` directory reads data from CSV metadata files to dynamically create resources.
* Below you can find all the modules.

### 5.1 Resource Group Module:

To configure the **Resource Group Module**, you need to upload the following metadata file:

### 🔹 File : resource\_group\_variables.csv

| Column Name               | Description                                          |
| ------------------------- | ---------------------------------------------------- |
| `environment`             | Name of the environment: `dev`, `test`, `prod`       |
| `project`                 | Name of the Project                                 |
| `resource_group_name`     | Name of the Azure resource group to be created       |
| `resource_group_location` | Azure region (e.g., `East US`, `West Europe`)        |
| `costcenter`              | Tagname               |
| `container_name`          | Container in the storage account for Terraform state |
| `storage_account_name`    | Azure Storage Account for storing Terraform state    |

🔎 **Purpose**: This defines your resource group, and the backend state configuration needed for Terraform.

---

### 5.2 Fabric Capacity Module:

To configure the **Fabric Capacity Module**, you need to upload the following metadata file:

### 🔹 File : fabric\_variables.csv

| Column Name   | Description   |
| ------------- | ------------- |
| `environment` | Same as above |
| `project`     | Same as above |
| `costcenter`  | Same as above |

🔎 **Purpose**: Used to tag resources and group them logically by environment or project.

### 🔹 File : capacities.csv

| Column Name | Description                                                                        |
| ----------- | ---------------------------------------------------------------------------------- |
| `name`      | Name of the capacity (e.g., `cap-dev`)                                             |
| `location`  | Azure region                                                                       |
| `size`      | Capacity size (e.g., `F64`, `F32`)                                                 |
| `members`   | Comma-separated list of Azure AD users (e.g., `"[""user1@jmangroup.com"", ""user2@jmangroup.com""]"`) |

🔎 **Purpose**: Creates and configures Fabric Capacity with assigned users.

> **Note:**
> If you need to create a capacity larger than 64 units, please contact your Azure subscription manager to upgrade your subscription tier accordingly.

---

### 5.3 Application Module:

To configure the **Application Module**, you need to upload the following metadata file:

### 🔹 File : application\_variables.csv

| Column Name                | Description                              |
| -------------------------- | ---------------------------------------- |
| `application_display_name` | Name of the Azure AD App                 |
| `current_user_upn`         | User email (e.g., `john.doe@domain.com`) |

🔎 **Purpose**: Used for creating Azure AD apps and assigning the User.

---

### 5.4 Key vault Module:

To configure the **Key vault Module**, you need to upload the following metadata file:

### 🔹 File : key\_vault\_variables.csv

| Column Name                            | Description                  |
| -------------------------------------- | ---------------------------- |
| `key_vault_name`                       | Azure Key Vault name         |
| `location`                             | Region                       |
| `resource_group_name`                  | RG where Key Vault is placed |
| `environment`, `project`, `costcenter` | Same as above                |



### 🔹 File : vault\_access\_policies.csv

| Column Name               | Description                        |
| ------------------------- | ---------------------------------- |
| `email`                   | Azure AD User or Service Principal |
| `key_permissions`         | The specific permissions granted to the user or service principal for managing keys in the Azure Key Vault e.g., `"[""Get"", ""List""]"`     |
| `secret_permissions`      | The specific permissions granted to the user or service principal for managing secrets in the Azure Key Vault e.g., `"[""Get"", ""list"", ""set""]"`    |
| `certificate_permissions` | The specific permissions granted to the user or service principal for managing certificates in the Azure Key Vault e.g., `"[""Get"", ""import""]"`      |

🔎 **Purpose**: Defines access to Key Vault secrets, keys, and certificates.

---

### 5.5 Roles Assignment Module:

### Role Assignments (IAM) in Azure

**IAM (Identity and Access Management)** helps define **who** has access to **what** in Azure.

### 🔑 Common IAM Roles

| Role Name                     | Permissions                             |
| ----------------------------- | --------------------------------------- |
| **Owner**                     | Full access + assign roles              |
| **Contributor**               | Full access except managing permissions |
| **Reader**                    | View-only access                        |
| **User Access Administrator** | Manage access permissions               |
| **Key Vault Contributor**     | Manage Key Vault content                |

📖 **Reference**:
👉 [Azure Built-in Roles List](https://learn.microsoft.com/en-us/azure/role-based-access-control/built-in-roles)

### 🔗 Example Role Assignment via Terraform

```hcl
resource "azurerm_role_assignment" "example" {
  scope                = azurerm_resource_group.example.id
  role_definition_name = "Contributor"
  principal_id         = data.azuread_user.user.object_id
}
```

To configure the **Roles Assignment Module**, you need to upload the following metadata file:

### 🔹 File : roles\_and\_users.csv

| Column Name | Description                                          |
| ----------- | ---------------------------------------------------- |
| `role`      | Role (e.g., `Contributor`, `Reader`)                 |
| `username`  | Mail address of the user                             |

---

## 6. Execution Guide

Follow the steps below to configure and run the deployment script for setting up infrastructure and workspaces:

### 6.1 Execute the Script

Once the configuration is complete, run the following command in Git Bash to start the script:

 ```bash
   ./run_teraform_and_python.sh
   ```

This command starts by cloning the fabric-kit repository, which contains the necessary Python scripts for deploying the Fabric infrastructure on Azure.
Here we need to update the metadata files that are needed to run the python script.

In the "workspace_role_assignment" csv file in the "terraform/fabric-kit/fabirc-kit-configs" folder.
   - Follow the general guidliness and provide access for project manager and contributor or viewer acess for the team members.
   - Note: Ensure to take user_id from Azure portal for the users by navigating to Microsoft Entra ID from Azure portal and fill it.
   - File Structure:

   | user_id              | access-level                    | user_name   |
   |----------------------|---------------------------------|-------------|
   |Take from Azure portal| Roles like Admin/Contributor    | User's email| 


### 6.2 Select an Option

After running the script, you will be prompted to select one of the following options:

* **1 → Infrastructure Setup + Fabric Workspace**
* **2 → Fabric Workspace Only**

Choose the appropriate option based on your project requirements.

#### Option 1: Infrastructure Setup + Fabric Workspace
- Provisions all required Azure infrastructure components.
- Sets up necessary resources like Resource Groups, Storage Accounts, Key Vaults, etc.
- Deploys a new Microsoft Fabric workspace within the provisioned infrastructure.
> Recommended for new or clean environments where infrastructure does not already exist.

#### Option 2: Fabric Workspace Only
- Skips infrastructure provisioning.
- Only deploys a Microsoft Fabric workspace using existing Azure infrastructure.
- Suitable when the necessary resources (e.g., Resource Group, Storage Account) are already provisioned.
> Ideal for environments where you want to reuse or manage infrastructure separately.

If you select option 2 you need to update the details in 'credentials.csv' in the "terraform/fabric-kit/fabirc-kit-configs" folder.
   - File Structure:

   | tenant_id                      | client_id                                          |
   |--------------------------------|----------------------------------------------------|
   |Identifier for the organization | Identifier for the application requesting access   |

---

## Script Functionality Options

### 1. Create Workspace and Onboard user
   Note: Make sure to obtain the capacity name from Azure, as it will be required during the deployment process.
     select "create workspace and onboard user" to create a workspace and onboard users.
   - Enter the capacity name for the connection and specify a name for the workspace.
   - Four workspaces (dev, prod, landing-dev, landing-prod) will be created, and users will be assigned to these workspaces based on the permissions defined.

### 2. Exit

* Select "Exit" to return to the previous menu or terminate the process.

---

After creating the workspace, navigate to each one individually, open Workspace Settings, and configure Git Integration to connect it with these details.

| Repo         | Branch | OrganizationName | ProjectName           | GitProviderType | RepositoryName        | DirectoryName          |
|--------------|--------|------------------|----------------------|-----------------|-----------------------|------------------------|
| dev          | dev    | jmangroupltd     | Data-Engineering-CoE | AzureDevOps     | Data-Engineering-CoE  | ws_dev_started_kit     |
| prod         | prod   | jmangroupltd     | Data-Engineering-CoE | AzureDevOps     | Data-Engineering-CoE  | ws_prod_starter_kit    |
| landing-dev  | dev    | jmangroupltd     | Data-Engineering-CoE | AzureDevOps     | Data-Engineering-CoE  | ws_dev_started_kit     |
| landing-prod | prod   | jmangroupltd     | Data-Engineering-CoE | AzureDevOps     | Data-Engineering-CoE  | ws_prod_starter_kit    |

After importing the files from Git, disconnect the Git integration from the workspace.

---

## 🛠️ 7. Common Issues and Fixes

| Issue                             | Fix                                                               |
| --------------------------------- | ----------------------------------------------------------------- |
| `az login` stuck or fails         | Run in PowerShell (not Git Bash), try `az logout` then login      |
| `terraform: command not found`    | Ensure Terraform path is set in environment variables             |
| Incorrect subscription being used | Use `az account set --subscription <id>`                          |
| CSV file not read                 | Ensure correct path and format (no extra commas or empty headers) |



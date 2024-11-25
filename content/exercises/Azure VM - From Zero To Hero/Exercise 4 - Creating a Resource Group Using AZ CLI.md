+++
title = "4. Creating a Resource Group Using AZ CLI"
weight = 4
date = 2024-11-25
draft = false
+++

# Exercise 4: Creating a Resource Group Using AZ CLI

---

## Overview
This exercise introduces the **Azure CLI** for managing Azure resources. You will use Azure CLI commands to create a resource group and verify its creation.

---

## **Step-by-Step Instructions**

### Step 1: Log in to Azure Cloud Shell
1. Open the [Azure Portal](https://portal.azure.com/) and click the **Cloud Shell** icon (top-right corner).
2. Select **Bash** as your shell environment if prompted.

   > 💡 **Information Box:**  
   > The Azure Cloud Shell is an integrated command-line interface in the Azure Portal that comes pre-configured with the Azure CLI. It enables you to run CLI commands without additional setup.

---

### Step 2: Create a Resource Group
1. Run the following command to create a resource group:
   ```bash
   az group create --name MyCLIDemoGroup --location northeurope
   ```
   - **`--name`**: Specifies the name of the resource group (e.g., `MyCLIDemoGroup`).
   - **`--location`**: Sets the region where the resources in the group will be located (e.g., `northeurope`).

2. Review the output to confirm successful creation. You should see a JSON response similar to:
   ```json
   {
     "id": "/subscriptions/<subscription-id>/resourceGroups/MyCLIDemoGroup",
     "location": "eastus",
     "name": "MyCLIDemoGroup",
     "properties": {
       "provisioningState": "Succeeded"
     }
   }
   ```

   > 💡 **Information Box:**  
   > A **Resource Group** acts as a container for all resources within Azure, making it easier to organize, manage, and delete resources as a single unit.

---

### Step 3: Verify the Resource Group
1. List all resource groups in your subscription:
   ```bash
   az group list --output table
   ```
2. Verify that `MyCLIDemoGroup` appears in the output. It should display details like the **name**, **location**, and **provisioning state**.

> ✅ **Verification Step:**  
> Ensure the table includes `MyCLIDemoGroup` with the status `Succeeded` under the **ProvisioningState** column.

---

### Step 4: Clean Up Resources (Optional)
1. Delete the resource group to avoid unnecessary costs:
   ```bash
   az group delete --name MyCLIDemoGroup --no-wait --yes
   ```
   - **`--no-wait`**: Ensures the command returns immediately without waiting for the operation to complete.
   - **`--yes`**: Confirms the deletion without additional prompts.

2. Verify that the resource group no longer exists:
   ```bash
   az group list --output table
   ```
   - Confirm that `MyCLIDemoGroup` no longer appears in the list.

   > 💡 **Information Box:**  
   > Always clean up unused resources to avoid unexpected charges in your Azure subscription.

---

## Exercise Complete!
You have successfully created and managed a resource group using Azure CLI, gaining hands-on experience with command-line operations in Azure.
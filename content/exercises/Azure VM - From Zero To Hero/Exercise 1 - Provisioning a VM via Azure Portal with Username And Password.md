+++
title = "1. Provisioning a VM via Azure Portal"
weight = 1
date = 2024-11-25
draft = false
+++

# Exercise 1: Provisioning a VM via Azure Portal with Username/Password

## Overview
This exercise introduces the Azure Portal, guiding you through the creation of a Virtual Machine (VM). You will configure a Linux VM with a username/password for access and verify that the VM is running successfully.

---

## **Step-by-Step Instructions**

### Step 1: Log in to Azure Portal
1. Open a web browser and navigate to the [Azure Portal](https://portal.azure.com/).
2. Sign in using your Azure account credentials.

### Step 2: Create a Resource Group
1. In the Azure Portal, search for **"Resource groups"** in the search bar.
2. Click **"Create"** and fill in the following:
   - **Subscription**: Choose your subscription.
   - **Resource Group Name**: `LabResourceGroup`
   - **Region**: Choose a region (e.g. `North Europe`).
3. Click **Review + Create** and then **Create**.

> 💡 **Information**<br>
> A **Resource Group** is a container that holds related resources for an Azure solution. It allows for easy management and deletion of all resources together.

### Step 3: Create a Virtual Machine
1. In the Azure Portal, search for **"Virtual Machines"** and click **"Create"** > **"Virtual machine"**.
2. Configure the VM:
   - **Subscription**: Select your subscription.
   - **Resource Group**: Choose `LabResourceGroup`.
   - **Virtual Machine Name**: `LabVM`
   - **Region**: Same as the resource group.
   - **Image**: Select `Ubuntu Server 24.04 LTS`.
   - **Size**: Choose a size like `Standard_B1s`.
   - **Authentication Type**: Select **Password**.
   - **Username**: `azureuser`
   - **Password**: Create a secure password and confirm it.
3. Configure inbound ports:
   - Check **Allow selected ports** and select **HTTP (80)** and **SSH (22)**.
4. Click **Review + Create**, and then **Create**.

> 💡 **Information**<br>
> - **VM Size:** Determines the CPU, memory, and storage capacity. For this exercise, `Standard_B1s` is cost-effective for learning purposes.  
> - **Inbound Ports:** Ensure HTTP (for web traffic) and SSH (for secure access) are open for connectivity.

### Step 4: Monitor Deployment
1. After clicking **Create**, Azure will begin provisioning the VM.  
2. Once completed, navigate to the **Virtual Machines** section and locate `LabVM`.

> ✅ **Verification Step:**<br>
> Ensure the VM status is **"Running"** in the Virtual Machines overview.

---

### Step 5: Connect to the VM Using Azure Cloud Shell
1. In the Azure Portal, click the **Cloud Shell** icon (top-right corner)
2. Select **Bash** as your shell environment
3. Connect to your VM via SSH:

   ```bash
   ssh azureuser@<VM_Public_IP>
   ```

   - Replace `<VM_Public_IP>` with the public IP address of your VM, found in the **Overview** tab of the VM.

   > 💡 **Information Box:**<br>
   > SSH (Secure Shell) is a protocol used for secure remote login to your server. Ensure you entered the correct public IP address and credentials.

   > ✅ **Verification Step:**<br>
   You should see a command prompt for `azureuser@LabVM`.

---

### Step 6: Update the System
1. Run the following commands to update the system packages:
   ```bash
   sudo apt update
   sudo apt upgrade -y
   ```

   > 💡 **Information Box:**<br>
   > The `apt update` command refreshes the list of available packages, while `apt upgrade` installs the latest updates. The `-y` flag confirms the operation without additional prompts.

---

### Step 7: Install Nginx (Optional, for Verification)
1. Install Nginx:

   ```bash
   sudo apt install nginx -y
   ```

2. Verify the installation:

   ```bash
   systemctl status nginx
   ```

   > ✅ **Verification Step:**<br>
   > Ensure the output shows `active (running)` for the Nginx service.

3. Open a web browser and enter the public IP of your VM. You should see the **"Welcome to Nginx!"** default page.

---

### Step 8: Clean Up Resources (Optional)

1. In the **Azure Portal**, navigate to the **Resource Groups** section by searching for "Resource Groups" in the search bar.  
   
2. Locate the resource group you created, `LabResourceGroup`. You can use the search bar in the Resource Groups section if you have multiple groups.

3. Click on the `LabResourceGroup` name to open the resource group details.

4. In the resource group overview, click the **Delete resource group** button at the top of the page.

5. A confirmation prompt will appear. Enter the exact name of the resource group, `LabResourceGroup`, in the confirmation box, and click **Delete**.

   > 💡 **Information Box:**  
   > Deleting the resource group will permanently remove all resources contained within it, including the virtual machine, network interface, and any associated disks or public IPs. This action helps avoid unnecessary costs and keeps your environment clean.

   > ✅ **Verification Step:**  
   > After deletion, go back to the **Resource Groups** page in the Azure Portal and confirm that `LabResourceGroup` no longer appears in the list. Refresh the page if necessary.

---

## Exercise Complete!
You have successfully provisioned a Virtual Machine in Azure using the portal and connected to it using Azure Cloud Shell. You also verified its operation by installing and testing Nginx.
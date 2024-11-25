+++
title = "3. Automating Nginx Installation with Custom Data Scripts"
weight = 3
date = 2024-11-25
draft = false
+++

# Exercise 3: Automating Nginx Installation with Custom Data Scripts

---

## Overview
This exercise introduces the use of **custom data scripts** to automate the installation of Nginx during VM provisioning. You will also edit the default Nginx index page to display custom content.

---

## **Step-by-Step Instructions**

### Step 1: Log in to Azure Portal
1. Open a web browser and navigate to the [Azure Portal](https://portal.azure.com/).
2. Sign in using your Azure account credentials.

---

### Step 2: Create a Resource Group
1. In the Azure Portal, search for **"Resource groups"**.
2. Click **"Create"** and fill in the following:
   - **Subscription**: Choose your subscription.
   - **Resource Group Name**: `LabCustomDataGroup`
   - **Region**: Choose a region (e.g. `North Europe`).
3. Click **Review + Create** and then **Create**.

> 💡 **Information Box:**  
> Resource groups simplify resource organization and management, making it easy to track, update, and delete related resources.

---

### Step 3: Create a Virtual Machine with Custom Data
1. Navigate to **Virtual Machines** in the Azure Portal and click **Create** > **Virtual machine**.
2. Configure the VM:
   - **Subscription**: Select your subscription.
   - **Resource Group**: Select `LabCustomDataGroup`.
   - **Virtual Machine Name**: `LabCustomDataVM`.
   - **Region**: Same as the resource group.
   - **Image**: Select `Ubuntu Server 24.04 LTS`.
   - **Size**: Choose `Standard_B1s`.
   - **Authentication Type**: Select **SSH Public Key**.
   - **Username**: `azureuser`.
   - **SSH Public Key Source**: Use an existing SSH key or generate a new one.
3. Under **Advanced** > **Custom Data**, paste the content.
   ```bash
   #!/bin/bash
   apt update
   apt install nginx -y
   systemctl start nginx
   systemctl enable nginx
   ```

   > 💡 **Information Box:**  
   > The **Custom Data** field allows you to execute a script during the initial boot of the VM, automating configurations and software installations.
   > 
   > This script:
   > - Updates the system's package list (`apt update`).  
   > - Installs the Nginx web server (`apt install nginx -y`).  
   > - Starts the Nginx service immediately (`systemctl start nginx`).  
   > - Configures the Nginx service to start automatically on boot (`systemctl enable nginx`).

4. Configure inbound ports:
   - Check **Allow selected ports** and select **HTTP (80)** and **SSH (22)**.
5. Click **Review + Create**, and then **Create**.

---

### Step 4: Connect to the VM
1. Connect to the VM using SSH:
   ```bash
   ssh -i <path-to-private-key> azureuser@<VM_Public_IP>
   ```
   - Replace `<path-to-private-key>` with your private key file path.
   - Replace `<VM_Public_IP>` with the VM's public IP address.

---

### Step 5: View and Edit the Nginx Default Page
1. Navigate to the Nginx default directory:
   ```bash
   cd /var/www/html/
   ```
2. Open `index.html` in a text editor:
   ```bash
   sudo nano index.html
   ```
3. Modify the content (e.g., add "Hello World").
4. Save and exit by pressing `CTRL+O`, `ENTER`, and `CTRL+X`.
5. Refresh the page in your browser or rerun the `curl` command to see the updated content.
6. Verify in the browser

---

### Step 6: Clean Up Resources (Optional)
1. In the Azure Portal, navigate to **Resource Groups** and locate `LabCustomDataGroup`.
2. Click **Delete resource group**.
3. Confirm deletion by typing `LabCustomDataGroup`.

> 💡 **Information Box:**  
> Resource cleanup prevents unnecessary costs. Ensure all critical data is saved before deletion.

> ✅ **Verification Step:**  
> Confirm that `LabCustomDataGroup` no longer appears in the Azure Portal.

---

## Exercise Complete!
You have successfully automated the installation of Nginx using a custom data script and edited the default web page to display custom content.
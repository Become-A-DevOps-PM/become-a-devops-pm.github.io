+++
title = "2. Provisioning a VM with SSH Keys and Exploring Linux"
weight = 2
date = 2024-11-25
draft = false
+++

# Exercise 2: Provisioning a VM with SSH Keys and Exploring Linux

---

## Overview
This exercise introduces secure SSH authentication using **Azure Portal's Generate Key Pair feature**. You will also connect to the VM using SSH and explore the Linux filesystem with basic command-line utilities.

---

## **Step-by-Step Instructions**

### Step 1: Log in to Azure Portal
1. Open a web browser and navigate to the [Azure Portal](https://portal.azure.com/).
2. Sign in using your Azure account credentials.

---

### Step 2: Create a Resource Group
1. In the Azure Portal, search for **"Resource groups"**.
2. Click **"Create"** and provide the following:
   - **Subscription**: Choose your subscription.
   - **Resource Group Name**: `LabSSHResourceGroup`
   - **Region**: Choose a region (e.g. `North Europe`).
3. Click **Review + Create** and then **Create**.

> 💡 **Information Box:**  
> A **Resource Group** organizes Azure resources into manageable containers.

---

### Step 3: Create a Virtual Machine with SSH Key Pair
1. Navigate to **Virtual Machines** and click **Create** > **Virtual machine**.
2. Configure the VM:
   - **Subscription**: Select your subscription.
   - **Resource Group**: Select `LabSSHResourceGroup`.
   - **Virtual Machine Name**: `LabSSHVM`.
   - **Region**: Same as the resource group.
   - **Image**: Select `Ubuntu Server 24.04 LTS`.
   - **Size**: Choose `Standard_B1s`.
   - **Authentication Type**: Select **SSH Public Key**.
   - **Username**: `azureuser`.
3. Under **SSH Public Key Source**, select **Generate new key pair**.
4. Provide a name for your key pair (e.g., `LabSSHKey`) and click **Download private key and create resource**.

   > 💡 **Information Box:**  
   > The **Generate new key pair** option allows you to securely create an SSH key pair. The private key will be downloaded to your computer and should be stored securely. Azure automatically applies the corresponding public key to the VM.

---

### Step 4: Secure the SSH Private Key
1. Locate the downloaded private key file (e.g., `LabSSHKey.pem`) on your local machine.
2. Follow these steps based on your operating system:       ```
   - **Mac/Linux Users (Terminal)**:
     - Change file permissions
       ```bash
       chmod 400 ~/Downloads/LabSSHKey.pem
       ```

   > 💡 **Information Box:**  
   > The `chmod 400` command restricts permissions on the private key file, ensuring it is readable only by the file's owner. This is required for secure SSH connections. Failure to set these permissions will result in SSH errors.

---

### Step 5: Connect to the VM
1. In the Azure Portal, navigate to your VM's **Overview** tab and copy the public IP address of `LabSSHVM`.
2. Open your terminal:
   - **Windows**: Use **Git Bash**.
   - **Mac/Linux**: Use your default terminal.
3. Connect to the VM using SSH:
   ```bash
   ssh -i ~/Downloads/LabSSHKey.pem azureuser@<VM_Public_IP>
   ```
   - Replace `<VM_Public_IP>` with the public IP address you copied.

> ✅ **Verification Step:**  
> Ensure you are logged into the VM and see the `azureuser@LabSSHVM` prompt.

---

### Step 6: Explore the Linux Filesystem
1. List the contents of the root directory:
   ```bash
   ls /
   ```
   - Key directories:
     - `/home`: User home directories.
     - `/etc`: Configuration files.
     - `/var`: Logs and variable data.
    
2. Check your current working directory:
   ```bash
   pwd
   ```

> 💡 **Information Box:**  
> The Linux filesystem is hierarchical, starting from `/` (root). Familiarity with this structure is essential for managing Linux systems.

---

### Step 7: Create and Execute a Bash Script to Install Nginx

1. **Create the Bash Script**:
   - While logged into the VM, create a new script file:
     ```bash
     nano install_nginx.sh
     ```
   - Add the following content to the file:
     ```bash
     #!/bin/bash
     apt update
     apt install nginx -y
     systemctl start nginx
     systemctl enable nginx
     ```
   - Save and exit by pressing `CTRL+O`, `ENTER`, and `CTRL+X`.

2. **Make the Script Executable**:
   ```bash
   chmod +x install_nginx.sh
   ```

3. **Run the Script with Elevated Privileges**:
   ```bash
   sudo ./install_nginx.sh
   ```

4. **Verify the Installation**:
   - Check if Nginx is running:
     ```bash
     systemctl status nginx
     ```
     - Ensure the status shows `active (running)`.
   - Test Nginx using `curl`:
     ```bash
     curl localhost
     ```
     - The output should include the default Nginx HTML content, such as `Welcome to nginx!`.

> 💡 **Information Box:**  
> **Systemctl and the Init System**  
> The `systemctl` command is used to interact with the system's **init system** (often `systemd` on modern Linux distributions). It is responsible for managing services, such as starting, stopping, enabling, or checking the status of services like Nginx.  
> - `start`: Starts the service immediately.  
> - `enable`: Configures the service to start automatically at boot.  
> - `status`: Displays the current status of the service.  
> The `systemctl` tool makes service management straightforward and consistent across various distributions.

---

### Step 8: Clean Up Resources (Optional)
1. In the Azure Portal, go to **Resource Groups** and locate `LabSSHResourceGroup`.
2. Click **Delete resource group**.
3. Confirm deletion by typing `LabSSHResourceGroup`.

> 💡 **Information Box:**  
> Deleting the resource group removes all associated resources, ensuring you don’t incur unnecessary costs. Always confirm that all essential data has been backed up before deletion.

> ✅ **Verification Step:**  
> Ensure that `LabSSHResourceGroup` no longer appears in the **Resource Groups** section of the Azure Portal.

---

## Exercise Complete!
You have successfully provisioned a VM using Azure's Generate Key Pair feature, connected securely using SSH, and explored the Linux filesystem and basic commands.
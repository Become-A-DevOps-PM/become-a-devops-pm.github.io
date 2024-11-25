+++
title = "5. Provisioning a VM Using AZ CLI, Configuring Nginx, and Allowing HTTP Traffic"
weight = 5
date = 2024-11-25
draft = false
+++

# Exercise 5: Provisioning a VM Using AZ CLI, Configuring Nginx, and Allowing HTTP Traffic

---

## Overview
This exercise introduces creating a Virtual Machine (VM) with **Azure CLI**, using default SSH keys for secure authentication, installing Nginx, and configuring the network to allow HTTP traffic. You will work exclusively in the **VSCode Integrated Terminal** (or **Git Bash** for Windows users).

---

## **Step-by-Step Instructions**

### Step 1: Set Up the Environment
1. Open **VSCode** and launch the **Integrated Terminal**:
   - On Windows, use **Git Bash** inside VSCode.
   - On Mac or Linux, use the default terminal.

2. Verify that Azure CLI is installed by running:
   ```bash
   az --version
   ```
   - If Azure CLI is not installed, follow the [official installation guide](https://learn.microsoft.com/en-us/cli/azure/install-azure-cli).

3. Log in to your Azure account:
   ```bash
   az login
   ```
   - Follow the on-screen instructions to complete the authentication.

---

### Step 2: Create a Resource Group
1. Set variables for the resource group and VM name:
   ```bash
   resource_group="MyCLIVMGroup"
   vm_name="MyCLIVM"
   ```

2. Create the resource group in the `northeurope` location:
   ```bash
   az group create --name $resource_group --location northeurope
   ```

3. Verify its creation:
   ```bash
   az group list --output table
   ```
   - Ensure `MyCLIVMGroup` appears with the location `northeurope`.

---

### Step 3: Create a VM
1. Create a VM using the default SSH keys:
   ```bash
   az vm create --name $vm_name --resource-group $resource_group \
       --image Ubuntu2204 --size Standard_B1s \
       --generate-ssh-keys
   ```
   - This command:
     - Creates a VM named `MyCLIVM`.
     - Uses the latest Ubuntu 22.04 image.
     - Generates SSH keys if not already available in `~/.ssh/id_rsa`.

2. Verify the VM's creation:
   ```bash
   az vm show --resource-group $resource_group --name $vm_name --query "provisioningState" --output table
   ```
   - Ensure the output shows `Succeeded`.

> 💡 **Information Box:**  
> If you don’t have default SSH keys (`~/.ssh/id_rsa`), Azure CLI will generate them automatically. You can also create them manually using:
> ```bash
> ssh-keygen -t rsa -b 2048
> ```
> This generates:
> - `~/.ssh/id_rsa` (private key)
> - `~/.ssh/id_rsa.pub` (public key)

---

### Step 4: Log in to the VM
1. Retrieve the public IP address of the VM:
   ```bash
   vm_ip=$(az vm show --resource-group $resource_group --name $vm_name --show-details --query publicIps -o tsv)
   ```

2. Log in to the VM using your default SSH key:
   ```bash
   ssh azureuser@$vm_ip
   ```

   > 💡 **Information Box:**  
   > The default SSH key located in `~/.ssh/id_rsa` is used for authentication. Ensure your key permissions are set correctly:
   > ```bash
   > chmod 600 ~/.ssh/id_rsa
   > ```

---

### Step 5: Install Nginx
1. Update the system and install Nginx:
   ```bash
   sudo apt update
   sudo apt install nginx -y
   ```

2. Verify the installation:
   ```bash
   systemctl status nginx
   ```
   - Ensure it shows `active (running)`.

3. Test locally on the VM:
   ```bash
   curl localhost
   ```
   - The output should include the default Nginx HTML content, such as `Welcome to nginx!`.

---

### Step 6: Test Nginx in a Browser
1. Open a browser and navigate to the VM's public IP address:
   ```bash
   echo "http://$vm_ip"
   ```
2. **Observe the Error**:
   - The browser will not display the Nginx page because port 80 is not open in the VM's network security group (NSG).

> 💡 **Information Box:**  
> Azure VMs block HTTP traffic on port 80 by default for security reasons. You need to explicitly allow HTTP traffic.

---

### Step 7: Open Port 80 for HTTP Traffic
1. Allow traffic on port 80:
   ```bash
   az vm open-port --resource-group $resource_group --name $vm_name --port 80
   ```

2. Verify the rule is applied:
   ```bash
   az network nsg rule list --resource-group $resource_group --nsg-name ${vm_name}NSG --output table
   ```
   - Ensure a rule exists for port 80 allowing inbound traffic.

---

### Step 8: Verify in Browser Again
1. Refresh the browser with the VM's public IP address (`http://<VM_Public_IP>`).
2. You should now see the default **Nginx Welcome Page**.

> ✅ **Verification Step:**  
> Confirm the Nginx page loads in the browser, indicating port 80 is open and traffic is allowed.

---

### Step 9: Clean Up Resources (Optional)
1. Delete the resource group to avoid unnecessary costs:
   ```bash
   az group delete --name $resource_group --no-wait --yes
   ```
2. Verify:
   ```bash
   az group list --output table
   ```
   - Ensure `MyCLIVMGroup` no longer appears.

> 💡 **Information Box:**  
> Always clean up unused resources to avoid incurring unexpected charges.

---

## Exercise Complete!
You have successfully provisioned a VM using Azure CLI, logged in with SSH, installed and tested Nginx, and configured port 80 for HTTP traffic.
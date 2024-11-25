+++
title = "6. Automating VM Creation, Nginx Installation, and Port Configuration Using a Bash Script"
weight = 6
date = 2024-11-25
draft = false
+++

# Exercise 6: Automating VM Creation, Nginx Installation, and Port Configuration Using a Bash Script

---

## Overview
In this exercise, you will create a Bash script that automates the provisioning of a resource group, a VM, and the opening of port 80. You will also use a **custom data file** to automatically install and configure Nginx on the VM during provisioning. This exercise enables a "one-click" solution for deploying a web server.

---

## **Step-by-Step Instructions**

### Step 1: Prepare the Custom Data File
1. Create a file named `custom_data.sh` in your working directory:
   ```bash
   nano custom_data.sh
   ```
2. Add the following content to the file:
   ```bash
   #!/bin/bash
   apt update
   apt install nginx -y
   systemctl start nginx
   systemctl enable nginx
   echo "Hello, World from Azure VM!" > /var/www/html/index.html
   ```
3. Save file.

   > 💡 **Information Box:**  
   > The custom data script:
   > - Installs and starts Nginx.
   > - Configures it to run on startup.
   > - Adds a custom message to the default `index.html` page.

---

### Step 2: Create the Automation Script
1. Create a Bash script named `deploy_vm.sh`:
   ```bash
   nano provision_vm.sh
   ```
2. Add the following content to the script:
   ```bash
   #!/bin/bash

   # Variables
   resource_group="MyOneClickGroup"
   vm_name="MyOneClickVM"
   location="northeurope"
   custom_data_file="custom_data.sh"

   # Create a resource group
   echo "Creating resource group: $resource_group..."
   az group create --name $resource_group --location $location

   # Create a virtual machine with custom data
   echo "Creating virtual machine: $vm_name..."
   az vm create \
       --name $vm_name \
       --resource-group $resource_group \
       --location $location \
       --image Ubuntu2204 \
       --size Standard_B1s \
       --generate-ssh-keys \
       --custom-data @$custom_data_file

   # Open port 80 for HTTP traffic
   echo "Opening port 80 for HTTP traffic..."
   az vm open-port --resource-group $resource_group --name $vm_name --port 80

   # Retrieve public IP address of the VM
   vm_ip=$(az vm show --resource-group $resource_group --name $vm_name --show-details --query publicIps -o tsv)

   echo "Deployment complete! Access your server at http://$vm_ip"
   ```
3. Save.

4. Make the script executable:
   ```bash
   chmod +x deploy_vm.sh
   ```

---

### Step 3: Execute the Automation Script
1. Run the script:
   ```bash
   ./provision_vm.sh
   ```

2. The script will:
   - Create a resource group.
   - Provision a VM with the custom data script to install and configure Nginx.
   - Open port 80 for HTTP traffic.
   - Output the public IP address of the VM.

---

### Step 4: Verify Deployment
1. Copy the public IP address displayed at the end of the script execution.
2. Open a browser and navigate to:
   ```
   http://<VM_Public_IP>
   ```
3. You should see the message:
   ```
   Hello, World from Azure VM!
   ```

> ✅ **Verification Step:**  
> Ensure the custom web page is displayed, indicating successful automation of the VM provisioning and Nginx setup.

---

### Step 5: Clean Up Resources (Optional)
1. If you no longer need the resources, delete the resource group:
   ```bash
   az group delete --name MyOneClickGroup --no-wait --yes
   ```
2. Verify the deletion:
   ```bash
   az group list --output table
   ```
   - Ensure `MyOneClickGroup` no longer appears in the list.

> 💡 **Information Box:**  
> Cleaning up resources prevents unnecessary charges and ensures a tidy Azure environment.

---

## Exercise Complete!
You have successfully created a "one-click" solution to deploy an Azure VM, configure Nginx, and enable HTTP traffic using Bash scripting and custom data.
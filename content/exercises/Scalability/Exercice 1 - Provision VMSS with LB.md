+++
title = "1. Provision VMSS with LB"
weight = 1
date = 2024-12-29
draft = false
+++

## Overview

This exercise will walk you through creating an Azure VM Scale Set (VMSS) using the Azure Portal. You will:

- configure Nginx on each VM instance through a **cloud-init** script
- set up a Load Balancer (LB) with public IP
- configure Network Security Group (NSG) rules to allow HTTP and SSH. 

By the end of this exercise, you will have a scalable group of VMs behind a Load Balancer (this is a level 4 load balancer)

### Step 1: Log in to Azure Portal
1. Open a web browser and navigate to the [Azure Portal](https://portal.azure.com/).
2. Sign in using your Azure account credentials.

### Step 2: Create a Virtual Machine Scale Set
1. In the Azure Portal, search for **"Virtual machine scale sets"** and click **"Create"**.
2. Configure the **Basics** tab:
   - **Subscription**: Select your subscription.
   - **Resource group**: Create new `LabResourceGroup`.
   - **Name**: `LabVMSS`
   - **Region**: `North Europe`
   - **Orchestration mode**: `Uniform` (for this exercise).
   - **Instance count**: `2` (or any small number suitable for testing).
   - **Image**: Choose **Ubuntu Server 24.04 LTS**.
   - **Authentication Type**: Select **SSH public key**.
     - **Username**: `azureuser`
     - If using SSH public key: alt 1) paste your public key; alt 2) download a new key
3. In the **Networking** tab:
   - **Virtual network**: Let the wizard create a new one for you (LabResourceGroup-vnet)
   - **Edit the NIC**:
   		- **Subnet**: Use the default subnet created.
   		- **Public inbound ports**: Allow selected ports (HTTP 80, SSH 22)
   		- **Public IP address**: **Disable** (we want to use a Load Balancer with a single public IP).
   		- **OK**
   - **Load balancing options**: Select **Azure load balancer** and choose **Create a new load balancer** if prompted.
   		- **Load balancer name**: `DemoLB`
   		- **Create**
4. In the **Management** tab:
    - **Upgrade mode**: `Automatic`
5. In the **Health** tab:
    - **Enable Health Probe**
6. In the **Advanced** tab:
	- **Custom data and cloud init** and paste the following script in the text box:
	
	```yaml
	#cloud-config
	package_update: true
	packages:
	  - nginx
	runcmd:
	  - systemctl start nginx
	  - systemctl enable nginx
	  - RANDOM_NUMBER=$(shuf -i 1000-9999 -n 1)
	  - sed -i "s/nginx/$RANDOM_NUMBER/" /var/www/html/index.nginx-debian.html
	
	```

7. Click **Review + create**, then **Create** to provision the VM Scale Set.

> 💡 **Information**
> 
> **cloud-init Script**: This script updates the system, installs Nginx, and replaces the text “nginx” on the default homepage with a random number to help distinguish each instance’s page.

> ✅ **Verification Step:**
> 
> After Azure finishes deployment (it may take a few minutes), navigate to **Virtual machine scale sets** in the Portal. You should see `LabVMSS` with a status of **"Running"**.
> 
> Browse to the Public IP address and verify that you can see the default nginx page with a unique number. Refresh the page and see if the number changes.

> 💡 **Configure NSG Rules for HTTP and SSH**
> 
> Even if you enabled **HTTP**/ **SSH** during creation, verify or add the inbound security rules in the VMSS’s network interface (NSG) to ensure inbound traffic can reach the VMs and that you can SSH in via NAT:
> 
> 1. Go to **Virtual machine scale sets** > **LabVMSS**.
> 2. Select **Networking** > **Network settings** (on the left menu).
> 3. Locate or add **Inbound port rules** for:
>    - **80 (HTTP)**: Source: `Any`, Destination: `Any`, Protocol: **TCP**, Action: **Allow**.
>    - **22 (SSH)**: Source: `Any`, Destination: `Any`, Protocol: **TCP**, Action: **Allow**.

> 💡 **Information**  
> 
> NSGs (Network Security Groups) govern inbound and outbound traffic. For web and SSH access, make sure inbound port 80 and 22 are open, respectively.  

> ✅ **Verification Step: Verify Load Balancer Configuration**
> 
> 1. In the **LabVMSS** pane, select **Instances** to see your VM instances. Notice each instance is now powered on.
> 2. From **LabVMSS** > **Settings** > **Load balancing**, click the link to open the Load Balancer’s details.
> 3. In the Load Balancer’s overview, note the **Frontend public IP address**. This is the single public IP shared by all instances behind the load balancer.
> 	- **Frontend test**: Copy the Load Balancer’s public IP address and paste it into your browser.  
>  	- Refresh the page multiple times; you should see the random numbers from each VM instance in place of the word “nginx.”
> 4. Click **Settings** > **Inbound NAT Rules** to see the automatically created rules that forward unique ports to each VM instance for SSH.
> 	- **NAT test**: Look for an **Inbound NAT Rule** that maps a random port (e.g. 50000) to port 22 on a specific VM instance. In your terminal:
> 
>   	```bash
>   	ssh azureuser@<LoadBalancer_Public_IP> -p 50000
>   	```
> 
>   	Replace `<LoadBalancer_Public_IP>` and `50000` with the NAT rule’s actual values.
> 5. Scaling
> 	- If you increase or decrease the VM count in the Scale Set (under **Scaling**), any new VM will use the same cloud-init, displaying a different random number when you refresh the site in the browser.


### Conclusion
- **Load Balancer**: Distributes traffic to multiple VM instances, each running Nginx.  
- **No Public IP on Each VM**: All instances share the LB’s public IP.  
- **NSG Rules**: Must allow **HTTP (80)** and **SSH (22)** inbound from the Internet.  
- **NAT**: Port forwarding is used for SSH to each individual VM instance.

This approach minimizes public endpoints and centralizes traffic management via the Load Balancer. Scaling the instance count up or down will automatically apply the same cloud-init, ensuring every VM in the Scale Set is similarly configured.


### (Optional) Clean Up Resources
1. In the **Azure Portal**, search for **"Resource Groups"** and open **LabResourceGroup**.
2. Click **Delete resource group**, type the resource group name to confirm, and then select **Delete**.

# Happy Cloud!
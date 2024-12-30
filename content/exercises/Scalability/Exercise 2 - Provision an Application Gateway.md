+++
title = "2. Provision an Application Gateway."
weight = 2
date = 2024-12-30
draft = false
+++


## Overview
In this exercise, you will extend the environment you created in the previous **Provision VMSS with LB** by adding an **Azure Application Gateway** (AGW)—a Layer 7 load balancer. The AGW will distribute HTTP traffic to the VM instances created earlier. You will create a new subnet specifically for the AGW and configure it to route requests to the VMSS backend.

> 💡 **Prerequisite**  
> Make sure you have completed the **Provision VMSS with LB** exercise, including its **Resource Group**, **VNet**, and **VM Scale Set**.

### Step 1: Log in to Azure Portal
1. Open a web browser and navigate to the [Azure Portal](https://portal.azure.com/).
2. Sign in with your Azure account credentials.

### Step 2: Add a New Subnet for the Application Gateway
1. In the Azure Portal, search for **"Virtual networks"**.
2. Select the existing VNet you created for your VM Scale Set (e.g., `LabVMSSVnet`).
3. Under **Settings**, click **Subnets**.
4. Click **+ Subnet**, then create a new subnet:
   - **Name**: `AGWSubnet`  
   - **Address range**: A suitable CIDR block (for example `10.0.1.0/24` if your VNet range allows).
5. Click **Save**.

> 💡 **Information** 
> 
> The Application Gateway must exist in its own dedicated subnet separate from your VMSS instances. This step ensures it has an isolated subnet to operate in.

### Step 3: Create an Azure Application Gateway
1. In the Azure Portal, search for **"Application gateways"** and click **"Create"**.
2. Under **Basics**:
   - **Subscription**: Select the same subscription used for your VMSS.
   - **Resource group**: Pick the same resource group, e.g. `LabResourceGroup`.
   - **Name**: `LabAppGw`.
   - **Region**: Must match the region of your existing resources (e.g. `North Europe`).
   - **Tier**: `Basic` (suitable for development).
   - **Virtual network**: Choose the one created earlier
   - **Subnet**: `AGWSubnet`
3. Under **Frontend**:
   - Add a new **Public IP address**:
     - **Name**: `LabAppGwPublicIP`.
   	  - **OK**
4. Under **Backends**:
   - **Add a backend pool**:
     - **Name**: `LabAppGwBackendPool`.
     - For **Target type**, select **VMSS** and choose your existing VM scale set (`LabVMSS`).  
     - **Add**
5. Under **Configuration**:
   - **Add a routing rule**:
     1. **Rule name**: `HTTPRule`.
     2. **Priority**: `100`
	  3. **Listener**:
	        - **Listener name**: `HTTPlistener`.
	        - **Frontend IP**: `LabAppGwPublicIP` (your newly created/selected IP).
	        - **Protocol**: **HTTP**.
	        - **Port**: `80`.
     3. **Backend target**:
	        - **Backend pool**: `LabAppGwBackendPool`.
	        - **Backend settings**: `Add new`
	        	- **Backend settings name**: `HTTPsetting`
	        	- **Add**
   - Click **Add** to finalize your rule.
6. Click **Review + create**. After validation, click **Create** to provision the Application Gateway.

> ✅ **Verification Step**:
> 
> - Once deployment completes, go to **Application gateways** > **LabAppGw** and confirm the status is **Running**.  
> - Make sure the AGW is deployed into the **`AGWSubnet`** you created in the previous step.
> 
> ✅ **Verify Application Gateway Routing**
> 
> 1. In **LabAppGw**, note the **Frontend public IP address** (under the Overview tab).
> 2. Open a web browser and navigate to that public IP.
> 3. You should see the Nginx welcome page, with the randomized numbers that indicate which VM instance you’ve landed on.  
> 4. **Refresh** the page multiple times; traffic should distribute across the VM instances in your VM Scale Set.
> 
> ✅ **Verification Step**  
> If you scale your VMSS up or down, the AGW will automatically include or remove instances from the backend pool, continuing to distribute traffic at the application layer.
> 
> 💡 **Information**  
> The Azure Application Gateway provides Layer 7 (HTTP/HTTPS) load-balancing features, allowing you to do **URL-based routing**, **path-based routing**, and more advanced HTTP features compared to the Layer 4 Azure Load Balancer.

### Conclusion
By completing this exercise, you have:

- **Added a dedicated AGW subnet**: Ensuring network isolation for the Application Gateway.  
- **Created an Application Gateway**: Acting as a Layer 7 load balancer for your VM Scale Set.  
- **Validated Routing**: Confirmed HTTP requests are distributed among your Nginx instances.


### (Optional) Clean Up Resources
If you want to remove the entire lab environment (both the VMSS and the Application Gateway):

1. In the **Azure Portal**, search for **"Resource Groups"**.
2. Locate **LabResourceGroup** (or the name you chose).
3. Click **Delete resource group**, type the name to confirm, then select **Delete**.

# Happy Cloud!

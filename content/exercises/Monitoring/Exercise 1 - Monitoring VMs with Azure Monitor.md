+++
title = "1. Monitoring Azure VMs with Azure Monitor"
weight = 1
date = 2025-01-03
draft = false
+++

## Overview

In this exercise, you will configure monitoring for an Azure Virtual Machine (VM) using Azure Monitor. The exercise covers:

- Monitoring **CPU**, **Memory**, and **Disk** metrics through Azure Monitor **Insights**.
- Collecting the **VM Syslog** via Azure Monitor.
- Redirecting **Nginx access logs** to Syslog and analyzing them with Azure Monitor using **KQL (Kusto Query Language)**.

By the end of this exercise, you will have hands-on experience with monitoring and log analytics for an Azure VM.

### Step 1: Log in to Azure Portal

1. Open a web browser and navigate to the [Azure Portal]\([https://portal.azure.com/](https://portal.azure.com/)).
2. Sign in using your Azure account credentials.

### Step 2: Create a Log Analytics Workspace (LAWS)

1. In the Azure Portal, search for **Log Analytics workspaces**.
2. Click **+ Create**.
3. Fill out the following details:
   - **Subscription**: Select your subscription.
   - **Resource Group**: Create new `MonitoringRG`.
   - **Name**: `MonitoringLAWS`.
   - **Region**: `North Europe`
4. Click **Review + Create** and then **Create**.

### Step 3: Provision a Virtual Machine

1. Search for **Virtual Machines** in the Azure Portal and click **+ Create**.
2. Configure the **Basics** tab:
   - **Subscription**: Select your subscription.
   - **Resource group**: `MonitoringRG`.
   - **Name**: `MonitoredVM`
   - **Region**: `North Europe`
   - **Image**: Choose **Ubuntu Server 24.04 LTS**.
   - **Size**: Standard B1s.
   - **Authentication Type**: Select **SSH public key**.
     - **Username**: `azureuser`
     - If using SSH public key: alt 1) paste your public key; alt 2) download a new key
   - **Select inbound ports**: 80 and 22
3. Click **Review + create**, then **Create** to provision the VM.

### Step 4: Enable Insights on the VM

1. Go to the VM in the Azure Portal.
2. Go to the left menu: Monitoring -> Insights
   - Click **Enable**
   - Enable guest performance: check (will Configure Performance Counters)
   - **Data collection rule**: Create New
      - **Data collection rule name**: `MonitoredVM` (same as the VM)
      - **Log Analytics workspaces**: `MonitoringLAWS`
      - **Create**
3. **Configure** (Will create a DCR and install AMA, including managed identity, on the VM)

> ✅ **Verification Step: Insights Data**
> 
> 1. Navigate to **VM** -> **Insights** -> Performance. (You might have to refresh the browser - not just the refresh button)
>    - Ensure metrics for **CPU**, **Memory**, and **Disk** are visible. (This can take a few minutes to show up)
> 2. Go to **VM** -> **Logs** and run the following queries:
>    - **Heartbeat**
>    - **InsightMetrics**:
> 3. Go to **Data collection rules**
>    - Select: `MSVMI-MonitoredVM`
>    - Go to Configuration -> Data sources. Verify _Performance Counters_
>    - Go to Configuration -> Resources. Verify _MonitoredVM_

### Step 5: Configure Syslog Collection

1. In the Azure Portal, create another Data Collection Rule:
   - Basic Tab:
      - **Name**: `MonitoredVMSyslogDCR`.
      - Resource group: `MonitoringRG`
      - Region: `North Europe`
      - Platform Type: `Linux`
      - Next: Resources
   - Resources Tab:
      - **Add resource**: `MonitoredVM`
      - **Apply**
      - Next: Collect and deliver
   - Collect and deliver Tab:
      - **Add data source**: Linux Syslog (check all facilities and set LOG\_INFO)
      - Next: Destination
      - **Destination**: Select `MonitoringLAWS`
      - Add data source
2. Click **Review + create**, then **Create**

> ✅ **Verification Step: Syslog Data**
> 
> 
> 1. Go to **Monitoring** > **Logs** on the VM.
> 2. Run a query to check Syslog data (It can take a few minutes before values appear)

### Step 6: Install Nginx And Redirect Access Logs to Syslog

1. Connect using SSH:

	```bash
	ssh <username>@<VM-public-IP>
	```

2. Update the package list and install Nginx:

	```bash
	sudo apt update && sudo apt install -y nginx
	```

> ✅ **Verification Step: Nginx**
> 
>    ```bash
>    curl localhost
>    ``` 
> 
> 2. Tail the Nginx access log. Browse to your Nginx site and verify the logs in the access.log:
> 
>    ```bash
>    tail -f /var/log/nginx/access.log
>    ```
> Click `Ctrl-C` to exit


3. Update Nginx Configuration

- Open the default Nginx configuration file:

   ```bash
   sudo nano /etc/nginx/sites-enabled/default
   ```

- Add the **access_log** directive into the server block:

   ```nginx
   access_log syslog:server=unix:/dev/log combined;
   ```

- Test and restart Nginx:

   ```bash
   sudo nginx -t
   sudo systemctl restart nginx
   ```

> ✅ **Verification Step: Nginx Logs in Syslog**
> 
> 
> 1. Tail the system log:
> 
>    ```bash
>    tail -f /var/log/syslog
>    ```
> 
> 2. Browse to your Nginx site and verify the logs in the syslog
> 
> 3. Run a query in **VM** > **Logs** to analyze Nginx logs: (You might need to wait a few minutes for the logs to appear)
> 
>    ```kql
>    Syslog | where Facility == "local7" | count 
>    ```

# Happy Cloud!

Feel free to explore more KQL queries and extend the monitoring setup!


Analyzing Nginx access logs can provide valuable insights into web traffic, user behavior, system performance, and security. Here are some interesting statistics and analyses you can perform using Nginx access logs:


### Traffic Analysis

- **Requests Over Time**:
  - Analyze traffic patterns by hour, day, or minute.

    ```kql
    Syslog | where Facility == "local7"
    | summarize RequestCount = count() by bin(TimeGenerated, 1m)
    ```

- **Top Requested URLs**:
  - Identify the most accessed pages or endpoints.

    ```kql
    Syslog | where Facility == "local7"
    | extend URL = extract("\"[A-Z]+ (/.*?) HTTP", 1, SyslogMessage)
    | summarize Count = count() by URL
    | sort by Count desc
    | take 10
    ```

### User Behavior

- **Top User Agents**:
  - Identify the browsers and devices your users are using.

   ```kql
	Syslog | where Facility == "local7"
	| extend UserAgent = extract(".*\"[^\"]+\"[ ]+\"([^\"]+)\"", 1, SyslogMessage)
	| summarize Count = count() by UserAgent
	| sort by Count desc
	| take 10
   ```

### Security Insights
- **Top 404 (Not Found) Errors**:
  - Identify missing pages or incorrect links users are requesting.

   ```kql
	Syslog | where Facility == "local7"
	| extend StatusCode = extract("\" ([0-9]{3}) ", 1, SyslogMessage)
	| where StatusCode == "404"
	| extend URL = extract("\"[A-Z]+ (/.*?) HTTP", 1, SyslogMessage)
	| summarize Count = count() by URL
	| sort by Count desc
	| take 10
   ```

- **Frequent Client Errors (4xx)**:
  - Monitor user errors such as bad requests or unauthorized access attempts.

    ```kql
    Syslog | where Facility == "local7"
    | extend StatusCode = extract("\" ([0-9]{3}) ", 1, SyslogMessage)
    | where StatusCode startswith "4"
    | summarize Count = count() by StatusCode
    ```

- **Top IPs Causing Errors**:
  - Detect potential malicious actors or misconfigured clients.

   ```kql
	Syslog | where Facility == "local7"
	| extend StatusCode = extract("\" ([0-9]{3}) ", 1, SyslogMessage)
	| where StatusCode startswith "4" or StatusCode startswith "5"
	| extend IP = extract("nginx: ([0-9.]+)", 1, SyslogMessage)
	| summarize Count = count() by IP
	| sort by Count desc
   ```

---

### Content and Resource Analysis
- **Top File Types**:
  - Understand what types of files (e.g., `.html`, `.js`, `.png`) are being requested most often.

    ```kql
    Syslog | where Facility == "local7"
    | extend FileType = extract("\\.(\\w+)$", 1, SyslogMessage)
    | summarize Count = count() by FileType
    | sort by Count desc
    ```

### Error Monitoring
- **Error Rate**:
  - Monitor the proportion of error responses (4xx and 5xx) over total responses.

    ```kql
    Syslog | where Facility == "local7"
    | extend StatusCode = extract("\" ([0-9]{3}) ", 1, SyslogMessage)
    | summarize TotalRequests = count(), Errors = countif(StatusCode startswith "4" or StatusCode startswith "5")
    | extend ErrorRate = Errors * 100 / TotalRequests
    ```



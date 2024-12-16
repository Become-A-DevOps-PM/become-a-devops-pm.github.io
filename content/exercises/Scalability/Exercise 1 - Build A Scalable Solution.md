+++
title = "1: Build a Scalable Solution in Azure"
weight = 1
date = 2024-12-16
draft = false
+++

## Overview

In this exercise, you will create a secure, multi-tier architecture in Azure using the Azure Portal. You will set up:

1. A **Virtual Network (VNet)** with four subnets for different tiers: AppGateway, Application, Database and BastionHost.
2. An **Azure Bastion Host** for secure remote access to resources within the VNet.
4. An **Azure Database for MySQL Flexible Server** in the Database subnet.
5. A **Virtual Machine Scale Set (VMSS)** in the Application subnet.
6. An **Application Gateway** in the Gateway subnet for managing HTTP traffic.
7. An **App Server** (used as Provisioning Server to configure the database)

![AzureScalableSolution](/AzureScalableSolution.png)

## 1. Create a Resource Group
1. Navigate to the Azure Portal.
2. Go to **Resource Groups** > **+ Create**.
3. Enter the following details:
   - **Resource Group Name**: `DemoRG`
   - **Region**: `North Europe`
4. Click **Review + Create** > **Create**.

## 2. Create a Virtual Network And Subnets
1. Navigate to **Virtual Networks** > **+ Create**.
2. Enter the following details:
   - **Name**: `DemoVnet`
   - **Region**: `North Europe`
   - **Address Space**: `10.0.0.0/16`
3. Click **Next: IP Addresses** and add the following subnets:
   - **Gateway Subnet**: (edit default)
     - Name: `DemoGatewaySubnet`
     - Address Range: `10.0.0.0/24`
   - **App Subnet**:
     - Name: `DemoAppSubnet`
     - Address Range: `10.0.1.0/24`
   - **Database Subnet**:
     - Name: `DemoDatabaseSubnet`
     - Address Range: `10.0.2.0/24`
   - **Bastion Subnet**:
     - Name: `AzureBastionSubnet`
     - Address Range: `10.0.3.0/27`
4. Click **Review + Create** > **Create**.

## 3. Create a Network Security Group For The App Subnet
1. Navigate to **Network Security Groups** > **+ Create**.
2. Enter the following details:
   - **Name**: `DemoAppSubnetNSG`
   - **Region**: `North Europe`
   - **Resource Group**: `DemoRG`
3. Click **Review + Create** > **Create**.

4. Add the following inbound rules:
   - **Allow SSH**:
     - Protocol: `TCP`
     - Port Range: `22`
     - Priority: `1000`
   - **Allow HTTP**:
     - Protocol: `TCP`
     - Port Range: `80`
     - Priority: `2000`

5. Associate the NSG with the **App Subnet**:
   - Navigate to **Virtual Networks** > **DemoVnet** > **Subnets**.
   - **+ Associate** > `DemoVnet` > `DemoAppSubnet` > **OK**

## 4. Create Public IPs
1. Navigate to **Public IP Addresses** > **+ Create**.
2. Create two public IPs:
   - **For Application Gateway**:
     - Name: `AppGateway-PublicIP`
     - SKU: `Standard`
   - **For Bastion Host**:
     - Name: `Bastion-PublicIP`
     - SKU: `Standard`

## 5. Create an Azure Database for MySQL Flexible Server
1. Navigate to **Azure Database for MySQL Flexible Server** > **+ Create**.
2. Advanced Create:
   - **Server Name**: `demodbcampus24`
   - **Resource Group**: `DemoRG`
   - **Region**: `North Europe`
   - **Administrator Username**: `adminuser`
   - **Password**: `SecurePassword123!`
   
   	Network Tab
   
   - **Connectivity method**: Private access (VNet Integration)
   - **VNet**: `DemoVnet`
   - **Subnet**: `DemoDatabaseSubnet`
3. Click **Review + Create** > **Create**.

## 6. Create a Virtual Machine Scale Set
1. Navigate to **Virtual Machine Scale Sets** > **+ Create**.
2. Enter the following details:
   - **Name**: `DemoAppVMSS`
   - **Resource Group**: `DemoRG`
   - **Region**: `North Europe`
   - **Orchestration mode**: `Uniform`
   - **Image**: `Ubuntu 22.04 LTS`
   - **Instance Count**: `2`

   Network:
   
   - **VNet**: `DemoVnet`

	   Edit NIC:
	   
	   - **Subnet**: `DemoAppSubnet`
	   - **Public IP**: None (optional for production).

   Management:
   
   - **Upgrade policy**: automatic

   Advanced:
   
   - **Custom data**
   
	```yaml
	#cloud-config
	package_update: true
	packages:
	  - nginx
	  - php-fpm
	  - php-mysql
	write_files:
	  - path: /var/www/html/info.php
	    content: |
	      <?php phpinfo(); ?>
	  - path: /var/www/html/index.html
	    content: |
	      <!DOCTYPE html>
	      <html lang="en">
	      <head>
	          <meta charset="UTF-8">
	          <meta name="viewport" content="width=device-width, initial-scale=1.0">
	          <title>Contact Form</title>
	      </head>
	      <body>
	          <h1>Contact Us (VMSS)</h1>
	          <form action="on_post_contact.php" method="post">
	              <label for="name">Name:</label><br>
	              <input type="text" id="name" name="name" required><br><br>
	
	              <label for="email">Email:</label><br>
	              <input type="email" id="email" name="email" required><br><br>
	
	              <label for="message">Message:</label><br>
	              <textarea id="message" name="message" rows="5" required></textarea><br><br>
	
	              <button type="submit">Submit</button>
	          </form>
	      </body>
	      </html>
	  - path: /var/www/html/on_post_contact.php
	    content: |
	      <?php
	      // Database credentials
	      $servername = "demodbcampus24.mysql.database.azure.com";
	      $username = "php_user";
	      $password = "secure_password";
	      $dbname = "contact_db";
	
	      // Establish database connection
	
	      // SSL configuration
	      $ssl_options = array(
	          MYSQLI_OPT_SSL_VERIFY_SERVER_CERT => true
	      );
	
	      $conn = mysqli_init();
	      $conn->options(MYSQLI_OPT_SSL_VERIFY_SERVER_CERT, true); // Verify the server certificate
	      $conn->ssl_set(null, null, null, null, null); // Default SSL setup
	      if (!$conn->real_connect($servername, $username, $password, $dbname, 3306, null, MYSQLI_CLIENT_SSL)) {
	          die("Connection failed: " . mysqli_connect_error() . "\n");
	      }
	
	      // Collect and sanitize form data
	      $name = htmlspecialchars($_POST['name']);
	      $email = htmlspecialchars($_POST['email']);
	      $message = htmlspecialchars($_POST['message']);
	
	      // Prepare and bind SQL statement
	      $stmt = $conn->prepare("INSERT INTO contacts (name, email, message) VALUES (?, ?, ?)");
	      $stmt->bind_param("sss", $name, $email, $message);
	
	      // Execute the query
	      if ($stmt->execute()) {
	          echo "Thank you! Your message has been sent.";
	      } else {
	          echo "Error: " . $stmt->error;
	      }
	
	      // Close connections
	      $stmt->close();
	      $conn->close();
	      ?>
	  - path: /etc/nginx/sites-available/default
	    content: |
	      server {
	          listen 80;
	          server_name _;
	
	          root /var/www/html;
	          index index.php index.html index.htm index.nginx-debian.html;
	
	          location / {
	              try_files $uri $uri/ =404;
	          }
	
	          location ~ \.php$ {
	              include snippets/fastcgi-php.conf;
	              fastcgi_pass unix:/var/run/php/php8.1-fpm.sock; # Adjust PHP version if necessary
	              fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
	              include fastcgi_params;
	          }
	      }
	
	runcmd:
	  - systemctl restart nginx
	  - sed -i "s/VMSS/$(shuf -i 1000-9999 -n 1)/" /var/www/html/index.html
	```
	
3. Click **Review + Create** > **Create**.

## 7. Create an Application Gateway
1. Navigate to **Load Balancers** > **Application Gateway** > **+ Create**.
2. Enter the following details:
   - **Name**: `DemoAppGateway`
   - **Resource Group**: `DemoRG`
   - **Region**: `North Europe`
   - **SKU**: `Standard_v2`
   - **Virtual network**: `DemoVnet`
   - **Subnet**: `DemoGatewaySubnet`
3. Configure Frontend IP:
   - **Public IP**: `AppGateway-PublicIP`
4. Configure Backend Pool:
	- **Add Backend Pool**: `backendpool`
	- **Target type**: `VMSS`
	- **Target**: `DemoAppVMSS`
	- **Add** 
4. Configuration:
	- **Add a routing rule**
	- **Rule name**: `rule`
	- **Priority**: `100`
	- **Listener name**: `listener`
	- **Target type**: `backendpool`
	- **Backend settings**: *Add new* -> `backendsettings` -> *Add*
	- **Add**
5. Click **Review + Create** > **Create**.

## 8. Create an App Server
1. Navigate to **Virtual Machines** > **Create**:
   - **Resource Group**: `DemoRG`.
   - **Virtual Machine Name**: `DemoAppServer`.
   - **Region**: `North Europe`.
   - **Image**: `Ubuntu 22.04 LTS`.
   - **Size**: `Standard_B1s` (suitable for small workloads).
   - **Administrator Account**:
     - **Authentication Type**: SSH public key.
     - **Username**: `azureuser`.
     - **SSH Public Key**: Generate new key pair.

	Configure Networking:
	
1. **Network Interface**:
   - **Virtual Network**: Select `DemoVnet`.
   - **Subnet**: Select `DemoAppSubnet`.
   - **Public IP**: No Public IP (use Bastion for access).
   - **NIC Network Security Group**: None (rely on the subnet-level NSG `DemoAppSubnetNSG`).

	Configure Advanced Settings:
	
1. **Custom Data**:
   
	```yaml
	#cloud-config
	package_update: true
	packages:
	  - mysql-client
	  - nginx
	  - php
	  - php-fpm
	  - php-mysql
	write_files:
	  - path: /var/www/html/info.php
	    content: |
	      <?php phpinfo(); ?>
	  - path: /var/www/html/index.html
	    content: |
	      <!DOCTYPE html>
	      <html lang="en">
	      <head>
	          <meta charset="UTF-8">
	          <meta name="viewport" content="width=device-width, initial-scale=1.0">
	          <title>Contact Form</title>
	      </head>
	      <body>
	          <h1>Contact Us</h1>
	          <form action="on_post_contact.php" method="post">
	              <label for="name">Name:</label><br>
	              <input type="text" id="name" name="name" required><br><br>
	
	              <label for="email">Email:</label><br>
	              <input type="email" id="email" name="email" required><br><br>
	
	              <label for="message">Message:</label><br>
	              <textarea id="message" name="message" rows="5" required></textarea><br><br>
	
	              <button type="submit">Submit</button>
	          </form>
	      </body>
	      </html>
	  - path: /var/www/html/on_post_contact.php
	    content: |
	      <?php
	      // Database credentials
	      $servername = "demodbcampus24.mysql.database.azure.com";
	      $username = "php_user";
	      $password = "secure_password";
	      $dbname = "contact_db";
	
	      // Establish database connection
	
	      // SSL configuration
	      $ssl_options = array(
	          MYSQLI_OPT_SSL_VERIFY_SERVER_CERT => true
	      );
	
	      $conn = mysqli_init();
	      $conn->options(MYSQLI_OPT_SSL_VERIFY_SERVER_CERT, true); // Verify the server certificate
	      $conn->ssl_set(null, null, null, null, null); // Default SSL setup
	      if (!$conn->real_connect($servername, $username, $password, $dbname, 3306, null, MYSQLI_CLIENT_SSL)) {
	          die("Connection failed: " . mysqli_connect_error() . "\n");
	      }
	
	      // Collect and sanitize form data
	      $name = htmlspecialchars($_POST['name']);
	      $email = htmlspecialchars($_POST['email']);
	      $message = htmlspecialchars($_POST['message']);
	
	      // Prepare and bind SQL statement
	      $stmt = $conn->prepare("INSERT INTO contacts (name, email, message) VALUES (?, ?, ?)");
	      $stmt->bind_param("sss", $name, $email, $message);
	
	      // Execute the query
	      if ($stmt->execute()) {
	          echo "Thank you! Your message has been sent.";
	      } else {
	          echo "Error: " . $stmt->error;
	      }
	
	      // Close connections
	      $stmt->close();
	      $conn->close();
	      ?>
	  - path: /etc/nginx/sites-available/default
	    content: |
	      server {
	          listen 80;
	          server_name _;
	
	          root /var/www/html;
	          index index.php index.html index.htm index.nginx-debian.html;
	
	          location / {
	              try_files $uri $uri/ =404;
	          }
	
	          location ~ \.php$ {
	              include snippets/fastcgi-php.conf;
	              fastcgi_pass unix:/var/run/php/php8.1-fpm.sock; # Adjust PHP version if necessary
	              fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
	              include fastcgi_params;
	          }
	      }
	
	runcmd:
	  - systemctl restart nginx
	  - |
	    mysql -h demodbcampus24.mysql.database.azure.com -u adminuser -pSecurePassword123! <<EOF
	    CREATE DATABASE contact_db;
	    USE contact_db;
	
	    CREATE TABLE contacts (
	      id INT AUTO_INCREMENT PRIMARY KEY,
	      name VARCHAR(100) NOT NULL,
	      email VARCHAR(100) NOT NULL,
	      message TEXT NOT NULL,
	      submitted_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
	    );
	
	    CREATE USER 'php_user'@'%' IDENTIFIED BY 'secure_password';
	    GRANT ALL PRIVILEGES ON contact_db.* TO 'php_user'@'%';
	    FLUSH PRIVILEGES;
	    EOF
	```
	
2. Click **Review + Create** > **Create**.

## 9. Create an Azure Bastion Host
1. Navigate to **Bastions** > **+ Create**.
2. Enter the following details:
   - **Name**: `DemoBastion`
   - **Resource Group**: `DemoRG`
   - **Region**: `North Europe`
   - **VNet**: `DemoVnet`
   - **Subnet**: `AzureBastionSubnet`
   - **Public IP**: `Bastion-PublicIP`

   Advanced
   
   - Check: Native client support
3. Click **Review + Create** > **Create**.


## Verify
1. Go to the public IP of the Application Gateway
	- Fill in the form and submit
2. SSH in to the App Server and verify that the record is saved in the database

	- Alt A: Login directly to the App Server

		```bash
		chmod 400 <key>
		ssh -i <key> azureuser@<ip>
		```
	
	- Alt B: Login via Bastion Host (change the subscription ID)

		```bash
		az account list --output table
		
		RESOURCE_GROUP="DemoRG"
		APP_SERVER_NAME="DemoAppServer"
		
		az network bastion ssh \
		    --name DemoBastion \
		    --resource-group $RESOURCE_GROUP \
		    --target-resource-id /subscriptions/<CHANGE SUBSCRIPTION ID>/resourceGroups/$RESOURCE_GROUP/providers/Microsoft.Compute/virtualMachines/$APP_SERVER_NAME \
		    --auth-type ssh-key \
		    --username azureuser \
		    --ssh-key ~/Downloads/DemoAppServer_key.pem
		```

	- Run the commands:

		```bash
		mysql -h demodbcampus24.mysql.database.azure.com -u adminuser -pSecurePassword123!
		```
			
		```sql
		USE contact_db;
		SELECT * FROM contacts;
		```

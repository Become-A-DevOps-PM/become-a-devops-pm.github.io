+++
title = "Install the LEMP Stack on an Ubuntu VM Using Azure CLI"
weight = 6
date = 2024-12-07
draft = false
+++

## Overview

This tutorial explains how to install the LEMP stack (Linux, Nginx, MySQL, PHP) on an Ubuntu Virtual Machine (VM).

### Step 1: Provision the Virtual Machine

1. Create a resource group:

	```bash
	az group create --name LempRG --location northeurope
	```

2. Provision the Ubuntu VM:

	```bash
	az vm create \
	  --resource-group LempRG \
	  --name LempVM \
	  --image Ubuntu2204 \
	  --size Standard_B1s \
	  --admin-username azureuser \
	  --generate-ssh-keys
	```

3. Open port 80 for HTTP traffic:

	```bash
	az vm open-port --resource-group LempRG --name LempVM --port 80
	```

3. Retreive the Public IP of the VM:

	```bash
	echo $(az vm show --resource-group LempRG --name LempVM --show-details --query publicIps -o tsv)
	```

### Step 2: Install the LEMP Stack

1. **Connect to the VM**:

   ```bash
   ssh azureuser@<VM_Public_IP>
   ```

   Replace `<VM_Public_IP>` with the public IP of your VM.

2. **Install the LEMP stack**:

   ```bash
   sudo apt update
   sudo apt install nginx -y
   sudo apt install mysql-server -y
   sudo apt install php-fpm php-mysql -y
   ```

3. **Verify all services are running**

	```bash
	sudo systemctl status nginx.service 
   	sudo systemctl status mysql.service 
   	sudo systemctl status php8.1-fpm.service
   	```
   	
   	Change the PHP version to the one you installed

4. **Configure Nginx to Use PHP**

	1. Open or create a configuration file for your site:
	
	   ```bash
	   sudo nano /etc/nginx/sites-available/default
	   ```
	   
	2. Add the following configuration:
	
		> /etc/nginx/sites-available/default
		
	   ```nginx
	   server {
	       listen 80;
	       server_name _;
	
	       root /var/www/html;
	       index index.php index.html index.htm;
	
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
	   ```
   		
   		Restart Nginx:
		   
		 ```bash
		 sudo systemctl restart nginx
		 ```


6. **Test PHP with Nginx**:

   - Create a test PHP file:
   
     ```bash
     echo "<?php phpinfo(); ?>" | sudo tee /var/www/html/info.php
     ```
   - Browse to `<VM_Public_IP>/info.php`. You should see the PHP information page.


### Cleanup (Optional)

To avoid unnecessary costs, delete the resource group and all associated resources:

```bash
az group delete --name LempRG --no-wait --yes
```


# TL;DR

> `provision_vm_lemp.sh`

```bash
#!/bin/bash

az group create --name LempRG --location northeurope

az vm create \
  --resource-group LempRG \
  --name LempVM \
  --image Ubuntu2204 \
  --size Standard_B1s \
  --admin-username azureuser \
  --generate-ssh-keys \
  --custom-data @cloud_init_lemp.yaml

az vm open-port --resource-group LempRG --name LempVM --port 80

echo $(az vm show --resource-group LempRG --name LempVM --show-details --query publicIps -o tsv)
```

> `cloud_init_lemp.yaml`

```bash
#cloud-config

packages:
  - nginx
  - mysql-server
  - php-fpm
  - php-mysql

write_files:
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

  - path: /var/www/html/info.php
    content: |
      <?php phpinfo(); ?>
      
runcmd:
  - systemctl restart nginx
```

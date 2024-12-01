+++
title = "Use A Reverse Proxy In Front of the App"
weight = 5
date = 2024-02-27
draft = false
+++

## Introduction

This tutorial is designed for developers and system administrators looking to configure **Nginx as a reverse proxy** in front of an application server on Azure. Both servers will reside within the same virtual network.

## Prerequisites

- An active Azure account. Sign up [here](https://azure.microsoft.com/) if you don't have one.
- Familiarity with Azure portal, cloud networking, and SSH.
- Basic knowledge of Nginx.

In order to "set the table" so that we can verify that the reverse proxy works as expected, let's create two VMs - one Application Server and one Reverse Proxy. These two servers should be connected to the same subnet in this example.

### 1. **Provision the Virtual Network**

- **Log into Azure Portal**: Access your Azure account at [Azure Portal](https://portal.azure.com/).

- **Create a Virtual Network**:
  - Navigate to "Virtual networks" and select "Create".
  - Fill in the details, including name (e.g., `DemoVNet`), region, and address space (e.g., `10.1.0.0/16`).
  - Create a subnet within the VNet with a specific address range (e.g., `10.1.0.0/24`). If you do it in the portal a `default` subnet will automatically be created.

### 2. **Deploy Two Azure VMs**

- **Create VM for Nginx Reverse Proxy**:
  - Go to "Virtual machines" > "Create" > "Virtual machine".
  - Choose the previously created resource group and virtual network.
  - Select an Ubuntu Server image (22.04 LTS recommended).
  - Size the VM according to your needs and set up SSH authentication.
  - **Assign the VM to the subnet within your virtual network.**
  - **Open port 80 for HTTP traffic.**

- **Create VM for Application Server**:
  - Repeat the above steps to create a second VM within the same virtual network and subnet. This VM will host your application.

### 3. **Set Up the Application Server**

- **SSH into Your App Server VM**:

  ```bash
  ssh username@app_server_vm_ip
  ```
  
- **Install Nginx and Configure Your Application** (for demonstration purposes):
  - Install Nginx and start the service

    ```bash
    sudo apt-get update && sudo apt-get install nginx -y
    ```

  - Edit the default web page so that you can identify it when you browse to it. This step is just to set it apart from the Reverse Proxy that also runs Nginx.

## Install and Configure Nginx on the Reverse Proxy Server

- **SSH into Your Reverse Proxy VM**:

  ```bash
  ssh azureuser@reverse_proxy_vm_ip
  ```
- **Install Nginx**:

  ```bash
  sudo apt update && sudo apt install nginx -y
  ```
- **Configure Nginx as a Reverse Proxy**:
  - Edit the Nginx server block in `/etc/nginx/sites-available/`
	  
	  ```bash
	  sudo nano /etc/nginx/sites-available/default
	  ```
  - Configure the server block to proxy requests to your application server:

    > /etc/nginx/sites-available/default
  
    ```nginx
    server {
      listen        80 default_server;
      location / {
        proxy_pass         http://<InternalIP>:<Port>/;
        proxy_http_version 1.1;
        proxy_set_header   Upgrade $http_upgrade;
        proxy_set_header   Connection keep-alive;
        proxy_set_header   Host $host;
        proxy_cache_bypass $http_upgrade;
        proxy_set_header   X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header   X-Forwarded-Proto $scheme;
      }
    }
    ```
	  
    > Note that you have to change the IP and port according to what your App Server has. Also make sure the firewall is open between the Reverse Proxy and the App Server.

  - Enable the configuration by linking it to the `sites-enabled` directory and reloading Nginx:
  
    ```bash
    sudo nginx -t
    sudo systemctl reload nginx
    ```

### 5. **Testing and Verification**

- **Access Your Application**: Open a web browser and navigate to your reverse proxy's public IP address. You should be served content from your application server via the Nginx reverse proxy.



# Happy Reverse Proxy! 🚀
+++
title = "Provision a Virtual Machine on Azure and Install an Nginx Web Server"
weight = 1
date = 2024-02-11
draft = false
+++

## Video

[See Video](https://vimeo.com/912243936/b7cadb99bc)

<div style="padding:56.25% 0 0 0;position:relative;"><iframe src="https://player.vimeo.com/video/912243936?h=b7cadb99bc&amp;title=0&amp;byline=0&amp;portrait=0&amp;badge=0&amp;autopause=0&amp;player_id=0&amp;app_id=58479" frameborder="0" allow="autoplay; fullscreen; picture-in-picture; clipboard-write" style="position:absolute;top:0;left:0;width:100%;height:100%;" title="Azure VM Nginx"></iframe></div><script src="https://player.vimeo.com/api/player.js"></script>

## Introduction

This tutorial is designed for individuals with a basic understanding of cloud concepts and who are interested in learning how to set up a virtual machine (VM) on Azure and deploy an Nginx web server. We will go step-by-step through the process, emphasizing best practices and troubleshooting.

## Method

- We will use the Azure web console to manually provision a VM running the Linux distribution Ubuntu as Operating System (OS)
- We will deploy an Nginx web server by using the Ubuntu package manager running _cloud-init_.
- We will use a locally installed terminal to access the server (GitBash on Windows)

## Prerequisites

- An Azure account. If you don't have one, sign up [here](https://azure.microsoft.com/).
- Basic knowledge of cloud computing and familiarity with command-line interfaces (CLI).
- For Windows users: GitBash or similar terminal (Mac and Linux users can use the pre-installed terminal)

## Provision a Virtual Machine

1. Log into Azure Portal
   - Go to the [Azure Portal](https://portal.azure.com/).
   - Enter your credentials to log in.

2. Create a New Resource Group (this step can also be done in the next step - in the wizard for creating a VM)
   - Within the portal, find and select "Resource groups".
   - Fill out the basic settings:
     - Select the Subscription
	 - Specify a resource group name
     - Select the Region (preferably one closest to you for reduced latency).
   - Click on "Review + create" and then "Create" to create the new resource group.

3. Create the VM
   - Go to "Virtual machines" and then click on "Create" -> "Azure virtual machine".
   - Fill out the basic settings:
     - Subscription and resource group (create a new one if you skipped step 2)
     - Virtual machine name.
     - Region (preferably one closest to you for reduced latency).
     - Availability options as per your requirement.
     - Image: Ubuntu Server 22.04 LTS - x64 Gen2.
     - Size: "Standard_B1ls" is the cheapest and often good for testing.
   - Under the "Administrator account" section, choose "SSH public key" if you're familiar with SSH, or "Password" for a simpler setup.
   - In the "Inbound port rules" section:
   	 - Select inbound ports: allow HTTP (port 80) and SSH (port 22).
   - Navigate to the "Advanced" tab, find the "Custom Data" section, and paste your cloud-init script (provided below).
		```bash
		#!/bin/bash
		apt-get update -y
		apt-get install nginx -y
		systemctl start nginx
		systemctl enable nginx

		```
   - Once all details are filled in, review the configuration by clicking "Review + create" and then click "Create".
   - Download the SSH private key from the popup dialog box (if you chose SSH above)

> - Choosing a region close to you or your users minimizes latency.
> - The "Standard_B1s size is cost-effective for small projects and testing environments, but you should choose based on your specific needs.
> - Ubuntu 22.04 LTS offers long-term support and stability.


## Accessing the VM
   - After the VM is set up, navigate to the VM page in the Azure portal.
   - Click on "Connect" and choose "Native SSH" for instructions on how to connect using an SSH client.
   - Open up a terminal and navigate to the directory containing the SSH key
   - On Mac and Linux you need to change the priviliges of the private SSH key by running `chmod 400 <SSH_key>` (this makes it read-only just for you)
   - SSH into the VM: `ssh -i <path/to/key> azureuser@<external_IP>`

> `azureuser` is the default user for azure VMs

## Verify Nginx Installation
   - Access your server using the public IP provided in the VM overview page.
   - You should see the Nginx welcome page by navigating to your VM's public IP address in a web browser. (this might take a minute or so)

## Additional Configuration (Optional)

- You can customize the served content by editing the default index file at `/var/www/html/index.nginx-debian.html` using a command like `sudo nano /var/www/html/index.nginx-debian.html`.
- You can also add a completely new `index.html` file, which will override the default one

## Troubleshooting

- **Firewall Issues**: Ensure the Azure network security group (NSG) rules allow HTTP and SSH traffic.
- **Nginx Service**: Verify Nginx is running with `systemctl status nginx`. If not, start it using the commands provided in the cloud-init script.
- Logs from `cloud-init` can be found here:
  -  `/var/log/cloud-init.log`
  -  `/var/log/cloud-init-output.log`

## Final Thoughts

Limitations:
- Content is served unencrypted over HTTP. Consider setting up SSL/TLS for secure communication.
- Access to the VM can be achieved through various methods, including Azure's cloud shell, SSH, or even Azure Bastion for a more secure, private connection without exposing SSH to the internet.

## Don't Forget

Azure services incur costs based on resource consumption, including VM instances, outbound data transfers, and more. It's important to monitor your usage and manage resources effectively to avoid unexpected charges.

Remember to delete resources you no longer need to prevent ongoing charges.

# Happy cloud computing! 🚀

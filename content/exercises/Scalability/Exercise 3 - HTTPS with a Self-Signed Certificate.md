+++
title = "3. Enabling HTTPS with a Self-Signed Certificate"
weight = 3
date = 2024-12-31
draft = false
+++

## Overview
In this exercise, you will **enable HTTPS** on the **Azure Application Gateway** (AGW) created in the previous exercise. You will generate a self-signed certificate, upload it to the Application Gateway’s HTTP settings, and create an HTTPS listener.  

> 💡 **Prerequisite**
> 
> - You must have an **Application Gateway** and **VM Scale Set** already set up, as described in the previous exercises.  

### Step 1: Generate a Self-Signed Certificate (Local Machine or Cloud Shell)

You can generate a self-signed certificate using **OpenSSL** (or PowerShell, if on Windows). Below is an example using OpenSSL (available by default in Azure Cloud Shell’s `Bash` environment):

```bash
openssl genrsa -out mySelfSigned.key 2048

openssl req -new -key mySelfSigned.key -out mySelfSigned.csr -subj "/C=SE/ST=-/L=Molndal/O=CampusMolndal/CN=labappgw"

openssl x509 -req -in mySelfSigned.csr -signkey mySelfSigned.key -out mySelfSigned.crt -days 365

openssl pkcs12 -export -out mySelfSigned.pfx -inkey mySelfSigned.key -in mySelfSigned.crt -passout pass:"MyStrongPassword123"
  
```

> 💡 **Information**
> 
> **Application Gateway** requires a **PFX** certificate with a private key. In this example, the password is `"MyStrongPassword123"`, which you’ll need when uploading to the gateway.

### Step 2: Upload the Certificate to Application Gateway
1. In the [Azure Portal](https://portal.azure.com/), search for **"Application gateways"** and select your existing gateway (e.g., `LabAppGw`).
2. Under **Settings**, click **Listeners**.
	- Add listener
		- Listener name: HTTPSListener
		- Protocol: HTTPS
		- Cert name: mySelfSignedCertificate
		- PFX certificate file: mySelfSigned.pfx
		- Password: MyStrongPassword123
		- **Add**
2. Under **Settings**, click **Rules**.
	- Add routing rule
		- Rule name: HTTPSRule
		- Priority: 101
		- Listener:
			- Listener: HTTPSListener
		- Backend targets:
			- Backend target: LabAppBackend pool
			- Backend settings: HTTPListener
		- Add
3. Click **Add** or **Save**.

> ✅ **Verification Step: Test Your HTTPS Endpoint**  

> 1. In **LabAppGw** > **Overview**, copy the **Frontend public IP address**.
> 2. In a web browser, navigate to `https://<ApplicationGateway_PublicIP>`.
> 3. Since you’re using a **self-signed certificate**, your browser may warn that the certificate is not from a trusted authority. Proceed to the site.
> 4. You should see the Nginx welcome page from your VM scale set via HTTPS, with the random numbers previously configured in your cloud-init script.

> 💡 **Information**
> 
> 	- You can also map a custom domain to your Application Gateway by creating a CNAME or A record in DNS, then importing a certificate that matches your domain name.  
> 	- For production, use a certificate from a **trusted Certificate Authority**.

### Step 3: Redirect HTTP to HTTPS
2. Under **Settings**, click **Rules**.
	- Click the HTTPRule
		- Go to Backend targets:
			- Target type: Redirection
			- Redirection target: Listener
			- Target listener: HTTPSListener
		- Save

> ✅ **Verification Step: Test Your HTTP Redirect to HTTPS**  
> 
> 1. In a web browser, navigate to `http://<ApplicationGateway_PublicIP>`.
> 2. Verify that you will end up with a secure HTTPS connection


### Conclusion
By completing this exercise, you have:

- **Generated a Self-Signed Certificate**: Using OpenSSL  
- **Configured Application Gateway for HTTPS**: Created an HTTPS listener, uploaded the PFX certificate, and linked it to the correct backend settings.  
- **Verified Secure Connections**: Confirmed the gateway can serve requests over **port 443** with your self-signed certificate.

### (Optional) Clean Up Resources
If you no longer need this environment:

1. In the **Azure Portal**, search for **Resource Groups**.
2. Locate **LabResourceGroup** (or the name you chose).
3. Click **Delete resource group**, confirm by typing the name, then **Delete**.

# Happy Cloud!

+++
title = "3. Provision CosmosDB on Azure with MongoDB API"
weight = 3
date = 2025-02-04
draft = false
+++

## Goal

In this tutorial, you will learn how to:

1. Provision an Azure CosmosDB account using the Azure Portal.
2. Configure the account to use the MongoDB API with serverless capacity and Request Units (RU).

### Prerequisites

- An active Azure subscription.
- Azure Portal access ([https://portal.azure.com](https://portal.azure.com)).
- Basic understanding of MongoDB concepts.

## Step-by-step Guide

### 1. Log in to Azure Portal

Steps:

1. Open a web browser and navigate to the [Azure Portal](https://portal.azure.com/).
2. Sign in using your Azure account credentials.

### 2. Create a Resource Group

We’ll create a resource group to organize the resources related to CosmosDB.

Steps:

1. In the Azure Portal, search for **"Resource groups"** in the search bar.
2. Click **"Create"** and fill in the following:
   - **Subscription**: Choose your subscription.
   - **Resource Group Name**: `NewsletterRG`
   - **Region**: Choose a region (e.g., `North Europe`).
3. Click **Review + Create** and then **Create**.

### 3. Provision CosmosDB with MongoDB API

We’ll now create a CosmosDB account configured to use the MongoDB API.

Steps:

1. In the Azure Portal, search for **Azure Cosmos DB** and click **Create**.
2. Select **Azure Cosmos DB for MongoDB** and click **Create**.
3. Select **ARequest unit (RU) database account** and click **Create**.
4. Configure the CosmosDB account:
   - **Subscription**: Select your subscription.
   - **Resource Group**: Choose `NewsletterRG`.
   - **Account Name**: `cosmosdb<date><time>` (must be globally unique).
   - **Location**: North Europe.
   - **Capacity Mode**: Select **Serverless**.
6. Click **Review + create** and then **Create**

> 💡 **Information**\
> The **MongoDB API** on CosmosDB allows you to use MongoDB tools and drivers.

### 4. Connect to the Database

Now we’ll connect to the CosmosDB using the MongoDB connection string.

Steps:

1. In your CosmosDB account, go to **Connection String** under **Settings**.
2. Copy the **Primary connection string**.

> 💡 **Information**\
> The connection string contains credentials, so keep it secure.

3. Login to your VM and set the connection string as an environment variable

```bash
sudo nano /etc/Newsletter/.env
``` 

```
ASPNETCORE_ENVIRONMENT=Production
ASPNETCORE_URLS="http://*:5000"
MongoDbSettings__ConnectionString=mongodb://cosmosdb1111...
DatabaseToUse=MongoDb
```

> 💡 **Note**
> Note how the json hierarchy in appsettings.json is replaced by dubble underscore (__).

> ```
> ...
> },
>   "MongoDbSettings": {
>     "ConnectionString": "SET PROD CONNECTION STRING",
>     ...
>   },
> ...
> ```

4. Restart the service running the dotnet app

```bash
sudo systemctl restart Newsletter.service
```

### 6. Insert and Query Data

Now browse to the app and add some Subscribers. Make sure it look alright in the app.

> 💡 **Verification**

1. Navigate to your **CosmosDB account**.
2. In the left-hand menu, select **Data Explorer**.
3. You should see the posts from the app

> 💡 **Information**\
> Collections are equivalent to tables in relational databases. Documents are stored in collections, and they are formatted as JSON.

# You now have a CosmosDB with MongoDB API running on Azure! 🚀

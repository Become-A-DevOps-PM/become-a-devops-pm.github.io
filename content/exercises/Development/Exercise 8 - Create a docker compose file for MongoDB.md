+++
title = "8. Create a Docker Compose File for MongoDB"
weight = 8
date = 2025-02-02
draft = false
+++

## Goal

In this tutorial, you will learn how to:

1.	Create a docker-compose.yaml file to run MongoDB in a container.
2.	Add Mongo Express to visually manage the MongoDB database through a web interface.
3.	Configure persistent storage for MongoDB data.
4.	Connect your ASP.NET application to the MongoDB instance running in Docker.
5.	Verify that the application correctly interacts with MongoDB.

## Step-by-step Guide

### 1. Create the Docker Compose File

We’ll use Docker Compose to define and manage multi-container applications. In this case, we’ll set up MongoDB along with Mongo Express, a web-based UI to manage MongoDB.

Steps:

1.	Navigate to the root of your project in the terminal.
2.	Create a new folder named `infra` to store infrastructure-related files
3.	Create a new file named `mongodb_and_express.yaml`
4.	Add the following content to the file:

> infra/mongodb_and_express.yaml

```yaml
services:
  # MongoDB service
  db:
    image: mongo
    restart: always
    ports:
      - "27017:27017"
    volumes:
      - mongodb-data:/data/db

  # Mongo Express service
  mongo-express:
    image: mongo-express
    restart: always
    ports:
      - "8081:8081"
    environment:
      ME_CONFIG_MONGODB_SERVER: db

volumes:
  mongodb-data: 
```

> Purpose:
> 
> - db Service: Runs the MongoDB database on port 27017 and stores data persistently in the mongodb-data volume.
> - mongo-express Service: Provides a web-based UI accessible on port 8081 to manage your MongoDB instance.
> - Volumes: Ensures MongoDB data persists across container restarts.

### 2. Start the MongoDB and Mongo Express Containers

Now that the Docker Compose file is ready, we’ll spin up the containers. Make sure Docker Desktop is running.

Steps:

1.	In your terminal, navigate to the project root directory (if you’re not already there).
2.	Run the following command to start the MongoDB and Mongo Express services in detached mode (-d):

	```bash
	docker compose -f ./infra/mongodb_and_express.yaml up -d  
	```
4.	Open your browser and navigate to http://localhost:8081 to access the Mongo Express interface. You should see your MongoDB instance running.

	- The credentials are: `admin:pass`

> 
> Purpose:
> 
> This step runs MongoDB in the background and provides a UI to easily view and manage your data.

### 3. Clean Up

If you want to stop the MongoDB containers, use the following command:

```bash
docker compose -f ./infra/mongodb_and_express.yaml down  
```

This will stop and remove the containers but keep the data in the mongodb-data volume. To remove the volume as well, run:

```bash
docker compose -f ./infra/mongodb_and_express.yaml down -v  
``` 

## Summary

In this tutorial, we:

1.	Created a Docker Compose file to run MongoDB and Mongo Express.
2.	Configured persistent storage for MongoDB data.
3.	Updated the ASP.NET application to connect to MongoDB running in Docker.
4.	Verified that the application can successfully add subscribers to the MongoDB database.
5.	Used Mongo Express to visually manage and inspect MongoDB data.

# Now you have MongoDB running 🚀
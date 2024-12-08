+++
title = "1. Setup Contact Form using LEMP Stack"
weight = 1
date = 2024-12-08
draft = false
+++

## Overview

In this exercise we will implement a simple Contact Form on a web page and save the contact information in the database. The solution is built on the LEMP stack (Linux, Nginx, MySQL, PHP).

### Prerequisites

- You have an Ubuntu VM with the LEMP stack installed.
- You can login to the VM using SSH


### Step 1. Set Up the MySQL Database

Our first step is to setup a database in the MySQL Database Instance. The database will have a schema with a table for the contacts and a user that can access the database from the PHP code.

1. **Log in to MySQL**:

   ```bash
   sudo mysql -u root
   ```
   
   You will see the `mysql>` terminal prompt
   
2. **Create a database**:


   ```sql
   CREATE DATABASE contact_db;
   ```
   
3. **Create a table for storing contact information**:


   ```sql
   USE contact_db;

   CREATE TABLE contacts (
       id INT AUTO_INCREMENT PRIMARY KEY,
       name VARCHAR(100) NOT NULL,
       email VARCHAR(100) NOT NULL,
       message TEXT NOT NULL,
       submitted_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
   );
   ```
   
4. **Create a new MySQL user and grant permissions**:


   ```sql
   CREATE USER 'php_user'@'localhost' IDENTIFIED BY 'secure_password';
   
   GRANT ALL PRIVILEGES ON contact_db.* TO 'php_user'@'localhost';
   
   FLUSH PRIVILEGES;
   EXIT;
   ```

> ✅ **Verification Step:**  
> 
> 
> 	Connect to the database
> 	
> 	```
> 	sudo mysql -u root
> 	```
> 	
> 	Insert test data
> 	
> 	```
> 	USE contact_db;
> 	INSERT INTO contacts (name, email, message)VALUES ('John Doe', 'john.doe@example.com', 'This is a test message.');
> 	SELECT * FROM contacts;
> 	```
> 	Expected result
> 	
> 	```
> 	mysql> SELECT * FROM contacts;
> 	+----+----------+----------------------+-------------------------+---------------------+
> 	| id | name     | email                | message                 | submitted_at        |
> 	+----+----------+----------------------+-------------------------+---------------------+
> 	|  1 | John Doe | john.doe@example.com | This is a test message. | 2024-12-07 10:55:57 |
> 	+----+----------+----------------------+-------------------------+---------------------+
> 	1 row in set (0.00 sec)
> 	```
> 
>  Exit back to bash
> 
>  ```sql
>  EXIT;
>  ```

### Step 2. Create the HTML Form

Now we need to create a web page with a Contact form. On submit the form will call PHP code on the backend using the HTTP POST method. The data of the form will be sent in the HTTP Request Body and processed by the PHP code when it arrives at the backend.

1. Create an HTML file in your web directory:

   ```bash
   sudo nano /var/www/html/index.html
   ```
   
2. Add the following code:

   ```html
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
   ```

### Step 3. Create the PHP Script to Process the Form

In this step we will implement the PHP code that receives the post message from the browser.

1. Create a PHP file to handle form submissions:

   ```bash
   sudo nano /var/www/html/on_post_contact.php
   ```
   
2. Add the following PHP code:

   ```php
   <?php
   // Database credentials
   $servername = "localhost";
   $username = "php_user";
   $password = "secure_password";
   $dbname = "contact_db";

   // Establish database connection
   $conn = new mysqli($servername, $username, $password, $dbname);

   // Check connection
   if ($conn->connect_error) {
       die("Connection failed: " . $conn->connect_error);
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
   ```

> ✅ **Verification Step:**  
> 
> 
> 1. **Verify the Web Page**
> 	
>     1. Open your browser and visit `http://<VM_Public_IP>`.
> 	   2. Fill out the form and submit it. If everything is configured correctly:
> 	      - You should see a success message.
> 	      - The submitted data will be stored in the `contacts` table in the `contact_db` database.
> 
> 2. **Verify the Stored Data**
> 
>     1. Log in to MySQL:
> 
>        ```
>        sudo mysql -u root
>        ```
>     
> 	   2. Check the data in the `contacts` table:
> 	
> 	      ```
> 	      USE contact_db;
> 	      SELECT * FROM contacts;
> 	      ```
> 	   
> 	   3. Expected result
> 
> 		   ```
> 		   Database changed
> 		   +----+----------+----------------------+-------------------------+---------------------+
> 		   | id | name     | email                | message                 | submitted_at        |
> 		   +----+----------+----------------------+-------------------------+---------------------+
> 		   |  1 | John Doe | john.doe@example.com | This is a test message. | 2024-12-08 11:31:38 |
> 		   |  2 | Jane     | jane@email.com       | Hello                   | 2024-12-08 11:43:10 |
> 		   +----+----------+----------------------+-------------------------+---------------------+
> 		   2 rows in set (0.00 sec)
> 		   ```
> 		
> 	      ```
> 	      EXIT;
> 	      ```

### Install phpMyAdmin (optional)

phpMyAdmin is a web based administrative tool for MySQL. It makes it much easier to look into the database and verify results than using the MySQL CLI tool in the terminal.

1. Download the `phpMyAdmin` code and put it as a subfolder to the nginx root directory

	```bash
	cd /var/www/html
	sudo wget https://www.phpmyadmin.net/downloads/phpMyAdmin-latest-all-languages.tar.gz
	sudo tar -xvzf phpMyAdmin-latest-all-languages.tar.gz
	sudo mv phpMyAdmin-*-all-languages phpmyadmin
	sudo rm phpMyAdmin-latest-all-languages.tar.gz
	```

2. Create a database user that the phpMyAdmin tool will use in order to connect to the database. The web UI of this tool is exposed on the Internet (in this example), so choose a good password.
	
	```bash
	sudo mysql -u root
	```
	
	```sql
	CREATE USER 'admin'@'localhost' IDENTIFIED BY 'S3cr3t';
	
	GRANT ALL PRIVILEGES ON *.* TO 'admin'@'localhost';
	
	FLUSH PRIVILEGES;
	EXIT;
	```

> ✅ **Verification Step:**  
> 
> 
> 1. Open your browser and visit `http://<VM_Public_IP>/phpmyadmin`.
> 2. Login using your credentials (admin / S3cr3t)
> 3. You can now see schemas, tables, users, etc in the local MySQL Database Instance
> 

### Install A Local Reverse Proxy (optional)

In order to create a more complete DEV environment for an app running on the LEMP stack, we can configure a local Reverse Proxy on the same VM

In order to have the reverse proxy listen to port 80 we need first to change the current Nginx server block to another port like 8080. For development purposes we also open port 8080 in the firewall in order to be able to directly access the application without going through the reverse proxy

1. Open NSG for port 8080

	```bash
	az vm open-port --resource-group LempRG --name LempVM --port 8080 --priority 1011
	```

2. Change site to listen to port 8080 and add a new server block for the Reverse Proxy

	```bash
	sudo nano /etc/nginx/sites-available/default
	```
	
	```
	server {
	  listen 8080;
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
	
	server {
	  listen 80;
	  location / {
	    proxy_pass http://localhost:8080/;
	    proxy_set_header Host $host;
	    proxy_set_header X-Real-IP $remote_addr;
	    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
	  }
	}
	```
	
	```bash
	sudo systemctl restart nginx
	```

> ✅ **Verification Step:**  
> 
> 
> 1. Open your browser and visit `http://<VM_Public_IP>`.
> 2. You should still be able to see your contact form page
> 
> 3. Open your browser and visit `http://<VM_Public_IP>:8080`.
> 4. You should be able to see your contact form page. In this case you bypass the reverse proxy and go directly to the app without going through the reverse proxy
> 

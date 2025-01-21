+++
title = "1. Create a web app and put it under version control"
weight = 1
date = 2025-01-21
draft = false
+++

## Overview

In this exercise you will create a new .Net MVC web application using the dotnet CLI "scaffolding" tool and put the code under version control in a local git repository

### Step 1. Git Configuration

- Open your terminal.
- Configure your Git username and email:

	```bash
	git config --global user.name "Your Name" 
	git config --global user.email "your_email@example.com"
	```

- Replace `"Your Name"` and `"your_email@example.com"` with your actual name and email1 address. This information will be associated with your commits.
- You only need to do this once on your laptop

Verify your configuration:

```bash
git config --list
```

### Step 2. Create a New Project Directory And a Web App

- Create a new directory for your project and create your web app:

	```bash
	mkdir HelloWorld
	cd HelloWorld
	
	dotnet new mvc
	dotnet new gitignore
	```

Verify your app:

```bash
dotnet run
```

### Step3. Initialize a Git Repository

- Initialize an empty Git repository in the current directory:

	```bash
	git init
	```

* This creates a hidden `.git` directory that contains all the necessary files for Git to track changes.

### Step 4. Stage the Files

- Add the files to the staging area:

	```bash
	git add .
	```

### Step 5. Commit the Changes

- Commit the staged changes with a meaningful message:

	```bash
	git commit -m "Initial commit"
	```

### Step 6. Check the Commit History

- View the commit history:
	
	```bash
	git log
	```

### Step 7. Make a change to your code

- Go to the file Views -> Home -> Index.cshtml
- Change _Welcome_ to _Hello World_ and save the file
- Add the change to stage and commit it to the repo
	
	```bash
	git add .
	git commit -m "Change welcome to hello world"
	```

# Happy Version!

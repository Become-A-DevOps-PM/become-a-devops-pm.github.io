+++
title = "1. Set Up Continuous Integration with GitHub Actions"
weight = 1
date = 2025-02-02
draft = false
+++

## Goal

In this tutorial, you will learn how to:

1.	Set up a Continuous Integration (CI) pipeline using GitHub Actions.
2.	Automate restoring dependencies, building, and publishing your ASP.NET application.
3.	Ensure that every push and pull request to the main branch triggers the CI pipeline.
4.	Utilize helpful VS Code extensions to streamline working with GitHub Actions.

### Prerequisites

- A GitHub repository containing your ASP.NET project.
- Basic knowledge of Git and GitHub (commits, branches, pull requests).


## Step-by-step Guide

### 1. Create a GitHub Actions Workflow File

GitHub Actions uses workflow files written in YAML to define automation steps. We’ll create a workflow that triggers on every _push_ or _pull request_ to the main branch.

Steps:

1.	In the root of your project, create the following folder structure `.github/workflows`
2.	Inside the workflows folder, create a new file named `cicd.yaml`
3.	Add the following content to the cicd.yaml file:

> .github/workflows/cicd.yaml

```yaml
name: CI/CD Pipeline # Name of the GitHub Actions workflow

on:
  push: # Trigger the workflow on push events
    branches:
      - main # Only trigger on pushes to the 'main' branch
  workflow_dispatch: # Enable manual triggering of the workflow

jobs:
  build:
    runs-on: ubuntu-latest # Use the latest Ubuntu runner

    steps:
    - name: Checkout repository # Check out the repository to the GitHub Actions runner
      uses: actions/checkout@v4

    - name: Setup .NET SDK # Set up the .NET SDK according to the specified version
      uses: actions/setup-dotnet@v4
      with:
        dotnet-version: '9.0.x' # Specify the .NET version to use

    - name: Restore, build and publish # Restore dependencies, build, and publish the app
      run: |
        echo "Restore the NuGet packages without using the cache"
        dotnet restore --no-cache

        echo "Build the app in Release configuration without restoring dependencies"
        dotnet build --configuration Release --no-restore
        
        echo "Publish the app to the 'publish' directory in the repository root"
        dotnet publish ./Newsletter.csproj --configuration Release --no-restore --output ./publish

    - name: Upload app artifacts # Upload the published app artifacts to the GitHub artifact repository
      uses: actions/upload-artifact@v4
      with:
        name: app-artifacts # Name the artifact 'app-artifacts'
        path: publish # Specify the path to the 'publish' directory in the repository root
```

> Purpose:
> 
> - `on.push` & `on.pull_request`: The workflow runs when code is pushed or a pull request is created targeting the main branch.
> - build job: Restores dependencies, builds the project, and publishes artifacts.
> - runs-on: ubuntu-latest: Uses the latest Ubuntu environment to run the workflow.

### 2. Push the Workflow to GitHub

Once the workflow file is created, we’ll commit and push it to trigger the CI pipeline.

Steps:

1.	Commit and push the new workflow file to github:

	```bash
	git add .github/workflows/cicd.yaml  
	git commit -m "Add CI workflow with GitHub Actions"
	git push
	```

2.	Go to your GitHub repository, click on the Actions tab, and you should see your workflow running under the name `CI/CD Pipeline`.

	GitHub Actions provides detailed logs for every step in the workflow, helping you troubleshoot issues if the build or tests fail.
	
	Steps:
	
	1.	Navigate to your repository on GitHub.
	2.	Click on the Actions tab.
	3.	Select the Continuous Integration workflow to see the running or completed jobs.
	4.	Click on any job to expand and view detailed logs for each step (restore, build, publish).

	> **Add Visual Studio Code Extensions for GitHub Actions**
	> 
	> VS Code offers extensions, like `GitHub Actions`,to improve your experience when working with GitHub Actions. These extensions can help you write, validate, and monitor your workflows directly from your editor.

3. Verify that you get the artifacts. You find them under _Summary_. Download the ZIP-file and check the content.

## Summary

In this tutorial, we:

1.	Created a GitHub Actions workflow file (cicd.yaml) to automate the CI process.
2.	Configured the pipeline to trigger on pushes and pull requests targeting the main branch.
3.	Automated restoring dependencies, building, and testing the ASP.NET application.
4.	Installed helpful VS Code extensions to improve workflow management.
5.	Verified that the CI pipeline works by pushing code changes and monitoring the workflow in GitHub.

Now you have a fully functional Continuous Integration pipeline that ensures your application is always tested and build-ready before merging any changes. 🚀
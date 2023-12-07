---
title: Online Hosted Instructions
permalink: index.html
layout: home
---

# MLOps Challenges

This repository contains hands-on challenges for end-to-end machine learning operations (MLOps) with Azure Machine Learning.

To complete these exercises, you’ll need a Microsoft Azure subscription. If your instructor has not provided you with one, you can sign up for a free trial at [https://azure.microsoft.com](https://azure.microsoft.com/).

## Challenges walkthrough

### 0: Convert a notebook to production code
Function split_data added to train.py.

Module mlflow added to train.py

### 1: Create an Azure Machine Learning job 

If **az ml -h** shows an error <em>“'ml' is misspelled or not recognized by the system.”</em>

- Upgrade Azure CLI (I upgraded to version 2.54).
- Upgrade Python (I upgraded to version 3.12).
- Remove previously installed ML extension.
- Add ML extension.

#### Login to Azure
- az login
- az account set --subscription "[subscription id]"
- az configure --defaults group="[resource group]"

#### Create an Azure Machine Learning workspace
- az ml workspace create --name "[workspace name]"
- az configure --defaults workspace="[workspace name]"

#### Create a compute instance
- az ml compute create --name "[compute instance name]" --size STANDARD_DS11_V2 --type ComputeInstance

#### Create an environment

Environment defined in two YAML files: src/environment.yml and src/conda-envs/basic-env-ml.yml.
- az ml environment create --file environment.yml

#### Create a data asset

- az ml data create --name diabetes-dev-folder --path experimentation/data

#### Create a job
Job definition is in the src/job.yml. The following command runs job in the Azure ML.
- az ml job create --file src/job.yml

### 2: Trigger the Azure Machine Learning job with GitHub Actions
**To complete the challenge, you need to have the authorization to create a service principal.**

 - az ad sp create-for-rbac --name "sp-mlops-contributor" --role contributor 
                              --scopes /subscriptions/[subscription id]/resourceGroups/[resource group] 
                              --sdk-auth

This command will return a Json. Copy the entire Json output, it is a value for the GitHub secret AZURE_CREDENTIALS_MLOPS. 

Compute cluster created in Azure portal. New job definition that uses cluster computational resource created in the file src/job-cluster.yml

The definition of the manual trigger is in this file: .github/workflows/02-manual-trigger-job.yml

### 3: Trigger GitHub Actions with feature-based development
- A dev branch created.
- Protection rule "Require pull request reviews before merging" applyed for the main branch.
- GitHub acction that triggered by opened/reopened pull request defined in the file .github/workflows/03-pr-check.yml

### 4: Work with linting and unit testing
In case you see this error when using **mlflow** <em>“ModuleNotFoundError: No module named 'pkg_resources'”</em>, try the following command: 
- **pip install setuptools**

Solution described here: https://stackoverflow.com/questions/7446187/no-module-named-pkg-resources

Also, .gitignore should be updated to ignore mlflow output stored locally.

The code check workflow is in the file .github/workflows/04-code-checks.yml

### 5: Work with environments

Dev envronment created in GitHub 'Environments'. Previously created service principal reused for ths environment as AZURE_CREDENTIALS_MLOPS secret.  

Command to create service principal for prod  environment: 

 - az ad sp create-for-rbac --name "sp-mlops-contributor-prd" --role contributor 
                              --scopes /subscriptions/[subscription id]/resourceGroups/[resource group] 
                              --sdk-auth

Prod environment created in GitHub 'Environments'. Required reviewers added to prod environment. Production service principal added as an environment secret AZURE_CREDENTIALS_PROD.

Command to add production data asset:
 - az ml data create --name diabetes-prod-folder --path production/data

 Pipeline for this challenge is in the file .github/workflows/05-environments.yml. Attribute "needs" creates a link between jobs. Parameter “--stream” (-s) displays job output.

 ### 6: Deploy and test the model
The definition for endpoint is in the src/create-endpoint.yml 

Commands to create managed online endpoint and get credentials:
- az ml online-endpoint create --name vp-mlflow-diabetes -f src/create-endpoint.yml
- az ml online-endpoint get-credentials --name vp-mlflow-diabetes

The production model registered manually in ML studio. I tried to automate this step and pass run id to the next step in workflow but without success :(

For successful deploy of the model you need container with sufficient memory. Memory optimized instance type Standard_E2s_v3 is enough for this challenge. Model deployment defined in this file: src/model-deployment.yml. Following command deploys the model:
- az ml online-deployment create -f src/model-deployment.yml

The running endpoint generates cost so better to remove it when it's not needed.  




Completed 7/7
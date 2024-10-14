# Azure DevOps MLOps Template for Classical Machine Learning

**Description:**  
This repository provides an Azure DevOps MLOps pipeline template for classical machine learning workflows. It is designed to automate key processes such as model training, evaluation, and deployment using Azure Machine Learning (AML) services. The pipeline operates on a self-hosted system pool instead of Azure's cloud-based pools.

This repository is based on the mlopsv2 accelerator repo by the Azure team https://github.com/Azure/mlops-v2/tree/main. It addresses and gets past certain errors encountered in the repo as well as uses an AutoML training pipeline for classical regression instead of random forest (still kept in train2.py). This is an aml-cli-v2 version focused on classical machine learning (other versions focused on a different type of machine learning/iac will be implemented at a later date).

## Table of Contents

1. [Getting Started](#getting-started)
2. [Prerequisites](#prerequisites)
3. [Installation and Setup](#installation-and-setup)
4. [Service Principal Setup](#service-principal-setup)
5. [Creating Azure DevOps Environments](#creating-azure-devops-environments)
6. [Directory Structure](#directory-structure)
7. [Project Structure](#project-structure)
8. [Python Pipeline Overview](#python-pipeline-overview) 
9. [MLOps Pipelines and CI/CD Architecture](#mlops-pipelines-and-cicd-architecture)
10. [Deploy and Execute Azure Machine Learning Pipelines](#deploy-and-execute-azure-machine-learning-pipelines)
11. [Usage](#usage)
12. [Contributing](#contributing)
13. [License](#license)

## Getting Started

This repository is designed for deploying and managing machine learning models using Azure DevOps, with a self-hosted agent for CI/CD. The templates automate the installation of required tools and facilitate end-to-end MLOps, from data registration to deployment.

## Prerequisites

Before running the pipeline, ensure that you have the following:

- **Self-hosted Azure DevOps Agent**: A self-hosted system to run Azure DevOps pipelines. (this repo uses a macOS self-hosted agent) https://learn.microsoft.com/en-us/azure/devops/pipelines/agents/osx-agent?view=azure-devops
- **Azure Subscription**: Access to necessary Azure Machine Learning resources.
- **Python 3.12** installed on the self-hosted system.
- **Azure CLI** and **AML CLI**, which will be installed automatically by the pipeline templates.

## Installation and Setup

The templates in the `aml-cli-v2` folder handle most of the installation and setup steps automatically.

- **Azure CLI and AML CLI**: Use `install-az-cli.yml` and `install-aml-cli.yml` to install Azure CLI, AML CLI, and required extensions.
- **Compute Cluster**: The `create-compute.yml` template ensures that the necessary Azure Machine Learning compute resources are provisioned.
- **Workspace Connection**: The `connect-to-workspace.yml` connects the self-hosted system to the Azure Machine Learning workspace, allowing it to manage and deploy models.

## Service Principal Setup

For Azure DevOps pipelines to create Azure Machine Learning infrastructure, deploy, and execute AML pipelines, it is necessary to create an Azure service principal for each Azure ML environment (Dev, Stage, and Prod). These service principals can be created using the Azure Cloud Shell:

1. **Launch the Azure Cloud Shell**
   - Open the [Azure Cloud Shell](https://shell.azure.com/).
   - If prompted, choose **Bash** as the environment.
   - If this is your first time using Cloud Shell, you will be required to create a storage account for it.

2. **Prepare and Run the Bash Commands**
   Update the variables in the following script and run it in the Cloud Shell:

   ```bash
   projectName="<your project name>"
   roleName="Contributor"
   subscriptionId="<subscription Id>"
   environment="<Dev|Stage|Prod>" # Capitalize the first letter
   servicePrincipalName="Azure-ARM-${environment}-${projectName}"

   echo "Using subscription ID $subscriptionId"
   echo "Creating SP for RBAC with name $servicePrincipalName, with role $roleName and in scope /subscriptions/$subscriptionId"

   az ad sp create-for-rbac --name $servicePrincipalName --role $roleName --scopes /subscriptions/$subscriptionId
   echo "Please ensure that the information created here is properly saved for future use."
   ```

3. **Save the Output**
   Save the output containing the service principal credentials securely.

4. **Repeat for Each Environment**
   Repeat these steps for each environment (Dev, Stage, and Prod).

5. **Configure Azure DevOps Service Connections**
   Use the service principal credentials to create service connections in Azure DevOps.

## Creating Azure DevOps Environments

The pipelines in each branch of your ML project repository will depend on Azure DevOps environments. These environments should be created before deployment.

To create the Dev, Stage, and Prod environments:

1. **Create New Environments**
   - In Azure DevOps, select Pipeline in the left menu and then select Environments.
   - Select New Environment.

2. **Name the Environments**
   - Create three environments named dev, stage, and prod.
   - Click Create for each.

The environments will initially be empty and indicate "Never deployed," but this status will update after the first deployment.

Once the environments are created, you are ready to deploy your Azure Machine Learning infrastructure and execute the ML training and model deployment pipelines.

## Directory Structure

```plaintext
aml-cli-v2/
  ├── allocate-traffic.yml              # YAML template for allocating traffic between model versions.
  ├── connect-to-workspace.yml          # Template to connect to an Azure ML workspace.
  ├── create-compute.yml                # Template to create necessary Azure ML compute resources.
  ├── create-deployment.yml             # YAML file to handle the deployment of models.
  ├── create-endpoint.yml               # Template to create endpoints for deployed models.
  ├── install-aml-cli.yml               # Template to install Azure Machine Learning CLI.
  ├── install-az-cli.yml                # Template to install Azure CLI.
  ├── register-data.yml                 # Template to register datasets in Azure ML.
  ├── register-environment.yml          # Template to register environments in Azure ML.
  ├── run-pipeline.yml                  # Template to run the main pipeline.
  ├── test-deployment.yml               # Template for testing the deployment of models.

data-science/
  ├── data/                              # Directory for datasets used in the project.
  │   ├── batch.csv                     # Batch input dataset for model training.
  │   ├── data.csv                      # Main dataset for training and evaluation.
  │   ├── data.yml                      # YAML file containing metadata for the main dataset.
  │   ├── request.json                  # Sample request file for batch scoring.
  │   ├── stage.csv                     # Test Dataset for staging environment.
  │   └── stage.yml                     # YAML file containing metadata for the staging dataset.
  ├── environment/                       # Directory for environment configurations.
  │   ├── automl-conda.yml              # Conda environment configuration for AutoML.
  │   ├── automl-env.yml                # YAML file for the AutoML environment.
  │   ├── train-conda.yml               # Conda environment configuration for training.
  │   └── train-env.yml                 # YAML file for the training environment.
  ├── src/                               # Source code for the project.
  │   ├── evaluate.py                   # Script to evaluate the performance of models.
  │   ├── prep.py                       # Script for data preparation and preprocessing.
  │   ├── register.py                   # Script to register trained models in Azure ML.
  │   ├── stageprep.py                  # Script for staging-specific data preparation.
  │   ├── train.py                      # Script for training models using AutoML.
  │   └── train2.py                     # Script for training models using RandomForestRegressor.

infrastructure/
  ├── env-variables/                     # Directory for environment variable configurations.
  │   ├── config-infra-dev.yml          # Configuration file for the Dev environment.
  │   ├── config-infra-prod.yml         # Configuration file for the Prod environment.
  │   └── config-infra-stage.yml        # Configuration file for the Stage environment.
  ├── modules/                           # Bicep modules for infrastructure setup.
  │   ├── aml_computecluster.bicep      # Bicep template for creating AML compute clusters.
  │   ├── aml_workspace.bicep           # Bicep template for creating AML workspaces.
  │   ├── application_insights.bicep    # Bicep template for setting up Application Insights.
  │   ├── container_registry.bicep      # Bicep template for creating a container registry.
  │   ├── key_vault.bicep               # Bicep template for creating Azure Key Vault.
  │   └── storage_account.bicep         # Bicep template for creating a storage account.
  ├── pipelines/                         # Directory for infrastructure deployment pipelines.
  │   └── bicep-ado-deploy-infra.yml    # YAML template for deploying infrastructure using Bicep.

mlops-pipelines/
  ├── deploy/                             # Directory for deployment pipelines.
  │   ├── batch/                         # Subdirectory for batch deployment pipelines.
  │   │   ├── batch-deployment.yml       # Pipeline for deploying batch models.
  │   │   └── batch-endpoint.yml         # Pipeline for setting up batch endpoints.
  │   └── online/                        # Subdirectory for online deployment pipelines.
  │       ├── online-deployment.yml      # Pipeline for deploying online models.
  │       └── online-endpoint.yml        # Pipeline for setting up online endpoints.
  ├── evaluate/                           # Directory for evaluation pipelines.
  │   ├── test/                          # Subdirectory for testing pipelines.
  │   │   └── stage-test.yml             # Pipeline for testing models in the staging environment.
  │   └── train/                         # Subdirectory for training pipelines.
  │       └── dev-train.yml              # Pipeline for training models in the development environment.
  ├── dev-model-training.yml             # Main development model training pipeline.
  ├── prod-batch-endpoint.yml            # Production batch endpoint deployment pipeline.
  ├── prod-online-endpoint.yml           # Production online endpoint deployment pipeline.
  ├── prod-register-in-production.yml    # Production model registration pipeline.
  ├── stage-batch-endpoint.yml           # Staging batch endpoint deployment pipeline.
  ├── stage-model-testing.yml            # Staging model testing pipeline.
  ├── stage-online-endpoint.yml          # Staging online endpoint deployment pipeline.
  └── stage-register-in-staging.yml      # Staging model registration pipeline.
```

## Project Structure

The repository is organized into several key directories:

### aml-cli-v2/
Contains YAML templates for managing Azure resources, including the creation of compute resources, deployments, and connections to the Azure Machine Learning workspace.

### data-science/
This directory includes all components related to data preparation and model training:
- **data/**: Contains datasets used for training and evaluation, along with metadata files.
- **environment/**: Holds configurations for the Conda environments used in training and AutoML.
- **src/**: Contains the main source code scripts responsible for data preparation, training, evaluation, and model registration.

### infrastructure/
This folder is dedicated to the infrastructure setup for the Azure resources:
- **env-variables/**: Contains configuration files for different environments (Dev, Stage, Prod).
- **modules/**: Houses Bicep modules for resource creation, such as Azure ML workspaces and compute clusters.
- **pipelines/**: Includes YAML files for deploying infrastructure using Bicep.

### mlops-pipelines/
This directory contains the main Azure DevOps pipelines for MLOps processes:
- **deploy/**: Subdirectories for batch and online deployment pipelines, including the necessary YAML files.
- **evaluate/**: Contains testing and training pipelines for evaluating and training models.
- **Main Pipelines**: YAML files for development, staging, and production pipelines that automate model training, deployment, and registration.

## Python Pipeline Overview

This repository includes several Python scripts that handle key stages of the machine learning workflow, including data preparation, evaluation, and model registration. These scripts are integrated into the Azure DevOps pipeline, facilitating automation and efficiency throughout the ML lifecycle.

### Key Python Scripts

1. **`train.py` - AutoML Training Pipeline**  
   This script utilizes Azure AutoML to train a classical regression model, selecting the best model and hyperparameters based on the dataset provided. 

2. **`train2.py` - Custom RandomForestRegressor Training**  
   This script focuses on training a RandomForest model for regression, allowing manual tuning and feature engineering.

3. **`prep.py` - Data Preparation**  
   This script is responsible for preprocessing the data before it is fed into the training pipelines. Key features include:
   - Data Cleaning
   - Feature Engineering
   - Normalization/Scaling
   - Dataset Splitting

4. **`evaluate.py` - Model Evaluation**  
   This script evaluates the performance of trained models based on validation metrics. Key functions include:
   - Performance Metrics Calculation
   - Model Comparison
   - Visualization

5. **`register.py` - Model Registration**  
   This script is used to register trained models into the Azure Machine Learning workspace. Key features include:
   - Model Versioning
   - Metadata Logging
   - Deployment Preparation

6. **`stageprep.py` - Staging Data Preparation**  
   Similar to `prep.py`, this script prepares data specifically for the staging environment.

### Integration into Azure DevOps Pipelines

All these scripts are integrated into the Azure DevOps pipeline, ensuring a seamless workflow from data preparation to model deployment.

### Customization Options

- **Data Processing Logic**: Each of the preparation scripts can be customized to include specific data transformations, feature selections, or additional preprocessing steps that are relevant to your use case.
  
- **Evaluation Criteria**: You can modify `evaluate.py` to include specific performance metrics that are more aligned with your project's goals.

## MLOps Pipelines and CI/CD Architecture

### MLOps Pipelines

The repository includes several pipelines that automate key stages of the machine learning lifecycle:

- **Development Pipeline**
  - `dev-model-training.yml`: Automates model training, data registration, and initial evaluation.

- **Staging Pipelines**
  - `stage-batch-endpoint.yml`: Deploys models to batch endpoints for staging.
  - `stage-online-endpoint.yml`: Deploys models to online endpoints for real-time inference in staging.
  - `stage-model-testing.yml`: Tests the deployed model in the staging environment.
  - `stage-register-in-staging.yml`: Registers models into the staging registry.

- **Production Pipelines**
  - `prod-batch-endpoint.yml`: Deploys the approved model to production batch endpoints.
  - `prod-online-endpoint.yml`: Deploys models to online endpoints for real-time inference in production.
  - `prod-register-in-production.yml`: Registers the final model into the production registry.

### CI/CD Architecture

The repository is designed around a CI/CD pipeline to streamline the development, testing, and deployment of machine learning models:

- **Continuous Integration (CI)**: The development pipeline is triggered with every code commit.
- **Continuous Deployment (CD)**: Deployment pipelines automate the process of moving models through different environments.
- **Model Registry**: Models are registered in different environments using dedicated pipelines.
- **Self-hosted Agent Pool**: The CI/CD process runs on a self-hosted agent pool for better control and integration with on-premise systems.

## Deploy and Execute Azure Machine Learning Pipelines

Now that your ML project is created, follow these steps to deploy and execute the pipelines:

1. **Deploy Azure Machine Learning Infrastructure**
   - Go to your project repository (e.g., taxi-fare-regression).
   - Customize the `config-infra-dev.yml`, `config-infra-stage.yml`, and `config-infra-prod.yml` files to define unique Azure resource groups and Azure ML workspaces for your project in each environment.
   - Run the infrastructure deployment pipeline (`bicep-ado-deploy-infra.yml`) to deploy the Azure Machine Learning resources (e.g., resource groups, workspaces, compute clusters) for each environment.

2. **Deploy and Manage Pipelines**
   - Once infrastructure is deployed, deploy the ML training and model deployment pipelines in the respective environments.
   - Manage the development of the model training pipeline in the Dev environment.
   - When the model is validated in the Dev environment, promote it to the Stage environment for further testing.
   - After successful validation in the Stage environment, promote the model to the Prod environment through pull requests and run the model deployment pipeline.

## Usage

To run the pipeline:

1. **Trigger the Development Pipeline**: Navigate to the Pipelines section in Azure DevOps and trigger the `dev-model-training.yml` pipeline.
2. **Deploy to Staging**: Use the staging pipelines to deploy your model to the staging environment.
3. **Deploy to Production**: After successful staging validation, use the production pipelines to deploy the model into the production environment.
4. **Register Models**: Use the registration pipelines to handle the registration of models in the respective environments.
  
## Contributing

To contribute:

1. Fork the repository.
2. Create a new branch (`git checkout -b feature-branch`).
3. Make your changes and commit them (`git commit -m 'Feature description'`).
4. Push to the branch (`git push origin feature-branch`).
5. Create a new Pull Request.

## License

This project is licensed under the MIT License. See `LICENSE` for more details.

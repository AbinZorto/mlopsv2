# Azure DevOps MLOps Template for Classical Machine Learning

## Description

This repository provides an Azure DevOps MLOps pipeline template for classical machine learning workflows. It is designed to automate key processes such as model training, evaluation, and deployment using Azure Machine Learning (AML) services. The pipeline operates on a self-hosted system pool instead of Azure's cloud-based pools.

## Table of Contents

1. [Getting Started](#getting-started)
2. [Prerequisites](#prerequisites)
3. [Installation and Setup](#installation-and-setup)
4. [Service Principal Setup](#service-principal-setup)
5. [MLOps Pipelines and CI/CD Architecture](#mlops-pipelines-and-cicd-architecture)
6. [Usage](#usage)
7. [Contributing](#contributing)
8. [License](#license)

## Getting Started

This repository is designed for deploying and managing machine learning models using Azure DevOps, with a self-hosted agent for CI/CD. The templates automate the installation of required tools and facilitate end-to-end MLOps, from data registration to deployment.

## Prerequisites

Before running the pipeline, ensure that you have the following:

- **Self-hosted Azure DevOps Agent**: A self-hosted system to run Azure DevOps pipelines.
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

This project is licensed under the MIT License. See LICENSE for more details.

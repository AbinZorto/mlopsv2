# Azure DevOps MLOps Template for Classical Machine Learning

**Description:**  
This repository provides an Azure DevOps MLOps pipeline template for classical machine learning workflows. It is designed to automate key processes such as model training, evaluation, and deployment using Azure Machine Learning (AML) services. The pipeline operates on a self-hosted system pool instead of Azure's cloud-based pools.

---

## Table of Contents

1. [Getting Started](#getting-started)
2. [Prerequisites](#prerequisites)
3. [Installation and Setup](#installation-and-setup)
4. [MLOps Pipelines and CI/CD Architecture](#mlops-pipelines-and-cicd-architecture)
5. [Usage](#usage)
6. [Contributing](#contributing)
7. [License](#license)

---

## Getting Started

This repository is designed for deploying and managing machine learning models using Azure DevOps, with a self-hosted agent for CI/CD. The templates automate the installation of required tools and facilitate end-to-end MLOps, from data registration to deployment.

---

## Prerequisites

Before running the pipeline, ensure that you have the following:

- **Self-hosted Azure DevOps Agent**  
  A self-hosted system to run Azure DevOps pipelines.
  
- **Azure Subscription**  
  Access to necessary Azure Machine Learning resources.

- **Python 3.12** installed on the self-hosted system.

- **Azure CLI** and **AML CLI**, which will be installed automatically by the pipeline templates.

---

## Installation and Setup

The templates in the `aml-cli-v2` folder handle most of the installation and setup steps automatically.

- **Azure CLI and AML CLI**:  
  Use `install-az-cli.yml` and `install-aml-cli.yml` to install Azure CLI, AML CLI, and required extensions.

- **Compute Cluster**:  
  The `create-compute.yml` template ensures that the necessary Azure Machine Learning compute resources are provisioned.

- **Workspace Connection**:  
  The `connect-to-workspace.yml` connects the self-hosted system to the Azure Machine Learning workspace, allowing it to manage and deploy models.

---

## MLOps Pipelines and CI/CD Architecture

### MLOps Pipelines

The repository includes several pipelines that automate key stages of the machine learning lifecycle, allowing for seamless model deployment and management across environments (development, staging, and production):

1. **Development Pipeline**  
   - `dev-model-training.yml`: Automates the training of models using Azure Machine Learning services. This pipeline registers data, trains the model, and performs an initial evaluation.

2. **Staging Pipelines**  
   - `stage-batch-endpoint.yml`: Deploys models to batch endpoints for staging.
   - `stage-online-endpoint.yml`: Deploys models to online endpoints for real-time inference in staging.
   - `stage-model-testing.yml`: Tests the deployed model in the staging environment to validate performance and accuracy.
   - `stage-register-in-staging.yml`: Registers models into the staging registry for future deployments.

3. **Production Pipelines**  
   - `prod-batch-endpoint.yml`: Deploys the approved model to production batch endpoints.
   - `prod-online-endpoint.yml`: Deploys models to online endpoints for real-time inference in production.
   - `prod-register-in-production.yml`: Registers the final model into the production registry after successful testing and evaluation.

### CI/CD Architecture

The repository is designed around a CI/CD pipeline to streamline the development, testing, and deployment of machine learning models.

- **Continuous Integration (CI)**:  
  The development pipeline (`dev-model-training.yml`) is triggered with every code commit. It automates data registration, model training, and evaluation.

- **Continuous Deployment (CD)**:  
  Once models pass evaluation, the deployment pipelines for staging (`stage-batch-endpoint.yml`, `stage-online-endpoint.yml`) and production (`prod-batch-endpoint.yml`, `prod-online-endpoint.yml`) automate the process of moving models through different environments, from development to production.

- **Model Registry**:  
  Models are registered in different environments using the `stage-register-in-staging.yml` and `prod-register-in-production.yml` pipelines, ensuring seamless transitions between stages.

### Self-hosted Agent Pool

The CI/CD process runs on a self-hosted agent pool, which provides more control over the environment and is ideal for custom setups or compliance-related restrictions. This setup allows for better integration with on-premise systems and offers full flexibility over resource management.

---

## Usage

To run the pipeline:

1. **Trigger the Development Pipeline**  
   Navigate to the Pipelines section in Azure DevOps and trigger the `dev-model-training.yml` pipeline to start training a model.

2. **Deploy to Staging**  
   Use the staging pipelines (`stage-batch-endpoint.yml`, `stage-online-endpoint.yml`) to deploy your model to the staging environment.

3. **Deploy to Production**  
   After successful staging validation, use the production pipelines (`prod-batch-endpoint.yml`, `prod-online-endpoint.yml`) to deploy the model into the production environment.

4. **Register Models**  
   The `stage-register-in-staging.yml` and `prod-register-in-production.yml` pipelines handle the registration of models in the respective environments.

---

## Contributing

To contribute, follow these steps:

1. Fork the repository.
2. Create a new branch (`git checkout -b feature-branch`).
3. Make your changes and commit them (`git commit -m 'Feature description'`).
4. Push to the branch (`git push origin feature-branch`).
5. Create a new Pull Request.

---

## License

This project is licensed under the MIT License. See `LICENSE` for more details.


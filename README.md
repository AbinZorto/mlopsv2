# Azure DevOps MLOps Template for Classical Machine Learning

**Description:**  
This repository provides an Azure DevOps MLOps pipeline template for classical machine learning workflows. It is designed to automate key processes such as model training, evaluation, and deployment using Azure Machine Learning (AML) services. The pipeline operates on a self-hosted system pool instead of Azure's cloud-based pools.

This repository is based on the mlopsv2 accelerator repo by the azure team https://github.com/Azure/mlops-v2/tree/main. It adresses gets passed certain errors encountered in the repo as well as uses an automl training pipeline for classical regression instead of random forest (still kept in train2.py). This an aml-cli-v2 version focused on classical machine learning, other versions focused on a different type of machine learning/iac


## Table of Contents

1. [Getting Started](#getting-started)
2. [Prerequisites](#prerequisites)
3. [Python Pipeline Overview](#python-pipeline-overview)
4. [Installation and Setup](#installation-and-setup)
5. [Service Principal Setup](#service-principal-setup)
6. [Creating Azure DevOps Environments](#creating-azure-devops-environments)
7. [MLOps Pipelines and CI/CD Architecture](#mlops-pipelines-and-cicd-architecture)
8. [Deploy and Execute Azure Machine Learning Pipelines](#deploy-and-execute-azure-machine-learning-pipelines)
9. [Usage](#usage)
10. [Contributing](#contributing)
11. [License](#license)

## Getting Started

This repository is designed for deploying and managing machine learning models using Azure DevOps, with a self-hosted agent for CI/CD. The templates automate the installation of required tools and facilitate end-to-end MLOps, from data registration to deployment.

## Prerequisites

Before running the pipeline, ensure that you have the following:

- **Self-hosted Azure DevOps Agent**: A self-hosted system to run Azure DevOps pipelines.
- **Azure Subscription**: Access to necessary Azure Machine Learning resources.
- **Python 3.12** installed on the self-hosted system.
- **Azure CLI** and **AML CLI**, which will be installed automatically by the pipeline templates.

## Python Pipeline Overview

This repository includes several Python scripts that handle key stages of the machine learning workflow, including data preparation, evaluation, and model registration. These scripts are integrated into the Azure DevOps pipeline, facilitating automation and efficiency throughout the ML lifecycle.

### Key Python Scripts

1. **`train.py` - AutoML Training Pipeline**  
   This script utilizes Azure AutoML to train a classical regression model, selecting the best model and hyperparameters based on the dataset provided. 

2. **`train2.py` - Custom RandomForestRegressor Training**  
   This script focuses on training a RandomForest model for regression, allowing manual tuning and feature engineering.

3. **`prep.py` - Data Preparation**  
   This script is responsible for preprocessing the data before it is fed into the training pipelines. Key features include:
   - **Data Cleaning**: Handles missing values, outliers, and inconsistent data entries.
   - **Feature Engineering**: Creates new features from existing data to enhance model performance.
   - **Normalization/Scaling**: Applies techniques to normalize or scale features, ensuring that the data is suitable for the models being trained.
   - **Dataset Splitting**: Splits the data into training and validation sets, optimizing the training process and model evaluation.

   The `prep.py` script ensures that the data is in the right format and quality for the training scripts, improving the overall efficiency and accuracy of the model.

4. **`evaluate.py` - Model Evaluation**  
   This script evaluates the performance of trained models based on validation metrics. Key functions include:
   - **Performance Metrics Calculation**: Computes metrics such as accuracy, precision, recall, F1-score, mean absolute error (MAE), and mean squared error (MSE).
   - **Model Comparison**: Compares the performance of different models trained during the same pipeline run, providing insights into which model performs best under specific conditions.
   - **Visualization**: Generates plots and visualizations (e.g., confusion matrices, ROC curves) to help stakeholders understand model performance.

   The `evaluate.py` script allows data scientists and ML engineers to analyze the model's performance and decide whether to proceed with deployment.

5. **`register.py` - Model Registration**  
   This script is used to register trained models into the Azure Machine Learning workspace. Key features include:
   - **Model Versioning**: Automatically manages model versions, ensuring that each new model is stored correctly in the workspace for tracking and comparison.
   - **Metadata Logging**: Records important metadata about the model, such as performance metrics, training parameters, and dataset versions.
   - **Deployment Preparation**: Prepares models for deployment by ensuring they are registered with the necessary configurations in Azure ML.

   The `register.py` script streamlines the model registration process, making it easy to manage different versions and ensure that only the best-performing models are deployed.

6. **`stageprep.py` - Staging Data Preparation**  
   Similar to `prep.py`, this script prepares data specifically for the staging environment. It may include additional steps specific to the testing or validation of models that are being promoted from Dev to Stage.

### Integration into Azure DevOps Pipelines

All these scripts are integrated into the Azure DevOps pipeline, ensuring a seamless workflow:

- **Data Preparation**: The `prep.py` script is executed as part of the `dev-model-training.yml` pipeline, ensuring that the data is clean and ready before training begins.
  
- **Model Training and Evaluation**: After data preparation, the pipeline executes `train.py` or `train2.py` to train the models, followed by `evaluate.py` to assess their performance based on predefined metrics.

- **Model Registration**: If the evaluation meets the required thresholds, `register.py` is called to register the model in the Azure ML workspace, making it available for deployment.

### Customization Options

- **Data Processing Logic**: Each of the preparation scripts (`prep.py`, `stageprep.py`) can be customized to include specific data transformations, feature selections, or additional preprocessing steps that are relevant to your use case.
  
- **Evaluation Criteria**: You can modify `evaluate.py` to include specific performance metrics that are more aligned with your project's goals, ensuring that the evaluation process captures all necessary aspects of model performance.

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

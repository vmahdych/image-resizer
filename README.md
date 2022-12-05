# Task: Create GitHub Actions workflow


## Requirements:
- [x] Job 1 - Setup environment: Storage account and Container Registry
- [x] Job 2 - Build / Push to ACR: build image, scan image, and push image to ACR
- [x] Job 3 - Deploy to ACI
- [x] Make sure Docker file is [secure](https://dev.to/tomoyamachi/how-to-keep-secure-your-docker-image-2hj2) enough


### Useful resources:

- [To create workflow for Azure environment setup](https://faun.pub/how-to-create-a-pipeline-in-github-workflow-to-deploy-your-app-on-azure-a6e14a184d26)
- [Configure a GitHub Action to create a container instance](https://learn.microsoft.com/en-us/azure/container-instances/container-instances-github-action)
- [Another example GH Actions to ACI ](https://github.com/marketplace/actions/deploy-to-azure-container-instances)
- [Detailed information of how to get all the needed secrets for the workflows](https://willvelida.medium.com/building-and-deploying-container-images-to-azure-container-apps-with-github-actions-5e8e11f1a03)
- [Deploing from ACR to ACI explained](https://learn.microsoft.com/en-us/azure/container-instances/container-instances-using-azure-container-registry)


### Azure CLI commands that have been used during environment setup:

- Login:
  ```bash
  az login
  ```
- Create resourceGroup:
  ```bash
  az group create --name <RG_NAME> --location <SUITABLE_LOCATION>
  ```
- Create ACR:
  ```bash
  az acr create --resource-group <RG_NAME> -n <ACR_NAME> --sku Basic
  ```
-  Create a service principal for GitHub Actions workflow:
  ```bash
  az ad sp create-for-rbac --name <SERVICE_PRINCIPAL_NAME> --role "contributor" --scopes /subscriptions/<SUBSCRIPTION_ID>/resourceGroups/<RESOURCE_GROUP_NAME> --sdk-auth
  ```
- Grant GitHub Actions service principal with the permissions to both pull and push images to Azure Container Registry:
  - Get the id of our Azure Container Registry
  ```bash
  registryId=$(az acr show --name <registry-name> --query id --output tsv)
  ```
  - Grant the 'AcrPush' role to our service principal
  ```bash
  az role assignment create --assignee <ClientId> --scope $registryId --role AcrPush
  ```
  - Grant the 'AcrPull' role to our service principal
  ```bash
  az role assignment create --assignee <ClientId> --scope $registryId --role AcrPull
  ```


### Issues I've faced with during the workflows creation:

- The ImageResizer app itself :smile:
  - Does not work on the latest node version;
  - Tests fail all the time;
- During deploing the process failed firs time. Error below:
  *Error: The subscription is not registered to use namespace 'Microsoft.ContainerInstance'*


### Just a thoughts of how the workflow can be improved:

- [ ] Add lint test for the "start.yml" workflow;
- [ ] Add an event to trigger workflows after adding tag to the app;
- [ ] Divide "deploy-to-azure.yml" jobs by *needs* keyword so the deploing will trigger after successful image build only.


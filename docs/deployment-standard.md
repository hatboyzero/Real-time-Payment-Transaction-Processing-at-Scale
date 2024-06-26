# Deployment - Standard

## Prerequisites

- Azure Subscription
- Subscription access to Azure OpenAI service. Start here to [Request Access to Azure OpenAI Service](https://aka.ms/oaiapply)

- Backend (Web API, Worker Service, Console Apps, etc.)
  - Visual Studio 2022 17.6 or later (required for passthrough Visual Studio authentication for the Docker container)
  - .NET 7 SDK
  - Docker Desktop (with WSL for Windows machines)
  - Azure CLI ([v2.49.0 or greater](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli))
  - [Helm 3.11.1 or greater](https://helm.sh/docs/intro/install/)
- Frontend (React web app)
  - Visual Studio Code
  - Ensure you have the latest version of NPM and node.js:
    - Install NVM from https://github.com/coreybutler/nvm-windows
    - Run nvm install latest
    - Run nvm list (to see the versions of NPM/node.js available)
    - Run nvm use latest (to use the latest available version)

To start the React web app:

1. Navigate to the `ui` folder
2. Run npm install to restore the packages
3. Run npm run dev
4. Open localhost:3000 in a web browser

## Deployment steps

Follow the steps below to deploy the solution to your Azure subscription.

1. Ensure all the prerequisites are installed.  

1. Clone the repository:

    > **Important:** Do not forget the `--recurse-submodules` parameter. This loads the `AKS-Construction` submodule that contains AKS-specific Bicep templates.

    ```bash
    git clone --recurse-submodules https://github.com/Azure/Real-time-Payment-Transaction-Processing-at-Scale.git
    ```

1. Run the following script to provision the infrastructure and deploy the API and frontend. This will provision all of the required infrastructure, deploy the API and web app services into AKS, and provision and load artifacts into a Synapse Analytics workspace.

    ```pwsh
    cd .\Real-time-Payment-Transaction-Processing-at-Scale
    ./deploy/powershell/Unified-Deploy.ps1 -resourceGroup <rg_name> -subscription <target_subscription_id>
    ```

>**NOTE**: Make sure to set the `-openAiLocation` value or the first region of `-locations` list to a region that supports Azure OpenAI services.  See [Azure OpenAI service regions](https://azure.microsoft.com/en-us/explore/global-infrastructure/products-by-region/?products=cognitive-services&regions=all) for more information.

### Deployment samples

1. Default deployment using Azure Container Apps in 3 regions `SouthCentralUS, NorthCentralUS and EastUS` and Azure Open AI Service in `EastUS`. 
    ```pwsh
    cd .\Real-time-Payment-Transaction-Processing-at-Scale
    ./deploy/powershell/Unified-Deploy.ps1 -resourceGroup <rg_name> -subscription <target_subscription_id>
    ```
1. Deployment using Azure Kubernetes Service in 2 regions `SouthCentralUS, NorthCentralUS` and Azure Open AI Service in `EastUS`. 
    ```pwsh
    cd .\Real-time-Payment-Transaction-Processing-at-Scale
    ./deploy/powershell/Unified-Deploy.ps1 -resourceGroup <rg_name> -subscription <target_subscription_id> -locations 'SouthCentralUS,NorthCentralUS' -openAiLocation 'EastUS' -deployAks $true
    ```
1. Deployment using an existing Azure Open AI Service. 
    ```pwsh
    cd .\Real-time-Payment-Transaction-Processing-at-Scale
    ./deploy/powershell/Unified-Deploy.ps1 -resourceGroup <rg_name> -subscription <target_subscription_id> -openAiRg <openai_rg_name> -openAiName <openai_service_name> -openAiCompletionsDeployment <openai_deployment>
    ```

### Enabling/Disabling Deployment Steps

The following flags can be used to enable/disable specific deployment steps in the `Unified-Deploy.ps1` script.

| Parameter Name | Description |
|----------------|-------------|
| stepDeployBicep | Enables or disables the provisioning of resources in Azure via Bicep templates (located in `./infrastructure`). Valid values are 0 (Disabled) and 1 (Enabled). See the `deploy/powershell/Deploy-Bicep.ps1` script.
| stepBuildPush | Enables or disables the build and push of Docker images into the Azure Container Registry (ACR). Valid values are 0 (Disabled) and 1 (Enabled). See the `deploy/infrastructure/BuildPush.ps1` script.
| stepDeployImages | Enables or disables deploying the Docker images from the `CoreClaims.WebAPI` and `CoreClaims.WorkerService` projects to AKS. Valid values are 0 (Disabled) and 1 (Enabled). See the `deploy/infrastructure/Deploy-Images-Aks.ps1` script.
| stepPublishSite | Enables or disables the build and deployment of the static HTML site to the hosting storage account in the target resource group. Valid values are 0 (Disabled) and 1 (Enabled). See the `deploy/infrastructure/Publish-Site.ps1` script.
| stepLoginAzure | Enables or disables interactive Azure login. If disabled, the deployment assumes that the current Azure CLI session is valid. Valid values are 0 (Disabled).

Example command:

```pwsh
cd deploy/powershell
./Unified-Deploy.ps1 -resourceGroup myRg `
                     -subscription 0000... `
                     -stepLoginAzure 0 `
                     -stepDeployBicep 0 `
                     -stepDeployFD 0 `
                     -stepBuildPush 1 `
                     -stepDeployImages 1 `
                     -stepPublishSite 1
```

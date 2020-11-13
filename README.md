# Automatically deploy `helium` using terraform

> This repository is used to automatically deploy [helium](https://github.com/retaildevcrews/helium) using [terraform](https://www.hashicorp.com/products/terraform)

![License](https://img.shields.io/badge/license-MIT-green.svg)

> Visual Studio Codespaces is the easiest way to evaluate helium as all of the prerequisites are automatically installed
>
> Follow the setup steps in the [Helium readme](https://github.com/retaildevcrews/helium) to setup Codespaces

## Prerequisites

- Azure subscription and permissions to create:
  - Resource Groups, Service Principals, Key Vault, Cosmos DB, Azure Container Registry, Azure Monitor, App Service or AKS (Subscription Owner Role)
- Bash shell (tested on Visual Studio Codespaces, Mac, Ubuntu, Windows with WSL2)
  - Will not work in Cloud Shell or WSL1
- Azure CLI ([download](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli?view=azure-cli-latest))
- Terraform 0.12+ ([download](https://www.terraform.io/downloads.html))
- Docker must be installed and running ([download](https://docs.docker.com/get-docker/))

## Setup

```bash

# Clone this repo if not using Codespaces
git clone https://github.com/retaildevcrews/helium-terraform

cd helium-terraform/src/root

```

### Login to Azure

```bash

az login

# show your Azure accounts
az account list -o table

# select the Azure subscription if necessary
az account set -s {subscription name or Id}

```

> All commands require you to be in `helium-terraform/src/root`

### Choose a unique DNS name

```bash
# this will be the prefix for all resources
# only use a-z and 0-9 - do not include punctuation or uppercase characters
# must be at least 5 characters long
# must start with a-z (only lowercase)
export He_Name=[your unique name]

### if true, change He_Name
az cosmosdb check-name-exists -n ${He_Name}

### if nslookup doesn't fail to resolve, change He_Name
nslookup ${He_Name}.azurewebsites.net
nslookup ${He_Name}.vault.azure.net
nslookup ${He_Name}.azurecr.io

```

### Set additional values

```bash

export He_Email=replaceWithYourEmail

# change the location (optional)
export He_Location=centralus

# change the repo (optional - valid: helium-csharp, helium-java, helium-typescript)
export He_Repo=helium-csharp

```

## Deploy `helium`

> Make sure you are in the `helium-terraform/src/root` directory

```bash

# create tfvars file
./create-tf-vars.sh

# initialize
terraform init

# validate
terraform validate

# If you have no errors you can create the resources
terraform apply -auto-approve

# This generally takes about 10 minutes to complete

```

## Verify the deployment

```bash

# check the App Service
curl https://${He_Name}.azurewebsites.net/version
curl https://${He_Name}.azurewebsites.net/api/genres

# check the docker logs from the webv tests
az container logs -g ${He_Name}-rg-webv -n ${He_Name}-webv-${He_Location}

# check Log Analytics
export He_LogAnalytics_Id='az monitor log-analytics workspace show -g ${He_Name}-rg-webv -n ${He_Name}-webv-logs --query customerId -o tsv'

az monitor log-analytics query -w $(eval $He_LogAnalytics_Id) --analytics-query "ContainerInstanceLog_CL | sort by TimeGenerated"

```

You can also log into the Azure portal and browse the new resources

> If the terraform plan command is redirected to a file, there will be secrets stored in that file!
>
> Be sure not to remove the ignore *tfplan* in the .gitignore file
>

## Module Documentation

Each module has a `README` file in the module directory under [`./src/modules/`](./src/modules/). You can reference the module documentation for the specific requirements of each module.

The main calling Terraform script can be found at [`./src/root/main.tf`](./src/root/main.tf)

Customizations are generated by `create-tf-vars.sh` and stored in `terraform.tfvars`.  This file should never be added to github and is included in the .gitignore file.

## Removing the deployment

>
> WARNING - this will delete everything with only one prompt
>

```bash

# this takes several minutes to run
terraform destroy

# delete the service principals
az ad sp delete --id http://${He_Name}-acr-sp
az ad sp delete --id http://${He_Name}-tf-sp

# remove state and vars files
rm terraform.tfstate*
rm terraform.tfvars

```

## Contributing

This project welcomes contributions and suggestions. Most contributions require you to agree to a Contributor License Agreement (CLA) declaring that you have the right to, and actually do, grant us the rights to use your contribution. For details, visit [Microsoft Contributor License Agreement](https://cla.opensource.microsoft.com).

When you submit a pull request, a CLA bot will automatically determine whether you need to provide a CLA and decorate the PR appropriately (e.g., status check, comment). Simply follow the instructions provided by the bot. You will only need to do this once across all repos using our CLA.

This project has adopted the [Microsoft Open Source Code of Conduct](https://opensource.microsoft.com/codeofconduct/). For more information see the [Code of Conduct FAQ](https://opensource.microsoft.com/codeofconduct/faq/) or contact [opencode@microsoft.com](mailto:opencode@microsoft.com) with any additional questions or comments.
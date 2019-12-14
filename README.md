# Azure Terraform Landing Zone Tutorial

## Terraform Tutorial

1. Create resource group "testResourceGroup"
```
$ cat test.tf
provider "azurerm" {
}
resource "azurerm_resource_group" "rg" {
        name = "testResourceGroup"
        location = "westus"
}

$ terraform init
$ terraform plan
$ terraform apply
```
2. Add variables.tf and outputs.tf
```
$ cat variables.tf
variable "location" {
 description = "This is azure region that this resource will reside"
 default = "koreacentral"
}

$ cat outputs.tf
output "resourcegroup_id" {
 value = azurerm_resource_group.rg.id
}

$cat terraform.tfvars
location = "westus"
```

2. Move backend to Azurerm backend (Azure Blob storage)
```
$ cat main.tf
terraform {
    backend "azurerm" {
        storage_account_name = "mytfbackend"
        container_name = "tfstate"
        key = "test.terraform.tfstate"
        access_key = "xxxxp5Nv8kTjdpYj9KwGIxeB+JkvmXKPLdpYjNY9/wE1pLM2RuOglxvuCA7RwLx7vdd2SFNCOCfIyyyyyyyyy=="
    }
}
```

3. Create Virtual Network
```
resource "azurerm_virtual_network" "example" {
  name                = "myVNet"
  location            = azurerm_resource_group.rg.location
  resource_group_name = azurerm_resource_group.rg.name
  address_space       = ["10.0.0.0/16"]

  subnet {
    name           = "subnet1"
    address_prefix = "10.0.1.0/24"
  }

  tags = {
    environment = "Production"
  }
}
```




## Setup environment 
WSL 2 (Windows Subsystem for Linux 2) installation is recommended. 
https://docs.microsoft.com/en-us/windows/wsl/wsl2-install

1. Ensure that you are running Windows 10 build 18917 or higher
C:\Users\azureuser>ver
Microsoft Windows [Version 10.0.18363.535]

If not, go to https://insider.windows.com/en-us/ and click "REGISTER TO GET THE PREVIEW"

Then, Go to Settings > Update & Security > Windows Insider Program and click Get Started to access the latest build.
Select the 'Fast' ring or the 'Slow' ring.

C:\Users\azureuser>ver
Microsoft Windows [Version 10.0.19041.1]

dism.exe /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux /all /norestart
dism.exe /online /enable-feature /featurename:VirtualMachinePlatform /all /norestart

On web browser, go to https://aka.ms/wslstore and install "Ubuntu"

```
Installing, this may take a few minutes...
Please create a default UNIX user account. The username does not need to match your Windows username.
For more information visit: https://aka.ms/wslusers
Enter new UNIX username: azureuser
Enter new UNIX password:
Retype new UNIX password:
passwd: password updated successfully
Installation successful!
To run a command as administrator (user "root"), use "sudo <command>".
See "man sudo_root" for details.

azureuser@win10:~$
```
Set a distro to be backed by WSL 2 using the command line
```
C:\WINDOWS\system32>wsl --set-version ubuntu 2
Conversion in progress, this may take a few minutes...
For information on key differences with WSL 2 please visit https://aka.ms/wsl2
Please enable the Virtual Machine Platform Windows feature and ensure virtualization is enabled in the BIOS.
For information please visit https://aka.ms/wsl2-install
```
Install Windows Terminal (Preview)
https://www.microsoft.com/en-us/p/windows-terminal-preview/9n0dx20hk701



1. Create service principal to running terraform
```
$ export SUBSCRIPTION_ID="xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
$ az account set --subscription="${SUBSCRIPTION_ID}"
$ az ad sp create-for-rbac --role="Contributor" --scopes="/subscriptions/${SUBSCRIPTION_ID}"
Creating a role assignment under the scope of "/subscriptions/87b7ed75-7074-41d6-9b53-3bf8894138bb"
{
  "appId": "aaaaaa-aaaa-aaaa-aaaa-aaaaaaaaaaaa",
  "displayName": "azure-cli-2019-12-13-08-53-51",
  "name": "http://azure-cli-2019-12-13-08-53-51",
  "password": "bbbbbb-bbbb-bbbb-bbbb-bbbbbbbbbbb",
  "tenant": "cccccccc-cccc-cccc-cccc-cccccccccccc"
}

$ export ARM_SUBSCRIPTION_ID=your_subscription_id
$ export ARM_CLIENT_ID=your_appId
$ export ARM_CLIENT_SECRET=your_password
$ export ARM_TENANT_ID=your_tenant_id

$ cat test.tf
provider "azurerm" {
}
resource "azurerm_resource_group" "rg" {
        name = "testResourceGroup"
        location = "westus"
}

$ terraform init

$ terraform plan

$ terraform apply
```


1. Prepare Bash shell environment (Windows Terminal with Windows Subsystem for Linux is recommended.)

2. Install terraform (v0.12.12), jq, git, docker
```
$ sudo apt-get update -y
$ wget https://releases.hashicorp.com/terraform/0.12.12/terraform_0.12.12_linux_amd64.zip
$ sudo unzip ./terraform_0.12.12_linux_amd64.zip -d /usr/local/bin/
$ terraform -v

$ sudo apt-get install -y jq
$ sudo apt install git-all
```
For docker daemon installation, refer https://docs.docker.com/install/linux/docker-ce/ubuntu/

3. Deploy Landing Zone Level0
```
# Install rover
$ wget -O - --no-cache https://raw.githubusercontent.com/aztfmod/rover/master/install.sh | bash
$ cd ~/git/github.com/aztfmod/rover

# clone repositories to local directory
~/git/github.com/aztfmod/rover$ make setup_dev_githttp


# Modify ../level0/launchpad_opensource/main.tf
locals {
  tfstate-blob-name = var.tf_name
  prefix      = "ebkr"
}

# Modify ../level0/launchpad_opensource/storage.tf
resource "azurerm_resource_group" "rg" {
  name     = "${local.prefix}-terraform-state"
  #name     = "${random_string.prefix.result}-terraform-state"

# Modify ../level0/launchpad_opensource/sp_tfstate.tf
resource "azuread_application" "tfstate" {
  name                       = "${local.prefix}tfstate"
  #name                       = "${random_string.prefix.result}tfstate"
}


# Modify ../level0/launchpad_opensource/sp_azure_devops.tf
resource "azuread_application" "devops" {
  name                       = "${local.prefix}devops"
  #name                       = "${random_string.prefix.result}devops"
}

# Modify ../level0/launchpad_opensource/keyvault.tf
resource "random_string" "kv_name" {
  length  = 23 - length(local.prefix)
  #length  = 23 - length(random_string.prefix.result)
  special = false
  upper   = false
  number  = true
}

locals {
    kv_name = "${local.prefix}${random_string.kv_middle.result}${random_string.kv_name.result}"
    #kv_name = "${random_string.prefix.result}${random_string.kv_middle.result}${random_string.kv_name.result}"
}

# Modify ../level0/launchpad_opensource/keyvault_secrets.tf
resource "azurerm_key_vault_secret" "tfstate_prefix" {
    provider      = azurerm.sp_tfstate

    name         = "tfstate-prefix"
    value        = local.prefix
    #value        = random_string.prefix.result
    key_vault_id = azurerm_key_vault.tfstate.id
}

# Modify ../level0/launchpad_opensource/sp_tfstate.tf
resource "azurerm_user_assigned_identity" "tfstate" {
  resource_group_name = azurerm_resource_group.rg.name
  location            = azurerm_resource_group.rg.location

  name = "${local.prefix}tfstate_msi"
  #name = "${random_string.prefix.result}tfstate_msi"
}

# make sure pre-requisites packages are installed
 - jq
 - docker
 - git
 
# start dockerd
$ sudo service docker start

# Load the rover with landing zones
$ make local

# login to Azure subscription
$ ./rover.sh login [subscription_guid] [tenantname.onmicrosoft.com or tenant_guid]

# confirm currently logged-in subscription
$ az account show

# install launchpad (level0) using rover
$./rover.sh

var.location
  Azure region to deploy the launchpad in the form or 'southeastasia' or 'westeurope'

  Enter a value: koreacentral

var.tf_name
  Name of the terraform state in the blob storage

  Enter a value: level0_launchpad.tfstate


# Or manually install launchpad (level0) without rover

$ cd level0/
$ 
$ cp level0/ level0_launchpad

  



$ git clone https://github.com/aztfmod/level0.git

```

4. Clone ebay Korea Landing Zones from repo

```
$ git clone https://github.com/hyundonk/ebkr-landing-zones.git
$ cd ebkr-landing-zones/
```

5. Deploy tranquility landing zone (level1)











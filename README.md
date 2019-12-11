# Azure Terraform Landing Zone Tutorial

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











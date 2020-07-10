# Ansible--Azure-keyvault

# 1- Providing Credentials to Azure Modules
# Azure PowerShell

# 2. Create an Azure Service Principal

$credentials = New-Object Microsoft.Azure.Commands.ActiveDirectory.PSADPasswordCredential `
-Property @{ StartDate=Get-Date; EndDate=Get-Date -Year 2024; Password='<PASSWORD>'};

$spSplat = @{
    DisplayName = 'sp-cs-ansible'
    PasswordCredential = $credentials
}

$sp = New-AzAdServicePrincipal @spSplat

Assign a Role to the Service Principal

$subId = (Get-AzSubscription -SubscriptionName 'NameOfSubscriptionHere').id

$roleAssignmentSplat = @{
    ObjectId = $sp.id
    RoleDefinitionName = 'Contributor'
    Scope = "/subscriptions/$subId"
}

New-AzRoleAssignment @roleAssignmentSplat

# 3. Create Azure credentials

$subscriptionId = (Get-AzSubscription -SubscriptionName 'NameOfSubscriptionHere').id

$servicePrincipalAppId = (Get-AzADServicePrincipal -DisplayName 'sp-cs-ansible').ApplicationId

$servicePrincipalPassword = 'Value Specified Previously'

$tenantId = (Get-AzSubscription -SubscriptionName 'NameOfSubscriptionHere').TenantId




# 4. Create a credentials file

mkdir ~/.azure
vi ~/.azure/credentials

4.1. Populate the required Ansible variables. Replace <Text> with actual values.

[default]
subscription_id=<subscription_id>
client_id=<security-principal-appid>
secret=<security-principal-password>
tenant=<security-principal-tenant>



# Option 2: Use Ansible Environment Variables

export AZURE_SUBSCRIPTION_ID=<subscription_id>
export AZURE_CLIENT_ID=<security-principal-appid>
export AZURE_SECRET=<security-principal-password>
export AZURE_TENANT=<security-principal-tenant>

# 5. Create a playbook file

vi playbook.yaml

example:

---
- hosts: localhost
  connection: local
  tasks:
    - name: Create resource group
      azure_rm_resourcegroup:
        name: rg-cs-ansible
        location: eastus
      register: rg
    - debug:
        var: rg
		

# 6. Run the playbook 

ansible-playbook playbook.yaml

Create azure keyvault

# Get Azure tenantId
az account show --subscription "MySubscriptionName" --query tenantId --output tsv

# Get objectId with AzCLI
az ad sp show --id <ApplicationID> --query objectId
# Get Azure tenantID with AzPowerShell
(Get-AzSubscription -SubscriptionName "MySubscriptionName").TenantId

# Get objectId with AzPowerShell
(Get-AzADServicePrincipal -ApplicationId <ApplicationID> ).id


# After run playbooks 1-4
# Add the Lookup Plugin

In order for you to add a lookup plugin to your Ansible code base, you first have to create a directory called lookup_plugins. This directory needs to be adjacent to your playbook. If you'd like to specify a different location for it, you can do that by modifying the ansible.cfg. With the lookup_plugins directory created you next need to create a file with the name of the lookup plugin which is azure_keyvault_secret.py. It is a Python script and requires the .py extension.

mkdir lookup_plugins

#PowerShell
Invoke-WebRequest `
-Uri 'https://raw.githubusercontent.com/Azure/azure_preview_modules/master/lookup_plugins/azure_keyvault_secret.py' `
-OutFile lookup_plugins/azure_keyvault_secret.py

#Curl
curl \
https://raw.githubusercontent.com/Azure/azure_preview_modules/master/lookup_plugins/azure_keyvault_secret.py \
-o lookup_plugins/azure_keyvault_secret.py


# Non-Managed Identity Lookups 

When using the azurekeyvaultsecret plugin with non-managed identities you have to provide additional information for the plugin to authenticate. In addition to the vault uri and vault secret, you also have to provide a service principal client id, service principal secret, and a tenant id.

# Managed Identity Lookups 

# Prerequisites

https://docs.microsoft.com/en-us/azure/key-vault/general/managed-identity

https://docs.microsoft.com/en-us/azure/active-directory/managed-identities-azure-resources/qs-configure-portal-windows-vm

https://docs.microsoft.com/en-us/azure/key-vault/general/managed-identity




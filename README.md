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

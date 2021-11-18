# azure-devops-ci-ansible-vpn-deployment
Automatically deploy both ends of a VPN tunnel using Ansible and Azure KeyVault.  At one end it deploys and Azure Virtual Network Gateway, and at the other it configures an ASA 5506 firewall.

The Azure DevOps Pieline runs on an Ubuntu Agent running on-prem, so it can access the ASA through its LAN interface. 

To logon to Azure Ansible uses credentials stored in the ~/.azure/credentials of the user running the Azure Pipelines Agent.

The credentials file has this format:

[default]
subscription_id=xxx
client_id=xxx
secret=xxx
tenant=xxx

The only time the service account secret is displayed is during the account creation with the command

az ad sp create-for-rbac --name service-account-name

After the service account is created to get a new password you can use the command its password with the command

az ad sp create-for-rbac --name service-account-name --query password -o tsv

To avoid writing usernames or passwords, playbook uses an Azure KeyVault to retrieve these secrets.  In addition to the secrets the KeyVault deployment also restricts access to a public IP and allows you to specify a user with read-only access and another with full access.   



[![Deploy to Azure](https://aka.ms/deploytoazurebutton)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FBetter-Computing-Consulting%2Fazure-devops-ci-ansible-vpn-deployment%2Fmaster%2FKeyVault.json)


The Azure DevOps Pieline runs on an Ubuntu Agent running on-prem, so it can access the ASA through its LAN interface,

# azure-devops-ci-ansible-vpn-deployment
Automatically deploy both ends of a VPN tunnel using Ansible and Azure KeyVault.  At one end it deploys and Azure Virtual Network Gateway, and at the other it configures an ASA 5506 firewall.

The Azure DevOps Pipeline runs on an Ubuntu Agent running on-prem, so it can access the ASA through its LAN interface. 

To logon to Azure Ansible uses credentials stored in the __~/.azure/credentials__ file of the user running the Azure Pipelines Agent.

The credentials file has this format:

```Text
[default]
subscription_id=xxx
client_id=xxx
secret=xxx
tenant=xxx
```

The only time the service account secret is displayed is during the account creation with the command

`az ad sp create-for-rbac --name service-account-name-here`

After the service account is created to get a new password use the command

`az ad sp create-for-rbac --name service-account-name --query password -o tsv`

Once you have your service account password, you can generate the file with these commands:

```Shell

mkdir ~/.azure
echo "[default]" > ~/.azure/credentials
echo "subscription_id=$(az account show --query '{subscriptionid:id}' -o tsv)" >> ~/.azure/credentials
echo "client_id=$(az ad sp list --display-name ansible-service-name-here --query '{clientId:[0].appId}' -o tsv)" >> ~/.azure/credentials
echo "secret=service-principal-password-here" >> ~/.azure/credentials
echo "tenant=$(az account show --query '{tenantId:tenantId}' -o tsv)" >> ~/.azure/credentials

```

To avoid writing usernames, passwords or the VPN shared key, the playbook retrieves these secrets from an Azure KeyVault.  The KeyVault used in the project also restricts access to one public IP and specifies a user with read-only access and another with full access. 

You can use the button below to deploy a KeyVault with the same properties as the one the playbook uses.  The KeyVault deployment proceess requires the objectid of each user.

To get the objectid for the read-only service principal run this command:

`az ad sp list --display-name ansible-service-name-here | grep objectId`

To get the objectid for the admin user run this command:

`az ad user list --upn admin-user@your-domain-here_com | grep objectId`

To get public ip of the server running the Azure Pipelines Agent run this command:

`dig +short myip.opendns.com @resolver1.opendns.com`

[![Deploy to Azure](https://aka.ms/deploytoazurebutton)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FBetter-Computing-Consulting%2Fazure-devops-ci-ansible-vpn-deployment%2Fmaster%2FKeyVault.json)


You will need to update the playbook variable __keyvaulturl__ with the url of your own KeyVault.

The repository includes a playbook __vpnrm.yml__ to undo all the changes made to the firewall and delete the Azure Resource Group, Virtual Network Gateway, and all other resources the project created.

There is a blog that explains with details some aspects of the playblooks, including how to setup the self-hosten Azure Pipelines agent.

https://bcc.bz/post/azure-devops-ci-ansible-vpn-deployment-between-virtual-network-gateway-and-cisco-asa

I have also posted a video that shows the deployment of the KeyVault and execution of the playbooks.

https://youtu.be/1uSOQlm5KsQ



:smiley:

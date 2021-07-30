# ansible
Ansible code for automating security group configuration changes that are implmented in the infrastructure created in the terraform repository.  

### Prerequisites ###

* Ansible installed, see https://docs.ansible.com/ansible/latest/installation_guide/index.html   

## Connect to Azure with Ansible ##
   
### Prerequisites ###
* An Azure subscription. Create a free account at https://azure.microsoft.com/en-us/free/
* Ansible installed for Azure, see https://docs.microsoft.com/en-us/azure/developer/ansible/install-on-linux-vm?tabs=azure-cli
* Azure PowerShell module installed, see https://docs.microsoft.com/en-us/powershell/azure/install-az-ps?view=azps-6.2.1    
    
### Prep Steps ###
1. Create an Azure Service Principal using Powershell   
      ```
      $credentials = New-Object Microsoft.Azure.Commands.ActiveDirectory.PSADPasswordCredential `
      -Property @{ StartDate=Get-Date; EndDate=Get-Date -Year 2024; Password='<PASSWORD>'};
      
      $spSplat = @{
          DisplayName = 'sp-ansible'
          PasswordCredential = $credentials
      }
      
      $sp = New-AzAdServicePrincipal @spSplat
      ```
2. Use PowerShell to get the required Azure information    
      ```
      @{
            subscriptionId = (Get-AzContext).Subscription.Id
            clientid = (Get-AzADServicePrincipal -DisplayName 'sp-ansible').ApplicationId.Guid
            tenantid = (Get-AzContext).Tenant.Id
      }
      ```    
3. Assign a Role to the Service Principal      
      ```
      $subId = <SUBSCRIPTION ID>

      $roleAssignmentSplat = @{
         ObjectId = $sp.id
         RoleDefinitionName = 'Contributor'
         Scope = "/subscriptions/$subId"
      }
      
      New-AzRoleAssignment @roleAssignmentSplat
      ```
3. Create Azure credentials file   
      ```
      mkdir ~/.azure
      vi ~/.azure/credentials
      ```    
Populate the required Ansible variables    
      ```
      [default]
      subscription_id=<subscription_id>
      client_id=<security-principal-appid>
      secret=<security-principal-password>
      tenant=<security-principal-tenant>
      ```
4. Now ready to run ansible playbooks    

- - -    
## Connect to AWS with Ansible ##
   
### Prerequisites ###
* An AWS subscription. Create a free account at https://aws.amazon.com/free/
* boto3 Python module installed on your control machine    
      ```pip3 install boto3```    
* AWS CLI installed. See https://docs.aws.amazon.com/cli/latest/userguide/install-cliv2-linux.html#cliv2-linux-install    
* Configure AWS CLI. Quick configuration with ```aws configure```

### Prep Steps ###
1. To use authentication with AWS-related modules, you need to specify your access and secret keys as either module arguments or environmental (ENV) variables. To store as environment variables:  

      ```
      export AWS_ACCESS_KEY_ID='AK123' 
      export AWS_SECRET_ACCESS_KEY='abc123'
      ```  
  
2. Install amazon.aws collection    
      ```ansible-galaxy collection install amazon.aws```    

3. Set region for outfield profile    
      ```aws configure set region eu-central-1 --profile outfield```    

4. Set output format for outfield profile    
      ```aws configure set output yaml --profile outfield```    

5. Set Profile    
      ```export AWS_PROFILE=outfield```    

6. Check that outfield profile is configured    
      ```aws configure list```    

7. Now ready to run ansible playbooks    

- - -  
## Run Ansible Playbooks ##

1. Clone or download the playbook files in this repository
2. Run Asible playbooks    
      ```ansible-playbook azure-sg-playbook-all.yml```    
      or    
      ```ansible-playbook aws-sg-playbook-all.yml```  

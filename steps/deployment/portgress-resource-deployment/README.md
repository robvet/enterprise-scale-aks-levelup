# Bicep option: Deploy Postgres database using Bicep

1. Make changes in the parameters file [here](https://github.com/mosabami/enterprise-scale-aks-levelup/blob/smartbrain/steps/deployment/portgress-resource-deployment/parameters.postgres.json) accordingly.

2. Deploy bicep using the modified parameters file

   ```bash
   az deployment sub create -n "ESLZ-SPOKE" -l "CentralUS" -f postgres.bicep -p parameters-postgres.json
   ```

If you get an error about changes to the configuration, go with the `-reconfigure` flag option.

:arrow_forward:[Return to deployment steps](../README.md)

# OPTIONAL: Deploy Postgres database using Terraform

If you happen to be using terraform, you can use instruction below instead.

1. Copy [this file](https://github.com/mosabami/enterprise-scale-aks-levelup/blob/main/steps/deployment/portgress-resource-deployment/postgres.tf) into [this folder](https://github.com/Azure/Enterprise-Scale-for-AKS/tree/main/Scenarios/AKS-Secure-Baseline-PrivateCluster/Terraform/07-AKS-cluster) in your cloned Enterprise-Scale-for-AKS repo you used to deploy the Azure resources (you may need to clone the enterprise-scale-aks-levelup repo into your local computer to do this)

2. Make changes to the variables as required

3. Terraform apply again in that directory you copied it into

   ```bash
   terraform init -backend-config="resource_group_name=$TFSTATE_RG" -backend-config="storage_account_name=$STORAGEACCOUNTNAME" -backend-config="container_name=$CONTAINERNAME"
   ```

   ```
   terraform apply
   ```

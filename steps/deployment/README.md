## Deploy Postgres and create images

Now that you have completed creating the main Azure resources, the next step is to create postgres database and setup the virtual machine you will use to deploy the workload.

1. Deploy postgress database into your landing zone using the instructions [here](./portgress-resource-deployment/README.md). Make the password "12345ASDf" and username postgres

2. Follow the instructions [here](https://github.com/Azure/Enterprise-Scale-for-AKS/blob/main/Scenarios/AKS-Secure-Baseline-PrivateCluster/Bicep/07-workload.md#provide-yourself-access-to-create-secrets-in-your-key-vault) to provide yourself access to create secrets in Key vault (don't deploy the workload)

3. SSH into the jumpbox vm you created, using Visual Studio code using the instructions [here](https://github.com/Azure/Enterprise-Scale-for-AKS/blob/main/Scenarios/AKS-Secure-Baseline-PrivateCluster/Terraform/08-workload.md#option-1-connecting-into-the-server-dev-linux-vm-using-ssh-and-vs-code). All the steps below need to be done from the jumpbox because your AKS, Key vault and ACR resources can only be accessed from within the same virtual network. We are using visual studio code because it is easy to manipulate files in your virtual machine using it.

4. Install the required tools in your new virtual machine using the instructions [here](./portgress-resource-deployment/setupVM.md)

5. Clone this repository and cd to the folder

   ```bash
   git clone https://github.com/mosabami/enterprise-scale-aks-levelup
   cd enterprise-scale-aks-levelup
   ```

6. Copy the content of the steps/starting-point folder in your cloned repo in the vm into the steps/deployment folder.

7. Get your ACR name and replace the placeholder below with your ACR name

   ```bash
   ACR_NAME=$(az deployment sub show -n "ESLZ-AKS-Supporting" --query properties.outputs.acrName.value -o tsv)
   KV_NAME=$(az deployment sub show -n "ESLZ-AKS-Supporting" --query properties.outputs.keyvaultName.value -o tsv)

   ```

8. Switch to the server folder (steps/deployment/server) and open the index.js file. (Make sure you have copied the content of steps/starting-point to steps/deployment if you havent yet before entering the command below)

   ```bash
   cd steps/deployment/smartbrainapi
   code server.js
   ```

9. Add "/api" to the beginning of the paths in line 58,60, 62, 64, 73, 81. This is because AGIC does not have the nginx ingress controller feature that allows you to rewrite target path. More details on this later. Save it.

10. Build the image for the server into the container registry you just created using the tag "v2". the original version image did not have the "/api" at the beginning of the paths.

    ```bash
    sudo az acr login -n $ACR_NAME
    ```

11. Log into the ACR using user name and password you can find in the portal (find the ACR in the spoke resource group) if it does't let you log in using AAD.

    ```bash
    sudo docker build -t $ACR_NAME.azurecr.io/smartbrainapi:v2 .
    ```

12. Build image of the calculator microservice

    ```bash
    cd ../smartbrainml
    code src/app.py
    ```

13. Add "/worker" to the beginning of the path in line 64.

    ```bash
    sudo docker build -t $ACR_NAME.azurecr.io/smartbrainworker:v2 .
    ```

14. Build the image of the front end microservice

    ```bash
    cd ../smartbrainclient
    sudo docker build -t $ACR_NAME.azurecr.io/smartbrainclient .
    ```

15. Push the built images into your private container registry

    ```bash
    sudo docker push $ACR_NAME.azurecr.io/smartbrainworker:v2
    sudo docker push $ACR_NAME.azurecr.io/smartbrainapi:v2
    sudo docker push $ACR_NAME.azurecr.io/smartbrainclient
    ```

16. Create the postgres database password as a secret in keyvault

    ```bash
    az keyvault secret set --name pgpassword --vault-name $KV_NAME --value "12345ASDf"
    ```

17. Copy the redis image from Docker hub to your ACR

    ```bash
    az acr import \
      --name $ACR_NAME \
      --source docker.io/library/redis \
      --image redis
    ```

18. Ensure you have all 4 images that will be required for the rest of this workshop

    ```bash
    az acr repository list -n $ACR_NAME
    ```

    Your output should look like this:

    ```output
    [
      "redis",
      "smartbrainapi",
      "smartbrainclient",
      "smartbrainworker"
    ]
    ```

:arrow_forward:[Deploy workload](./Deploy-workload.md)

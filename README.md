This README provides a step-by-step guide to managing Azure Kubernetes Service (AKS) clusters using Terraform and Azure CLI within the Azure Cloud Shell environment. Follow these instructions to deploy and manage your Kubernetes cluster on Azure.

# Deploying AKS with Terraform

This guide provides instructions on deploying Azure Kubernetes Service (AKS) using Terraform. It includes steps for creating an Azure Service Principal, initializing Terraform, planning and applying the Terraform configuration, and cleaning up resources.

## Prerequisites

- Azure CLI
- Terraform

## Steps

### 1. Create Azure Service Principal

Create an Azure Service Principal for Terraform with the Contributor role. Replace `<service_principal_name>` and `<subscription_id>` with your desired service principal name and Azure subscription ID.

```bash
az ad sp create-for-rbac --name <service_principal_name> --role Contributor --scopes /subscriptions/<subscription_id>
```

Example:

```bash
az ad sp create-for-rbac --name "mpn-aks" --role Contributor --scopes /subscriptions/<subscription_id>/
```

### 2. Initialize Terraform

Initialize the Terraform environment to download necessary providers and set up the workspace.

```bash
terraform init -upgrade
```

### 3. Plan Deployment

Generate an execution plan for Terraform. This step allows you to review the actions Terraform will perform before making any changes to your Azure resources.

```bash
terraform plan -out main.tfplan
```

### 4. Apply Deployment

Apply the Terraform plan to deploy the AKS cluster. This step will create or update resources based on the Terraform configuration.

```bash
terraform apply main.tfplan
```

### 5. Optional: Destroy Resources

To clean up and remove the resources created by Terraform, you can plan and apply a destruction plan.

Generate the destruction plan:

```bash
terraform plan -destroy -out main.destroy.tfplan
```

Apply the destruction plan:

```bash
terraform apply main.destroy.tfplan
```

### 6. Delete Service Principal

After you're done with the deployment and have destroyed the resources (if desired), you can remove the Azure Service Principal to clean up your Azure AD.

Retrieve the Service Principal ID:

```bash
sp=$(terraform output -raw sp)
```

Delete the Service Principal:

```bash
az ad sp delete --id $sp
```

# Application Deployment


### Prerequisites

- An Azure account with an active subscription.
- Terraform installed in your Azure Cloud Shell.
- Azure CLI installed and configured in your Azure Cloud Shell.

### Steps

1. **Get the Azure Resource Group Name**

   Extract the name of the resource group created by Terraform.

   ```console
   resource_group_name=$(terraform output -raw resource_group_name)
   ```

2. **List Kubernetes Clusters**

   Display the name of your Kubernetes cluster(s) in the specified resource group.

   ```azurecli
   az aks list \
     --resource-group $resource_group_name \
     --query "[].{\"K8s cluster name\":name}" \
     --output table
   ```

3. **Retrieve Kubernetes Configuration**

   Get the Kubernetes cluster configuration from the Terraform state and save it to a file for `kubectl` use.

   ```console
   echo "$(terraform output kube_config)" > ./azurek8s
   ```

4. **Verify Configuration File**

   Ensure that the `azurek8s` file does not contain ASCII EOT characters.

   ```console
   cat ./azurek8s
   ```

   - **Important:** If you see `<< EOT` at the beginning and `EOT` at the end of the file, remove these characters. Presence of these characters may lead to an error message: `error: error loading config file "./azurek8s": yaml: line 2: mapping values are not allowed in this context`.

5. **Set Kubernetes Configuration Environment Variable**

   Set an environment variable so that `kubectl` can use the correct configuration file.

   ```console
   export KUBECONFIG=./azurek8s
   ```

6. **Verify Cluster Health**

   Check the health and status of your Kubernetes cluster nodes.

   ```console
   kubectl get nodes
   ```

### Key Points

- **Azure Resource Group:** The container that holds related resources for an Azure solution.
- **Terraform:** An open-source tool for building, changing, and versioning infrastructure safely and efficiently.
- **AKS (Azure Kubernetes Service):** A managed container orchestration service based on Kubernetes.
- **kubectl:** A command-line tool for interacting with Kubernetes clusters.

By following these steps, you will have deployed a Kubernetes cluster using Terraform and managed its configuration using Azure CLI and kubectl within the Azure Cloud Shell environment.





This README guides you through testing an application deployed on Kubernetes, specifically focusing on verifying pod status and accessing the application via a public IP address.

### Prerequisites

- An application deployed on Kubernetes.
- `kubectl` command-line tool installed and configured to communicate with your Kubernetes cluster.



### To apply code
kubectl apply -f aks-store-quickstart.yaml



### Steps

1. **Check Pods Status**

   Before accessing your application, ensure that all deployed pods are in the `Running` state. This verification ensures that your application components are functioning correctly.

   ```console
   kubectl get pods
   ```

   Wait for all pods to show the status `Running`. If any pods are not in the `Running` state, investigate and resolve any issues before proceeding.

2. **Monitor Service for Public IP**

   Your application's front end is exposed to the internet through a Kubernetes service. To access the application, you need the service's public IP address. This IP address assignment can take a few minutes.

   To monitor the assignment of the public IP address to your `store-front` service, use the following command:

   ```azurecli
   kubectl get service store-front --watch
   ```

   Initially, the `EXTERNAL-IP` for the `store-front` service will display as `<pending>`.

   **Example Initial Output:**
   ```
   NAME          TYPE           CLUSTER-IP    EXTERNAL-IP   PORT(S)        AGE
   store-front   LoadBalancer   10.0.100.10   <pending>     80:30025/TCP   4h4m
   ```

3. **Obtain Public IP Address**

   Once the `EXTERNAL-IP` changes from `<pending>` to an actual public IP address, you can stop the watch process by pressing `CTRL-C`.

   **Example Output with Valid Public IP:**
   ```
   NAME          TYPE           CLUSTER-IP    EXTERNAL-IP    PORT(S)        AGE
   store-front   LoadBalancer   10.0.100.10   20.62.159.19   80:30025/TCP   4h5m
   ```

4. **Access the Application**

   With the public IP address now assigned to your service, open a web browser and navigate to the external IP address to view the Azure Store app.

   ```
   http://<EXTERNAL-IP>
   ```

   Replace `<EXTERNAL-IP>` with the actual public IP address of your service.

### Key Points

- Ensure all Kubernetes pods are running smoothly before attempting to access the application.
- Monitoring the service for the public IP address assignment is crucial for accessing the application externally.
- Accessing the application through its public IP address allows you to interact with it as end-users would.

By following these steps, you can effectively test and verify that your application is successfully deployed and accessible on Kubernetes.
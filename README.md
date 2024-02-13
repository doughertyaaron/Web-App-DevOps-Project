# Web-App-DevOps-Project

Welcome to the Web App DevOps Project repo! This application allows you to efficiently manage and track orders for a potential business. It provides an intuitive user interface for viewing existing orders and adding new ones.

## Table of Contents

- [Features](#features)
- [Getting Started](#getting-started)
- [Technology Stack](#technology-stack)
- [Contributors](#contributors)
- [License](#license)
- [Containerisation](#containerisation)
- [IAC](#iac)
- [Kubernetes deployment to AKS](#kubernetes-deployment-to-aks)
- [CI/CD Pipeline (Azure DevOps)](#cicd-pipeline-azure-devops)
- [AKS Cluster Monitoring](#aks-cluster-monitoring)
- [Azure Key Vault](#azure-key-vault)
- [End-to-End Testing](#end-to-end-testing)

## Features

- **Order List:** View a comprehensive list of orders including details like date UUID, user ID, card number, store code, product code, product quantity, order date, and shipping date.
  
![Screenshot 2023-08-31 at 15 48 48](https://github.com/maya-a-iuga/Web-App-DevOps-Project/assets/104773240/3a3bae88-9224-4755-bf62-567beb7bf692)

- **Pagination:** Easily navigate through multiple pages of orders using the built-in pagination feature.
  
![Screenshot 2023-08-31 at 15 49 08](https://github.com/maya-a-iuga/Web-App-DevOps-Project/assets/104773240/d92a045d-b568-4695-b2b9-986874b4ed5a)

- **Add New Order:** Fill out a user-friendly form to add new orders to the system with necessary information.
  
![Screenshot 2023-08-31 at 15 49 26](https://github.com/maya-a-iuga/Web-App-DevOps-Project/assets/104773240/83236d79-6212-4fc3-afa3-3cee88354b1a)

- **Data Validation:** Ensure data accuracy and completeness with required fields, date restrictions, and card number validation.

- **Delivery Date:** Allows the specification of a delivery date when adding an order. This ensures that when an order is created, a delivery date can be tagged to the order.

## Getting Started

### Prerequisites

For the application to succesfully run, you need to install the following packages:

- flask (version 2.2.2)
- pyodbc (version 4.0.39)
- SQLAlchemy (version 2.0.21)
- werkzeug (version 2.2.3)

### Usage

To run the application, you simply need to run the `app.py` script in this repository. Once the application starts you should be able to access it locally at `http://127.0.0.1:5001`. Here you will be meet with the following two pages:

1. **Order List Page:** Navigate to the "Order List" page to view all existing orders. Use the pagination controls to navigate between pages.

2. **Add New Order Page:** Click on the "Add New Order" tab to access the order form. Complete all required fields and ensure that your entries meet the specified criteria.

## Technology Stack

- **Backend:** Flask is used to build the backend of the application, handling routing, data processing, and interactions with the database.

- **Frontend:** The user interface is designed using HTML, CSS, and JavaScript to ensure a smooth and intuitive user experience.

- **Database:** The application employs an Azure SQL Database as its database system to store order-related data.

## Contributors 

- [Maya Iuga]([https://github.com/yourusername](https://github.com/maya-a-iuga))
- [Aaron Dougherty]([https://github.com/yourusername](https://github.com/doughertyaaron))

## License

This project is licensed under the MIT License. For more details, refer to the [LICENSE](LICENSE) file.

## Containerisation
This app has been developed to be run from a container using Docker. This allows the application to ported and run from any divice that supports docker and also allows for deployment on clooud services and Kubernetes clusters.

### Containerization Process
This web application is a containerised application - designed to be hosted in and run from a docker container. The 'dockerfile' in the repository contains the outline of how to set up and run a container instance of this application. 

This file is specified to run in a MacOS environment using an M1/M2/M3 chip. The WORKDIR defines which folder will be open within the container on start up, while the COPY command defines exactly what will be copied from the repo into the container (and where it will be stored). This can be edited to allow the application to run on different computers and with different base images, however, this application does need to be run using Python.

The RUN commands define exactly what needs to be run and installed on container start up, while EXPORT defines which port to connect to (this must match the port provided in your docker run command once the container has been created and the ports specified in your code - in this instance, it is set to 5001 by default). The RUN commands should not be edited or changed, however, the port can be adjusted as needed. Any adjustments made to the port must be followed up byb updating the port named in script.js and app.py.

Finally, the CMD command specifies what command line argument to run once the container has completed set up, and will initiate the web application from within the container. This should not be edited, except for the prupose of adjusting the file path, if the WORKDIR is changed.

To build a local image instance of this container locally, simply run `docker build -t <name of the image> .` in the comand line, where the full stop references the file path location of the dockerfile.

Once built, to spin up a container instance of the image and run the application, run `docker run -p 5000:5000 <name of the image>` and then navigate to *http://127.0.0.1:5001* to access the web application.

To tag and push the docker image to Docker Hub (where the image can be saved and version controlled by Docker), run `docker tag <name of the image> <docker-hub-username>/<image-name>:<tag>` and then `docker push <docker-hub-username>/<image-name>:<tag>`.

## IAC

Terraform is used to provide infrastructure as code (IAC) to define and provision the necessary cloud resources on Azure to host the web application on a Kubernetes cluster. This Terraform project is split into two modules, one for defining and managing the cluster and the other for the cluster's networking. The file structure can be seen in detail below.

.
├── aks-terraform               # main project folder
    ├── main.tf                 # defines the resources for the module
    ├── outputs.tf          §   # defines the outputs from the provisioning of resources
    ├── variables.tf            # user defined variables to be used in main.tf
    ├── aks-cluster-module
       ├── main.tf
       ├── outputs.tf
       └── variables.tf
    └── networking-module
        ├── main.tf
        ├── outputs.tf
        └── variables.tf

### Networking module
This module defines the cloud network/platform used to host the Kubernetes cluster and subsequent web application on Azure's cloud. for this project we have define the following networking resources:

* **Resource group:** Logical grouping of Azure resources for logging, monitoring, billing and group analytics and control
* **Virtual network:** Cloud-based IP network or platform for the application to run in
* **Subnet:** Subdivision of the IP network for separation of resources whilst maintaining easy communication when needed. Our application defines one for the AKS control pane and the other for worker nodes
* **NSG (Network Security group):** Security boundary for inbound and outbound connection rules between different Azurew resources and different requests and communication attempts from outside the apoplication or network
* **Inbound rules:** Network security rules limiting network traffic to only the minimum our application needs for security purposes. This application holds two inbound rule - meaning only inbound traffic of certain specified types can enter the network. These are SSH and communication ot the kube-apiserver

### AKS clustering module
This module defines the AKS cluster that will be launched to host the wedb application. For this project, the key elements are:

* **AKS cluster:**  The group of machines (nodes) working together to run our containerized application on Azure
* **Node pool:** The default number of nodes in a cluster
* **Service principal:** The authentication required for the network linked with the AKS cluster

### Setting up permissions
When using terraform to create the necessary resources, we need to first log in to the Azure CLI using:
`az login --tenant <azure_tenant> --allow-no-subscriptions`. This will provide us with our SubscriptionId and TenantId required to set up our azurerm provider. The JSON response we get back in our command line will list these under "id" and "tenantId" respectively. Running `az account list --output table` Will show all accounts and their relevant IDs to allow you to check you are using the correct IDs. You may also run `az account set --subscription <subscriptionId>` if you wish to ensure your Azure CLI is referencing the correct account going forward.

We can then run az ad sp create-for-rbac -`-name <choose_name> --role contributor --scopes /subscriptions/{your-subscription-id}` to create a new app registration. This will provide an Application (client) ID and Client Secret required to create the resource group and associated resources using Terraform. After this has been created, you will receive another JSON response to you command line, inclusing the "appId" (Client ID), "password" (Client Secret), "displayName" (the name of your registration) and "tenant" (your tenant ID - you should already have this).

Finally, you can add the following variables to your main.tf file under your azurerm provider block: client_id, client_secret, subscription_id and tenant_id. These should be taken from defined variables in your variables.tf file and can be automatically brought in when running terraform apply via your .zshrc file. This file should include set terraform variables in the following format: `export TF_VAR_variable_name_in_terraform_variables_file="<azure ID or secret>"`. Ensure to run `source .zshrc` after editing the file.

### Creating the resources with Terraform
Once permissions have been created and added to your .zshrc file, you can run `terraform init` in each module and main foolder in terraform to initialise the modules and main file path. Then  run `terraform plan` and finally `terraform apply` to provision the required resources.

Now retrieve the kubeconfig file via: `az aks get-credentials --resource-group <your-resource-group> --name <your-aks-cluster-name>` in the command line. This will return the message `Merged "<aks cluster name>" as current context in /Users/<user name>/.kube/config`. Running `kubectl config get-contexts` will confirm the existence of the cluster locally.

## Kubernetes deployment to AKS
To create and deploy the kubernetes cluster to your provisioned AKS resources on Azure, run `kubectl apply -f <absolute file path to manifest.yaml file>`. To check this has worked and view all pods, run `kubectl get pods -w`. Finally, to access the webapplication, port forward into the application using: `kubectl port-forward flask-app-deployment-67f878f7db-g2z76 5001:5001` and going to this address in your web browser: http://127.0.0.1:5000

## CI/CD Pipeline (Azure DevOps)
A CI/CD pipeline has been set up on Azure DevOps. This is stored under the project "APD DevOps Pipeline" This includes the creation of a pipeline (defined in `azure-pipelines.yaml`) and three connections to:
* GitHub (allows certain tasks to be triggered based on activities in GitHub)
* Docker Hub (automatically builds the docker image from the dockerfile and pushes to Docker Hub on new pushes to main)
* Kubernetes - AKS (automatically deploys kubernetes cluster from the deployment manifest file on new pushes to main)

This pipeline has been validate on Azure DevOps. Further, the AKS cluster has been accessed to ensure that it is working correctly and the dockerfile has been downloaded from DockerHub and ran to ensure that this is updated correctly.

## AKS Cluster Monitoring
AKS cluster monitoring is handled within Azure AKS through the creation of dashboards, logs and alarms. These are set up following enabling Container Insights.

The required process to achieve this on any aks cluster is as follows:

1. Run `az aks update -g {resource-group-name} -n {aks-cluster-name} --enable-managed-identity` using the CLI to enable IAM on the cluster
2. Under your subscription associated with your cluster, add IAM permissions under Acess Control  to make changes to Insights and monitoring. This is achieved via 'Add role assignment' for three roles (see below). Ensure to assign the role to you Service Principal which set up the cluster.
    * Monitoring Metrics Publisher (Grants permission to publish monitoring metrics to Azure Monitor. This is important for applications and services that need to push metrics to Azure Monitor)
    * Monitoring Contributor (Grants broad permissions for monitoring and managing monitoring resources in Azure, including permissions to read and write monitoring settings, access monitoring data and manage monitoring resources)
    * Log Analytics Contributor (Grants permissions to read and write access to Log Analytics workspaces. Includes permissions to query and analyze log data stored in those workspaces)
3. In your cluster in AKS, under 'Monitoring' -> 'Insights' -> 'Configure Monitoring', select 'Enable container logs'

A dashboard showing 4 charts was created by creating, configuring and saving the relevant charts in Metrics Explorer. These charts include:

* Average Node CPU Usage: This chart allows you to track the CPU usage of your AKS cluster's nodes. Monitoring CPU usage helps ensure efficient resource allocation and detect potential performance issues.
* Average Pod Count: This chart displays the average number of pods running in your AKS cluster. It's a key metric for evaluating the cluster's capacity and workload distribution.
* Used Disk Percentage: Monitoring disk usage is critical to prevent storage-related issues. This chart helps you track how much disk space is being utilized.
* Bytes Read and Written per Second: Monitoring data I/O is crucial for identifying potential performance bottlenecks. This chart provides insights into data transfer rates.

In the cluster, under 'Logs', the following queries were run and saved under the Containers category and under the same label:

* Average Node CPU Usage Percentage per Minute: This configuration captures data on node-level usage at a granular level, with logs recorded per minute
* Average Node Memory Usage Percentage per Minute: Similar to CPU usage, tracking memory usage at node level allows you to detect memory-related performance concerns and efficiently allocate resources
* Pods Counts with Phase: This log configuration provides information on the count of pods with different phases, such as Pending, Running, or Terminating. It offers insights into pod lifecycle management and helps ensure the cluster's workload is appropriately distributed.
* Find Warning Value in Container Logs: By configuring Log Analytics to search for warning values in container logs, you proactively detect issues or errors within your containers, allowing for prompt troubleshooting and issues resolution
* Monitoring Kubernetes Events: Monitoring Kubernetes events, such as pod scheduling, scaling activities, and errors, is essential for tracking the overall health and stability of the cluster

Finally, three alert rules were created and set to an alarm which automatically emails the relevant DevOps engineer if certain conditions are broken. These alerts are:

* If the disk percentage in the AKS cluster exceeds 90% (checking every 5 minutes with a loopback period of 15 minutes)
* If the CPU usage or memory working set percentage exceed 80% (checking every minute with a loopback period of 5 minutes)

The dashboard, logs and alerts allow the monitoring of the health of the AKS cluster. Specifically, these metrics inform the user whether the application is coming under strain and may need additional nodes/compute resources, an increase in shared memory or an increase of fallback instances should the traffic become too much, the application start to perform too slowly or come too close to crashing.

## Azure Key Vault
This application requires a number of credentials to access the backend database, including the name of the server, the name of the database and the username and the password to log in. To prevent this being hard coded into the app.py file, these are saved in Azure Key Vault as secrets and accessed using the Azure Python libraries (azure-identity and azure-keyvault-secrets).

The key vault was created in "Key vaults" on Azure and the secrets were added under the "Secrets" tab under "Objects". Prior to creating any secrets, the Key Vault Administrator role was added to the Microsoft Entra ID user in order to grant the necessary permissions for managing secrets within the Key Vault. These roles included:

* Key Vault Administrator: The highest level of access, the Key Vault Administrator role grants full control over the Key Vault. Administrators can manage access policies, configure advanced settings, and perform any operation within the Key Vault.

* Key Vault Contributor: Key Vault Contributors have broad permissions within the Key Vault, allowing them to manage all aspects except for access policies and advanced settings. They can add, update, and delete secrets, keys, and certificates.

* Key Vault Reader: Readers have read-only access to the Key Vault. They can view configurations, secrets, and certificates but are unable to make modifications.

To then allow AKS to interact with the Key Vault and access the required secrets, user managed identity for the AKS has been enabled. First, a user-assigned managed identity was created by running `az identity create -g {existing resource group to manage} -n {name of managed identity to create}`. Then, the AKS resource to be managed can be updated to add this identity: `az aks update --resource-group <resource group> --name <aks resource name> --enable-managed-identity`. Alternatively, not specifying the identity will automatically create and assign a system-assigned managed identity. The primary differences are detailed further below.

Finally, Role-Based Action Control (RBAC) has been granted to the AKS cluster. RBAC is a critical component of Azure's security, providing a structured approach to managing permissions. RBAC ensures that users, services, and applications have precisely the required level of access, reducing the risk of unauthorized actions and enhancing overall security. The Key Vault Secrets Officer role is assigned to the Managed Identity. This role provides permissions to read, list, set and delete secrets, certificates, and keys within the specified Azure Key Vault. The CLI command used was:

`az role assignment create --role "Key Vault Secrets Officer" \ --assignee <managed-identity-client-id> \ --scope /subscriptions/{subscription-id}/resourceGroups/{resource-group}/providers/Microsoft.KeyVault/vaults/{key-vault-name}`


### Managed Identity Types
1. System-assigned Managed Identity: When enabling a system-assigned managed identity:
    * Created directly on specific Azure resources (e.g., Virtual Machines or Azure App Service). Only the specified resource can use this managed identity to request tokens from Microsoft Entra ID.
    * A unique service principal is created in Microsoft Entra ID, tied to the Azure resource's lifecycle
    * The identity is automatically deleted when the associated Azure resource is deleted
2. User-assigned Managed Identity: When enabling an user-assigned managed identity:
    * Created independently as a standalone Azure resource
    * A dedicated service principal is established in Microsoft Entra ID, managed separately
    * This identity persists independently of the resources using it and must be explicitly deleted
    * Can be shared across multiple Azure resources, offering flexibility in identity management

## End-to-End Testing
End-to-End testing has been completed through checking the pipeline has run successfully after pushing the final code changes to GitHub and portforwarding using Kubectl to access the deployed cluster locally and checking it is working as normal with no errors. Automated testing could also be added to GitHub actions or Azure DevOps in the future.

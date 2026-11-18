# Lab 8 - CST8915 Full-stack Cloud-native Development: Deploying and Managing the Algonquin Pet Store (On Steroids)

Welcome to Lab 8! In this lab, you will deploy and manage a microservices-based application, **Algonquin Pet Store (On Steroids)**, into a Kubernetes cluster. 

> **Note:** This lab will incur an approximate cost of $3 to your Azure credit. Be sure to clean up your resources after completing the lab to avoid additional charges.

---
## Lab Objectives
1. Deploy the Algonquin Pet Store (On Steroids) application to a Kubernetes cluster.
2. Configure and manage essential Kubernetes resources like StatefulSets, Secrets, ConfigMaps, and Deployments.
3. Test the application's features, including backend services, frontend interfaces, and AI integration.
4. Scale services and monitor application health.
5. Simulate customer and worker tasks using Virtual Customer and Virtual Worker services.
---

## Understanding Key Kubernetes Resources: StatefulSets, Deployments, Secrets, and ConfigMaps
In this section, you will learn about essential Kubernetes resources used to deploy and manage applications in a cluster.

### **Deployments**
A **Deployment** is a Kubernetes resource that ensures a specified number of pod replicas are running. It provides mechanisms for rolling updates, scaling, and version management.

#### **Key Features of Deployments**:
1. **Replica Management**:
Ensures a defined number of pod replicas are running at all times.
2. **Rolling Updates**:
Updates pods incrementally to ensure minimal downtime.
3. **Self-healing**:
Automatically replaces failed pods.
#### **Use Cases**:
- Stateless applications like web servers or APIs.
- Backend microservices.

### **StatefulSets**
A **StatefulSet** is a Kubernetes resource used to manage stateful applications. Unlike Deployments, StatefulSets are designed for applications that require unique network identifiers, stable storage, and consistent state across pod restarts.

#### **Key Features of StatefulSets**:
1. **Stable Pod Names**:
   Pods in a StatefulSet have predictable names, such as `pod-name-0`, `pod-name-1`.
2. **Persistent Storage**:
   Each pod can be associated with its own Persistent Volume, ensuring data is retained across restarts.
3. **Ordered Scaling and Updates**:
   Pods are created, updated, and deleted in a controlled order.

#### **Use Cases**:
- Databases like MongoDB, MySQL, or PostgreSQL.
- Message queues like RabbitMQ.

### **Secrets**
A **Secret** is a Kubernetes resource used to store sensitive information, such as API keys, passwords, or tokens. Use Kubernetes access controls to limit who can read Secrets. Anyone who can read their Base64-encoded values can decode them.

#### **Key Features of Secrets**:
1. **Base64 Encoding**:
Secret data is stored in Base64-encoded format.
2. **Environment Variables or Volumes**:
Secrets can be injected into pods as environment variables or mounted as files.
3. **Access Control**:
Secrets are scoped to specific namespaces and can only be accessed by authorized workloads.

#### **Use Cases**:
- Storing database credentials or API keys.
- Managing certificates and TLS data.

### **ConfigMap**
A **ConfigMap** is a Kubernetes resource used to store non-sensitive configuration data, such as application settings, environment variables, or file-based configurations.

#### **Key Features of ConfigMap**:
1. Decouples Configuration from Code:
Enables dynamic updates to configuration without modifying the application code.
2. Flexible Usage:
Can be injected into pods as environment variables, command-line arguments, or mounted as configuration files.
3. Version Management:
Easy to update and manage configurations independently of application code.

#### **Use Cases**:
- Application settings like database URLs or feature flags.
- Storing configuration for external services.

## Step 1: Clone the Algonquin Pet Store Repository

Clone the course repository containing this handout and open its [Deployment Files](./Deployment%20Files/) directory. Use the manifests from this Lab 8 release throughout the exercise.

 **Review the Deployment Files**:
   - Navigate to the `Deployment Files` folder
   - This folder contains YAML files for deploying all necessary Kubernetes resources, including services, deployments, StatefulSets, ConfigMaps, and Secrets.


## Step 2: Set Up the AKS Cluster
Create an AKS cluster with two worker nodes for this exercise. For this step, you can refer to **Lab 6**

## Step 3: Set Up the AI Backing Services
**If your Azure for Students subscription does not include access to the Azure AI resources or model deployments required for this lab, follow the [Student Guide for OpenAI](./Student_Guide_OpenAI.md).** Use the two credentials provided through Brightspace for the direct OpenAI API, then resume at Step 4. Students who have Azure access can use the following steps.

### Deploy the text and image models on Azure

1. Open Microsoft Foundry and select or create an Azure OpenAI resource in a region where your subscription has quota for the models. The text and image deployments can use different resources.
2. Deploy a Chat Completions model for product descriptions (for example, `gpt-4.1-mini` where available) and a GPT Image model for product photos (for example, `gpt-image-2`). Record the **deployment name** you gave each model and the resource endpoint shown in the portal. [Azure image model availability and setup](https://learn.microsoft.com/en-us/azure/foundry/openai/how-to/dall-e)
3. In the `ai-service` section of `aps-all-in-one.yaml`, set `USE_AZURE_OPENAI` to `"True"`. Fill in the following environment variables. The service appends the Azure OpenAI v1 API path to each resource endpoint.

   | Variable | Value |
   | --- | --- |
   | `AZURE_OPENAI_GPT_ENDPOINT` | Text resource endpoint, e.g. `https://<resource>.openai.azure.com` |
   | `AZURE_OPENAI_GPT_DEPLOYMENT_NAME` | Your text deployment name |
   | `AZURE_OPENAI_DALLE_ENDPOINT` | Image resource endpoint |
   | `AZURE_OPENAI_DALLE_DEPLOYMENT_NAME` | Your GPT Image deployment name |

4. In `Deployment Files`, copy the template to `secrets.yaml`.

   ```bash
   cp secrets.example.yaml secrets.yaml
   ```

5. Edit `secrets.yaml` and replace each placeholder with the corresponding Azure resource key. If both deployments use the same resource, enter its key in both fields. The example uses Kubernetes `stringData`, so enter the keys as supplied. Keep credentials out of your submission, screenshots, and video.

## Step 4: Deploy the ConfigMap and Secrets

Run these commands from `Deployment Files`, where you saved `secrets.yaml`. The lab's `.gitignore` already excludes this file. Keep it out of Git commits.

```bash
kubectl apply -f config-maps.yaml
kubectl apply -f secrets.yaml
kubectl get configmap rabbitmq-enabled-plugins
kubectl get secret openai-gpt-api-secret openai-dalle-api-secret
```

## Step 5: Deploy the Application
   ```bash
   kubectl apply -f aps-all-in-one.yaml
   ```
### Validate the Deployment
- Check Pods and Services:
   ```bash
   kubectl get pods
   kubectl get services
   ```
- Test Frontend Access:
   - Locate the external IPs for store-front and store-admin services:
   ```bash
   kubectl get services
   ```
   - Access the Store Front app at the external IP on port 80.
   - Access the Store Admin app at the external IP on port 80.
## Step 6: Deploy Virtual Customer and Worker
   ```bash
   kubectl apply -f admin-tasks.yaml
   ```
- Monitor Virtual Customer:
   ```bash
   kubectl logs -f deployment/virtual-customer
   ```
- Monitor Virtual Worker:
   ```bash
   kubectl logs -f deployment/virtual-worker
   ```

## Step 7: Scale and Monitor Services
### Scale Deployments:
- Scale the `order-service` to 3 replicas:
```bash
kubectl scale deployment order-service --replicas=3
```
- Check Scaling:
```bash
kubectl get pods
```
- Monitor Resource Usage:

   - Enable metrics server for resource monitoring.
   - Use kubectl top to monitor pod and node usage:
   ```bash
   kubectl top pods
   kubectl top nodes
   ```

## Step 8: Explore Advanced Features
### AI-Generated Descriptions and Images:
- In Store Admin, open the form to add a product. Enter a name, a positive price, and keywords. Select **Ask AI Assistant**, then **Generate Image**.
- Save the product, refresh Store Front, and open its product detail page. Verify that both the generated description and the photo appear.
### RabbitMQ Management:
- Access the RabbitMQ management UI:
   ```bash
   kubectl port-forward service/rabbitmq 15672:15672
   ```
   The kubectl port-forward command is used to forward a local port to a port on a Kubernetes resource (e.g., a Pod or Service). This allows you to access the application running in the cluster from your local machine without exposing it externally.


- Login with the default credentials (`username`/`password`).

### MongoDB Shell Access and Database Exploration
In this section, you will use the MongoDB shell to interact with the `orderdb` database, which stores order information for the Algonquin Pet Store application. Follow the steps below to connect to the MongoDB pod and explore its contents.

#### **1- Access the MongoDB Shell**
Run the following command to connect to the MongoDB shell inside the running MongoDB pod:
```bash
kubectl exec -it mongodb-0 -- mongosh
```
Explanation: This command uses kubectl exec to open an interactive shell (-it) inside the MongoDB pod and starts the MongoDB shell program (`mongosh`).

#### **2- List All Databases**
Once inside the MongoDB shell, run:
```bash
show dbs
```
Explanation: The show dbs command lists all databases available on the MongoDB server. You should see a list that includes the orderdb, which stores order-related data for the application.
#### **3- Switch to the Order Database**
```bash
use orderdb
```
Explanation: The use orderdb command selects the orderdb database, making it the active database for subsequent queries and commands.
#### **4- List Collections in the Database**
Display all collections in the orderdb database:
```bash
show collections
```
Explanation: The show collections command lists all collections (similar to tables in relational databases) in the current database. The orders collection contains the order data.
#### **5- Query the Orders Collection**
Retrieve all documents in the orders collection:
```bash
db.orders.find()
```
Explanation: The db.orders.find() command fetches and displays all documents (records) in the orders collection. This allows you to view the stored order data, including details such as customer information, products, and order status.

#### By following these steps, you will:
- Connect to the MongoDB shell in the Kubernetes pod.
- Explore the databases and collections used by the application.
- Query the orders collection to examine the data structure and stored records.

# Lab Assignment Tasks
## **Task 1: Build, Push, and Deploy Your Own Docker Images**

You are asked to fork the necessary service repositories, build Docker images for each service, push them to your own Docker Hub account, and update the Kubernetes configuration file to use your images.

---

### **Step 1: Fork the Repositories**

1. **Fork Each Repository**:
   - Visit the GitHub repositories for the services listed below and fork them into your own GitHub account:
   
   | Service            | Description                              | Github Repo                                                                 |
   | ------------------ | ---------------------------------------- | --------------------------------------------------------------------------- |
   | `store-front`      | Web app for customers to place orders    | [store-front-L8](https://github.com/ramymohamed10/store-front-L8)           |
   | `store-admin`      | Web app for store employees              | [store-admin-L8](https://github.com/ramymohamed10/store-admin-L8)           |
   | `order-service`    | Handles order placement                  | [order-service-L8](https://github.com/ramymohamed10/order-service-L8)       |
   | `product-service`  | Handles CRUD operations on products      | [product-service-L8](https://github.com/ramymohamed10/product-service-L8)   |
   | `makeline-service` | Processes and completes orders           | [makeline-service-L8](https://github.com/ramymohamed10/makeline-service-L8) |
   | `ai-service`       | AI-based product descriptions and images | [ai-service-L8](https://github.com/ramymohamed10/ai-service-L8)             |
   | `virtual-customer` | Simulates customer order creation        | [virtual-customer-L8](https://github.com/ramymohamed10/virtual-customer-L8) |
   | `virtual-worker`   | Simulates order completion               | [virtual-worker-L8](https://github.com/ramymohamed10/virtual-worker-L8)     |

2. **Clone the Forked Repositories**:
---
### **Step 2: Build and Push Docker Images**
---
### **Step 3: Update the deployment images**

Use your own image tags for the six application services in `aps-all-in-one.yaml` and for both simulators in `admin-tasks.yaml`. Apply both files and verify the running image names.
---
### **Step 4: Test the Application**

## **Task 2: Investigate Azure Managed Services**

- **Investigate what Azure managed services could replace self-hosted MongoDB and RabbitMQ.**
   - For each service:
      - Give its name and purpose.
      - Explain why it’s a good fit and what problem it solves.

## Submission

### What to Submit
1. **Demo Video (Max 5 minutes)**  
   - Show how you deploy the Algonquin Pet Store application using your updated `aps-all-in-one.yaml` file from Task 1.

2. **GitHub Repository (Submission Repo)**  
   - You must create **one GitHub repository** for your lab submission.  
   - This submission repository must include:  
     - A `README.md` file with:  
         - The YouTube demo video link
         - Your findings on Azure managed services from Task 2
         - Your updated `aps-all-in-one-Task1.yaml` file used for Task 1
         - Your updated `admin-tasks.yaml` with both simulator image tags

### How to Submit
- Push your work to a **public GitHub repository** (the submission repository).  
- Submit the link to your submission repository as your final lab deliverable in **Brightspace**.  
# Algonquin Pet Store (On Steroids)
Welcome to the Algonquin Pet Store (On Steroids) application. Start with the [Lab 8 instructions](./Lab8_Instructions.md).

This sample demo app consists of a group of containerized microservices that can be easily deployed into a Kubernetes cluster. This is meant to show a realistic scenario using a polyglot architecture, event-driven design, and common open source back-end services (eg - RabbitMQ, MongoDB). The application also leverages OpenAI's models to generate product descriptions and images. This can be done using either [Azure OpenAI](https://learn.microsoft.com/azure/ai-services/openai/overview) or [OpenAI](https://openai.com/).

This application is inspired by Azure Kubernetes Service (AKS) quickstart demo [Azure Kubernetes Service (AKS) Docs](https://learn.microsoft.com/en-us/azure/aks/).

> [!NOTE]
> This is not meant to be an example of perfect code to be used in production, but more about showing a realistic application running in kubernetes. 

## Architecture

The application has the following services: 

| Service | Description | Github Repo | Docker Image |
| --- | --- | --- | --- |
| `store-front` | Web app for customers to place orders (Vue.js) | [store-front-L8](https://github.com/ramymohamed10/store-front-L8) | [ramymohamed/store-front:lab8](https://hub.docker.com/r/ramymohamed/store-front/tags?name=lab8) |
| `store-admin` | Web app used by store employees to view orders in queue and manage products (Vue.js) | [store-admin-L8](https://github.com/ramymohamed10/store-admin-L8) | [ramymohamed/store-admin:lab8](https://hub.docker.com/r/ramymohamed/store-admin/tags?name=lab8) |
| `order-service` | This service is used for placing orders (Javascript) | [order-service-L8](https://github.com/ramymohamed10/order-service-L8) | [ramymohamed/order-service:lab8](https://hub.docker.com/r/ramymohamed/order-service/tags?name=lab8) |
| `product-service` | This service is used to perform CRUD operations on products (Rust) | [product-service-L8](https://github.com/ramymohamed10/product-service-L8) | [ramymohamed/product-service:lab8](https://hub.docker.com/r/ramymohamed/product-service/tags?name=lab8) |
| `makeline-service` | This service handles processing orders from the queue and completing them (Golang) | [makeline-service-L8](https://github.com/ramymohamed10/makeline-service-L8) | [ramymohamed/makeline-service:lab8](https://hub.docker.com/r/ramymohamed/makeline-service/tags?name=lab8) |
| `ai-service` | Optional service for adding generative text and graphics creation (Python) | [ai-service-L8](https://github.com/ramymohamed10/ai-service-L8) | [ramymohamed/ai-service:lab8](https://hub.docker.com/r/ramymohamed/ai-service/tags?name=lab8) |
| `rabbitmq` | RabbitMQ for an order queue | [rabbitmq](https://github.com/docker-library/rabbitmq) | [rabbitmq:3-management](https://hub.docker.com/_/rabbitmq) |
| `mongodb` | MongoDB instance for persisted data | [mongodb](https://github.com/docker-library/mongo) | [mongo:8.0.38](https://hub.docker.com/_/mongo) |
| `virtual-customer` | Simulates order creation on a scheduled basis (Rust) | [virtual-customer-L8](https://github.com/ramymohamed10/virtual-customer-L8) | [ramymohamed/virtual-customer:lab8](https://hub.docker.com/r/ramymohamed/virtual-customer/tags?name=lab8) |
| `virtual-worker` | Simulates order completion on a scheduled basis (Rust) | [virtual-worker-L8](https://github.com/ramymohamed10/virtual-worker-L8) | [ramymohamed/virtual-worker:lab8](https://hub.docker.com/r/ramymohamed/virtual-worker/tags?name=lab8) |


![Logical Application Architecture Diagram](assets/Algonquin%20Pet%20Store%20On%20Steroids.png)

## Run the app on Azure Kubernetes Service (AKS)

You can use the kubernetes YAML files provided in the [Deployment Files](./Deployment%20Files/) folder to deploy the app to an AKS cluster.




---

## Lab 8 - My Deployment

I deployed the Algonquin Pet Store application to Azure Kubernetes Service (AKS).

I created an AKS cluster with two worker nodes in Mexico Central. I built eight Docker images from my forked repositories and pushed them to my Docker Hub account, `justinjunior`.

I updated the Kubernetes YAML files to use my own Docker images.

### Task 1 - Deployment Files

- [Updated application deployment YAML](./Deployment%20Files/aps-all-in-one-Task1.yaml)
- [Updated virtual customer and worker YAML](./Deployment%20Files/admin-tasks.yaml)

I tested the Store Front and Store Admin applications. I also verified that the virtual customer creates orders and the virtual worker processes them.

I scaled the order-service deployment to three replicas and checked CPU and memory usage using kubectl.

Finally, I tested the AI service by generating a product description and image. I saved the new product and verified that it appeared in the Store Front application.

## Task 2 - Azure Managed Services

### 1. Azure Cosmos DB for MongoDB

Azure Cosmos DB for MongoDB is a managed database service that supports MongoDB-compatible applications.

In this lab, MongoDB stores application data. Azure Cosmos DB for MongoDB could replace the MongoDB container running in Kubernetes.

This is a good solution because Azure manages much of the database infrastructure, availability, backups, and scaling. It reduces the work needed to maintain a database server.

It also helps improve reliability because the database does not depend on a single MongoDB container in the AKS cluster.

Some connection settings and compatibility requirements would need to be checked before migration.

### 2. Azure Service Bus

Azure Service Bus is a managed messaging service that allows applications to communicate using queues and topics.

In this lab, RabbitMQ helps services exchange messages for order processing.

Azure Service Bus could provide managed queues for sending orders between services.

It is a good solution because Azure manages the messaging infrastructure. It supports reliable message delivery, retries, and dead-letter queues.

This reduces the need to install, monitor, and maintain a RabbitMQ server inside Kubernetes.

However, Azure Service Bus is not a direct RabbitMQ replacement. The applications would need changes to their messaging clients and configuration.

### Conclusion

Using managed Azure services can reduce maintenance and improve reliability. It also allows developers to focus more on their applications instead of managing databases and messaging servers.

## Demo Video

YouTube video: To be added after recording.

## References

- [Azure Cosmos DB for MongoDB documentation](https://learn.microsoft.com/en-us/azure/cosmos-db/mongodb/introduction)
- [Azure Service Bus documentation](https://learn.microsoft.com/en-us/azure/service-bus-messaging/service-bus-messaging-overview)
- [Azure Kubernetes Service documentation](https://learn.microsoft.com/en-us/azure/aks/)


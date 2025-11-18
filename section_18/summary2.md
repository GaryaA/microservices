# Section 18: Deploying microservices into cloud K8s cluster

This section does not contain any source code or configuration files.

Based on the title, this section of the course likely covers the process and best practices for deploying the previously containerized and Helm-charted application into a managed Kubernetes service in the cloud, such as:

*   **Google Kubernetes Engine (GKE)**
*   **Amazon Elastic Kubernetes Service (EKS)**
*   **Azure Kubernetes Service (AKS)**

The topics covered would probably include:
*   Setting up a cloud Kubernetes cluster.
*   Configuring `kubectl` to connect to the cloud cluster.
*   Pushing the microservice Docker images to a cloud container registry (like Google Container Registry, Amazon ECR, or Azure Container Registry).
*   Modifying Helm `values.yaml` files for a cloud environment, for example, changing `Service` types from `ClusterIP` to `LoadBalancer` to expose services to the internet.
*   Using Helm to deploy the entire application stack to the cloud cluster.
*   Managing cloud-specific resources and configurations.

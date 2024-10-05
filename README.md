# Spring PetClinic Application

## Overview

This project aims to create a complete CI/CD Pipeline for microservice-based applications using the [Spring Petclinic Application](https://github.com/spring-projects/spring-petclinic). The application leverages modern containerization and orchestration tools like Docker and Kubernetes. The CI/CD Pipeline is set up using a GitHub Actions server deployed  which automatically builds and deploys each update to the application.


## Application Features

- **Manage Pets**: Add, update, delete pets.
- **Manage Owners**: Add, update, delete pet owners.
- **Manage Visits**: Schedule and manage visits for pets.
- **MySQL Database**: Uses MySQL for storing data.


## Requirements

To run and deploy the **Spring PetClinic** application, you need to have the following tools installed on your system:

### 1. Docker
   - Docker is used to containerize the application and run it inside containers.

### 2. Docker Compose
   - Docker Compose is used to manage multi-container Docker applications (the Spring PetClinic app with MySQL) in a single configuration.

### 3. Kubernetes (Local or Cloud-based Kubernetes Cluster)
   - Kubernetes is used to manage and run applications in containers across a cluster.

### 4. Helm
   - Helm is a package manager for Kubernetes, used to easily manage application deployments.

### 5. Git
   - Git is a version control system used to manage source code changes.

### 6. GitHub Actions
   - GitHub Actions is used to automate CI/CD pipelines for building, testing, and deploying the application.


## Project Structure

```plaintext
spring-petclinic-app/
├── application/               # Contains the Spring PetClinic application and Docker configurations
│   ├── Dockerfile             # Dockerfile for building the application image
│   ├── docker-compose.yml     # Docker Compose file to run the app and MySQL
│   ├── src/                   # Application source code (Java)
│   └── pom.xml                # Maven project file
├── helm/                      # Helm charts for deploying the Spring PetClinic application
├── kubernetes-manifest/       # Contains Kubernetes manifests for the application and MySQL
│   ├── application-manifest/  # Kubernetes manifests for the Spring PetClinic application
│   ├── mysql-manifest/        # Kubernetes manifests for the MySQL database
├── .github/                   # GitHub Actions workflows for CI/CD
│   └── workflows/             # Workflow YAML files for automation
└── README.md                  # Project documentation
```


## STEP 1 - Prepare Dockerfile for Image

A **Dockerfile** is a script that contains a series of instructions on how to build a Docker image for the Spring PetClinic application. The image is a packaged version of the application with all its dependencies, configuration, and environment needed to run it consistently across different environments.

### Dockerfile Structure

To optimize the size of the Docker image, we use a **multi-stage build**. Multi-stage builds allow us to build the application in one stage and create a lightweight image for production by copying only the necessary files, reducing the final image size.



# STEP 2 - Create Docker Compose File for Spring PetClinic Application and Database

**Docker Compose** is a tool that allows you to define and manage multi-container Docker applications. For the Spring PetClinic application, we use Docker Compose to run both the application and the MySQL database together, making it easier to manage dependencies and services with a single command.

## Services Used in Docker Compose

In the **docker-compose.yaml** file, we define two services:

1. **Application Service (app)**: This service runs the Spring PetClinic application using the optimized Docker image built from the Dockerfile. 

2. **MySQL Service (mysql)**: This service runs a MySQL database, which is used to store the application’s data.


### Dependency Management with **depends_on**

In the Docker Compose file, we use the **depends_on** directive to ensure that the **app** service (Spring PetClinic application) starts only after the **mysql** service is running. This ensures that the database is fully operational before the application attempts to connect to it. 

```yaml
depends_on:
  - mysql
```

### Volume Usage

We use a **volume** in the MySQL service to ensure that the data is persistent. The volume maps the MySQL data directory inside the container (`/var/lib/mysql`) to a directory on the host machine. This way, the data is not lost when the container is stopped or removed.

- **Volume**: `/home/ubuntu/mysql_data:/var/lib/mysql`
  - The host path `/home/ubuntu/mysql_data` is where the MySQL data is stored, ensuring persistence even after the container is stopped.

This setup allows us to manage both the application and the database together, while ensuring that the database data is preserved across container restarts.



# STEP 3 - Set Up a Kubernetes Cluster

Due to local storage constraints, we have set up a Kubernetes cluster on AWS, consisting of one **master** node and one **worker** node. This setup provides a scalable and reliable environment to run the Spring PetClinic application.

### Setting Up the Kubernetes Cluster on AWS

We use AWS EC2 instances to create a Kubernetes cluster. The setup includes:

- **1 Master Node**: Responsible for managing the cluster and controlling the worker node.
- **1 Worker Node**: Executes the workloads and runs the Spring PetClinic application.


# STEP 4 - Prepare Kubernetes Manifests for the Application and Database

For the database setup, we used Kubernetes manifest files to deploy MySQL in the cluster. This ensures that the database is properly configured with persistent storage and secure access through Kubernetes secrets and service accounts. Below are the files used to deploy the MySQL database.

### MySQL Kubernetes Manifests

1. **mysql-deployment.yaml**: This file defines the **Deployment** for the MySQL database, including the container image and environment variables for the database configuration.

2. **mysql-persistentvolume.yaml**: The **PersistentVolume** (PV) defines the physical storage for the MySQL database. It ensures that data is stored persistently outside the container.

3. **mysql-persistentvolumeclaim.yaml**: The **PersistentVolumeClaim** (PVC) requests storage for the MySQL deployment. It binds to the PersistentVolume to ensure that the MySQL data is saved even when the pod is restarted.

4. **mysql-secret.yaml**: The **Secret** stores sensitive data such as the MySQL root password and database credentials. This ensures that these credentials are securely passed to the MySQL container without hardcoding them in the manifest files.

5. **mysql-service.yaml**: The **Service** defines how the MySQL deployment is exposed within the Kubernetes cluster. It allows the application to communicate with the database through a stable network endpoint.

6. **mysql-serviceaccount.yaml**: The **ServiceAccount** is used to grant MySQL specific permissions within the Kubernetes cluster, providing security and access control.

---

With these manifest files, the MySQL database is securely and persistently deployed on the Kubernetes cluster, ready to interact with the Spring PetClinic application.


### Application Kubernetes Manifests

1. **app-deployment.yaml**: Defines the **Deployment** for the Spring PetClinic application, specifying the container image, replicas, and environment configurations needed to run the application.

2. **app-service.yaml**: The **Service** object is used to expose the Spring PetClinic application within the Kubernetes cluster. It allows the application to be accessed through a stable endpoint (IP/port).

3. **app-configmap.yaml**: The **ConfigMap** stores configuration data that is injected into the application at runtime, ensuring externalized configurations.

4. **app-secret.yaml**: The **Secret** stores sensitive information such as the application's credentials or database connection details, ensuring secure handling of this data.

5. **app-serviceaccount.yaml**: The **ServiceAccount** object defines the permissions for the Spring PetClinic application to interact with other Kubernetes resources, improving security by limiting access.

6. **app-hpa.yaml**: The **Horizontal Pod Autoscaler (HPA)** automatically scales the number of pods based on CPU utilization or other resource metrics. This ensures that the application scales up or down based on demand, improving performance and resource efficiency.

7. **app-ingress.yaml**: The **Ingress** object is used to manage external access to the Spring PetClinic application. It provides routing rules and can expose the application via a domain name, allowing traffic to be routed to the appropriate service.

---

With these manifest files, the Spring PetClinic application is securely deployed and can scale automatically to meet demand, while being easily accessible through an Ingress for external traffic.



# STEP 5 - Prepare Helm Charts for Application Kubernetes Manifests

After preparing the Kubernetes manifests for the Spring PetClinic application, we transitioned these manifests into Helm charts. Helm makes it easier to manage and deploy applications by providing a package manager for Kubernetes resources. By using Helm, we can parameterize our configuration files and simplify deployment and upgrades.

### Helm Chart Files for the Application

Below is the list of manifest files that have been converted into Helm charts:

1. **configmap.yaml**: Defines the application’s **ConfigMap**, which stores external configuration values for the application. This ensures that configurations are kept outside the container and can be easily updated without changing the image.

2. **deployment.yaml**: Configures the **Deployment** for the Spring PetClinic application, including replicas and container configurations. With Helm, you can dynamically adjust the number of replicas or the image version.

3. **hpa.yaml**: Sets up the **Horizontal Pod Autoscaler (HPA)**. This file configures autoscaling policies for the application, allowing Helm to dynamically adjust the number of running pods based on resource usage such as CPU and memory.

4. **ingress.yaml**: The **Ingress** configuration routes external traffic to the application’s service. Helm allows you to easily manage different routing rules or domain names based on the environment (development, production, etc.).

5. **secret.yaml**: Helm is used to manage the **Secret** objects for securely storing sensitive information such as database credentials or application secrets.

6. **service.yaml**: The **Service** file defines how the application is exposed within the cluster. Helm allows you to dynamically configure ports or service types (ClusterIP, LoadBalancer, etc.).

7. **serviceaccount.yaml**: The **ServiceAccount** configuration provides the necessary permissions for the application within the Kubernetes cluster. Helm enables easy updates to these permissions if needed.


Using these Helm charts, we can now deploy the Spring PetClinic application in a more structured and manageable way, while also gaining flexibility for future updates and scaling.
---



# STEP 6 - CI/CD Pipeline with GitHub Actions

We have set up a CI/CD pipeline using **GitHub Actions** to automate the process of building Docker images, pushing them to Docker Hub, and deploying the Spring PetClinic application to a Kubernetes cluster using Helm.

### CI/CD Pipeline Overview

Here are the steps involved in the CI/CD pipeline:

1. **Trigger the Pipeline**:
   - The pipeline is triggered whenever there is a push to the `main` branch.
   - Additionally, the pipeline can be triggered manually using the `workflow_dispatch` event.

2. **Checkout the Code**:
   - The repository code is checked out from the `main` branch to ensure that the latest version of the code is being used for the build and deployment process.

3. **Set Up Docker Buildx**:
   - Docker Buildx is set up to enable advanced image building features, such as multi-architecture builds.

4. **Build Docker Image**:
   - The Docker image for the Spring PetClinic application is built using the **Dockerfile** located in the `application/` directory.
   - The image is tagged using the first 7 characters of the Git SHA to ensure unique versioning.

5. **Login to Docker Hub**:
   - The pipeline logs in to Docker Hub using stored credentials from GitHub Secrets.

6. **Push Docker Image to Docker Hub**:
   - The newly built Docker image is pushed to Docker Hub with the SHA tag.

7. **Set Up Kubeconfig**:
   - The Kubeconfig file is set up to authenticate and interact with the Kubernetes cluster. This allows the pipeline to deploy the application to the cluster.

8. **Update Helm Chart Image Tag**:
   - The `values.yaml` file in the Helm chart is updated with the new Docker image tag, ensuring that the correct image version is deployed.

9. **Deploy to Kubernetes Cluster**:
   - The updated Helm chart is deployed to the Kubernetes cluster using the `helm upgrade --install` command, ensuring that the latest version of the application is running.

---


## Conclusion

This project demonstrates how to containerize and deploy the Spring PetClinic application using **Docker**, **Docker Compose**, and **Kubernetes**. With the help of **Helm charts**, the application can also be easily deployed in scalable environments like Kubernetes clusters. 

Additionally, the project is integrated with **GitHub Actions** to automate the CI/CD pipeline. This pipeline automates the process of:

- Building Docker images for the application.
- Pushing these images to Docker Hub.
- Deploying the application to a Kubernetes cluster using Helm.

With this setup, the application can be continuously built and deployed in a streamlined and efficient manner, ensuring that updates are automatically rolled out to the Kubernetes environment.

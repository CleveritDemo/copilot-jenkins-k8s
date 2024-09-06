# My Node Application with Kubernetes

This project demonstrates how to deploy a simple Node.js application to a local Kubernetes cluster using **Kind** (Kubernetes in Docker). The Kubernetes manifests include a **Deployment** to run the application and a **Service** to expose the application via a **NodePort**.

## Prerequisites

Before running the application, ensure you have the following installed on your machine:

- [Docker](https://www.docker.com/products/docker-desktop)
- [Kind](https://kind.sigs.k8s.io/) (Kubernetes in Docker)
- [kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl/) (Kubernetes command-line tool)

## Setting Up the Local Kubernetes Cluster

### Step 1: Create a Kind Cluster

To run Kubernetes locally, we will create a Kind cluster.

    # Create a new Kind cluster named 'jenkins'
    kind create cluster --name jenkins

### Step 2: Build the Docker Image Locally

Before deploying to Kubernetes, build the Docker image for the Node.js application.

    # Build the Docker image
    docker build -t my-node-app:latest .

### Step 3: Load the Docker Image into the Kind Cluster

Since Kind runs Kubernetes inside Docker containers, you need to load the Docker image into the Kind cluster.

    # Load the local Docker image into the Kind cluster
    kind load docker-image my-node-app:latest --name jenkins

### Step 4: Apply Kubernetes Manifests

Now that the image is loaded into the Kind cluster, apply the Kubernetes manifests to deploy the application.

    # Create a namespace and deploy the application
    kubectl apply -f k8s.yaml

### Step 5: Verify Deployment

Check the status of the pods to ensure the application is running:

    # Get the status of the pods
    kubectl get pods -n jenkins-ns

You should see two pods running.

### Step 6: Expose the Application

The service is exposed via **NodePort**. You can access the application on port `30080`.

    # Port forward the service to make it accessible in the browser
    kubectl port-forward service/my-app-service 8080:80 -n jenkins-ns

You can now access the application in your browser at:

    http://localhost:8080

### Step 7: Verify the Service

To check the status of the service, run the following command:

    kubectl get svc -n jenkins-ns

You should see something like:

    NAME             TYPE       CLUSTER-IP     EXTERNAL-IP   PORT(S)        AGE
    my-app-service   NodePort   10.96.179.44   <none>        80:30080/TCP   5m

### Cleaning Up

To delete the cluster and clean up resources:

    # Delete the Kind cluster
    kind delete cluster --name jenkins
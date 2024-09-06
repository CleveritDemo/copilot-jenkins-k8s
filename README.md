# My Node Application with Kubernetes, Jenkins, and Copilot

This project demonstrates how to deploy a simple Node.js application to a local Kubernetes cluster using **Kind** (Kubernetes in Docker), **Jenkins** for continuous integration, and **GitHub Copilot** to automate development tasks.

## Prerequisites

Ensure the following are installed on your machine:

- [Docker](https://www.docker.com/products/docker-desktop) (for containerization)
- [Kind](https://kind.sigs.k8s.io/) (Kubernetes in Docker)
- [kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl/) (Kubernetes command-line tool)
- [Jenkins](https://hub.docker.com/repository/docker/nicolasandrescalvo/jenkins/general) (for CI/CD pipelines)

---

## Setting Up Jenkins Locally

### Step 1: Pull the Pre-built Docker Image

    docker pull nicolasandrescalvo/jenkins:v1.0

### Step 2: Run Jenkins Locally

    docker run -d --name jenkins \
      -p 8080:8080 \
      -p 50000:50000 \
      -v $(pwd)/jenkins_home:/var/jenkins_home \
      --privileged \
      -v /var/run/docker.sock:/var/run/docker.sock \
      nicolasandrescalvo/jenkins:v1.0

### Step 3: Extract Admin Secret

    docker exec jenkins cat /var/jenkins_home/secrets/initialAdminPassword

![Admin Password Screenshot](./images/pass.png)

Access Jenkins at `http://localhost:8080`, and use the admin secret to log in.

![Jenkins Login Screenshot](./images/jenkins.png)

---

## Setting Up the Local Kubernetes Cluster

### Step 1: Create a Kind Cluster

    kind create cluster --name jenkins

### Step 2: Configure the Local Cluster

To configure the Kubernetes context:

    kubectl config get-contexts
    kubectl config set-cluster kind-jenkins

---

## Workflow with GitHub Copilot

### Step 1: Generate the Workspace

Use Copilot Chat to generate the project workspace:

> @workspace /new inside copilot-jenkins-k8s repository in the copilot directory. Create a simple Node.js app, a Dockerfile to run it, and Kubernetes manifests to run it locally.

![Create Workspace Screenshot](./images/new.png)

### Step 2: Generate the Jenkinsfile

Use Copilot Chat to generate the Jenkinsfile

> @workspace now i need a Jenkinsfile to build, and push this solution

![cop-jenkins](./images/cop-jenkins.png)

### Step 3: Update Jenkinsfile

Use Copilot Chat to update the Jenkinsfile

> @workspace lets update it a little jenkinsfile. Repository is: https://github.com/NicolasAndresCalvo/copilot-jenkins-k8s. And Docker Registry to push image is: nicolasandrescalvo/copilot-jenkins-k8s. Docker Credential is a PAT stored it in Jenkins under this ID: DOCKERHUB-PAT

![cop-jenkins-2](./images/cop-jenkins-2.png)

---

## Test App Locally with Docker and Kubernetes

### Step 1: Build the Docker Image Locally

Before deploying to Kubernetes, build the Docker image for the Node.js application.

    # Build the Docker image
    docker build -t my-node-app:latest .

### Step 2: Load the Docker Image into the Kind Cluster

Since Kind runs Kubernetes inside Docker containers, you need to load the Docker image into the Kind cluster.

    # Load the local Docker image into the Kind cluster
    kind load docker-image my-node-app:latest --name jenkins

### Step 3: Apply Kubernetes Manifests

Now that the image is loaded into the Kind cluster, apply the Kubernetes manifests to deploy the application.

    # Create a namespace and deploy the application
    kubectl apply -f k8s.yaml

### Step 4: Verify Deployment

Check the status of the pods to ensure the application is running:

    # Get the status of the pods
    kubectl get pods -n jenkins-ns

You should see two pods running.

### Step 5: Expose the Application

The service is exposed via **NodePort**. You can access the application on port `30080`.

    # Port forward the service to make it accessible in the browser
    kubectl port-forward service/my-app-service 8080:80 -n jenkins-ns

You can now access the application in your browser at:

    http://localhost:8080

### Step 6: Verify the Service

To check the status of the service, run the following command:

    kubectl get svc -n jenkins-ns

You should see something like:

    NAME             TYPE       CLUSTER-IP     EXTERNAL-IP   PORT(S)        AGE
    my-app-service   NodePort   10.96.179.44   <none>        80:30080/TCP   5m

### Cleaning Up

To delete the cluster and clean up resources:

    # Delete the Kind cluster
    kind delete cluster --name jenkins
---



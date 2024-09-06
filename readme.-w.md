# My Node Application with Kubernetes, Jenkins, and GitHub Copilot

This project demonstrates how to deploy a simple Node.js application to a local Kubernetes cluster using **Kind** (Kubernetes in Docker), **Jenkins** for continuous integration, and **GitHub Copilot** to automate development tasks.

---

## Prerequisites

Ensure the following are installed on your machine:

- [Docker](https://www.docker.com/products/docker-desktop) - For containerization
- [Kind](https://kind.sigs.k8s.io/) - Kubernetes in Docker
- [kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl/) - Kubernetes command-line tool
- [Jenkins](https://hub.docker.com/repository/docker/nicolasandrescalvo/jenkins/general) - For CI/CD pipelines

---

## Setting Up Jenkins Locally

### Step 1: Pull the Pre-built Jenkins Docker Image

Pull the pre-built Docker image for Jenkins:

    docker pull nicolasandrescalvo/jenkins:v1.0

### Step 2: Run Jenkins Locally

Run the Jenkins container:

    docker run -d --name jenkins \
      -p 8080:8080 \
      -p 50000:50000 \
      -v $(pwd)/jenkins_home:/var/jenkins_home \
      --privileged \
      -v /var/run/docker.sock:/var/run/docker.sock \
      nicolasandrescalvo/jenkins:v1.0

### Step 3: Extract Admin Secret

After starting Jenkins, extract the admin password:

    docker exec jenkins cat /var/jenkins_home/secrets/initialAdminPassword

![Admin Password Screenshot](./images/pass.png)

Access Jenkins at `http://localhost:8080`, and use the admin password to log in:

![Jenkins Login Screenshot](./images/jenkins.png)

---

## Setting Up the Local Kubernetes Cluster

### Step 1: Create a Kind Cluster

To run Kubernetes locally, create a Kind cluster named `jenkins`:

    kind create cluster --name jenkins

### Step 2: Configure the Local Kubernetes Cluster

Use `kubectl` to configure the Kubernetes context:

    kubectl config get-contexts
    kubectl config set-cluster kind-jenkins

---

## Workflow with GitHub Copilot

### Step 1: Generate the Workspace

Ask GitHub Copilot Chat to generate the project workspace:

> @workspace /new inside copilot-jenkins-k8s repository in the copilot directory. Create a simple Node.js app, a Dockerfile to run it, and Kubernetes manifests to run it locally.

![Create Workspace Screenshot](./images/new.png)

### Step 2: Generate the Jenkinsfile

Use GitHub Copilot to generate the `Jenkinsfile`:

> @workspace now I need a Jenkinsfile to build and push this solution.

![Generate Jenkinsfile Screenshot](./images/cop-jenkins.png)

### Step 3: Update Jenkinsfile

Ask GitHub Copilot Chat to update the `Jenkinsfile` to include Docker registry settings:

> @workspace update the `Jenkinsfile`. The repository is: https://github.com/NicolasAndresCalvo/copilot-jenkins-k8s, and the Docker Registry to push the image is: `nicolasandrescalvo/copilot-jenkins-k8s`. Docker Credential is stored in Jenkins under this ID: `DOCKERHUB-PAT`.

![Update Jenkinsfile Screenshot](./images/cop-jenkins-2.png)

---

## Testing the App Locally with Docker and Kubernetes

### Step 1: Build the Docker Image

Before deploying to Kubernetes, build the Docker image for the Node.js application:

    docker build -t my-node-app:latest .

### Step 2: Load the Docker Image into the Kind Cluster

Since Kind runs Kubernetes inside Docker containers, load the image into the Kind cluster:

    kind load docker-image my-node-app:latest --name jenkins

### Step 3: Apply the Kubernetes Manifests

Apply the Kubernetes manifests to deploy the application. Ensure the deployment includes `imagePullPolicy: Never` to use the local image:

    kubectl apply -f k8s.yaml

### Step 4: Verify the Deployment

Check the status of the pods:

    kubectl get pods -n jenkins-ns

You should see two pods running.

### Step 5: Expose the Application

The application is exposed via a **NodePort**. Use `kubectl` to port forward and access the app in your browser:

    kubectl port-forward service/my-app-service 8080:80 -n jenkins-ns

Access the app in your browser at:

    http://localhost:8080

---
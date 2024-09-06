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

Use Copilot Chat to generate the Jenkinsfile

> @workspace lets update it a little jenkinsfile. Repository is: https://github.com/NicolasAndresCalvo/copilot-jenkins-k8s. And Docker Registry to push image is: nicolasandrescalvo/copilot-jenkins-k8s. Docker Credential is a PAT stored it in Jenkins under this ID: DOCKERHUB-PAT

    pipeline {
        agent any

        environment {
            DOCKER_IMAGE = 'nicolasandrescalvo/copilot-jenkins-k8s'
            IMAGE_TAG = "v1.0.${env.BUILD_ID}"
            DOCKERHUB_USERNAME = 'nicolasandrescalvo'
        }

        parameters {
            string(name: 'BRANCH_NAME', defaultValue: 'main', description: 'Branch to build')
        }

        stages {
            stage('Checkout Code') {
                steps {
                    git branch: "${params.BRANCH_NAME}", url: 'https://github.com/NicolasAndresCalvo/copilot-jenkins-k8s.git'
                }
            }

            stage('Build Docker Image') {
                steps {
                    script {
                        sh "docker build -t ${DOCKER_IMAGE}:${IMAGE_TAG} ."
                    }
                }
            }

            stage('Login to Docker Hub') {
                steps {
                    script {
                        withCredentials([string(credentialsId: 'DOCKERHUB-PAT', variable: 'DOCKERHUB_PAT')]) {
                            sh "echo ${DOCKERHUB_PAT} | docker login -u ${DOCKERHUB_USERNAME} --password-stdin"
                        }
                    }
                }
            }

            stage('Push Docker Image') {
                steps {
                    script {
                        sh "docker push ${DOCKER_IMAGE}:${IMAGE_TAG}"
                    }
                }
            }
        }
    }
---



# Optimized DevOps with Kubernetes, Docker, Jenkins + GitHub Copilot 🤖

In this training, we will leverage GitHub Copilot to build a DevOps work environment that allows us to create a fully automated workflow from start to finish, incorporating the best practices of the SDLC.

The goal is to set up a local environment with Jenkins and publish a web application generated from scratch with GitHub Copilot to a Docker Registry, with the aim of deploying our application in a local Kubernetes environment.

# Final Result
The completed application developed in this practical exercise is located in this [public repository](https://github.com/CleveritDemo/copilot-static-web-demo-app/tree/solved). The final changes are in the `solved` branch.

The application contains:
- Source code of the website.
- Kubernetes manifest.
- Pipeline file `jenkinsfile`.
- `Dockerfile`.

# Next Steps:
## 🚧 Technical Requirements

The following programs are **mandatory** to complete this practical exercise.

1. [Docker](https://www.docker.com/products/docker-desktop): Used as a container execution tool.
2. [Kind](https://kind.sigs.k8s.io/): Tool for creating Kubernetes clusters using Docker containers.
3. [Jenkins](https://www.jenkins.io/): Open-source automation server for building, testing, and deploying applications.
4. [GitHub](https://github.com/): Used for code versioning and repository hosting.
5. [Docker Hub](https://hub.docker.com/): Service for hosting Docker container images.
6. [NodeJS](https://nodejs.org): Server-side JavaScript runtime environment.

## 🛠️ 1. Setting up Jenkins locally using Docker

- Open a terminal session in the `jenkins` folder located in this repository.
    ```shell
    cd "repository_folder/jenkins"
    ```

- Once the terminal (Bash|Powershell) is open, start the Jenkins service using Docker Compose with the following commands:
    ```shell
    docker compose build # Downloads the necessary Docker images and builds the containers.

    docker compose up -d # Initializes and starts the Docker containers. Runs the Jenkins application.
    ```
- Extract the Jenkins security password (needed to unlock and configure Jenkins). The password can be extracted using the following command:
    ```shell
    docker exec jenkins cat /var/jenkins_home/secrets/initialAdminPassword

    # The returned value will be a string similar to this:
    # 3ef9768dafa8429c870e45b68d96d651
    ```
- Open your browser and navigate to: `http://localhost:8080`. Enter the unlock key obtained in the previous step.
    ![Jenkins Lock Screen](./images/jenkins.png).

> Jenkins User 💡:  
Alternatively, Jenkins will ask to create a user and password to manage the tool. This is optional for this practical exercise, and the admin user can be used.

## 🖥️ 2. Setting up Kubernetes locally using Kind

First, create a new cluster using Kind with the following command:
```sh
kind create cluster --name devops-demo
```

Next, obtain the cluster context, which is a file that allows us to connect to the cluster control API using the `kubectl` command line.

Execute the following command:
```sh
kubectl cluster-info --context kind-devops-demo
```

To validate that the context has been configured and is active, you can verify by running the following command:
```sh
kubectl config get-contexts
```
This will return an output similar to the following:
```sh
CURRENT   NAME               CLUSTER            AUTHINFO           NAMESPACE
*         kind-devops-demo   kind-devops-demo   kind-devops-demo
```
If you have multiple contexts, ensure that the `kind-devops-demo` context is marked with an asterisk (*).

## 🚀 3. Creating a repository on GitHub

In this step, we will create a new public repository on our GitHub account. The purpose of this repository is to host the web application that Copilot will generate for us.

We will use this repository in our local Jenkins instance to deploy the web application.

We can ask GitHub Copilot to guide us through the steps to create a repository using the following prompt:

_Example Prompt:_
```
Can you provide the steps to create a new public repository on GitHub called "demo-solar-system-app"? The repository should contain a README and a .gitignore. What steps should I follow?
```

**Follow the suggestions provided by GitHub Copilot at this point.**

Once this step is completed, clone the created repository to your computer and open it with Visual Studio Code.

## 🧑🏼‍💻 4. Creating a test web application

In this step, we will create a simple web application that will serve as a test service for this practical exercise. This application will be hosted in the GitHub repository created in the previous step.

We will use GitHub Copilot extensions, specifically `@workspace`, and the `Copilot Edits` feature to make dynamic changes to our application's files.

> **RECOMMENDATION** 🚧  
> For better project generation results, use @workspace /new with the OpenAI O1 (Preview) model.

**PROMPT TO USE:**
```
@workspace /new I want you to build the structure of a new web project for a static website. The site consists of three main pages: Index, About, and Contact Us. The website is intended to display information about the solar system and is named "app". Therefore, you should include information about the planets that make up the solar system: Mercury, Venus, Earth, Mars, Jupiter, Saturn, Uranus, and Neptune. Some parts recognize Pluto as a planet, so include information about Pluto as well. Don't forget to include the Sun and all its relevant information on the index page. On the about us page, state that we are a small group of astronomy enthusiasts with a mission to educate and share information about the planets. (As an AI model, you should generate rich and useful content here). Finally, on the Contact Us page, create a contact form that collects the following information: First Name, Last Name, Email, Country, and a large text field for users to write their anecdotes.

This website should be developed using HTML5, CSS3, JavaScript, and the latest version of Bootstrap as the style and component library. To provision this website, use NodeJS with the ExpressJS framework to serve and make this static website available. The entire structure of this application should be created within a folder called "app" located in the root directory of this repository. It is very important not to duplicate the web pages, i.e., do not generate the same page twice, and above all, do not forget to generate the .gitignore file for this project.
```

**Troubleshooting: .gitignore**  
If the `.gitignore` file is not generated during the command execution, simply create a new `.gitignore` file inside the `app` folder and use the `ctrl`+`i` key combination on Windows or `cmd`+`i` on MacOS to open the Copilot chat "in-line". Write the following prompt for Copilot to generate a basic .gitignore structure for you.

**PROMPT TO USE**
```
Add a basic structure for a .gitignore file
```

### Containerizing the application

We need to run the application created by Copilot, so we will ask the chat to help us create a Dockerfile. This file will contain all the necessary dependencies to run the application and will also allow us to deploy it later in the Kubernetes cluster created earlier.

_Prompt to containerize the application:_
```
@workspace Build a Dockerfile to deploy the static website located in the "app" folder #file:server.js
```

1. Create a new `Dockerfile`.
2. Copy Copilot's suggestion into the Dockerfile.
3. Save the changes.

### Running the application

To test the application, we will ask Copilot for the commands needed to build the Docker image and run the web application using the following prompt.

**PROMPT TO USE**
```
@workspace What Docker commands do I need to build and run my web application declared in the following Dockerfile #file:Dockerfile
```

Copilot will most likely suggest the following commands:
```sh
docker build -t solar-system-app:latest . # Builds the Docker image locally on our computer

docker run -p 3000:3000 --name solar-system-website solar-system-app:latest # Creates and runs a container, launching the website
```

## 🔧 5. Creating the Jenkinsfile

Within the web application repository, we need to create a `jenkinsfile` that will contain the code and definition of the pipeline we will use in Jenkins.

_Prompt to create the Jenkinsfile:_
```
@workspace Create a Jenkinsfile to build and publish the Docker image of this web application. The pipeline should include stages for dependency installation, Docker image packaging, and delivery to a container registry. We will use Docker Hub as the registry, so include the use of credentials in the Docker steps. #file:Dockerfile
```

With this, Copilot will suggest a basic pipeline structure that we will modify later.

## 🖥️ 6. Creating Kubernetes manifest (Deployment)

Using GitHub Copilot, we will request the creation of the Kubernetes manifest that we will use to deploy the web application within the cluster we created earlier. Use the following prompt:

_Prompt: Kubernetes manifest:_
```
@workspace Create a k8s.yaml file. This Kubernetes manifest should contain a deployment of the Docker image we created earlier #file:Dockerfile and should match the Docker repository URL in #file:jenkinsfile. Cluster IP should be implemented as a service, and this web application should only be accessible internally.
```

Executing this prompt, Copilot will suggest the necessary YAML structure for the deployment.

Save the suggested code in a new file called `k8s.yaml` in the root of the repository.

## ⚡ 7. Configuring Docker Registry

First, we will configure our Docker Registry or Container Registry, which will serve as the storage repository for the Docker image of our web application. We will use **Docker Hub**. Using GitHub Copilot, we will ask how to use Docker Hub with our Docker repository.

To make these modifications, we will use **GitHub Copilot Edits**, a feature of GitHub Copilot that allows modifying multiple files in a workspace simultaneously.

It is important to add the `jenkinsfile` and `k8s.yaml` files to the Copilot Edits working set.

```
I want to configure my Docker Hub account as a Docker image repository. Modify the Jenkins file and the Kubernetes deployment file to include the following Docker Hub user in the Docker image path as follows: macmoi/solar-system-app:latest
```

Copilot will make modifications within the `jenkinsfile` and `k8s.yaml` files, specifically changing the lines of code where we indicate the image path in the repository. This will correctly configure our Docker user.

> 🚧 **Important: Docker User**  
> In this example, a test Docker user is used. For personal executions, replace this user with your own. Example: For a user named `test`, the image path would be: `test/solar-system-app:latest`

## ☁️ 8. Pushing changes to the repository

Once all configurations and modifications to the application are completed, we proceed to push the changes to the created Git repository.

Simply execute the following commands:

```sh
git status # Check the current branch status.
git add . # Add all changes made.
git commit -m "Initial changes" # Record a change incorporation
git push # Push the changes to the Git repository
```
> 🎯 **Note:** **Incorporations**
> Depending on your repository's authentication method, you may be asked for a key or to configure SSH for authentication. In that case, follow your version control tool's steps for authentication.
> For GitHub, for example, these are the steps:
> [Create a new SSH key pair](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent) and 
> [add SSH key to GitHub account](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/adding-a-new-ssh-key-to-your-github-account)

## 🔨 9. Configuring Jenkins Pipeline and GitHub

At this point, we will use GitHub Copilot as an assistant to guide us in configuring a new pipeline in Jenkins using the following prompt.

```
@workspace I need you to provide the steps to configure this application in an automated Jenkins pipeline #file:jenkinsfile
```

After this, GitHub Copilot will suggest the steps to create a new Pipeline in Jenkins. If you encounter issues, it is recommended to review the steps in the following troubleshooting.

> **IMPORTANT**  
> Ensure that the Docker and NodeJS plugins are installed when creating the pipeline. If not installed, go to Jenkins plugin configurations, search for the NodeJS and Docker plugins, select and install them.  
> Also, configure the Tool to have NodeJS enabled and available for use in the pipelines. Follow the steps in this guide: [NodeJS Configuration](https://plugins.jenkins.io/nodejs/)

## 🚀 10. Deploying in a local environment using Kind

In this step, we will use GitHub Copilot to guide us through the steps to deploy our application locally. Use the following prompt:

_Prompt:_
```
@workspace I want you to provide the steps to correctly deploy this Kubernetes deployment manifest in my local cluster created with Kind. This application should be accessible from my local computer.
```

At this point, Copilot will suggest the steps to follow and provide the necessary Kubectl commands to execute the deployment.

```sh
# Commands suggested by Copilot.

kubectl apply -f app/k8s.yaml # Applies the Kubernetes manifest in the cluster
kubectl get deployments # Displays the deployments
kubectl get services # Displays the services
```

To access the application locally from our browser, we need to map ports. We will ask Copilot to provide the "port forwarding" command to perform this task.

_Prompt_
```
@workspace I need to access my web application deployed using the manifest #file:k8s.yaml from my local computer. Provide the port forwarding command I need to execute to make my application available and accessible from my web browser on my local machine. Remember, I am running this from KIND.
```

Copilot will most likely suggest the following command:

```sh
kubectl port-forward service/app-service 3000:3000
```

Once this command is executed, we can access the application from our browser using the following URL: http://localhost:3000

> **IMPORTANT**  
> If the port forwarding command cannot map port 3000, check if the port is being used by another application, or simply change the port value to another one.

## 🗑️ 11. Cleaning up resources

Congratulations, you have reached the end of this practical exercise. Use the following commands to remove the resources used on your computer.

### Destroy Docker Compose resources
```sh
docker compose down -v # Removes Docker containers and their volumes (NOT REVERSIBLE).
```

### Destroy the local Kubernetes cluster
```sh
kind get clusters # Lists available clusters.
kind delete clusters "copilot-devops-demo" # Deletes the cluster used in this exercise
```
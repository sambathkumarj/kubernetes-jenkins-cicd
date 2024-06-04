# kubernetes-jenkins-cicd

# Jenkins
Jenkins is an open-source automation server written in Java. It helps automate parts of the software development process with continuous integration and continuous delivery (CI/CD). Jenkins supports building, deploying, and automating software projects. It can integrate with various version control systems and development, testing, and deployment technologies.

# Git
 A distributed version control system (VCS) that allows developers to track code changes, collaborate effectively, and revert to previous versions if necessary. It's a popular choice for managing source code in software projects.

 # Docker
A containerization platform that packages an application with its dependencies and configuration into a standardized unit called a container. Containers are lightweight and portable, ensuring consistent execution across environments.


# Kubernetes 
An open-source container orchestration system that automates container deployment, scaling, and management. It groups containers into logical units called pods, facilitates scaling based on resource needs, and handles service discovery for containerized applications.

# prerequisites

1. Jenkins - As a service
2. Github Repo - Web application & docker file & Manifest file
3. Docker Hub Account
4. Existing Kubernetes Cluster - Microk8s


# Jenkins Installation as a service
To install Jenkins as a service on an Ubuntu machine, follow these steps:

# Step 1: Update Your System

Before installing any new package, it's good practice to update your package index.

```
sudo apt update
```

# Step 2: Install Java

Jenkins requires Java to run. You can install OpenJDK, which is the recommended version.

```
sudo apt install openjdk-11-jdk -y
```

# Step 3: Add Jenkins Repository

1. Import the GPG key:

   ```
   curl -fsSL https://pkg.jenkins.io/debian-stable/jenkins.io.key | sudo tee /usr/share/keyrings/jenkins-keyring.asc > /dev/null
   ```

2. Add the Jenkins repository:

   ```
   echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] https://pkg.jenkins.io/debian-stable binary/ | sudo tee /etc/apt/sources.list.d/jenkins.list > /dev/null
   ```

# Step 4: Install Jenkins

After adding the repository, install Jenkins.

```
sudo apt update
sudo apt install jenkins -y
```

# Step 5: Start and Enable Jenkins

Once Jenkins is installed, start the service and enable it to start on boot.

```
sudo systemctl start jenkins
sudo systemctl enable jenkins
```

# Step 6: Adjust Firewall

If you have a firewall enabled, allow traffic on port 8080, which is the default port Jenkins runs on.

```
sudo ufw allow 8080
sudo ufw status
```

# Step 7: Setup Jenkins

1. Access Jenkins: Open your web browser and go to `http://your_server_ip_or_domain:8080`.

2. Retrieve the Initial Admin Password: To unlock Jenkins, you'll need the initial admin password. You can find it in the following file:

   ```
   sudo cat /var/lib/jenkins/secrets/initialAdminPassword
   ```

3. Finish the Setup: Follow the on-screen instructions to finish setting up Jenkins. You can install suggested plugins and create your first admin user.

# Step 8: Install and Configure Plugins

After the initial setup, you can install necessary plugins like Git, Docker, and Kubernetes.

1. Git Plugin: Go to `Manage Jenkins` > `Manage Plugins` > `Available`, search for "Git plugin", and install it.

2. Docker Plugin: Similarly, search for and install the "Docker plugin".

3. Kubernetes Plugin: Search for and install the "Kubernetes plugin".
   
# Step 9: Configure Jenkins

1.  Global Tool Configuration:
- Go to `Manage Jenkins` > `Global Tool Configuration`.
- Set up JDK installations if needed.
- Set up Git installations by specifying the Git executable if it's not installed by default.

2.  Credentials:
- Go to `Manage Jenkins` > `Manage Credentials`.
- Add credentials for accessing your Git repository, Docker registry, and Kubernetes cluster.

3. Job Setup:
-  Create a new Jenkins job and configure it to pull from your Git repository, build Docker images, and deploy to your Kubernetes cluster using the pipeline script you define.


# First, we will understand the requirement

1. The team of developers working on new features will merge their code to a GitHub repo.
2. As soon as the code reaches GitHub, using a CI (Continuous Integration) pipeline, setup in Jenkins, automated builds will be triggered
3. The automated builds will frequently deploy new features to the production website
4. Every build will prepare a Dockerfile and push docker images to docker-hub
5. Every docker image will be deployed (Continuous Deployment) to a Kubernetes cluster.

So now let’s make the architecture of our application. This will represent the workflow of the CI-CD pipeline.

![kube-jenkin-cicd](https://github.com/sambathkumarj/kubernetes-jenkins-cicd/assets/42794636/71a775a0-3baa-42d0-aead-6b3fe147ad7c)

1. Developer to push code on the Source code repository
2. Jenkins to fetch the code and build the docker image
3. Push Docker image to Dockerhub
4. Kubernetes to pull the same image from Dockerhub and deploy it to Kubernetes Cluster

# Workflow with Git, Jenkins, Docker, and Kubernetes

Create the Multi-Branch Pipeline to deploy workflow;

1. Code Changes in Git: commit code changes to a Git repository, typically hosted on platforms like GitHub, GitLab, or Bitbucket.

2. Triggering Jenkins Job: An event in the Git repository, such as a push or pull request, triggers a corresponding Jenkins job. This can be configured using plugins like the Git plugin or webhooks provided by the Git hosting platform.
     
3. Jenkins Fetches Code: The triggered Jenkins job initiates by cloning the code from the Git repository using the provided URL and credentials.

   ![git_repo_cred](https://github.com/sambathkumarj/kubernetes-jenkins-cicd/assets/42794636/8e3ff712-2bcb-4310-bccf-e2630184327b)
    
4. Building Docker Image: Jenkins executes the commands specified in a Dockerfile (a text file defining the container image build process).

The Dockerfile typically includes steps to:
• Define a base image (e.g., Node.js image for a web application)

• Copy the application code from the Git repository into the container
• Install dependencies (e.g., using npm install or apt-get install)
• Expose the application's port (e.g., EXPOSE 8080 for a web server running on port 8080)
• Define the application's entry point (e.g., the command to start the web server)
◦ Jenkins builds and pushes the Docker image to a Docker registry (e.g., Docker Hub, a private registry).

5. Deployment to Kubernetes:
◦ After successful image building, Jenkins uses plugins like the Kubernetes plugin to interact with your Kubernetes cluster.
◦ Jenkins submits deployment or pod creation manifests (often in YAML format) to the Kubernetes API. These manifests describe how the containerized application should be deployed, including:

# Manifest file (YAML files):

Pod.yaml, secret.yaml,& svc.yaml create these files with the docker image which is built in CI/CD 

# Pod.yaml

```
microk8s kubectl run kube-jenkin-cicd --image=docker.io/sambathkumarj/kube-jenkin-cicd --dry-run=client -o yaml > pod.yaml
```

# Secret.yaml

```
microk8s kubectl create secret docker-registry dockerhub --docker-server=docker.io --docker-username='qwertyuiop' --docker-password='asdfghjkl' --dry-run=client -o yaml > secret.yaml
```

# svc.yaml

```
microk8s kubectl create svc nodeport kube-jenkin-cicd --tcp=80:80 --dry-run=client -o yaml > svc.yaml
```

▪ Pod specifications (number of replicas, resource requests/limits, volume mounts)
▪ Service specifications (defining how to expose the application externally, e.g., using a LoadBalancer service)

6. Kubernetes Manages Deployment:
◦ Kubernetes receives the manifests, schedules the containers across available nodes in the cluster, ensures pod health and restarts if necessary, and manages service discovery (making the running application accessible).

# Credentials:

- Go to `Manage Jenkins` > `Manage Credentials`.
- Add credentials for accessing your Git repository, Docker registry, and Kubernetes cluster.

Github -> github (Mentioned while creating Multi-branch Pipeline)
Docker-Registry ->  dockerhub
Kubernetes-Cluster -> Kube-config-k8s (Secret file - credential)

![cred_store](https://github.com/sambathkumarj/kubernetes-jenkins-cicd/assets/42794636/9844ff9e-bd84-4a48-9776-d3072eb7abc4)


# Job Pipeline (Groovy Syntax - Multibranch Pipeline)

```

pipeline{

	agent any

	environment {
		DOCKERHUB_CREDENTIALS=credentials('dockerhub')
	}

	stages {
		stage('Build') {

			steps {
				sh 'docker build -t sambathkumarj/kube-jenkin-cicd:latest .'
			}
		}

		stage('Login') {

			steps {
				sh 'echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin'
			}
		}

		stage('Push') {

			steps {
				sh 'docker push sambathkumarj/kube-jenkin-cicd:latest'
			}
		}

		stage('Deploy to Kubernetes'){
			steps{
				withKubeConfig([credentialsId: 'Kube-config-k8s']) {
					sh 'kubectl apply -f pod.yaml'
					sh 'kubectl apply -f svc.yaml'
					sh 'kubectl apply -f secret.yaml'
				}
			}
		}	
	}
}

```

![project_success_2](https://github.com/sambathkumarj/kubernetes-jenkins-cicd/assets/42794636/1ddbd705-6f17-4d47-bf7e-672f08ee5fb0)



# Key Points and Benefits

• Repeatable Builds: Jenkins ensures consistent build environments across different machines.
• Automated Testing: Integration with testing frameworks allows for automated execution of tests during the build process.
• Fast Rollbacks: Easy rollbacks to previous versions if deployments encounter issues.
• Scalability: Kubernetes facilitates scaling applications up or down based on resource demands.
• Streamlined Workflow: This integration empowers developers to focus on coding while automation handles the build, test, and deployment process.

![k8s_res](https://github.com/sambathkumarj/kubernetes-jenkins-cicd/assets/42794636/3093b2cb-7afc-4d6a-9912-c95c02cac8df)



# Additional Considerations

• Security: Securely store Git credentials and Docker registry access in Jenkins.
• Version Control: Implement versioning in Docker images to track changes and facilitate rollbacks.
• Monitoring: Monitor deployed applications in Kubernetes for performance and resource usage.

![output](https://github.com/sambathkumarj/kubernetes-jenkins-cicd/assets/42794636/df135313-43f6-497d-85d6-1baac4d81e0c)



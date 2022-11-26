## Deploying a Spring Boot application on AKS Cluster using Jenkins and Storing the Artifacts on ACR.

### Pre-requisites:
  - AKS cluster needs to be up running.
  - Jenkins should be up and running.
  - Make sure to Install Docker, Docker pipeline and Kubectl CLI plug-ins are installed in Jenkins.
  - Install Docker in Jenkins and Jenkins have proper permission to perform Docker builds.
  - Install Kubectl on Jenkins instance, as we will use that Jenkins Instance to deploy the manifests onto our cluster on to AKS.
  - Make sure that ACR is also setup in Azure cloud.
  - Make sure AKS has pull access from ACR.
  - Dockerfile created along with the application source code for springboot App, for building the docker image.
  - For building the image onto the Jenkins Instance, docker should be installed on Jenkins Instance and make sure to add the jenkins user to docker group.
  - Install Azure CLI on your local machine. (We will be creating the AKS cluster from our local machine)
  
## STEP: 1
### Create Credentials to connect to ACR from Jenkins:
  - Go to Azure Portal console, go to container registry
  - Settings--> Access keys
  - Get the username and password from there.
  - Now, Go to Jenkins-> Manage Jenkins. Create credentials.
    
![image](https://user-images.githubusercontent.com/92631457/204107836-205849f4-ed56-41c1-b78f-6c1235c7e362.png)
    
  - Enter ID as ACR( This ID is important as It is used to make the Jenkins Pipeline) and enter some text for description and Save.
  
## STEP: 2
### Create Credentials for connecting to AKS cluster using Kubeconfig:
  - In the same way add another credentials for accessing the kubernetes cluster from Jenkins instance, As we want to deploy our manifests using Jenkins only. 
    
    

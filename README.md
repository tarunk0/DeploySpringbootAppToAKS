## Deploying a Spring Boot application on AKS Cluster using Jenkins and Storing the Artifacts on ACR.

### Flow of the Pipeline:

![image](https://user-images.githubusercontent.com/92631457/204109168-526bbe11-39f8-4640-a6e5-8f6b771e923d.png)


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
  - Go to Jenkins UI, click on Credentials and select the secret file type credentials and choose the kubeconfig file and Give the ID as K8S, and Some description and Save it.
  
  ![image](https://user-images.githubusercontent.com/92631457/204108693-7ef05cd4-d4c0-4757-82d3-cea2832aec3f.png)

  ![image](https://user-images.githubusercontent.com/92631457/204108724-1d678a0f-9a17-4751-b662-de50b19351f5.png)

 ## STEP: 3
 ### Create a pipeline in Jenkins:
 - Create a new pipeline job.
 - Copy use the below pipeline code and change the respective values. 
 
```sh
   pipeline {
    tools {
        maven "Maven3"
    }
    agent any
    
    environment {
    registryName = "myacrrepo8459"
    registryCredential = 'ACR'
    dockerImage = ''
    registryUrl = 'myacrrepo8459.azurecr.io'
    }
    stages {
        stage('Checkout') {
            steps {
                checkout([$class: 'GitSCM', branches: [[name: '*/main']], extensions: [], userRemoteConfigs: [[url: 'https://github.com/tarunk0/DeploySpringbootAppToAKS.git']]])
        }
    }
        stage('Build Code and Make Jar File') {
            steps {
                sh "mvn clean install"
            }
        }
        stage('Build Docker Image') {
            steps {
                script {
                   dockerImage = docker.build registryName
                }
                
            }
        }
        stage('Push Docker Image to ACR') {
            steps {
                script {
                docker.withRegistry( "http://${registryUrl}", registryCredential ) {
                dockerImage.push()
                }
                
            }
        }
      } 
        stage('Deploy to AKS Cluster') {
            steps {
                script {
                withKubeConfig([credentialsId: 'K8S', serverUrl: '']) {
                sh ('kubectl apply -f  jenkins-aks-deploy-from-acr.yaml')
                }
                
            }
        }
      } 
    }
  }
```
    
![Screenshot 2022-11-27 at 12 35 41 AM](https://user-images.githubusercontent.com/92631457/204108876-61ad318f-4963-4969-a853-8bc899447089.png)

![Screenshot 2022-11-27 at 12 42 38 AM](https://user-images.githubusercontent.com/92631457/204108893-88c4d2a2-20fb-4c28-bf56-2b1ed6ed0ca2.png)

![Screenshot 2022-11-27 at 12 35 41 AM](https://user-images.githubusercontent.com/92631457/204108910-70860cf4-0116-49dd-8733-ecad896a630e.png)

![Screenshot 2022-11-27 at 12 20 55 AM](https://user-images.githubusercontent.com/92631457/204108915-2c72faec-13cb-455a-9f3e-25e6c2448b04.png)

![Screenshot 2022-11-27 at 12 20 33 AM](https://user-images.githubusercontent.com/92631457/204108919-e215fbf1-cc16-4bcb-b0fc-51bb087422c2.png)

![Screenshot 2022-11-27 at 12 19 05 AM](https://user-images.githubusercontent.com/92631457/204108922-1e595786-b0dc-4ec0-a5d9-9b195ac23d69.png)


![Screenshot 2022-11-27 at 12 11 26 AM](https://user-images.githubusercontent.com/92631457/204108925-4e01acee-453d-49a5-ad6c-05cb6611fc3d.png)


![Screenshot 2022-11-27 at 1 41 23 AM](https://user-images.githubusercontent.com/92631457/204108928-4d0ddfd4-1fc7-4ad7-bcec-a2e3d265db58.png)


![Screenshot 2022-11-27 at 1 41 01 AM](https://user-images.githubusercontent.com/92631457/204108931-dad58758-ef87-4178-bfdb-cac79b66b7ab.png)


![Screenshot 2022-11-27 at 1 39 43 AM](https://user-images.githubusercontent.com/92631457/204108936-de7d8be7-650e-4e57-ba3e-5d19f75bb58d.png)


![Screenshot 2022-11-27 at 1 18 46 AM](https://user-images.githubusercontent.com/92631457/204108942-ce8a54d6-5e88-4a2b-96c5-f37a3bf56cfa.png)

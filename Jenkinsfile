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
# BlueGreenDeployment on AWS EKS with Jenkins Using Terraform



## Project Overview

This project demonstrates how to:

- Provision an **Amazon EKS Kubernetes cluster** with **Terraform**
- Set up **Jenkins** on Kubernetes (EKS)
- Deploy a **Node.js application** with a **blue-green deployment strategy**
- Use **kubectl service patching** to switch traffic between blue and green versions

- Install required tools on a fresh Ubuntu EC2 instance (Java, Maven, Docker, jenkins, kubectl)
1.aws configure
# Enter Access Key, Secret, region (e.g., us-east-1), and json
2 Update and Install required Applications Java, Mvn, Docker, Kubectl,jenkins
Upadte instance
* sudo apt update && sudo apt upgrade -y
Insatll Java
*sudo apt install -y openjdk-17-jdk
* java -version
Install mvn
*sudo apt install -y maven
* mvn -version

Install Docker
sudo apt install -y docker.io
sudo systemctl enable docker
sudo systemctl start docker
# Add current user to docker group (no need for sudo)
sudo usermod -aG docker $USER
newgrp docker  # Apply group change
docker –version

Created git repository  :https://github.com
Repository name reported : bluegreendeployment
cd your-project-folder/
git init
git add .
git commit -m "Initial commit"
Connted to GitHub remote.
git remote add origin https://github.com/maddinenisravan/BlueGreenDeployment.git 
New branch created 
git branch -M main
git push -u origin main



Provision EKS with Terraform (from cluster/ folder)

cd cluster/
terraform init
terraform apply -auto-approve


connect to eks cluster

aws eks --region us-east-1 update-kubeconfig --name devopsshack-cluster
kubectl get nodes


Destory cluster:

terraform delete 
terraform destroy -auto-approve.

Install Jenkins:

sudo apt update
sudo apt install -y Jenkins
Jenkins -version


Add Jenkins GPG Key and Repository:

add key: 
 wget -q -O - https://pkg.jenkins.io/debian-stable/jenkins.io.key | sudo tee \
  /usr/share/keyrings/jenkins-keyring.asc > /dev/null


add repo:

echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
  https://pkg.jenkins.io/debian-stable binary/ | \
  sudo tee /etc/apt/sources.list.d/jenkins.list > /dev/null

Adjust Firewall (if UFW is enabled)

sudo systemctl start jenkins
sudo systemctl enable jenkins
sudo systemctl status Jenkins

initial setup

Open your web browser and navigate to http://your_server_ip_or_domain:8080. 

Get initial admin password
sudo cat /var/lib/jenkins/secrets/initialAdminPassword

Create your first admin user.
login & password


Install Required Jenkins Plugins

Kubernetes plugin

Pipeline plugin

Git plugin

Docker Pipeline

Deploy Blue-Green Node.js App


Jenkins Pipeline (Groovy) for Blue-Green Deployment

pipeline {
    agent any
    
    parameters {
        choice(name: 'DEPLOY_ENV', choices: ['blue', 'green'], description: 'Choose which environment to deploy: Blue or Green')
        choice(name: 'DOCKER_TAG', choices: ['blue', 'green'], description: 'Choose the Docker image tag for the deployment')
        booleanParam(name: 'SWITCH_TRAFFIC', defaultValue: false, description: 'Switch traffic between Blue and Green')
    }
    
    environment {
        IMAGE_NAME = "maddinenisravan/blue-green-deployment"
        TAG = "${params.DOCKER_TAG}"  // The image tag now comes from the parameter
        KUBE_NAMESPACE = 'webapps'
        SCANNER_HOME= tool 'sonar-scanner'
    }

    stages {
        stage('Git Checkout') {
            steps {
                checkout scmGit(branches: [[name: '*/main']], extensions: [], userRemoteConfigs: [[url: 'https://github.com/maddinenisravan/Blue-Green-Deployment.git']])
            }
        }
        stage('compile') {
            steps {
                sh "mvn compile"
            }
        }
        stage('Test') {
            steps {
                sh "mvn test -DskipTests=true"
            }
        } 
        stage('Build') {
            steps {
                sh "mvn package -DskipTests=true"
            }
        }
        stage('Docker build') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker-cred') {
                        sh "docker build -t ${IMAGE_NAME}:${TAG} ."
                    }
                }
            }
        }
        stage('Docker Push Image') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker-cred') {
                        sh "docker push ${IMAGE_NAME}:${TAG}"
                    }
                }
            }
        }
        
        stage('Deploy MySQL Deployment and Service') {
            steps {
                script {
                    withKubeConfig(caCertificate: '', clusterName: 'devopsshack-cluster', contextName: '', credentialsId: 'k8-token', namespace: 'webapps', restrictKubeConfigAccess: false, serverUrl: 'https://D743070524A400E79C6AD2AF7967C4A9.gr7.us-east-1.eks.amazonaws.com') {
                        sh "kubectl apply -f mysql-ds.yml -n ${KUBE_NAMESPACE}"  // Ensure you have the MySQL deployment YAML ready
                    }
                }
            }
        }
        
        stage('Deploy SVC-APP') {
            steps {
                script {
                    withKubeConfig(caCertificate: '', clusterName: 'devopsshack-cluster', contextName: '', credentialsId: 'k8-token', namespace: 'webapps', restrictKubeConfigAccess: false, serverUrl: 'https://D743070524A400E79C6AD2AF7967C4A9.gr7.us-east-1.eks.amazonaws.com') {
                        sh """ if ! kubectl get svc bankapp-service -n ${KUBE_NAMESPACE}; then
                                kubectl apply -f bankapp-service.yml -n ${KUBE_NAMESPACE}
                              fi
                        """
                   }
                }
            }
        }
        
        stage('Deploy to Kubernetes') {
            steps {
                script {
                    def deploymentFile = ""
                    if (params.DEPLOY_ENV == 'blue') {
                        deploymentFile = 'app-deployment-blue.yml'
                    } else {
                        deploymentFile = 'app-deployment-green.yml'
                    }

                    withKubeConfig(caCertificate: '', clusterName: 'devopsshack-cluster', contextName: '', credentialsId: 'k8-token', namespace: 'webapps', restrictKubeConfigAccess: false, serverUrl: 'https://D743070524A400E79C6AD2AF7967C4A9.gr7.us-east-1.eks.amazonaws.com') {
                        sh "kubectl apply -f ${deploymentFile} -n ${KUBE_NAMESPACE}"
                    }
                }
            }
        }
        
        stage('Switch Traffic Between Blue & Green Environment') {
            when {
                expression { return params.SWITCH_TRAFFIC }
            }
            steps {
                script {
                    def newEnv = params.DEPLOY_ENV

                    // Always switch traffic based on DEPLOY_ENV
                    withKubeConfig(caCertificate: '', clusterName: 'devopsshack-cluster', contextName: '', credentialsId: 'k8-token', namespace: 'webapps', restrictKubeConfigAccess: false, serverUrl: 'https://D743070524A400E79C6AD2AF7967C4A9.gr7.us-east-1.eks.amazonaws.com') {
                        sh '''
                            kubectl patch service bankapp-service -p "{\\"spec\\": {\\"selector\\": {\\"app\\": \\"bankapp\\", \\"version\\": \\"''' + newEnv + '''\\"}}}" -n ${KUBE_NAMESPACE}
                        '''
                    }
                    echo "Traffic has been switched to the ${newEnv} environment."
                }
            }
        }
        
        stage('Verify Deployment') {
            steps {
                script {
                    def verifyEnv = params.DEPLOY_ENV
                    withKubeConfig(caCertificate: '', clusterName: 'devopsshack-cluster', contextName: '', credentialsId: 'k8-token', namespace: 'webapps', restrictKubeConfigAccess: false, serverUrl: 'https://D743070524A400E79C6AD2AF7967C4A9.gr7.us-east-1.eks.amazonaws.com') {
                        sh """
                        kubectl get pods -l version=${verifyEnv} -n ${KUBE_NAMESPACE}
                        kubectl get svc bankapp-service -n ${KUBE_NAMESPACE}
                        """
                    }
                }
            }
        }
    }
}



Install Kubectl
curl -LO "https://dl.k8s.io/release/$(curl -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
kubectl version –client



Switch Traffic Between Blue & Green

kubectl patch svc node-service -p '{"spec": {"selector": {"app": "node", "version": "green"}}}'


Destory cluster:

terraform delete 
terraform destroy -auto-approve.











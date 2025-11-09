# DevSecOps With Docker Scout Hotstar Clone

# Implementing a Full DevSecOps CI/CD Pipeline for Microservices on Kubernetes

This project demonstrates the complete end-to-end automation of microservice deployment using a modern DevOps toolchain. The architecture integrates Continuous Integration (CI) and Continuous Deployment (CD) pipelines built with Jenkins, SonarQube, Trivy, ArgoCD, and Kubernetes. Each component ensures that source code from GitHub is automatically built, scanned, tested, containerized, and deployed onto Kubernetes clusters with continuous monitoring via Prometheus and Grafana. This approach enhances deployment speed, quality assurance, and system observability while reducing manual intervention.

**Project Deployment Architecture**

## Tech stack used in this project:

- GitHub (Code)
- Docker (Containerization)
- Jenkins (CI)
- OWASP (Dependency check)
- SonarQube (Quality)
- Trivy (Filesystem Scan)
- ArgoCD (CD)
- AWS EKS (Kubernetes)
- Helm (Monitoring using grafana and prometheus)

### How pipeline will look after deployment:

- CI pipeline to build and push

- CD pipeline to update application version
- Docker Image after tag & push in JioHotstar-CI pipeline

- ArgoCD application for deployment on EKS

### Deployed Hotstar Clone

### Pre-requisites to implement this project:

This project is implemented on Asia pacific Mumbai (ap-south-1).

- Create 1 Master machine (microservice-project-goutm) on AWS with 2CPU, 8GB of RAM (t3.large) and 30 GB of storage

- Open the below ports in security group of master machine and also attach same security group to Jenkins worker node (We will create worker node shortly) 

We are creating this master machine (microservice-project-goutm) because we will configure Jenkins, eksctl, EKS cluster creation from here.

Go to master machine (microservice-project-goutm) server and give the below commands :

1.  **Install & Configure Docker by using below commands**

sudo apt-get update

sudo apt-get install docker.io -y  
sudo usermod -aG docker ubuntu && newgrp docker

\# To provide permission to docker socket so that docker build and push command do not fail give below command

sudo chmod 777 /var/run/docker.sock

1.  **Install and configure Jenkins**

sudo apt update -y  
sudo apt install fontconfig openjdk-17-jre -y  
<br/>sudo wget -O /usr/share/keyrings/jenkins-keyring.asc \\  
https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key  
<br/>echo "deb \[signed-by=/usr/share/keyrings/jenkins-keyring.asc\]" \\  
https://pkg.jenkins.io/debian-stable binary/ | sudo tee \\  
/etc/apt/sources.list.d/jenkins.list > /dev/null  
<br/>sudo apt-get update -y  
sudo apt-get install jenkins -y

- Now, access Jenkins on the browser on port 8080 and configure it. (&lt;public-ip&gt;:8080)

1.  **Create EKS Cluster on AWS**
    - Create IAM user with **access keys and secret access keys**
    - AWSCLI should be configured by using below commands

- curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"  
    sudo apt install unzip  
    unzip awscliv2.zip  
    sudo ./aws/install  
    aws configure

- 
- 

- - Install **kubectl**
- curl -o kubectl https://amazon-eks.s3.us-west-2.amazonaws.com/1.19.6/2021-01-05/bin/linux/amd64/kubectl  
    chmod +x ./kubectl  
    sudo mv ./kubectl /usr/local/bin  
    kubectl version --short –client
- - Install **eksctl**
- curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)\_amd64.tar.gz" | tar xz -C /tmp  
    sudo mv /tmp/eksctl /usr/local/bin  
    eksctl version
- - Create EKS Cluster using eksctl command

eksctl create cluster --name=hotstar-clone --version 1.30 --zones=ap-south-1a,ap-south-1b,ap-south-1c --without-nodegroup

- - Associate IAM OIDC Provider

eksctl utils associate-iam-oidc-provider --region ap-south-1 --cluster hotstar-clone –approve

- - Create Nodegroup using eksctl command

eksctl create nodegroup --cluster=hotstar-clone --region=ap-south-1 --name=hotstar-clone-ng-1 --node-type=t2.large --nodes=2 --nodes-min=2 --nodes-max=3 --node-volume-size=29 --ssh-access --ssh-public-key=eks-nodegroup-key-mumbai --managed --asg-access --external-dns-access --full-ecr-access --appmesh-access --alb-ingress-access

Make sure the ssh-public-key “eks-nodegroup-key-mumbai” is available in your aws account.

1.  **Install and configure SonarQube (Master machine)**

docker run -itd --name SonarQube-Server -p 9000:9000 sonarqube:lts-community

and give “docker ps -a” command to check if the container is up or not.

1.  **Install Trivy (Master machine)**

sudo apt-get install wget apt-transport-https gnupg lsb-release -y  
wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key | sudo apt-key add -  
echo deb https://aquasecurity.github.io/trivy-repo/deb $(lsb_release -sc) main | sudo tee -a /etc/apt/sources.list.d/trivy.list  
sudo apt-get update -y  
sudo apt-get install trivy -y

1.  **Install Helm (Master Machine)**

curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3  
chmod 700 get_helm.sh  
./get_helm.sh  
helm version

1.  **Install argocd through Helm (Master Machine)**

helm repo add argo-cd https://argoproj.github.io/argo-helm

helm repo update  
kubectl create namespace argocd  
helm install argocd argo-cd/argo-cd -n argocd

- - Check argocd services
- kubectl get all -n argocd
    - Change argocd server’s service from ClusterIP to NodePort
- kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "LoadBalancer"}}'
    - Confirm service is patched or not
- kubectl get all -n argocd
- - Access it on browser, click on advance and proceed with the load balancer dns.
    - Fetch the initial password of argocd server
- kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d; echo

1.  **Steps to add email notification**
    - Go to your Jenkins Master EC2 instance and allow 465 port number for SMTPS
    - Now, we need to generate an application password from our gmail account to authenticate with Jenkins
    - Open gmail and go to Manage your Google Account –> Security > \[!Important\] > Make sure 2 step verification must be on

- - Search for App password and create a app password for Jenkins

- - Once, app password is create and go back to jenkins Manage Jenkins –> Credentials to add username and password for email notification

- - Go back to Manage Jenkins –> System and search for Extended E-mail Notification
    - Scroll down and search for E-mail Notification and setup email notification -> Enter your gmail and password which we copied recently in password field E-mail Notification –> Advance -> Save

## Steps to implement the project:

- Go to Jenkins Master and click on Manage Jenkins –> Plugins –> Available plugins install the below plugins:
    - Pipeline: Stage View
    - Docker Pipeline
    - Docker Eclipse Temurin Installer
    - NodeJS
    - SonarQube Scanner
    - OWASP Dependency-Check

After installing restart the Jenkins

- Now move to Manage jenkins –> Tools
    - For NodeJs use version NodeJs 16.2.0
    - For Jdk17 use version jdk-17.0.8.1+1
    - For Dependency Check use latest version
    - For Sonar Scanner use latest version
- Login to SonarQube server and create the credentials and webhook for jenkins to integrate with SonarQube
    - Navigate to Administration –> Security –> Users –> Token 

- - Now go to Administration –> Webhook and click on create webhook to integrate with Jenkins

- Now, go to Manage Jenkins –> credentials and add Sonarqube credentials using kind as Secret text: 
- Go to Manage Jenkins –> credentials and add Github credentials to push updated code from the pipeline:

- - While adding github credentials add Personal Access Token in the password field.
- Go to Manage Jenkins –> System and search for SonarQube installations:
    - Add name -> Sonarqube server link (&lt;public-ip&gt;:9000) -> sonar-token (sonarqube credentials)
- Navigate to Manage Jenkins –> credentials and add credentials for docker login to push docker image: 
    - While adding DockerHub credentials add Personal Access Token in the password field.
- Create a JioHotstar-CI pipeline and JioHotstar-CD pipeline
    - For pipeline scripts go to https://github.com/goutham1404/hotstar-clone.git

- Go to argoCD and connect your github repo
    - Go to Settings -> Repositories -> connect repo

- - Make sure that connection status is successful
- Now, go to Applications and click on New App your application is deployed on AWS EKS Cluster

- The deployed JioHotstar web application

- Email Notification

# 

## To monitor EKS cluster, kubernetes components and workloads using prometheus and grafana via HELM (On Master machine)

- Add Helm Stable Charts for Your Local Client

helm repo add stable https://charts.helm.sh/stable

- Add Prometheus Helm Repository

helm repo add prometheus-community https://prometheus-community.github.io/helm-charts

- Create Prometheus Namespace

kubectl create namespace prometheus

kubectl get ns

- Install Prometheus using Helm

helm install stable prometheus-community/kube-prometheus-stack -n prometheus

- Expose Prometheus and Grafana to the external world through Node Port and change it from Cluster IP to NodePort after changing make sure you save the file and open the assigned nodeport to the service.

kubectl edit svc stable-kube-prometheus-sta-prometheus -n Prometheus

- Verify service

kubectl get svc -n prometheus

- Now,let’s change the SVC file of the Grafana and expose it to the outer world

kubectl edit svc stable-grafana -n prometheus

- Get a password for grafana

kubectl get secret --namespace prometheus stable-grafana -o jsonpath="{.data.admin-password}" | base64 --decode ; echo

- Now, view the dashboard in Prometheus

- Now, view the Dashboard in Grafana 

# 

## To Clean Up EKS Cluster

- Delete eks cluster

eksctl delete cluster --name=hotstar-clone --region=ap-south-1

- To Delete eks cluster nodes

eksctl delete nodegroup --cluster hotstar-clone --name hotstar-clone-ng-1

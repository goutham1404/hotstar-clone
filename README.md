
# Implementing a Full DevSecOps CI/CD Pipeline for Microservices on Kubernetes

This project demonstrates the complete end-to-end automation of microservice deployment using a modern DevOps toolchain. The architecture integrates Continuous Integration (CI) and Continuous Deployment (CD) pipelines built with Jenkins, SonarQube, Trivy, ArgoCD, and Kubernetes. Each component ensures that source code from GitHub is automatically built, scanned, tested, containerized, and deployed onto Kubernetes clusters with continuous monitoring via Prometheus and Grafana. This approach enhances deployment speed, quality assurance, and system observability while reducing manual intervention.

**Project Deployment Architecture**
<img width="1125" height="633" alt="image" src="https://github.com/user-attachments/assets/7907aee1-6163-46d6-a286-44fe9702db2c" />

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

<img width="963" height="431" alt="image" src="https://github.com/user-attachments/assets/668919fb-030c-4da1-a5dc-6213c77bd207" />

- CD pipeline to update application version

<img width="983" height="320" alt="image" src="https://github.com/user-attachments/assets/83b1165a-4d9f-41a9-9abd-be943d05abc3" />

- Docker Image after tag & push in JioHotstar-CI pipeline

<img width="883" height="457" alt="image" src="https://github.com/user-attachments/assets/3dc27d77-d123-4f97-8d9a-8e58238ba02a" />

- ArgoCD application for deployment on EKS

<img width="948" height="428" alt="image" src="https://github.com/user-attachments/assets/c3416015-604a-4b33-a8b1-fa7bf55e7077" />

### Deployed Hotstar Clone

<img width="961" height="557" alt="image" src="https://github.com/user-attachments/assets/6ac08a04-b319-48d5-b1c2-5f733c0a352e" />

### Pre-requisites to implement this project:

This project is implemented on Asia pacific Mumbai (ap-south-1).

- Create 1 Master machine (microservice-project-goutm) on AWS with 2CPU, 8GB of RAM (t3.large) and 30 GB of storage
  
<img width="871" height="180" alt="image" src="https://github.com/user-attachments/assets/5341eb2f-f006-454e-a820-8d78f76d0702" />

- Open the below ports in security group of master machine
 
<img width="875" height="359" alt="image" src="https://github.com/user-attachments/assets/7fce01e1-0984-4acd-a719-25be80fe980f" />

We are creating this master machine (microservice-project-goutm) because we will configure Jenkins, eksctl, EKS cluster creation from here.

Go to master machine (microservice-project-goutm) server and give the below commands :

1.  **Install & Configure Docker by using below commands**
'''
sudo apt-get update

sudo apt-get install docker.io -y  
sudo usermod -aG docker ubuntu && newgrp docker
'''
To provide permission to docker socket so that docker build and push command do not fail give below command

sudo chmod 777 /var/run/docker.sock

<img width="975" height="149" alt="image" src="https://github.com/user-attachments/assets/744ab9de-1f68-4f90-a912-8ba6b8834714" />

2.  **Install and configure Jenkins**

sudo apt update -y  
sudo apt install fontconfig openjdk-17-jre -y  
<br/>sudo wget -O /usr/share/keyrings/jenkins-keyring.asc \\  
https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key  
<br/>echo "deb \[signed-by=/usr/share/keyrings/jenkins-keyring.asc\]" \\  
https://pkg.jenkins.io/debian-stable binary/ | sudo tee \\  
/etc/apt/sources.list.d/jenkins.list > /dev/null  
<br/>sudo apt-get update -y  
sudo apt-get install jenkins -y

<img width="975" height="412" alt="image" src="https://github.com/user-attachments/assets/a73ed62f-e9b3-4865-ae8f-b3c093ed13fc" />

- Now, access Jenkins on the browser on port 8080 and configure it. (&lt;public-ip&gt;:8080)

3.  **Create EKS Cluster on AWS**
    - Create IAM user with **access keys and secret access keys**
    - AWSCLI should be configured by using below commands

- curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"  
    sudo apt install unzip  
    unzip awscliv2.zip  
    sudo ./aws/install  
    aws configure
  
<img width="843" height="192" alt="image" src="https://github.com/user-attachments/assets/dead8a0a-eb7c-4b1c-817a-47e1b888916c" />

<img width="847" height="154" alt="image" src="https://github.com/user-attachments/assets/b17f7cd3-b7ee-4ad4-9eec-7977676e3b59" />

- - Install **kubectl**
- curl -o kubectl https://amazon-eks.s3.us-west-2.amazonaws.com/1.19.6/2021-01-05/bin/linux/amd64/kubectl  
    chmod +x ./kubectl  
    sudo mv ./kubectl /usr/local/bin  
    kubectl version --short –client
  
<img width="817" height="219" alt="image" src="https://github.com/user-attachments/assets/2a35e1a6-2b48-4997-a422-e2f316d4902e" />

- - Install **eksctl**
- curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)\_amd64.tar.gz" | tar xz -C /tmp  
    sudo mv /tmp/eksctl /usr/local/bin  
    eksctl version
  
<img width="830" height="113" alt="image" src="https://github.com/user-attachments/assets/f5840dcd-d89a-40a7-9a28-f54e8af717af" />

- - Create EKS Cluster using eksctl command

eksctl create cluster --name=hotstar-clone --version 1.30 --zones=ap-south-1a,ap-south-1b,ap-south-1c --without-nodegroup

<img width="896" height="165" alt="image" src="https://github.com/user-attachments/assets/4d623c66-e0d6-483d-8b23-3a7bb11acacb" />

<img width="894" height="218" alt="image" src="https://github.com/user-attachments/assets/aaf7ce68-ee6b-40d1-a2e7-43e9d1bc08ea" />


- - Associate IAM OIDC Provider

eksctl utils associate-iam-oidc-provider --region ap-south-1 --cluster hotstar-clone –approve

<img width="892" height="68" alt="image" src="https://github.com/user-attachments/assets/d259a083-0868-4055-85b0-a840b7c5e238" />

- - Create Nodegroup using eksctl command

eksctl create nodegroup --cluster=hotstar-clone --region=ap-south-1 --name=hotstar-clone-ng-1 --node-type=t2.large --nodes=2 --nodes-min=2 --nodes-max=3 --node-volume-size=29 --ssh-access --ssh-public-key=eks-nodegroup-key-mumbai --managed --asg-access --external-dns-access --full-ecr-access --appmesh-access --alb-ingress-access

<img width="889" height="122" alt="image" src="https://github.com/user-attachments/assets/5bad28c2-1c55-445f-aae9-7c16e4b7a4c2" />

Make sure the ssh-public-key “eks-nodegroup-key-mumbai” is available in your aws account.

<img width="892" height="261" alt="image" src="https://github.com/user-attachments/assets/2a571f2a-01ff-4a35-acd2-ece85fb9fc8f" />

<img width="889" height="223" alt="image" src="https://github.com/user-attachments/assets/d8002816-f1ae-46b3-93b2-9e7c4cd53c91" />


4.  **Install and configure SonarQube (Master machine)**

docker run -itd --name SonarQube-Server -p 9000:9000 sonarqube:lts-community

and give “docker ps -a” command to check if the container is up or not.

<img width="970" height="236" alt="image" src="https://github.com/user-attachments/assets/05b42a64-55f9-4511-b4a2-74cea1924d78" />

5.  **Install Trivy (Master machine)**

sudo apt-get install wget apt-transport-https gnupg lsb-release -y  
wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key | sudo apt-key add -  
echo deb https://aquasecurity.github.io/trivy-repo/deb $(lsb_release -sc) main | sudo tee -a /etc/apt/sources.list.d/trivy.list  
sudo apt-get update -y  
sudo apt-get install trivy -y

<img width="976" height="170" alt="image" src="https://github.com/user-attachments/assets/858716d1-5138-464d-96bc-4b4f98ebf027" />


6.  **Install Helm (Master Machine)**

curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3  
chmod 700 get_helm.sh  
./get_helm.sh  
helm version

<img width="958" height="202" alt="image" src="https://github.com/user-attachments/assets/a04313ad-1e29-4c30-80b4-c3443fa71cbb" />

7.  **Install argocd through Helm (Master Machine)**

helm repo add argo-cd https://argoproj.github.io/argo-helm
helm repo update  
kubectl create namespace argocd  
helm install argocd argo-cd/argo-cd -n argocd

<img width="986" height="354" alt="image" src="https://github.com/user-attachments/assets/fddef9fe-8cc2-41c9-8a0e-7139c324d2a2" />

- - Check argocd services
- kubectl get all -n argocd
    - Change argocd server’s service from ClusterIP to NodePort
- kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "LoadBalancer"}}'
    - Confirm service is patched or not
- kubectl get all -n argocd
  
<img width="854" height="356" alt="image" src="https://github.com/user-attachments/assets/565572dc-8f7e-4c9d-9055-0f44062d3c3d" />

- Access it on browser, click on advance and proceed with the load balancer dns.
- Fetch the initial password of argocd server
   - kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" |       base64 -d; echo

8.  **Steps to add email notification**
    - Go to your Jenkins Master EC2 instance and allow 465 port number for SMTPS
    - Now, we need to generate an application password from our gmail account to authenticate with Jenkins
    - Open gmail and go to Manage your Google Account –> Security > \[!Important\] > Make sure 2 step verification must be on
      
<img width="905" height="407" alt="image" src="https://github.com/user-attachments/assets/284f9c66-e3ee-405b-ad36-3e80b11089ec" />

<img width="806" height="333" alt="image" src="https://github.com/user-attachments/assets/9b3cf9a8-2363-460d-9377-236c0ea6154b" />

- - Search for App password and create a app password for Jenkins

- - Once, app password is create and go back to jenkins Manage Jenkins –> Credentials to add username and password for email notification
    
<img width="1013" height="52" alt="image" src="https://github.com/user-attachments/assets/b0f49398-3ad3-4665-9559-0227460a354c" />

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
  
<img width="824" height="358" alt="image" src="https://github.com/user-attachments/assets/aba45cd5-5a85-4f24-b6ab-7e893dae3ff8" />

After installing restart the Jenkins

- Now move to Manage jenkins –> Tools
    - For NodeJs use version NodeJs 16.2.0
    - For Jdk17 use version jdk-17.0.8.1+1
    - For Dependency Check use latest version
    - For Sonar Scanner use latest version
- Login to SonarQube server and create the credentials and webhook for jenkins to integrate with SonarQube
    - Navigate to Administration –> Security –> Users –> Token

<img width="875" height="223" alt="image" src="https://github.com/user-attachments/assets/01619127-ee59-4670-b67e-caa1b0a78ef0" />

- Now go to Administration –> Webhook and click on create webhook to integrate with Jenkins
  
<img width="878" height="292" alt="image" src="https://github.com/user-attachments/assets/50974b81-4359-4a11-954a-4bcabd78bef2" />

- Now, go to Manage Jenkins –> credentials and add Sonarqube credentials using kind as Secret text:
  
<img width="1008" height="50" alt="image" src="https://github.com/user-attachments/assets/f51cda37-cac7-45f2-adce-a5b8e9b21f86" />

- Go to Manage Jenkins –> credentials and add Github credentials to push updated code from the pipeline:
  
<img width="1014" height="48" alt="image" src="https://github.com/user-attachments/assets/29eab77a-34b2-43c3-80ce-b11d27434d28" />

    - While adding github credentials add Personal Access Token in the password field.
- Go to Manage Jenkins –> System and search for SonarQube installations:
    - Add name -> Sonarqube server link (&lt;public-ip&gt;:9000) -> sonar-token (sonarqube credentials)
- Navigate to Manage Jenkins –> credentials and add credentials for docker login to push docker image:
  
<img width="1003" height="45" alt="image" src="https://github.com/user-attachments/assets/8974e80c-c9bc-42d0-9753-a19f2d6f55c3" />

    - While adding DockerHub credentials add Personal Access Token in the password field.
- Create a JioHotstar-CI pipeline and JioHotstar-CD pipeline
    - For pipeline scripts go to https://github.com/goutham1404/hotstar-clone.git
      
<img width="890" height="272" alt="image" src="https://github.com/user-attachments/assets/b8d5244d-e751-43a0-8c62-7c8cd9d08d91" />

- Go to argoCD and connect your github repo
    - Go to Settings -> Repositories -> connect repo

<img width="875" height="158" alt="image" src="https://github.com/user-attachments/assets/260fb2be-ce89-4e1e-b761-9adcc8b6c243" />

- - Make sure that connection status is successful
- Now, go to Applications and click on New App your application is deployed on AWS EKS Cluster
  
<img width="875" height="396" alt="image" src="https://github.com/user-attachments/assets/e7d02119-7037-499f-8542-9003a026ba91" />

- The deployed JioHotstar web application
  
<img width="874" height="492" alt="image" src="https://github.com/user-attachments/assets/b380a636-1d78-4e2f-b6d8-5feeeea82f1e" />

- Email Notification
  
<img width="804" height="409" alt="image" src="https://github.com/user-attachments/assets/c76fb836-48e0-4bfd-b928-52491ba996d0" />

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

<img width="989" height="332" alt="image" src="https://github.com/user-attachments/assets/050dc60b-278c-4e41-996f-ddb83154f526" />

- Expose Prometheus and Grafana to the external world through Node Port and change it from Cluster IP to NodePort after changing make sure you save the file and open the assigned nodeport to the service.

kubectl edit svc stable-kube-prometheus-sta-prometheus -n Prometheus

- Verify service

kubectl get svc -n prometheus

- Now,let’s change the SVC file of the Grafana and expose it to the outer world
  
<img width="984" height="509" alt="image" src="https://github.com/user-attachments/assets/866ce0b4-5274-426c-9898-d3402ab3de3d" />

kubectl edit svc stable-grafana -n prometheus

- Get a password for grafana

kubectl get secret --namespace prometheus stable-grafana -o jsonpath="{.data.admin-password}" | base64 --decode ; echo

- Now, view the dashboard in Prometheus
  
<img width="819" height="420" alt="image" src="https://github.com/user-attachments/assets/6df4dde9-ad26-4d54-9e7f-23bfb2739a45" />

- Now, view the Dashboard in Grafana
  
<img width="842" height="436" alt="image" src="https://github.com/user-attachments/assets/af1e2af6-648c-47a3-8ed9-d376e8507ed3" />

<img width="845" height="434" alt="image" src="https://github.com/user-attachments/assets/9de4e2af-af75-43c5-8ce6-182881433eba" /> 

## To Clean Up EKS Cluster

- Delete eks cluster

eksctl delete cluster --name=hotstar-clone --region=ap-south-1

- To Delete eks cluster nodes

eksctl delete nodegroup --cluster hotstar-clone --name hotstar-clone-ng-1

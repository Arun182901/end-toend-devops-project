🚀 Building a Complete End-to-End DevOps CI/CD Pipeline from Scratch
Arvind Verma
Arvind Verma

Follow
6 min read
·
2 days ago
52


1



GitHub ➜ Jenkins ➜ OWASP ➜ SonarQube ➜ Docker ➜ Trivy ➜ ArgoCD ➜ Kubernetes ➜ Prometheus ➜ Grafana 🔥
I wanted to understand how real DevOps pipelines work in companies instead of learning tools separately.

So I built a complete beginner-friendly DevOps project from scratch using:

✅ GitHub
✅ Jenkins
✅ OWASP Dependency Check
✅ SonarQube
✅ Docker
✅ Trivy
✅ ArgoCD
✅ Kubernetes
✅ Prometheus
✅ Grafana

This post is not just theory.

It is a complete beginner walkthrough showing:

how to install everything
how to configure everything
why each step exists
how the tools connect together
I’m explaining it like someone is learning DevOps for the first time.

🖥️ SYSTEM REQUIREMENTS
Before starting, I used:

RequirementValueRAMMinimum 16 GBCPU4 cores recommendedOSUbuntu 22.04InternetRequiredDockerHub AccountRequiredGitHub AccountRequired

This project can also run on AWS EC2.

🧠 FIRST UNDERSTAND THE FLOW
Imagine this like an automated food delivery system 🍔

Developer writes code → quality checked → security checked → packaged → deployed → monitored automatically.

Flow:

Developer
   ↓
GitHub
   ↓
Jenkins CI
   ↓
OWASP Scan
   ↓
SonarQube Analysis
   ↓
Docker Build
   ↓
Trivy Scan
   ↓
DockerHub Push
   ↓
Jenkins CD
   ↓
GitHub Manifest Update
   ↓
ArgoCD
   ↓
Kubernetes Deployment
   ↓
Prometheus Monitoring
   ↓
Grafana Dashboard
🔧 STEP 1: INSTALL DOCKER
Docker is needed because:

Jenkins runs in Docker
SonarQube runs in Docker
Application builds Docker image
📦 Install Docker
sudo apt update
sudo apt install docker.io -y
Start Docker:

sudo systemctl start docker
Enable Docker:

sudo systemctl enable docker
Check Docker:

docker --version
🔓 GIVE USER DOCKER ACCESS
Without this, you’ll get permission errors.

Run:

sudo usermod -aG docker $USER
Then logout and login again.

Check:

docker ps
If no sudo required → success.

📚 STEP 2: INSTALL GIT
sudo apt install git -y
Verify:

git --version
☸️ STEP 3: INSTALL KUBERNETES (MINIKUBE)
We need Kubernetes cluster.

For beginners:
Minikube is easiest.

📦 Install kubectl
sudo snap install kubectl --classic
Verify:

kubectl version --client
📦 Install Minikube
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube
Verify:

minikube version
🚀 Start Kubernetes Cluster
minikube start --driver=docker
This may take several minutes.

Check nodes:

kubectl get nodes
You should see:

Ready
📄 STEP 4: CREATE SIMPLE PYTHON APP
Create folder:

mkdir devops-project
cd devops-project
📄 Create app.py
nano app.py
Paste:

from flask import Flask
app = Flask(__name__)
@app.route('/')
def home():
    return "DevOps Pipeline Working Successfully 🚀"
if __name__ == "__main__":
    app.run(host='0.0.0.0', port=5000)
Save:
CTRL + X → Y → Enter

📄 Create requirements.txt
nano requirements.txt
Paste:

flask
🐳 STEP 5: CREATE DOCKERFILE
Create Dockerfile:

nano Dockerfile
Paste:

FROM python:3.10
WORKDIR /app
COPY . .
RUN pip install -r requirements.txt
EXPOSE 5000
CMD ["python", "app.py"]
🧪 STEP 6: TEST DOCKER LOCALLY
Build image:

docker build -t flask-devops .
Run container:

docker run -d -p 5000:5000 flask-devops
Open browser:

http://localhost:5000
If app opens → success.

🌐 STEP 7: PUSH CODE TO GITHUB
Create repository on GitHub.

Example:

devops-project
🔑 Configure Git
git config --global user.name "YOUR_NAME"
git config --global user.email "YOUR_EMAIL"
🚀 Push Code
git init
git add .
git commit -m "initial commit"
git branch -M main
git remote add origin YOUR_GITHUB_REPO
git push -u origin main
🤖 STEP 8: INSTALL JENKINS
Run Jenkins container:

docker run -d \
--name jenkins \
-p 8080:8080 \
-p 50000:50000 \
-v jenkins_home:/var/jenkins_home \
jenkins/jenkins:lts
Check:

docker ps
🌐 ACCESS JENKINS
Open:

http://YOUR_SERVER_IP:8080
🔑 GET JENKINS PASSWORD
docker exec jenkins cat /var/jenkins_home/secrets/initialAdminPassword
Copy password.

Paste in Jenkins UI.

Install suggested plugins.

Create admin user.

🔌 STEP 9: INSTALL JENKINS PLUGINS
Go to:

Manage Jenkins → Plugins
Install:

✅ Docker Pipeline
✅ SonarQube Scanner
✅ OWASP Dependency Check
✅ Email Extension Plugin
✅ Kubernetes CLI Plugin

Restart Jenkins.

🔍 STEP 10: INSTALL SONARQUBE
Run SonarQube container:

docker run -d \
--name sonarqube \
-p 9000:9000 \
sonarqube:lts-community
🌐 ACCESS SONARQUBE
Open:

http://YOUR_SERVER_IP:9000
Default:

username: admin
password: admin
🔑 CREATE SONARQUBE TOKEN
Go to:

Administration → Security → Users → Tokens
Generate token.

Copy it.

Get Arvind Verma’s stories in your inbox
Join Medium for free to get updates from this writer.

Enter your email
Subscribe

Remember me for faster sign in

Very important.

🔗 CONNECT SONARQUBE TO JENKINS
Go to Jenkins:

Manage Jenkins → System
Find:

SonarQube Servers
Add:

Name: sonarqube
URL: http://YOUR_SERVER_IP:9000
Token: paste token
Save.

🛡️ STEP 11: CONFIGURE OWASP
Go to:

Manage Jenkins → Tools
Find:

Dependency Check
Add installer.

Save.

🔐 STEP 12: INSTALL TRIVY
sudo apt install wget apt-transport-https gnupg lsb-release -y
wget https://github.com/aquasecurity/trivy/releases/latest/download/trivy_0.50.1_Linux-64bit.deb
sudo dpkg -i trivy_0.50.1_Linux-64bit.deb
Check:

trivy --version
📦 STEP 13: CREATE DOCKERHUB ACCOUNT
Create account:

DockerHub.com
Create repository:

flask-devops
🔑 ADD DOCKERHUB CREDENTIALS TO JENKINS
Go to:

Manage Jenkins → Credentials
Add:

username
password
ID:

dockerhub
⚙️ STEP 14: CREATE JENKINS PIPELINE
Go to:

New Item
Select:

Pipeline
Name:

devops-pipeline
📄 CREATE JENKINSFILE
Inside project:

nano Jenkinsfile
Paste:

pipeline {
    agent any
    environment {
        IMAGE_NAME = "YOUR_DOCKERHUB_USERNAME/flask-devops"
    }
    stages {
        stage('Pull Code') {
            steps {
                git 'YOUR_GITHUB_REPO'
            }
        }
        stage('OWASP Dependency Check') {
            steps {
                dependencyCheck additionalArguments: '--scan ./'
            }
        }
        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonarqube') {
                    sh '''
                    sonar-scanner \
                    -Dsonar.projectKey=flask-app \
                    -Dsonar.sources=. \
                    -Dsonar.host.url=http://YOUR_SERVER_IP:9000 \
                    -Dsonar.login=YOUR_TOKEN
                    '''
                }
            }
        }
        stage('Docker Build') {
            steps {
                sh 'docker build -t $IMAGE_NAME:$BUILD_NUMBER .'
            }
        }
        stage('Trivy Scan') {
            steps {
                sh 'trivy image $IMAGE_NAME:$BUILD_NUMBER'
            }
        }
        stage('Docker Push') {
            steps {
                withDockerRegistry([credentialsId: 'dockerhub']) {
                    sh 'docker push $IMAGE_NAME:$BUILD_NUMBER'
                }
            }
        }
    }
}
Commit and push Jenkinsfile.

▶️ STEP 15: RUN JENKINS PIPELINE
In Jenkins:

Build Now
Watch stages execute.

If successful:
Docker image pushed to DockerHub.

☸️ STEP 16: CREATE KUBERNETES MANIFESTS
Create deployment.yaml

nano deployment.yaml
Paste:

apiVersion: apps/v1
kind: Deployment
metadata:
  name: flask-app
spec:
  replicas: 2
  selector:
    matchLabels:
      app: flask-app
  template:
    metadata:
      labels:
        app: flask-app
    spec:
      containers:
      - name: flask-app
        image: YOUR_DOCKERHUB_USERNAME/flask-devops:1
        ports:
        - containerPort: 5000
Create service.yaml

nano service.yaml
Paste:

apiVersion: v1
kind: Service
metadata:
  name: flask-service
spec:
  type: NodePort
  selector:
    app: flask-app
  ports:
  - port: 5000
    targetPort: 5000
    nodePort: 30007
Apply:

kubectl apply -f deployment.yaml
kubectl apply -f service.yaml
Check pods:

kubectl get pods
🚀 STEP 17: INSTALL ARGOCD
Create namespace:

kubectl create namespace argocd
Install:

kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
Wait few minutes.

🌐 ACCESS ARGOCD
Port forward:

kubectl port-forward svc/argocd-server -n argocd 8081:443
Open:

https://localhost:8081
🔑 GET ARGOCD PASSWORD
kubectl get secret argocd-initial-admin-secret -n argocd -o yaml
Decode password.

Login:

username: admin
🔄 STEP 18: CONNECT GITHUB TO ARGOCD
In ArgoCD:
Create Application.

Fill:

repo URL
branch
deployment path
Enable auto sync.

Now whenever YAML changes in GitHub:

ArgoCD automatically deploys latest version.

📈 STEP 19: INSTALL PROMETHEUS & GRAFANA
Install Helm:

sudo snap install helm --classic
Add repo:

helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
Update:

helm repo update
Create namespace:

kubectl create namespace monitoring
Install stack:

helm install prometheus prometheus-community/kube-prometheus-stack -n monitoring
Wait several minutes.

🌐 ACCESS GRAFANA
Port forward:

kubectl port-forward svc/prometheus-grafana 3000:80 -n monitoring
Open:

http://localhost:3000
🔑 GET GRAFANA PASSWORD
kubectl get secret -n monitoring prometheus-grafana -o jsonpath="{.data.admin-password}" | base64 --decode
Username:

admin
📧 STEP 20: CONFIGURE EMAIL ALERTS
In Grafana:

Alerting → Contact Points
Add Gmail SMTP.

SMTP:

smtp.gmail.com:587
Use Gmail App Password.

Now Grafana can send alerts automatically.

🚨 FINAL RESULT
Now complete flow works:

✅ Developer pushes code
✅ Jenkins pipeline starts
✅ OWASP scans dependencies
✅ SonarQube checks code quality
✅ Docker builds image
✅ Trivy scans vulnerabilities
✅ Docker image pushed
✅ ArgoCD deploys automatically
✅ Kubernetes runs application
✅ Prometheus monitors infrastructure
✅ Grafana visualizes metrics
✅ Alerts sent automatically

🎯 BIGGEST THING I LEARNED
Real DevOps is not:
“installing tools”

It is understanding:

✅ automation
✅ deployment flow
✅ monitoring
✅ security
✅ scaling
✅ reliability

This project helped me understand how modern production-grade DevOps environments actually work. 🚀

#devops #jenkins #docker #kubernetes #argocd #prometheus #grafana #trivy #sonarqube #owasp #gitops #devsecops #automation #cloud #linux #python #flask #opensource #monitoring #cicd

Press enter or click to view image in full size


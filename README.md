Hello friends, we will be deploying a React Js 2048 Game. We will be using Jenkins as a CICD tool and deploying our application on a Docker container and Kubernetes Cluster. I Hope this detailed blog is useful.

[CLICK HERE FOR GITHUB REPOSITORY](https://github.com/Aj7Ay/2048-React-CICD.git)

**Steps:-**

Step 1 — Launch an Ubuntu(22.04) T2 Large Instance

Step 2 — Install Jenkins, Docker and Trivy. Create a Sonarqube Container using Docker.

Step 3 — Install Plugins like JDK, Sonarqube Scanner, Nodejs, and OWASP Dependency Check.

Step 4 — Create a Pipeline Project in Jenkins using a Declarative Pipeline

Step 5 — Install OWASP Dependency Check Plugins

Step 6 — Docker Image Build and Push

Step 7 — Deploy the image using Docker

Step 8 — Kubernetes master and slave setup on Ubuntu (20.04)

Step 9 — Access the Game on Browser.

Step 10 — Terminate the AWS EC2 Instances.

**Now, let's get started and dig deeper into each of these steps:-**

### STEP1:Launch an Ubuntu(22.04) T2 Large Instance

Launch an AWS T2 Large Instance. Use the image as Ubuntu. You can create a new key pair or use an existing one. Enable HTTP and HTTPS settings in the Security Group and open all ports (not best case to open all ports but just for learning purposes it's okay).

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1694159201292/35d8cf58-7ba8-4dc0-a1f8-9cc017439910.png?auto=compress,format&format=webp&auto=compress,format&format=webp)

### Step 2 — Install Jenkins, Docker and Trivy

### 2A — To Install Jenkins

Connect to your console, and enter these commands to Install Jenkins

```
vi jenkins.sh
```

```
#!/bin/bash
sudo apt update -y
#sudo apt upgrade -y
wget -O - https://packages.adoptium.net/artifactory/api/gpg/key/public | tee /etc/apt/keyrings/adoptium.asc
echo "deb &#91;signed-by=/etc/apt/keyrings/adoptium.asc] https://packages.adoptium.net/artifactory/deb $(awk -F= '/^VERSION_CODENAME/{print$2}' /etc/os-release) main" | tee /etc/apt/sources.list.d/adoptium.list
sudo apt update -y
sudo apt install temurin-17-jdk -y
/usr/bin/java --version
curl -fsSL https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key | sudo tee \
                  /usr/share/keyrings/jenkins-keyring.asc > /dev/null
echo deb &#91;signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
                  https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
                              /etc/apt/sources.list.d/jenkins.list > /dev/null
sudo apt-get update -y
sudo apt-get install jenkins -y
sudo systemctl start jenkins
sudo systemctl status jenkins
```

```
sudo chmod 777 jenkins.sh
./jenkins.sh 
```

Once Jenkins is installed, you will need to go to your AWS EC2 Security Group and open Inbound Port 8080, since Jenkins works on Port 8080.

Now, grab your Public IP Address

```
EC2 Public IP Address:8080
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1694159420565/1d6ebde5-9948-4cf1-b2da-9ac2c2b33711.png?auto=compress,format&format=webp&auto=compress,format&format=webp)

Unlock Jenkins using an administrative password and install the suggested plugins.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1694159444242/ecdd527f-6183-4651-9480-e0c1f89f5f05.png?auto=compress,format&format=webp&auto=compress,format&format=webp)

Jenkins will now get installed and install all the libraries.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1694159480758/a30da1cc-f1e4-45bf-8182-a5f32d6f6a35.png?auto=compress,format&format=webp&auto=compress,format&format=webp)

Create a user click on save and continue.

Jenkins Getting Started Screen.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1694159545473/ea177d4e-7182-4601-8ec1-4ad6bb7fa1c6.png?auto=compress,format&format=webp&auto=compress,format&format=webp)

### 2B — Install Docker

```
sudo apt-get update
sudo apt-get install docker.io -y
sudo usermod -aG docker $USER
newgrp docker
sudo chmod 777 /var/run/docker.sock
```

After the docker installation, we create a sonarqube container (Remember to add 9000 ports in the security group).

```
docker run -d --name sonar -p 9000:9000 sonarqube:lts-community
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1694159658559/a607bab7-4ee0-4802-bf77-e9716ac33838.png?auto=compress,format&format=webp&auto=compress,format&format=webp)

Now our sonarqube is up and running

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1694159822624/f07bd773-5992-4b88-b849-ffcea2891b8e.png?auto=compress,format&format=webp&auto=compress,format&format=webp)

Enter username and password, click on login and change password

```
username admin
password admin
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1694159867860/7425ab62-8978-4dbb-a5c5-d0eb3362c15f.png?auto=compress,format&format=webp&auto=compress,format&format=webp)

Update New password, This is Sonar Dashboard.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1694159887297/6e055b5c-13ea-405f-bc13-1234b05bf2ff.png?auto=compress,format&format=webp&auto=compress,format&format=webp)

### 2C — Install Trivy

```
vi trivy.sh
```

```
sudo apt-get install wget apt-transport-https gnupg lsb-release -y
wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key | gpg --dearmor | sudo tee /usr/share/keyrings/trivy.gpg > /dev/null
echo "deb &#91;signed-by=/usr/share/keyrings/trivy.gpg] https://aquasecurity.github.io/trivy-repo/deb $(lsb_release -sc) main" | sudo tee -a /etc/apt/sources.list.d/trivy.list
sudo apt-get update
sudo apt-get install trivy -y
```

Next, we will log in to Jenkins and start to configure our Pipeline in Jenkins

### Step 3 — Install Plugins like JDK, Sonarqube Scanner, NodeJs, OWASP Dependency Check

### 3A — Install Plugin

Goto Manage Jenkins →Plugins → Available Plugins →

Install below plugins

1 → Eclipse Temurin Installer (Install without restart)

2 → SonarQube Scanner (Install without restart)

3 → NodeJs Plugin (Install Without restart)

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1694160106164/d829e70f-9a23-4d03-a427-887d779aa141.png?auto=compress,format&format=webp&auto=compress,format&format=webp)

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1695227031117/e7a88b82-e007-465f-8c7f-b911c0e5f658.png?auto=compress,format&format=webp)

### 3B — Configure Java and Nodejs in Global Tool Configuration

Goto Manage Jenkins → Tools → Install JDK(17) and NodeJs(16)→ Click on Apply and Save

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1694160282666/7565037d-cf4f-4034-b55a-7028e580e3f8.png?auto=compress,format&format=webp&auto=compress,format&format=webp)

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1695403120569/3010e514-d64c-438a-85eb-d340ed5d3331.png?auto=compress,format&format=webp)

### 3C — Create a Job

create a job as 2048 Name, select pipeline and click on ok.

### Step 4 — Configure Sonar Server in Manage Jenkins

Grab the Public IP Address of your EC2 Instance, Sonarqube works on Port 9000, so <Public IP>:9000. Goto your Sonarqube Server. Click on Administration → Security → Users → Click on Tokens and Update Token → Give it a name → and click on Generate Token

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1694160792404/611ddf2a-9c2c-414a-ad3a-2ba84f8942ca.png?auto=compress,format&format=webp&auto=compress,format&format=webp)

click on update Token

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1694160866134/49f34dd2-de41-455c-aee0-f1ffe13d8488.png?auto=compress,format&format=webp&auto=compress,format&format=webp)

Create a token with a name and generate

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1694160967625/a96320b2-3eda-411e-bb44-20092b8a79ec.png?auto=compress,format&format=webp&auto=compress,format&format=webp)

copy Token

Goto Jenkins Dashboard → Manage Jenkins → Credentials → Add Secret Text. It should look like this

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1694161147409/35b423f8-e7e4-402e-895b-cd1869b8a170.png?auto=compress,format&format=webp&auto=compress,format&format=webp)

You will this page once you click on create

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1694161221410/9a7a6f3f-fd40-4bbb-8857-bf2ffb8e6895.png?auto=compress,format&format=webp&auto=compress,format&format=webp)

Now, go to Dashboard → Manage Jenkins → System and Add like the below image.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1694161318710/738bd6ea-c374-4c76-9234-5ec09cfe754f.png?auto=compress,format&format=webp&auto=compress,format&format=webp)

Click on Apply and Save

**The Configure System option** is used in Jenkins to configure different server

**Global Tool Configuration** is used to configure different tools that we install using Plugins

We will install a sonar scanner in the tools.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1694161414550/efa6ef74-29e5-41c4-a81e-e6c528048c9f.png?auto=compress,format&format=webp&auto=compress,format&format=webp)

In the Sonarqube Dashboard add a quality gate also

Administration--> Configuration-->Webhooks

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1694242716931/c913bcc3-07c6-4e68-b73b-5ec04c63d4d6.png?auto=compress,format&format=webp&auto=compress,format&format=webp)

Click on Create

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1694242794858/85015d2b-1c59-435a-b058-1db98f215b18.png?auto=compress,format&format=webp&auto=compress,format&format=webp)

Add details

```
#in url section of quality gate
http:&#47;&#47;jenkins-public-ip:8080/sonarqube-webhook/
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1694253241413/c6d214b6-86ad-4561-8d49-3739ac5fe66f.png?auto=compress,format&format=webp&auto=compress,format&format=webp)

Let's go to our Pipeline and add the script in our Pipeline Script.

```
pipeline{
    agent any
    tools{
        jdk 'jdk17'
        nodejs 'node16'
    }
    environment {
        SCANNER_HOME=tool 'sonar-scanner'
    }
    stages {
        stage('clean workspace'){
            steps{
                cleanWs()
            }
        }
        stage('Checkout from Git'){
            steps{
                git branch: 'master', url: 'https://github.com/Aj7Ay/2048-React-CICD.git'
            }
        }
        stage("Sonarqube Analysis "){
            steps{
                withSonarQubeEnv('sonar-server') {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=Game \
                    -Dsonar.projectKey=Game '''
                }
            }
        }
        stage("quality gate"){
           steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'Sonar-token'
                }
            }
        }
        stage('Install Dependencies') {
            steps {
                sh "npm install"
            }
        }
    }
}
```

Click on Build now, you will see the stage view like this

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1695231229497/f9d97aa9-90ba-49cf-b082-9df24ee86c4b.png?auto=compress,format&format=webp)

To see the report, you can go to Sonarqube Server and go to Projects.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1695231249646/14b74238-739c-4cdf-9627-9f8601e8d8d2.png?auto=compress,format&format=webp)

You can see the report has been generated and the status shows as passed. You can see that there are 838 lines. To see a detailed report, you can go to issues.

### Step 5 — Install OWASP Dependency Check Plugins

GotoDashboard → Manage Jenkins → Plugins → OWASP Dependency-Check. Click on it and install it without restart.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1694162570631/a0da6454-e85a-4e27-908a-255b024bf834.png?auto=compress,format&format=webp&auto=compress,format&format=webp)

First, we configured the Plugin and next, we had to configure the Tool

Goto Dashboard → Manage Jenkins → Tools →

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1694162653334/158f6b94-7c92-4556-9151-6213a003b431.png?auto=compress,format&format=webp&auto=compress,format&format=webp)

Click on Apply and Save here.

Now go configure → Pipeline and add this stage to your pipeline and build.

```
stage('OWASP FS SCAN') {
            steps {
                dependencyCheck additionalArguments: '--scan ./ --disableYarnAudit --disableNodeAudit', odcInstallation: 'DP-Check'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        stage('TRIVY FS SCAN') {
            steps {
                sh "trivy fs . > trivyfs.txt"
            }
        }
```

The stage view would look like this,

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1695231335082/b5330f0d-67a5-43df-866d-c41d64627f9f.png?auto=compress,format&format=webp)

You will see that in status, a graph will also be generated and Vulnerabilities.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1695231385639/64d49427-ce7b-4723-a432-faf02dbca838.png?auto=compress,format&format=webp)

### Step 6 — Docker Image Build and Push

We need to install the Docker tool in our system, Goto Dashboard → Manage Plugins → Available plugins → Search for Docker and install these plugins

`Docker`

`Docker Commons`

`Docker Pipeline`

`Docker API`

`docker-build-step`

and click on install without restart

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1694162971794/4e289fe1-c3b9-48d0-8efd-2b8b47118e71.png?auto=compress,format&format=webp&auto=compress,format&format=webp)

Now, goto Dashboard → Manage Jenkins → Tools →

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1694163030620/59df4527-29d4-41df-8dcd-501e04fda7eb.png?auto=compress,format&format=webp&auto=compress,format&format=webp)

Add DockerHub Username and Password under Global Credentials

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1694163085161/2dfb909a-61a8-4122-9292-87a2742bb8d9.png?auto=compress,format&format=webp&auto=compress,format&format=webp)

Add this stage to Pipeline Script

```
stage("Docker Build & Push"){
            steps{
                script{
                   withDockerRegistry(credentialsId: 'docker', toolName: 'docker'){
                       sh "docker build -t 2048 ."
                       sh "docker tag 2048 sevenajay/2048:latest "
                       sh "docker push sevenajay/2048:latest "
                    }
                }
            }
        }
        stage("TRIVY"){
            steps{
                sh "trivy image sevenajay/2048:latest > trivy.txt"
            }
        }
```

You will see the output below, with a dependency trend.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1695231457022/078e9995-b27f-47fd-9add-bffe2426107b.png?auto=compress,format&format=webp)

When you log in to Dockerhub, you will see a new image is created

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1695231503187/ee941c32-b83c-4956-8e06-a2db8c4fd4e6.png?auto=compress,format&format=webp)

Now Run the container to see if the game coming up or not by adding below stage

```
stage('Deploy to container'){
            steps{
                sh 'docker run -d --name 2048 -p 3000:3000 sevenajay/2048:latest'
            }
        }
```

stage view

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1695231634338/483ee533-5144-4424-b55f-a74ebc38769f.png?auto=compress,format&format=webp)

`<Jenkins-public-ip:3000>`

You will get this output

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1695231649923/fc1c3997-0854-493a-8a43-ccc7817dd77e.png?auto=compress,format&format=webp)

Play the game and make it 2048

### Step 8 — Kuberenetes Setup

Connect your machines to Putty or Mobaxtreme

**Take-Two Ubuntu 20.04 instances one for k8s master and the other one for worker.**

Install Kubectl on Jenkins machine also.

### Kubectl is to be installed on Jenkins also

```
sudo apt update
sudo apt install curl
curl -LO https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
kubectl version --client
```

### Part 1 ----------Master Node------------

```
sudo hostnamectl set-hostname K8s-Master
```

### ----------Worker Node------------

```
sudo hostnamectl set-hostname K8s-Worker
```

### Part 2 ------------Both Master & Node ------------

```
sudo apt-get update
sudo apt-get install -y docker.io
sudo usermod –aG docker Ubuntu
newgrp docker
sudo chmod 777 /var/run/docker.sock
sudo curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
sudo tee /etc/apt/sources.list.d/kubernetes.list : --token  --discovery-token-ca-cert-hash 
```

Copy the config file to Jenkins master or the local file manager and save it

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1694164048426/b0b0152d-aca6-474f-8d88-a9c28dd9d703.png?auto=compress,format&format=webp&auto=compress,format&format=webp)

copy it and save it in documents or another folder save it as secret-file.txt

Note: create a secret-file.txt in your file explorer save the config in it and use this at the kubernetes credential section.

Install Kubernetes Plugin, Once it's installed successfully

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1694163896514/5edda1d6-b9a3-45c2-9bcf-e677939191cf.png?auto=compress,format&format=webp&auto=compress,format&format=webp)

goto manage Jenkins --> manage credentials --> Click on Jenkins global --> add credentials

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1694163948237/0e01b94e-c1e4-4fd2-8f5d-730df240a5b5.png?auto=compress,format&format=webp&auto=compress,format&format=webp)

final step to deploy on the Kubernetes cluster

```
stage('Deploy to kubernets'){
            steps{
                script{
                    withKubeConfig(caCertificate: '', clusterName: '', contextName: '', credentialsId: 'k8s', namespace: '', restrictKubeConfigAccess: false, serverUrl: '') {
                       sh 'kubectl apply -f deployment.yaml'
                  }
                }
            }
        }
```

stage view

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1695231979645/41227fb6-e1da-4f60-a120-00d424d73fb9.png?auto=compress,format&format=webp)

In the Kubernetes cluster give this command

```
kubectl get all
kubectl get svc #use anyone
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1694165240130/0cec6712-24b4-4715-8143-2083a367016e.png?auto=compress,format&format=webp&auto=compress,format&format=webp)

### STEP9:Access from a Web browser with

`<public-ip-of-slave:service port>`

output:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1695232200237/8812357c-9ed6-4227-b9fb-f9dbcacfeaa9.png?auto=compress,format&format=webp)

### Step 10: Terminate instances.

### Complete Pipeline

```
pipeline{
    agent any
    tools{
        jdk 'jdk17'
        nodejs 'node16'
    }
    environment {
        SCANNER_HOME=tool 'sonar-scanner'
    }
    stages {
        stage('clean workspace'){
            steps{
                cleanWs()
            }
        }
        stage('Checkout from Git'){
            steps{
                git branch: 'master', url: 'https://github.com/Aj7Ay/2048-React-CICD.git'
            }
        }
        stage("Sonarqube Analysis "){
            steps{
                withSonarQubeEnv('sonar-server') {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=Game \
                    -Dsonar.projectKey=Game '''
                }
            }
        }
        stage("quality gate"){
           steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'Sonar-token'
                }
            }
        }
        stage('Install Dependencies') {
            steps {
                sh "npm install"
            }
        }
        stage('OWASP FS SCAN') {
            steps {
                dependencyCheck additionalArguments: '--scan ./ --disableYarnAudit --disableNodeAudit', odcInstallation: 'DP-Check'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        stage('TRIVY FS SCAN') {
            steps {
                sh "trivy fs . > trivyfs.txt"
            }
        }
        stage("Docker Build & Push"){
            steps{
                script{
                   withDockerRegistry(credentialsId: 'docker', toolName: 'docker'){
                       sh "docker build -t 2048 ."
                       sh "docker tag 2048 sevenajay/2048:latest "
                       sh "docker push sevenajay/2048:latest "
                    }
                }
            }
        }
        stage("TRIVY"){
            steps{
                sh "trivy image sevenajay/2048:latest > trivy.txt"
            }
        }
        stage('Deploy to container'){
            steps{
                sh 'docker run -d --name 2048 -p 3000:3000 sevenajay/2048:latest'
            }
        }
        stage('Deploy to kubernets'){
            steps{
                script{
                    withKubeConfig(caCertificate: '', clusterName: '', contextName: '', credentialsId: 'k8s', namespace: '', restrictKubeConfigAccess: false, serverUrl: '') {
                       sh 'kubectl apply -f deployment.yaml'
                  }
                }
            }
        }
    }
}
```

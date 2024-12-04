https://www.youtube.com/watch?v=xAjledixxTE&t=25s
GitOps Masterclass | DevOps CICD Project

all 3 repos go hand in hand: infra AWS EKS / app src (CI) / yaml manifest (CD)

ArgoCD-EKS-Terraform-BankAPP
Multi-Tier-BankApp-CI 
Multi-Tier-BankApp-CD

Gitops is a process to automate infra and application deployment, using git as a single source of truth.

4 principles

1. git as a single source of truth
2. EKS / K8s cluster: declarative desired state 
(declarative: writing final state / 2 worker / 30GB at the end
imperative: step by step approach)
3. automated reconciliation (suppose, 2 repos: 1 for infra and 2 for deployment, at one point of time, I need: change worker 2 to 3, 0r pods 4 to 5, automatically it should reflect in deployment app)
4. pull based deployment (argoCD: any new changes in Git, pull from repo and make changes)


t2.med/infraserver

sudo apt update

install aws cli - 3 steps

curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
sudo apt install unzip
unzip awscliv2.zip 
sudo ./aws/install

aws --version

aws configure

change private key in variables

git clone https://github.com/pvkraja227/ArgoCD-EKS-Terraform-BankAPP.git
install terraform: sudo snap install terraform --classic
cd ArgoCD-EKS-Terraform-BankAPP
terraform init
terraform plan - 17
terraform apply --auto-approve

aws eks --region ap-south-1 update-kubeconfig --name devopsshack-cluster

kubectl get nodes - 3 worker

kubectl create ns argocd
curl -sSfL https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml | kubectl apply -n argocd -f -

kubectl get all -n argocd

kubectl get svc -n argocd
kubectl edit svc argocd-server -n argocd (change type: ClusterIP to LoadBalancer)
kubectl get svc -n argocd (now, external IP is present, paste in browser/((takes time))click on advance/proceed to ....) admin/paste pwd

kubectl get secrets -n argocd


(((kubectl get secret -n argocd argocd-initial-admin-secret -o yaml (copy encrypted secret)

echo <encrypted secret> | base64 -d)))  OR OR OR

kubectl get secret -n argocd argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d ... copy pwd (before ubuntu)

dashboard: edit user info/change pwd logout n login
pwd: 8LeOzkPatrwoouVm


ec2 -3/Jenkins/SonarQube/nexus - t2.med


Jenkins:

sudo apt update
install java17: type java/install java17
install Jenkins - install Jenkins on ubuntu
sudo systemctl enable Jenkins
sudo systemctl start Jenkins
chrome: publicIP:8080

sudo cat .... (get admin pwd)/install plugins

chrome: install docker on ubuntu: 2 steps
sudo usermod -aG docker jenkins
to reflect the changes: go to browser /restart

Install Trivy
(no need of Maven and SonarQube: if installed directly no need to worry, if not and they are present in plugins, then only we have to define in tools section)

SonarQube:

sudo apt update
chrome: install docker on ubuntu: 2 steps
sudo usermod -aG docker $USER (or ubuntu)
newgrp docker

docker run -d -p 9000:9000 sonarqube:lts-community
chrome: publicIP:9000 (admin/admin)

token: squ_c3d818884576f3bcaedc1dd9e68c589c63dc349f

Nexus:

sudo apt update
chrome: install docker on ubuntu: 2 steps
sudo usermod -aG docker $USER (or ubuntu)
newgrp docker

docker run -d -p 8081:8081 sonatype/nexus3
chrome: publicIP:8081 (admin/paste pwd)
docker ps (container is running)
docker exec -it containerID /bin/bash
cat sonatype-work/nexus3/admin.password (copy pwd)
exit


CI Pipeline:

Manage Jenkins/Plugins

SonarQube scanner
pipeline stage view
pipeline maven integration
maven integration
config file provider (store artifact in Nexus: from Jenkins to Nexus we have to configure Nexus by providing pwds: thru this file provider only we can see "Managed Files" in Manage Jenkins)
docker pipeline

Manage Jenkins/tools

add SonarQube scanner/name: sonar-scanner
add maven/name: maven3
(no need of docker and java as we have already installed in our machine)

manage jenkins/Credentials:

Global/add credentials
GitHub username with pwd/ID: git-cred and in desc (check token expired)
docker username with pwd/ID: docker-cred and in desc
SonarQube username/token/ID: sonar-cred and in desc

Manage Jenkins/system: (where we configure servers)

SonarQube servers/add sonar/name: sonar/copy url no "/"  / select sonar-cred / save

(((install trivy in jenkins
(no need of Maven and SonarQube: if installed directly (trivy) no need to worry, if not and they are present in plugins (sonar and maven), then only we have to define in tools section) )))

change nexus URL in pom.xml
in nexus/repositories/hosted/deployment policy/allow redeploy

manage jenkins/managed files/add new config/global maven settings.xml/ID:maven-settings/in content

add 2 servers in servers section

-->
<server>
	<id>maven-releases</id>
      	<username>admin</username>
      	<password>nexus</password>
</server>
	
<server>
      	<id>maven-snapshots</id>
      	<username>admin</username>
      	<password>nexus</password>
</server>

then, submit (we can communicate with Nexus)

new item: create pipeline
name: CI_Pipeline (no space)
discard old builds: 2

create stages: in Jenkinsfile


build with parameters: change parameter to p1

result: in repo CD/version is changed to p1

CD Part: (ArgoCD)

settings/repositories/connect repo/via https/type git/url CD/username pvkraja227/pwd token/connect

status: successful

applications/new app/name: bankapp/project name: default/sync policy: automatic/click "self heal"/click "auto create namespace"/click "skip schema validation"/

repository URL/revision: main/path: "." 

destination: URL: https://kubernetes.dafault.svc
namespace: xyz (any value, bcz we haven't created any namepsace)

Directory: click on Directory recurse

create

we have bankapp - 4 services (ep (end point)/eps (end point slice)/bank app - 2 pods/mysql - 1 pod)

for suppose in CD manifest, change pods 2 to 1 --> immediately it reflects in ArgoCD if not .. we have to add webhook

since we have enabled auto sync, it shld work .. orelse,

copy url of argoCD ..github/settings/webhooks/
URL / content type: application/json
if http, make sure to disable ssl verification
add webhook
click on webhook/recent deliveries tab/ tick mark (works fine)

now goto jenkins pipeline/build with parameters: v4/build
error, bcz in workspace previous repo is present .. so, first delete it

stage('clean workspace') {
steps {
cleanWs() // clean the workspace
}
}

in Jenkinsfile add - clean workspace

build

ArgoCD - 2 replicasets are present

in CD manifest file: mentioned revisionHistoryLimit: 1 
(so, p1 last deployment replicaset and new deployment replicaset both are present in argoCD)

ex: 
previous: p1/3 pods
current: p3/3 pods + p1 rs also will be present

version p1: in ArgoCD

bank app --> 4 (bankapp-service/mysql-service/bankapp/MySQL)

bankapp-service - 2 (ep bankapp-service/eps bankapp-service-kjhul)
mysql-service - 2 (ep mysql-service/eps mysql-service-ygtfd)
bankapp - 1 rs --> 3 pods
MySQL - 1 rs --> 1 pod

after changing version to p3:

bankapp-service - 2 (ep bankapp-service/eps bankapp-service-kjhul)
mysql-service - 2 (ep mysql-service/eps mysql-service-ygtfd)
bankapp - 2 rs --> 3 pods only
(((in CD manifest file: mentioned revisionHistoryLimit: 1)))
MySQL - 1 rs --> 1 pod only

ep: endpoint
eps: endpointslice

# Spring Boot based Java web application
 
This is a simple Sprint Boot based Java application that can be built using Maven. Sprint Boot dependencies are handled using the pom.xml 
at the root directory of the repository.

This is a MVC architecture based application where controller returns a page with title and message attributes to the view.

## Execute the application locally or on an EC2 instance with docker container and access it using your browser

* Create an EC2 instance, allow ports 22, 8010 in inbound rules. 8010 is the port where application runs
* SSH into instance, install java, maven to build application and docker for the application to run in the docker way using docker container

Checkout the repo and move to the directory

```
git clone https://github.com/PavanaParadesi15/java-maven-sonar-argocd-helm-k8s-application.git
cd java-maven-sonar-argocd-helm-k8s/sprint-boot-app
```

Java is prerequisite for Jenkins
Command to install Java
```
sudo apt update
sudo apt install openjdk-17-jre
java -version
```

Command to install Maven
```
sudo apt install maven
maven -version
```

Command to install docker
```
sudo apt install docker.io
sudo su -  # switch to root user to grant ubuntu user permissions to run docker
usermod -aG docker ubuntu   # grant ubuntu user permissions to docker . to create and run contianers
usermod -aG docker jenkins   # we will need this while building CICD through jenkins 
systemctl restart docker   # restart docker for the permissions to apply
```

Command to build the application using Maven to generate the artifacts
```
mvn clean package
```

Command to build the docker image
```
docker build -t ultimate-cicd-pipeline:v1 .   # ultimate0cicd0pipeline - is the docker image name I gave and v1 is the version of the image
```

Command to run the docker container using docker image
```
docker run -d -p 8010:8080 -t ultimate-cicd-pipeline:v1
```

Access the application through browser 
```
http://<EC2-instance-publicIP>:8010/
```
![image](https://github.com/user-attachments/assets/6d61a50e-056e-4361-a74e-683153db980c)

### ## **To run on local machine**
/*** The above maven target stroes the artifacts to the `target` directory. You can either execute the artifact on your local machine
(or) run it as a Docker container.

** Note: To avoid issues with local setup, Java versions and other dependencies, I would recommend the docker way. **

#### Execute locally (Java 11 needed) and access the application on http://localhost:8080

```
java -jar target/spring-boot-web.jar
```
***/


## Next Steps

### Next steps to build complete CI/CD for the application and run it on Kubernetes
####  Install Jenkins

```
curl -fsSL https://pkg.jenkins.io/debian/jenkins.io-2023.key | sudo tee \
  /usr/share/keyrings/jenkins-keyring.asc > /dev/null
echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
  https://pkg.jenkins.io/debian binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null
sudo apt-get update
sudo apt-get install jenkins

```
** **Note:** By default, Jenkins will not be accessible to the external world due to the inbound traffic restriction by AWS. Open port 8080 in the inbound traffic rules on the EC2 Instance. Jenkins runs on port 8080 by default.

* Login to Jenkins using the below URL:
  
http://EC2Instance-publicIP:8080

* After login to Jenkins, - Run the command in terminal of EC2 instance to copy the Jenkins Admin Password - 
```sudo cat /var/lib/jenkins/secrets/initialAdminPassword ```
* Enter the Administrator password
* Click on Install suggested plugins
* Create First Admin User and password
* Jenkins Installation is Successful. 

#### In the Jenkins
* We can create jenkins pipeline in the "pipeline" way rather than "Freestyle project"(which was a legacy way of creating pipeline) 
* In the pipeline way , we can write jenkins pipeline script in a declarative way and can store it in Github, where others can view it and collaborate. 
* We can write jenkins file at the main (root of the application) in github and store it there, and can give Github Repository URL and credentials in the jenkins for it to connect to Github and pick the Jenkinsfile. Give the path of the Jenkinsfile. 
* Save the jenkins pipeline build
 
*** **Note:**  This Jenkinsfile have all the steps for Continuous Integration only (Jnekins is a CI tool only), its not a CD tool. 

* Install "docker pipeline" and "Sonarqube scanner" plugin in Jenkins
* Go to Manage Jenkins > Manage Plugins.
* In the Available tab, search for "Docker Pipeline".
* Select the plugin and click the Install button.
* Restart Jenkins after the plugin is installed.
 
#### Configure a Sonar Server in EC2 instance . It has to connect with AWS . sonar server runs on port 9000
```
sudo su -           # switch to root user to install sonar server 
adduser sonarqube    # Provide password 
apt install unzip    # install unzip 
sudo su - sonarqube    # switch to the sonarqube user 
wget https://binaries.sonarsource.com/Distribution/sonarqube/sonarqube-9.4.0.54424.zip        # command to download sonar binary
unzip *     # sonarqube folder is extracted 
chmod -R 755 /home/sonarqube/sonarqube-9.4.0.54424
chown -R sonarqube:sonarqube /home/sonarqube/sonarqube-9.4.0.54424
cd sonarqube-9.4.0.54424/bin/linux-x86-64/
./sonar.sh start           # sonarqube server starts at port 9000 by default
```

* Login to sonarqube  - http://ec2-instance-publicIP:9000

__**Note: Difference between adduser and useradd **__

_**adduser:**_
    _User-friendly script. It is a high-level, interactive script that simplifies the process of adding a user._
    _Prompts for details. It asks for user information like password, full name, and home directory during the process._
    _Automatically sets defaults. It handles creating the user's home directory, copying configuration files, and setting default permissions._

_**useradd:**_
    _Low-level command. It is a basic command with fewer automatic features and requires manual configuration._
    _No prompts. You need to specify options explicitly_


#### Authenticate Sonarqube with Jenkins
* Generate token in Sonarqube . 
* My Account -> Security -> Give token name -> Click on Generate -> Generates a token.
* Copy this token and authenticate sonarqube with Jenkins using this token.
* Go to Jenkins -> Manage Jenkins -> Credentials -> System -> Global credentials -> Click on Add credentials -> Add sonarqube token as Secrect text here. 
* This will authenticate sonarqube with jenkins. 

## CD - Continuous Deployment

### Next Step - Installing Kubernetes

### Install Minikube

Follow the steps in the below link

https://minikube.sigs.k8s.io/docs/start/?arch=%2Flinux%2Fx86-64%2Fstable%2Fbinary+download


* We need Kubectl to interact with a Minikube Kubernetes cluster. **kubectl** is the Kubernetes Command-Line Tool. It is used to communicate with your Kubernetes cluster for operations like deploying applications, checking cluster resources, and managing configurations.
* While Minikube sets up a local Kubernetes cluster, kubectl is the tool you use to interact with it.

#### Install Kubectl using below link

https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/

### ARGO CD 

* Argo CD is a declarative, GitOps continuous delivery tool for Kubernetes.
* To install Argo CD, we need to install Argo CD controller first. 
* For any K8s controller installation should take place using K8s Operators which manages lifecycle of K8s controllers, like upgrades of controllers etc. 
* Argo CD installation is done through operator

To check for operators - https://operatorhub.io/

To install ArgoCD operator -- https://operatorhub.io/operator/argocd-operator
* OLM installation is one time installation. Next time we just need to install required operators 

Install argocd through the steps in the above link


### JenkinsFile

CI part is done through Jenkins and CD part is done through ArgoCD which is GitOps tools to deploy application on K8s
Difference between mvn clean install and mvn clean package 
mvn clean install - If we want to push the jar/war/ear artifact created after the build process to some artifactory then we can use 'install' command
mvn clean package - When we just want the artifact to create docker image and publish that image to docker hub/image repo.

**Pom.xml** - File which contains all the project dependencies needed to run the application . Maven downloads all the dependencies in pom.xml and builds the java application to create jar/war/ear file. which are java/web/enterprise archive files. 
* This artifact is saved inside target folder 
* Docker file copies this artifact and executes it to create docker image
* Docker image is pushed to Docker Hub
* deployment.yaml file is created where Argo CD pulls the deployment file from the git repository and deploys the application to the K8s cluster.
* Jenkins updates the updated image to the deployment.yaml. For this we need to store docker and github credentials in Docker. So that jenkins should connect with docker and github
* Get the docker image created and updated in the deployment.yml file in git repo and deploy it on K8s cluster

### Create ArgoCD Controller
https://argocd-operator.readthedocs.io/en/latest/usage/basics/

Copy the code in this link and create a file names "argocd-basic.yml" in Linux VM 
Copy that  code in this file and execute the file with this command below
``` 
"argocd-basic.yml"
nano argocd-basic.yml 
kubectl apply -f argocd-basic.yml 
```

It created Argo CD controller and workloads

#### NOTE :- 
* To change Type of "example-argocd-server" from Cluster IP to NodePort as we have to access ArgoCD on browser, we have to edit "example-argocd-server"
* example-argocd-server -- is the Argocd service
* we can try to edit the "example-argocd-server" using this command below:
```kubectl edit svc example-argocd-server````
* But that is not changin Type from clusterIP to NodePort. "example-argocd-server"  service is automatically getting updated to default setting i.e, Type to changing to ClusterIP by default even after editing it. This is a Bug in service 
https://github.com/argoproj-labs/argocd-operator/pull/1546
* So to fix this, while creating ArgoCD controller, specify the "service: type: NodePort" . This created "example-argocd-server" as Tyoe Nodeport.

To check if minikube is running and start and stop it, below commands
```
minikube status
minikube start
minikube stop
```

#### Kubectl commands
```
kubectl get pods    // gets running pods
kubectl get pods -w    // see pods in watch mode
kubectl get svc    // list down services . lists Argo CD servers
kubectl get csv -n operators        // shows installed operators
kubectl create deployment
kubectl expose deployment
```


```kubectl get svc    // lists down the services
* example-argocd-server      -- This is the Argo CD server
* Edit this argocd-server to change the type from ClusterIP to NodePort to run Argo cd in the browser 

``` 
```
kubectl edit svc example-argocd-server 
kubectl get svc  // can see the updated port for argocd-server service 
minikube service argocd-server   // minikube generates URL for Argo cd so that we can access ArgoCD in the browser
minikube service list   // list the services with the port and URL . We can see argocd-server URL to access argocd through browser. 
kubectl get pods    // before accessing argocd through browser, check is Argocd pods are up and running  
```











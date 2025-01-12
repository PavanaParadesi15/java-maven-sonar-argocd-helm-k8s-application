# Jenkins Pipeline for Java based application using Maven, SonarQube, Argo CD, Helm and Kubernetes

![Screenshot 2023-03-28 at 9 38 09 PM](https://user-images.githubusercontent.com/43399466/228301952-abc02ca2-9942-4a67-8293-f76647b6f9d8.png)


Here are the step-by-step details to set up an end-to-end Jenkins pipeline for a Java application using SonarQube, Argo CD, Helm, and Kubernetes:

Prerequisites:

   -  Java application code hosted on a Git repository
   -   Jenkins server
   -  Kubernetes cluster
   -  Helm package manager
   -  Argo CD

Steps:

    1. Install the necessary Jenkins plugins:
       1.1 Git plugin
       1.2 Maven Integration plugin
       1.3 Pipeline plugin
       1.4 Kubernetes Continuous Deploy plugin

    2. Create a new Jenkins pipeline:
       2.1 In Jenkins, create a new pipeline job and configure it with the Git repository URL for the Java application.
       2.2 Add a Jenkinsfile to the Git repository to define the pipeline stages.

    3. Define the pipeline stages:
        Stage 1: Checkout the source code from Git.
        Stage 2: Build the Java application using Maven.
        Stage 3: Run unit tests using JUnit and Mockito.
        Stage 4: Run SonarQube analysis to check the code quality.
        Stage 5: Package the application into a JAR file.
        Stage 6: Deploy the application to a test environment using Helm.
        Stage 7: Run user acceptance tests on the deployed application.
        Stage 8: Promote the application to a production environment using Argo CD.

    4. Configure Jenkins pipeline stages:
        Stage 1: Use the Git plugin to check out the source code from the Git repository.
        Stage 2: Use the Maven Integration plugin to build the Java application.
        Stage 3: Use the JUnit and Mockito plugins to run unit tests.
        Stage 4: Use the SonarQube plugin to analyze the code quality of the Java application.
        Stage 5: Use the Maven Integration plugin to package the application into a JAR file.
        Stage 6: Use the Kubernetes Continuous Deploy plugin to deploy the application to a test environment using Helm.
        Stage 7: Use a testing framework like Selenium to run user acceptance tests on the deployed application.
        Stage 8: Use Argo CD to promote the application to a production environment.

    5. Set up Argo CD:
        Install Argo CD on the Kubernetes cluster.
        Set up a Git repository for Argo CD to track the changes in the Helm charts and Kubernetes manifests.
        Create a Helm chart for the Java application that includes the Kubernetes manifests and Helm values.
        Add the Helm chart to the Git repository that Argo CD is tracking.

    6. Configure Jenkins pipeline to integrate with Argo CD:
       6.1 Add the Argo CD API token to Jenkins credentials.
       6.2 Update the Jenkins pipeline to include the Argo CD deployment stage.

    7. Run the Jenkins pipeline:
       7.1 Trigger the Jenkins pipeline to start the CI/CD process for the Java application.
       7.2 Monitor the pipeline stages and fix any issues that arise.

This end-to-end Jenkins pipeline will automate the entire CI/CD process for a Java application, from code checkout to production deployment, using popular tools like SonarQube, Argo CD, Helm, and Kubernetes.





### Explanation of Jenkins file inside **"spring-boot-app"** folder

* One of the best ways to write jenkinsfile is to use docker container as agent for jenkins pipeline. Its main use is to lessen the configuration. 
* All the jenkins pipeline stages runs in this docker container. If we run all these stages inside a new slave EC2 instance, its downside is, where there are no code changes/commits in future, its waste of running a EC2 instance when there are no pipeline jobs to run in jenkins. 
* There is unncessary costing involved from the AWS. And we have to do lot of configurations/application dependencies on the EC2 instances like installing Maven, docker etc. 
* Instead when we use docker image to run docker container which helps to run all the jenkins pipeline stages , we just write the container configuration once and all the application dependencies/configurations are already present in the docker container and all the pipeline stages mentioned in the jenkins file gets executed in the docker container..
* As soon as jenkins pipeline gets triggered, it creates docker container, and pipeline executes all the pipeline stages in this container and once all the stages are executed and successful, docker container is killed/deleted. 
* In this way many pipelines/jenkins jobs can run parallely on a single EC2 instance as many containers can run on a instance. Once a container gets killed after its work, there is enough resources on the EC2 instance for the other containers to run
* So in this approach there is no EC2 instance resource wastage, no need to create multiple instances for each pipeline job and which leads to less costing. 
* So to use docker as an agent, we need to install "docker pipeline" plugin in jenkins and also 
* Write docker file to create a docker image first with all the necessary configurations (maven, docker installed) . We can use that image as agent in jenkins pipeline and mention it in jenkinsfile to trigger docker container which have maven and docker in it


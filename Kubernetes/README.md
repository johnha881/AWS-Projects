# Kubernetes

## Table of Contents
   - [Introduction](#introduction)
   - [EC2](#ec2)
      - [Create EC2 instance](#create-ec2-instance)
   - [Install Jenkins](#install-jenkins)
      - [Connect to EC2 ](#connect-to-ec2)
   - [Configure Jenkins](#configure-jenkins)
      - [Credentials](#credentials)
      - [Plugins for Jenkins](#plugins-for-jenkins)
   - [Jenkins jobs](#jenkins-jobs)
      - [Creating Jenkins jobs](#creating-jenkins-jobs)
      - [Executing Jenkins jobs](#executing-jenkins-jobs)
   - [Setting up AWS EKS creation/access](#setting-up-aws-eks-creation-access)
   - [Install ARGO CD](#install-agro-cd)
   - [Configuring Argo CD for Kubernetes Deployment](#configuring-argo-cd-for-kubernetes-deployment)
   - [Automation of CICD when code is updated](#automation-of-cicd-when-code-is-updated)

## Introduction
This tutorial focuses on automating the creation and deployment of Docker containers to Kubernetes using GitOps principles. It walks through setting up a CI/CD pipeline with Jenkins, from writing Dockerfiles and Jenkinsfiles to installing Jenkins and configuring jobs for container automation. The integration of ArgoCD ensures seamless deployment to Kubernetes clusters. You'll also learn how to automate GitHub with Jenkins using webhooks, allowing for automated updates and container management. Prerequisites include setting up GitHub and DockerHub accounts.


<!--GitOps
Dockerfile Jenkinsfile workflow
Jenkins installation
Jenkins jobs setup
ArgoCD  (GitOps)your  installation 
ArgoCD  (GitOps) Cd
automating github with Jenkins using webhooks

Prerreq: have github and dockerhub set up. -->

## EC2

### Create EC2 instance
  1. select or enter the following:
     - name: kubernetes with jenkins
     - AMI: Amazon Linux
     - 
![1](https://github.com/user-attachments/assets/cbfb4915-0355-4d31-b45e-de4439583b57)

     - Instance type: t2.micro
     - Key Pair name: Kubernetes-keypair
![2](https://github.com/user-attachments/assets/0ec3abf9-ceec-4521-bcce-6d85ea93b0cd)
      
      - Storage: 30 GiB
      - Launch instance after this is completed
    

## Install Jenkins

### Connect to EC2 

  1. We will connect to EC2 instance with the "Connect" feature our AWS console provides.

![4](https://github.com/user-attachments/assets/146aac61-b9e4-4b08-84f3-1afcb505918e)

Note: Enable auto assign public IP address to avoid issues using "Connect"

  2. Select:
     - Connect using EC2 Instance Connect
     - ec2-user
     - Connect

![6](https://github.com/user-attachments/assets/c6f32d13-5a10-4f7c-b944-823c02b11038)

Installing the Jenkins, Git and Docker. Refer to the following:

https://www.jenkins.io/doc/tutorials/tutorial-for-installing-jenkins-on-AWS/#prerequisites


install git with - 
```
sudo yum install git -y
```
install docker  with - 
```
sudo yum install -y docker
```

## Configure Jenkins

Refer to previous project for initial Jenkins setup:
https://github.com/johnha881/AWS-Projects/tree/main/Cloudformation%20with%20Jenkins#jenkins-access

### Credentials
  
  We will set our credentials so Jenkins is able to access other resources. 

  1. Head to: Dashboard-> Manage Jenkins-> Credentials -> System-> Global credentials(unrestricted) -> add credentials

![18](https://github.com/user-attachments/assets/c9aa9883-ccff-44e3-b4ec-6ac19ff0b95a)

  2. Before we proceed, create accounts on Hub.Docker.com and Github.com
  3. For Docker Hub we will enter the following:
     - Kind: Username and password
     - Scope: Global
     - Username: Your user name
     - Password: Your Docker Hub password
     - ID: dockerhub

  ![10](https://github.com/user-attachments/assets/4cbd0b2c-519f-4753-9e87-151699007ae7)

  4. For Github we will enter the following:
     - Kind: Username and password
     - Scope: Global
     - Username: Your user name
     - Password: Your GitHub password(Token)
     - ID: github

     - To obtain a token to use as password from github:
       - In github select your user icon on the top right
       - Select settings
       - Select developer settings

      ![11](https://github.com/user-attachments/assets/e2bee6d1-131a-4708-b1b4-b2a6d7a9db10)

       - Select Personal access token - > Tokens (Classic)
       - Generate new token -> Generate new token (Classic)
       - Note: gitops
       - Check the follow:
         - repo
         - admin:repo_hook
         - notification

![19](https://github.com/user-attachments/assets/6c8d236f-f161-4dc8-b39b-aff1427df7b2)

![12](https://github.com/user-attachments/assets/ae132a09-52c8-439b-8dc9-69fe4b04e3ab)

![13](https://github.com/user-attachments/assets/7760948a-989d-4f5e-9c34-1fdb8231cc8e)

  5. A token is generated for you to use as your github password for Jenkins. 

  6. You should see the following:

![15](https://github.com/user-attachments/assets/3a73d4eb-7dbf-4beb-a373-c9ec7db4c76c)

  Note: I selected "Treat username as secret" when creating our credentials.

### Plugins for Jenkins

  1. We will need a few plugins for our Jenkinsto interact other resources.
  2. Select " Manage Jenkins" -> "Plugins" - > "Available plugins"
  3. Search and select the following: 
    - EC2
    - Docker plugin
    - Docker pipeline
    - Github integration plugin
    - Parameterized trigger plugin

![17](https://github.com/user-attachments/assets/b790e751-3beb-410a-9078-659270cc442e)

## Jenkins jobs

### Creating Jenkins jobs
- We will create 2 Jenkins job. First one will create our Docker container. Second one will store and update our container to be pushed to a cluster. 

1. Head back to Jenkinsand select "New Item"
2. Enter/select the following to create a new job:
   - Enter an item name: buildimage
   - Select an item type: Pipepline

![20](https://github.com/user-attachments/assets/e87a95bb-1eb4-411b-84df-2a04d8aa5522)

3. Enter/select the follow on the pipelines page:
   - Definition: Pipeline script from SCM
   - SCM: Git
   - Repository URL: https://github.com/johnha881/kubernetescode.git
       (this repository contains our Dockerfile, JenkinsFile, app.py(container Code), requirements.txt)
   - Credentials: github credentials we created earlier

![21](https://github.com/user-attachments/assets/9496fb86-c255-464c-8c7d-512a9fde7eb8)

4. We will repeat the steps for our second job
   - Enter an item name: updatemanifest
   - Select an item type: Pipepline

![22](https://github.com/user-attachments/assets/d9619652-f1d5-406e-8d92-74ba8f3f9519)

5. Select "this project is parameterized"
   - Name: DOCKERTAG
   - Default value: latest

![23](https://github.com/user-attachments/assets/3139a18d-eac0-4236-a10a-e0883aa116c2)

   - Definition: Pipeline script from SCM
   - SCM: Git
   - Repository URL: https://github.com/johnha881/kubernetesmanifest.git
       (this repository contains our Jenkinsfile and depolyment file)
   - Credentials: github
   - Branch Specifier: */main

![24 5](https://github.com/user-attachments/assets/cdd1fa79-8936-4d90-bf79-12756bc0cc5a)

### Executing Jenkins jobs
  
  1. Head back to dashboard and you should see the following:

![26 5](https://github.com/user-attachments/assets/8c54117b-2ea3-4d9c-8fcc-6c2271d5ddff)

  2. Select "buildimage"
  3. Select "Build Now"

![25 5](https://github.com/user-attachments/assets/861e5b36-d17a-46ce-b7b6-cda72dd58ed8)

  4. While the job is running you can view what is being created by selecting the number corresponding with the job:

![32](https://github.com/user-attachments/assets/37bbfe8f-0cb1-42cd-99f8-4f875912c018)

  5. Select "Console Output" to see the pipepline steps:

![29](https://github.com/user-attachments/assets/7d86a861-9fc7-4c10-9a19-2260a4e54cae)

  6. If everything works you should see "SUCCESS" at the end

![30](https://github.com/user-attachments/assets/b7bf7931-1358-4f74-b435-a1e274959a4a)

  7. Check your dockerhub and you will see our updated docker image saved. 

![31](https://github.com/user-attachments/assets/79e4c794-b71c-4156-a1a5-3f48bca6d37e)

## Setting up AWS EKS creation/access

1. Install Chocolately 
  - Follow the instructions here to install : https://chocolatey.org/install

2. Install AWS CLI  
  - Follow the instructions here to install : https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html

3. Install eksctl with the follow command in powershell(run as admistrator):
   ```
   choco install eksctl
   ```
4. Us the following command to see if we have eksctl installed by checking the current version:
   ```
   eksctl version
   ```

5. Enter the following to create a cluster:
   ```
   eksctl create cluster 
   ```

6. You can check your Cloudformation console to see its progression.

![36](https://github.com/user-attachments/assets/7fac41ae-e858-4802-ad84-bf657a362a98)

## Install ARGO CD
  The following steps can be find here: https://argo-cd.readthedocs.io/en/stable/getting_started

1. Run the following command lines to install ARGO CD:
   ```
   kubectl create namespace argocd
   kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
   ```
![37](https://github.com/user-attachments/assets/81bf740e-9e28-4f80-a07b-2b85a419bf97)

2. Run the following command line to install ARGO CD CLI:
   ```
   choco install argocd-cli --force
   ```

![38](https://github.com/user-attachments/assets/0c1909fb-0cd1-445b-bd0a-e8c01a1cef31)

3. Run the following command line to expose Argo CD API server:
   ```
   kubectl port-forward svc/argocd-server -n argocd 8080:443
   ```

4. Open a browser and entering the following to access ARGO CD: localhost:8080
   - You should see the following:

![40](https://github.com/user-attachments/assets/88b56529-f12d-4a51-8f20-fb88052ea5f1)

5. Admin will be the user name. We will need to find the password now.
  - Open a new terminal to retrieve the password with the following command:
    ```
    argocd admin initial-password -n argocd
    ```

![41](https://github.com/user-attachments/assets/80543d05-ba54-47b6-aa23-45ea0dd5fa02)

6. Successful login:

![42](https://github.com/user-attachments/assets/0b02d931-3e83-4cf8-90f8-2e5e90319cac)

## Configuring Argo CD for Kubernetes Deployment

1. Find the link to kubernetesmantifest repository that will be used as source:
   - https://github.com/johnha881/kubernetesmanifest.git

![44](https://github.com/user-attachments/assets/4d4cecdf-b25d-44b4-8e9d-d9418bad75ec)

2. In ARGO select " + New App"

![43](https://github.com/user-attachments/assets/170c793d-347a-47e7-8879-63b801a60752)

3. Enter the following for app creation:
  - Application name: "flaskdemo"
  - Project Name: "default"
  - Sync Policy: Automatic
  - Repository URL: "https://github.com/johnha881/kubernetesmanifest.git"
  - Path: "./" (Depending on how your pathing is set up in github)
  - Cluster URL: https://kubernetes.default.svc
  - namespace: "default"

![45](https://github.com/user-attachments/assets/ce8806fa-1891-44f6-a624-6c51732066b1)

![46](https://github.com/user-attachments/assets/dde20c6e-460a-4356-9140-a6caa6284651)

4. Select "Create" after entering all the correct information.
  - flaskdemo app should now be displayed like the following:

![47](https://github.com/user-attachments/assets/1fd8e0f5-9812-4d5f-b873-f8d3bd9c9d73)

5. Kubernetes is now being deployed and you can see the flow when you click on flaskdemo.

![48](https://github.com/user-attachments/assets/709764ca-cbe1-4613-9932-1200368f2b83)

6. Head back to your terminal to view the current kubernetes pods running.
   - Enter the following command to view current pods:
     ```
     kubectl get pods
     ```
![49](https://github.com/user-attachments/assets/6f8e60b6-27a2-4439-86d2-e2229754e3d5)

7. To get the load balancer URL to view our kubernetes deployment enter the following command:
   ```
   kubectl get svc
   ```

![50](https://github.com/user-attachments/assets/05df257d-1f64-4694-9c13-6ff03db5c9be)

8. Enter the loadbalancer URL in your browser to see if our application was successful.

![51](https://github.com/user-attachments/assets/8fd17050-84f3-4ac4-ab78-3a3c6c56746f)

## Automation of CICD when code is updated

1. Head to kubernetescode repository to set up webhook for automatic triggering of the stack.
  - Go to settings -> Webhooks -> Add Webhook

![52](https://github.com/user-attachments/assets/3bb1ae11-00a6-4bee-8a70-bfaf588ee869)

![53](https://github.com/user-attachments/assets/7317c978-f986-408f-9771-37b0353ce909)

  - Enter the following:
    - Payload URL: https://54.210.3.37:8080/github-webhook/
    - Content type: application/json
    - Which events would you like to trigger this webhooks?: Just the push event

![54](https://github.com/user-attachments/assets/a2d97f08-2ab8-478b-a61e-5cfb418b8f8b)

2. Head back to Jenkins to configure out buildimage job
   - Select buildimage -> configure -> Build Triggers
     - Check mark Github hook trigger for GITScm polling
    
![55](https://github.com/user-attachments/assets/759b4eb3-fd7d-465a-adb0-1820233a1e91)

![56](https://github.com/user-attachments/assets/ecbfd72a-1c35-460e-a12c-7d91ca1d9bdb)

3. Now change the output of our python code by going to:
  - https://github.com/johnha881/kubernetescode/blob/main/app.py
    - Change the return from "Hello, Docker!" to a different text output.

![65](https://github.com/user-attachments/assets/613df17d-8c00-42e6-aee8-9b3b70a7bdd7)
    - Save code change

4. Jenkins will now react to the changes and trigger the correct jobs.
5. To see the jobs run correctly, head back to jenkins and select Console Output for our buildimage job.

![57](https://github.com/user-attachments/assets/cfdd7d81-7877-46a0-896f-5b56b8aa95ff)

![58](https://github.com/user-attachments/assets/df5db317-b747-438c-a328-296aebe2d946)

![59](https://github.com/user-attachments/assets/faf03884-65dd-411e-be97-0c5c9a7ba1b9)

6. When the Jenkins jobs are finished, the updated docker image should be pushed into DockerHub.

![60](https://github.com/user-attachments/assets/3afb473b-84fb-4df4-99f9-23f9651223c9)

7. We can now check if our change propagated to our kubernetes by going back to our browser and entering our loadbalancer URL again.

![62](https://github.com/user-attachments/assets/e134c08a-08cd-42da-8343-951994720dcf)

8. Check our ARGO CD for updated pods.

![63](https://github.com/user-attachments/assets/31f355e9-6fd5-4891-8ee2-95bbcc24fe19)

9. Check for updated pods through terminal.

![64](https://github.com/user-attachments/assets/107a867b-1dfc-4839-9d83-361f14d4e3c0)



## Issues I ran into
  - Forgot password for Jenkins login. Had to alter files inside ec2 to regain access.
  - Jenkins - temp space/swap space default size is too small. Warning threashold had to be increased. 
  - Old version of AWS CLI gave me credentials issue. Updating AWS CLI resolved issue.
  - eksctl create cluster did not work when setting instance size to t3.micro (was aiming for free tier). t5.large is the default size.

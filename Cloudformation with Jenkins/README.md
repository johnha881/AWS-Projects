# CloudFormation with Jenkins

- [Introduction](#introduction)
- [Amazon Machine Image with Jenkins](#amazon-machine-image-with-jenkins)
- [Jenkins access](#jenkins-access)
- [Creating first job with Jenkins](#creating-first-job-with-jenkins)
  - [Installing CloudFormation Plugin ](#installing-cloudformation-plugin)
  - [GitHub Jenkins connection](#github-jenkins-connection)
  - [Build Jenkins Job](#build-jenkins-job)
- [Installing AWS CLI with systems manager (SSM)](#installing-aws-cli-with-systems-manager-ssm)
- [Jenkins with EC2](#jenkins-with-ec2)
  - [Create a Jenkins file in our repository](#create-a-jenkins-file-in-our-repository)
  - [Roles for EC2 to create AWS services](#roles-for-ec2-to-create-aws-services)
  - [Creating an S3 bucket using a Jenkins pipeline](#creating-an-s3-bucket-using-a-jenkins-pipeline)
- [Issues I ran into](#issues-i-ran-into)

## Introduction
This guide walks you through using Jenkins to manage AWS infrastructure as code. Begin by configuring Jenkins with the necessary plugins and access permissions. Set up a Jenkins job that triggers the creation of AWS CloudFormation stacks. As you execute the job, Jenkins orchestrates the deployment of your infrastructure on AWS, ensuring efficient management and consistent results.


## Amazon Machine Image with Jenkins

  1. Search for Marketplace in AWS console.
  2. On AWS Marketplace look for "Discover products".

![1](https://github.com/user-attachments/assets/4726114e-4041-4e14-8226-e1e0dbafd745)

  3. Search for "Jenkins" and select your AMI of choice. I will use "Jenkins by Cloudeya" as it has an option for micro size instances for free tier pricing.
 
 ![3](https://github.com/user-attachments/assets/62e652a6-3fe9-4ca1-b354-292772477ac0)

  4. On the next page, you will see an option to select your instance size and pricing. We will use t3.micro and select us-east for N. Virginia
  
![4](https://github.com/user-attachments/assets/97ded223-8980-42d7-a6ac-767cc11892d8)

  5. Scroll back to the top and select "Subscribe".

![5](https://github.com/user-attachments/assets/da56aa94-18c0-41b7-a978-98dd92ebadb7)

  6. Accept the terms and your subscription will start to initiate.

![6](https://github.com/user-attachments/assets/d5ee30a4-9e15-43d7-8dcf-37e2f3f874cc)

  6. Once initiating has finished, select “Configuration” at the top.

![7](https://github.com/user-attachments/assets/abceaaeb-c22d-4505-af06-f08140753b71)

  7. Leave most of the options as default, but be sure that t3.micro is your EC2 Instance Type.

![9](https://github.com/user-attachments/assets/18d90c5a-3117-4623-9958-5cc213a0d40c)

  8. For Security Group Settings we will select “Create New Based on Seller Settings”. We will then name it “jenkins ec2 SG”. We will also create a key pair and name it “Jenkins”.

![10](https://github.com/user-attachments/assets/368552da-2860-4d1e-847a-81b6d6d5c3c0)

## Jenkins access

  1. Launch the instance and we will see this page:

![11](https://github.com/user-attachments/assets/bbd833da-5930-491f-90b6-6e8db4971e9b)

  2. Head back to EC2 page and look for our newly created instance. I will rename it "Jenkins".
  3. We will now begin to set up jenkins in our newly created instance. 
  4. We can now access “Jenkins” through our browser thanks to our security group. Port 8080 will be open to us.
  - First we need to find our Public IP address:

      ![13](https://github.com/user-attachments/assets/e3c1130d-23ad-4f63-a2f6-fb3fe7070864)
    
  - We then can head to our browser and enter the follow to access Jenkins: http://3.87.105.180:8080/
  - You should see this page:

![15](https://github.com/user-attachments/assets/b53f4a87-b91a-4d46-ad9b-8e4791885ef3)

  5. Now that we know how to access our Jenkins from the browser, we need to get our Administrator password from our instance by SSH in.
  6. Head to "CloudShell" so we can SSH into our instance.
  7. First we will need to upload our key pair to our CloudShell:
      - select "Action" -> "Upload"
      - Find "Jenkins.pem" on your local machine and upload it
      - Note the file path it was uploaded to:

![21](https://github.com/user-attachments/assets/01cc4b20-868a-4cf5-9170-44c216d6199e)

  8. Enter the follow so we have access permission for our key pair:

    chmod 400 /home/cloudshell-user/Jenkins.pem

  9. Now enter the following SSH access command to our instance::

    ssh -i /home/cloudshell-user/Jenkins.pem ec2-user@3.87.105.180

![17](https://github.com/user-attachments/assets/110d1f48-ac74-48f1-b410-66004b73ba4b)

  10. Enter the following to retrieve our administrator password for Jenkins:

    sudo cat /var/lib/jenkins/secrets/initialAdminPassword

  11. You should see a printout of the Administration password we need to Jenkins:

![18](https://github.com/user-attachments/assets/3d9fea24-7ab3-459f-b8c2-82924b0a10b7)

  12. Now that we have our password, head back to our browser and enter the password

![15](https://github.com/user-attachments/assets/e7f5afaf-8b36-46c0-98be-c48dd6fbc92c)

  13. Next select "Install suggested plugins".

![22](https://github.com/user-attachments/assets/96cf5ef2-0b92-48c0-b926-447eac669ad1)

  14. Once the plugins are installed you will see this:

![19](https://github.com/user-attachments/assets/3f1da207-4fe7-428f-bc52-37944efaeaa7)

  15. Save and finish.
  16. If all goes well you will see this:

![20](https://github.com/user-attachments/assets/565f03cf-f9d2-4cac-8777-8b17b82c202f)

## Creating first job with Jenkins

### Installing CloudFormation Plugin 
  
  1. Select "Manage Jenkins" on the left pane and then select "Plugins"

![23](https://github.com/user-attachments/assets/e62ee1b8-7f20-44f2-a06a-b0b715094889)

  2. Select "Available plugins" and search for "Cloudformation". Select "Cloudformation" and then click "install" on the top right corner.

![24](https://github.com/user-attachments/assets/6cdcbcf9-e0f6-4607-ae28-7ef2f787a71a)

### GitHub Jenkins connection

  1. We will need a github repository with the follow Json file. This file is a simple CloudFormation template which will create an S3 Bucket:

  ![25](https://github.com/user-attachments/assets/28b851da-4ae0-4d04-b4d6-8314eae19b17)

  2. Return to DashBoard and select "New item".

![26-1](https://github.com/user-attachments/assets/6611100a-dff3-4033-b5e5-6ce1b105c2f8)

  3. On the "New item" page will will enter and select the following:
     - Name: ctfdemo-jenkinsplugin
     - Freestyle project
     - Click: OK

  4. On the next page we will link our Github cloud formation template with Jenkins:
      (Before you continue, be sure Git is installed on your Ec2 instance)
     -Description: CloudFormation Simple template to create S3

![28](https://github.com/user-attachments/assets/69f2bf3d-6d9f-4969-9d62-954bb7611961)

  - Source: Git
  - Repository URL: URL where repository containing your JSON file is located
  - Credentials: Your Username and password for your github
  - Branches to build: change from default of "Master" to "Main" as github has new naming conventions

![29](https://github.com/user-attachments/assets/071ce2ee-34ce-485c-93b6-dad6ab08cd5d)

  - Build Environment: Create AWS Cloud Formation stack
    - AWS Region: US East (Northern Virginia) Region
    - Cloud Formation recipe file/S3 URL (json): simplests3cft.json
    - Stack name: jenkinscftplugin.s3
    - ASW Access Key: xxxxxxxxxxxx
    - AWS Secret Key: xxxxxxxxxx
      (You will need to create an IAM user that has programmatic access with these policies:
      - AWScloudformationfullaccess
      - Amazons3fullaccess
      )
    - Find Access/Secret key from user you created
    - Click: Save

![31](https://github.com/user-attachments/assets/8d929f2f-b3aa-4577-a5c3-d609e1d68d3f)

### Build Jenkins Job
  
  1. Click "Build Now" to see our project in motion

![32](https://github.com/user-attachments/assets/160b375a-9cb7-4f97-910a-eb8898577cee)

  2. The project will attempt to build our cloudformation.
  3. We will then click on the number next to the project being built and look at "Console Output"

![34](https://github.com/user-attachments/assets/8514796d-3005-4995-b312-b028d78dc0de)

  4. With receiving a successful build, we can go back to our CloudFormation page to see if one was created:

![35](https://github.com/user-attachments/assets/c76a9e9f-9411-4285-afc9-80bdb9811cd4)

## Installing AWS CLI with systems manager (SSM)
  Using SSM to sign into our instance without the need of key pairs

  1. First we need to give EC2 the policy role to interact with "Systems Manager"

![Screenshot 2024-08-30 at 2 20 20 PM](https://github.com/user-attachments/assets/0671f5e9-007e-4c17-98a3-8d0d8a9ca78f)

![Screenshot 2024-08-30 at 2 21 29 PM](https://github.com/user-attachments/assets/86d65c0a-5099-4725-90a9-038fa14294d1)

  2. Head IAM and roles. Create a new role with this policy attached to it:       
    "AmazonSSMManagedInstanceCore"

![Screenshot 2024-08-30 at 2 22 19 PM](https://github.com/user-attachments/assets/c96246a0-e0c8-4220-8957-ebb74827f41a)

  3. Next attach the policy to our EC2 instance.
 
![Screenshot 2024-08-30 at 2 24 37 PM](https://github.com/user-attachments/assets/6b6a2a54-b69c-4eb5-94eb-2c23b29439c7)

![Screenshot 2024-08-30 at 2 25 02 PM](https://github.com/user-attachments/assets/bad2b045-4bd2-4dd0-b058-d8d841dc828a)

  
  4. Head to Systems Manager console and select "Quick Setup".
  
  ![Screenshot 2024-08-30 at 1 43 23 PM](https://github.com/user-attachments/assets/bd995855-87ef-4d7b-8917-feb001a55a2a)

  5. Under host management, select "Create"

  ![Screenshot 2024-08-30 at 1 48 40 PM](https://github.com/user-attachments/assets/7c99ba86-bade-4927-ac63-a8f71cb15f49)

  6. Configure the following:
     - Targets: Entire Organization
  
![Screenshot 2024-08-30 at 1 49 41 PM](https://github.com/user-attachments/assets/a48431d6-597c-48f1-96d5-9dd5d1093c91)

  7. On the left pane look for "Node Management"
  8. Look for "Session Manager" and then select "Start session"
  
![37](https://github.com/user-attachments/assets/c57aa29d-c0d0-4296-a2da-339e242d551c)

  9. Select "Jenkins" instance and start session.

![Screenshot 2024-08-30 at 2 43 28 PM](https://github.com/user-attachments/assets/fc7a5c86-936b-46d8-b414-f3cd577202d0)
    
  10. In the shell terminal, run: sudo yum update
  
![Screenshot 2024-08-30 at 3 01 52 PM](https://github.com/user-attachments/assets/e7ff8fc7-713f-439b-aff1-7515d8e69030)

  
  11. Next install AWS CLI with : sudo yum install awscli ( we have it installed since we used amazon linux).

## Jenkins with EC2

### Create a Jenkins file in our repository

  1. Head to our github repository and create a jenkins file (defines the pipeline and stages).

 1. Create a Jenkins file in our repository that also has our simplests3cft.json file with. We will name it jenkinsfile.

    ```
    pipeline {
    agent any
    stages {
        stage('Submit Stack') {
            steps {
            sh "aws cloudformation create-stack --stack-name s3bucket --template-body file://simplests3cft.json --region 'us-east-1'"
              }
             }
            }
            }

    ```
### Roles for EC2 to create AWS services 

 1. We need to provide our EC2 with the proper roles to create cloudFormation and S3 Bucket.
 2. Head to IAM again and and go to our roles
 3. Look for "SSM-EC2-Role" and select it.

 ![Screenshot 2024-08-30 at 3 48 22 PM](https://github.com/user-attachments/assets/2074949c-4e31-413b-a00b-05d8b1e0a162)

 4. Click on "Add Permissions" and select "attach policy".
 
![Screenshot 2024-08-30 at 3 46 06 PM](https://github.com/user-attachments/assets/b0292f7b-e48e-4c70-9253-68efa9796659)

 5. Enter "Cloudformation" and look for "AWSCloudFormationFullAccess". Select that and click attach policy.
 
![Screenshot 2024-08-30 at 3 47 20 PM](https://github.com/user-attachments/assets/f13f22fa-7a79-4324-bf1b-fb7b4e13d0a1)

 6. We will also attach another policy for S3. Look for AmazonS3FullAccess

![Screenshot 2024-08-30 at 3 57 02 PM](https://github.com/user-attachments/assets/3337557b-12cd-42d4-af46-272854846ece)

![Screenshot 2024-08-30 at 3 57 29 PM](https://github.com/user-attachments/assets/703375a3-dc1a-4b10-afe2-d58f53db8860)


### Creating an S3 bucket using a Jenkins pipeline
 
 1. Head back to Jenkins to configure our new job
  - http://3.87.105.180:8080
 2. Select "New Item"
 3. New Item page:
    - Name: Pipeline-s3-cft
    - Select an item type: Pipeline
    - Select "Ok" to continue
 
![Screenshot 2024-08-30 at 4 19 32 PM](https://github.com/user-attachments/assets/61f9edea-0a35-43f7-8b5b-4cc32474dda4)
 
 4. On the Configuration page:
    - Pipeline: Pipeline script from SCM
    - SCM: Git
    - Repositories:
      - Repository URL: https://github.com/johnha881/cloudformationdemo
      - Credentials: johnha881/*****
      - Branch Specifier: */main
    -Script Path: Jenkinsfile
    - Save
 
![Screenshot 2024-08-30 at 4 26 56 PM](https://github.com/user-attachments/assets/9b9a955a-52ce-455f-b5a0-5107ebf945ae)

![Screenshot 2024-08-30 at 4 27 12 PM](https://github.com/user-attachments/assets/0784d1a7-e0ba-474b-b3b1-26ad3b3ed0c6)
  
  5. Select build and click on the #1 on our build

![Screenshot 2024-08-30 at 4 36 39 PM](https://github.com/user-attachments/assets/12027a29-eaee-4b21-9f82-8001a1205fbb)

 6. From there select "Console output" and you should see this if successful:

![Screenshot 2024-08-30 at 4 36 59 PM](https://github.com/user-attachments/assets/8322b2a2-fc3e-4826-a79b-afc0ecd299f0)

7. Head over to CloudFormation page and check if our stack was created.

![Screenshot 2024-08-30 at 4 41 04 PM](https://github.com/user-attachments/assets/a12b5fb2-c323-4ac3-a03c-08753abd025d)

## Issues I ran into

  - AMI Instance size needs to be taken into consideration for its cost
  - Git needs to be installed on our EC2 instance for Jenkins to communicate with github
    - sudo yum install git -y
  - Need to restart EC2 instance after applying role policy before Systems Manager was able to detect our instance. 

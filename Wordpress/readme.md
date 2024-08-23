# Wordpress

- [Introduction](#introduction)
- [Creating VPC](#creating-vpc)
  - [VPC](#vpc)
  - [Subnets](#subnets)
  - [Internet Gateway](#internet-gateway)
  - [Route Tables](#route-tables)
- [EC2](#ec2)
  - [Creating EC2](#creating-ec2)
- [RDS](#rds)
  - [Create RDS Instance](#create-rds-instance)
- [EFS](#efs)
- [Installing wordpress to EC2](#installing-wordpress-to-ec2)
- [Mount EFS to EC2](#mount-efs-to-ec2)
- [Connecting RDS and EC2](#connecting-rds-and-ec2)
- [Load balancer](#load-balancer)
  - [Create load balancer](#create-load-balancer)
- [Check if EFS is working with uploads to wordpress](#check-if-efs-is-working-with-uploads-to-wordpress) 
- [Scaling](#scaling)
- [Availability](#availability)
- [Cost Optimization](#cost-optimization)
- [Security](#security)
- [Issues I ran into](#issues-i-ran-into)

## Introduction
This tutorial guides you through setting up a high-availability WordPress environment on AWS. Begin by creating a custom VPC with subnets in multiple Availability Zones, attaching an Internet Gateway, and configuring route tables for internet access. Next, launch an Amazon RDS instance with Multi-AZ deployment for reliable database access, and adjust security group settings to permit communication with EC2 instances. Then, set up Amazon EFS for shared storage, creating a file system and configuring mount targets within your VPC. Launch EC2 instances to host WordPress, install and configure it, and ensure it connects to the RDS database and uses EFS for uploads. Add an Application Load Balancer to distribute traffic. Finally, test your WordPress site to confirm its functionality and monitor performance, adjusting scaling settings as needed.

## Creating VPC

  ### VPC
  
  1. Go to VPC page in AWS.
  2. Find "Create VPC" button and click it.
  3. Enter "MyVPC" for the name.
  4. Choose a CIDR block pattern. We will use "10.0.0.0/16".
  5. Leave the rest as default and click "Create VPC".

![2](https://github.com/user-attachments/assets/aafa2b9a-ca8f-40d6-b83c-67fc4bf05842)

   ### Subnets
  
  1. Click on subnet on the left pane and then select "create subnet"

  ![3](https://github.com/user-attachments/assets/18a41375-59a8-45d7-91d7-e635251a7271)

  2. For VPC ID select the one we just created(MyVPC).
  3. Next, choose a subnet name. We will use "my-first-custom-subnet".
  4. Select a Availability Zone. We will use "us-east-1a" for this example.
  5. VPC CIDR block will be "10.0.0.0/16" again.
  6. For IPv4 subnet CIDR block, we will use 10.0.1.0/24.
  7. Leave everthign else as default.
  8. Create subnet.

![4](https://github.com/user-attachments/assets/2fc7ab65-a01e-462e-a4e8-20911517270e)


  ### Internet Gateway

  1. Click on "Internet gateways" on the left pane and then select "create internet gateway".
  2. Name the Internet gateway "my-custom-Internet-gateway", then select "create".

![5](https://github.com/user-attachments/assets/3c996e34-b76c-4447-adc0-8f6f4ac0b3ef)

  3. Select te internet gateway you just created and click on "Actions" drop down menu.

![6](https://github.com/user-attachments/assets/807d1019-3870-4003-8ae3-ec77704b937d)
     
  4. Choose the VPC you created and click "Attach"

![7](https://github.com/user-attachments/assets/b351b2bc-8650-412d-802c-cc36de629022)

  ### Route Tables

  1. Click on "Route tables" on the left pane and then select "Create route table".
  
![8](https://github.com/user-attachments/assets/dfe23ab8-5a65-4c44-8ea5-7f316d96f906)

  2. Name the route table "my-cust-route-table."
  3. Under VPC select "MyVPC".
  4. Click "Create route table".

  ![9](https://github.com/user-attachments/assets/838a5dea-5d7c-4132-aa57-73c3471330d9)

  5. While having "my-custom-route-table" selected, go to "Actions" selection and select "Edit routes"
  Enter the following for edit routes
    - Destination: 0.0.0.0/0
    - Target: Internet Gateway
       - Select the internet gateway you created.
    - Save changes
  
  ![10](https://github.com/user-attachments/assets/2d4d38f9-040e-4bfb-a3e6-ee76bd3fb959)

  6. In the route table settings, select "Subnet Assocdiations" and then click on "Edit subnet associations"
 
![11](https://github.com/user-attachments/assets/9ff0e938-7ccc-44ed-8a8f-96bfb54b298e)
  
  7. On the subsequential page, select "my-first-custom-subnet", then click "Save associations"

![12](https://github.com/user-attachments/assets/efc26905-8dfb-44fb-a68b-3faa6b9ae086)

## EC2

  ### Creating EC2
  
  1. Go to EC2 page in AWS.
  2. Click on "Instances" on the left pane and then select "Launch instances"
  3. Select the following on the "Launch an instance page":
     - Name: WordPress instance
     - AMI(Quick Start): Amazon Linux
     - Aamazon Linux 2023 AMI(Free tier eligible)
     - Architecture: 64-bit (x86)
      ![23](https://github.com/user-attachments/assets/2bcf5d2b-2c57-4495-b0e7-bb7298ab78fa)

     - Instance type: t2.micro
    ![24](https://github.com/user-attachments/assets/24722f73-f7cb-4693-84fd-30669a9c9f88)

     - Key Pair: Create new key pair
         - Key pair name: wordpressKeyPair
         - key pair type: RSA
         - Private key file format: .pem
         - Create key pair
         - A file will be downloaded, which contains your key pair value
  ![25](https://github.com/user-attachments/assets/14759ce3-b121-433a-9049-528e2db7e451)
![26](https://github.com/user-attachments/assets/319075d6-cdef-4f14-adcf-64cc8cfd11a2)


     - Network settings
       - VPC: "MyVPC" we created
       - Subnet: my-first-custom-subnet-us-east-1a
       - Firewall(Security Group: Create security group
       - Security group name: wordpress-instance-SG
       - Description: wordpress-instance-SG created 2024-08-14T17:40:40.792Z
![27](https://github.com/user-attachments/assets/662de477-088f-4660-a408-b663789e9901)

       - Inbound security: Add security group rule:
           - Http:
             - Type: http
             - Port range: 80
             - Source: 0.0.0.0/0
             - Description: http
           - Https:
             - Type: https
             - Port range: 443
             - Source: 0.0.0.0/0
             - Description: http
           - Http:
             - Type: NFS
             - Port range: 2049
             - Source: 0.0.0.0/0
             - Description: http

  ![28](https://github.com/user-attachments/assets/01a05e93-a9cf-463d-8b5c-208a1f9a3025)

  5. Leave everything as default and click "Launch instance"

     **********make sure ec2 can talk to rds
     ****Configure Access Points (Optional):

        If you need to create access points, go to the EFS dashboard and click on “Access Points.”
        Click “Create access point” and follow the prompts.
     created access point for EFS

  6. Cloudshell to connect
     - Connect using ssh -i wordPressKeyPair.pem ec2-user@34.194.21.221 ******* adjust ip address for public view
     - mysql -h database-for-wordpress.chmkyke6k5vy.us-east-1.rds.amazonaws.com -P 3306 -u admin -p connect works to connect to RDS


## RDS

  ### Create RDS Instance

  1. Go to RDS page in AWS.
  2. Click "Create database".
  
  ![13](https://github.com/user-attachments/assets/607fdf43-3412-4eb1-87b3-d675caeec5e8)

  3. Choose MySQL.
  
![14](https://github.com/user-attachments/assets/34e8860c-b8bb-44bf-acb2-c1d697fee0ba)
  
  4. For "Templates" we will use "Free tier".
  5. For DB instance identifier" we will use "database-for-wordpress".
  
  ![15](https://github.com/user-attachments/assets/88543945-9e6f-451a-a944-81a6c8cf5900)

  6. Under Credentials we will use the follow:
    - Master username: admin
    - Credentials management: Self managed
    - Master password: password of your choice
  7. Under "Instance configuration" we will choose "db.t3.micro".

![16](https://github.com/user-attachments/assets/e1d29aa5-4963-4efa-befa-6383dd8e16db)

  8. Under "Connectivity", select the following:
     - Connect to an EC2 computer resource
     - EC2 instance: WordPress instance

  ![rds cmputer source](https://github.com/user-attachments/assets/727097b0-1fac-4fad-bf7d-5383f575a3b9)

  9. Leave everyhing as default and click "Create database".

## EFS

  1. Go to EFS page in AWS.
  2. Click "create file system".
  
  ![20](https://github.com/user-attachments/assets/d0433e07-bd0f-4ea8-a923-500cbfef1452)

  3. Select "Customize".
  
  ![efs customize](https://github.com/user-attachments/assets/d0cbba63-537a-4721-b017-c7fa81e2d27c)

  4. Set the following options:
     - Name: efs-for-wordpress
     - File system type: One Zone
     - Availability zone: us-east-1a
     - Lifecycle management: None
  
  ![22](https://github.com/user-attachments/assets/83c3bfb1-6294-4a25-a151-3e62237de364)

  5. Leave the rest as default for now and create your EFS.
  6. Guide to connecting EC2 and EFS can be found here:
     - https://medium.com/@Devin007/connecting-ec2-to-efs-db9856778172
  
## Installing wordpress to EC2

  1. Connect to EC2 with SSH
     - We will use AWS Cloudshell to connect to our instance
       - Click on action and select up "upload file"
       - Select and upload the key-pair key : wordpressKeyPair.pem
![perm](https://github.com/user-attachments/assets/517b63a3-7922-4794-a2b6-88eb5d87c146)

       - Next enter: ssh -i wordPressKeyPair.pem ec2-user@<your ec2 public ip address>
       - Note the public ip address we are using that as been assigned to our EC2 instance

  2. Update linux with this command: sudo yum update -y
  3. Next, install apache: sudo yum install -y httpd
  4. Start apache: sudo systemctl start httpd
  5. Enble it to start on boot: sudo systemctl enable httpd
  6. Next, install PHP: sudo yum install php

  7. Download WordPress with this command:

    wget https://wordpress.org/latest.tar.gz
    tar -xzf latest.tar.gz
    sudo mv wordpress/* /var/www/html/
    sudo chown -R apache:apache /var/www/html/
    

  8. wp-config.php is not to be find in our diretory: cd /var/www/html/
  9. we will isntead copy sample config file: cp wp-config-sample.php wp-config.php
  10.  open wp-config.php :  sudo nano wp-config.php
    - change the following to match our RDS DB information:
    - define('DB_NAME', 'your_database_name');
    - define('DB_USER', 'your_database_user');
    - define('DB_PASSWORD', 'your_database_password');
    - define('DB_HOST', 'your_rds_endpoint');
  
  11. Restart apache with this command: sudo systemctl restart httpd    

## Mount EFS to EC2
    
  1. Install NFS client with: sudo yum install -y nfs-utils
  2. Create a directory to mount the EFS file system: sudo mkdir /mnt/efs
  3. Mount the EFS file system to the directory: sudo mount -t nfs4 -o nfsvers=4.1 <file-system-id>.efs.us-east-1.amazonaws.com:/ /mnt/efs
  4. Check if EFS is mounted: df -h

## Connecting RDS and EC2

  Use this documentation on how to connect your EC2 instanc and RDS:
  https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/ec2-rds-connect.html

## Load balancer

### Create load balancer
 
  1. Go to EC2 page in AWS.
  2. Click on Load balancer" on the left pane and then select "Create load balancer".
  3. Look for "Application Load Balancer" and click on "Create".
  
  ![33](https://github.com/user-attachments/assets/1156643f-651f-44c7-9bdf-0100acd11ede)

  4. On the "Create Application Load Balancder" page:
     - Load balancer name: wordpress-load-balancer
     - Scheme: Internet-facing
     - Load balancer IP address type: IPv4
     
      ![34](https://github.com/user-attachments/assets/f41ed8ce-6d1c-4c20-a004-9a10dbcc4bbd)

     - VPC: MyVPC
     - Mappings: us-east-1a, us-east-1b, us-east-1c
![35](https://github.com/user-attachments/assets/b693e8bb-9ced-493d-8ec2-0091b3d02436)

     - Health Check protcol: HTTPS
     - Security groups: wordpress-instance-SG

      - Select newly created target group: Load-balacer-target-ec2
      - Create load balancer

  5. Look for target group under load balancer on left pane and select "create target group".
     - Choose a target type: Instances
     - Target group name: load-balancer-target-ec2
     - Port: HTTPS: 443
  
  ![36](https://github.com/user-attachments/assets/5355178d-465d-40e9-b5b9-f3330a45b314)
       
  - IP Address type: IPv4
  - VPC: MyVPC
      
  ![37](https://github.com/user-attachments/assets/72a6ffc3-c620-4c9c-8452-13f936ffc48e)
     
  - Leave rest as default and click "Next"
  - Select WordPress instance
  - Click on "Include as pending below"
  - Note that our wordpress instance is now a target
  - Select "Create target group"

  ![target group select](https://github.com/user-attachments/assets/08393aa3-b5f7-44a4-92c7-73d90d550b06)

  6. Check target groups for healths status:
     - Select "wordpress-target-group
     - Under details you should see health status:

![health check](https://github.com/user-attachments/assets/3a9d8afe-d88a-47c9-ae65-1f0a51dd1fb6)


## Check if EFS is working with uploads to wordpress
 1.  Go to wordpress website and upload a image.
 2.  After thinkering with folder paths and permissions I was able to upload to the proper EFS folder to host on wordpress
    ![happy face](https://github.com/user-attachments/assets/63dc48ac-526d-4ea3-b717-66e08154dece)

## Scaling

### EC2
  - Use Auto scaling groups
  - Use CloudWatch to handle scaling policies (i.e when CPU exceeds 80% usage)

### RDS
  - Read Replicas to help with read traffic
  - Increase instance size will help with:  CPU, memory, storage, and networking capacity

### EFS 
  - Scales storage automatically
  - Choose the correct throughput modes: Elastic, Provisioned, and Bursting

### Load balancer
  - Routes traffic depending on the conditions set
  - With the help of ec2 ASG, load balancers will send traffic to healthy instances
  - Multi-AZ to route traffic for best performance

## Availability

### EC2
  - Deployable across multiple Availability Zones
  - Associate an Elastic IP with our EC@ instance to maintain a static public IP address, allowing failover
  - Auto scaling group and Load balancing higher availability

### RDS
  - Multi-AZ deployment - instance is replicated 
  - Stand by will be promoted to Primary if original instance fails.
  - Automatic backups and snapshots
  - Read Replicas

### EFS
  - Storage across multi-AZ
  - Automatically handles failover

### Load balancer
  - Distribute traffic through multi-AZ
  - Send traffic to heahtly instances

## Cost Optimization

### EC2
  - Correct instance size
  - Spot and Reserve instances over On-Demand for cheaper pricing
  - Auto Scaling Groups to create and terminate instances based on usage

### RDS
  - Reserve instances for savings instead of On-Demand
  - Storage type: Provisioned IOPS SSD( Most expensive), General Provisioned, Magnetic(cheapest option)
  - Back-up retention period
  - Snapshot retention period

### EFS 
  - Infrequent Access Storage for cost savings
  - Life Cycle Management to move less-used data to IA

### Load balancer
  - Choose the correct type: Application load balancer (Web apps) vs Network load balancer (Low latency)

## Security

### EC2
  - Security groups
  - Network access control
  - Least privilege policy
  - IAM roles
  - SSH key pair for access
  - MFA for SSH access
  - Cloudwatch and CloudTrail
  - Patching

### RDS
  - IAM roles
  - Security groups
  - AWS Key Management Service (KMS) for at rest and in transit of data
  - Encrypted backups and snapshots
  - Database Activity Streams: Use RDS Database Activity Streams for monitoring database activities and logging access attempts.
  - Credentials management with AWS Secrets Manager for rotating secrets

### EFS
  - IAM roles
  - Security groups
  - AWS Key Management Service (KMS) for at rest and in transit of data

### Load balancer
  - Security groups
  - SSL/TLS
  - AWS WAF

## Issues I ran into
  - Making sure you are using free tier template for RDS creation
  - public ip address for EC2 was set to disabled by me during creation, which caused me to use an elastic ip address. This was an error was elastic ip address incur fees. The work-around was to make in image(Amazon Machine Images) of my current WordPress EC2 instance and launch the instace with auto-assign ip enabled.
  - Installing MySQL on EC2 to test RDS was not as straight forward as other applications. I had to follow these instructions on how to do so: https://dev.to/aws-builders/installing-mysql-on-amazon-linux-2023-1512 
  - Ports needed to be open for EFS on port 2049

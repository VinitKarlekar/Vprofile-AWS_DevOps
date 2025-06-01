# VProfile Project

This repository contains the setup and deployment instructions for the **VProfile** application, a web application deployed on AWS using various services. The project is divided into two main parts: **Initial Deployment** using EC2 instances and AWS services, and **Rearchitecting** the application using PaaS and SaaS solutions like Elastic Beanstalk, RDS, Elastic Cache, and Amazon MQ.

## Table of Contents
1. [Project Overview](#project-overview)
2. [Initial Deployment](#initial-deployment)
   - [Security Groups and Key Pair](#security-groups-and-key-pair)
   - [EC2 Instances](#ec2-instances)
   - [Route53 Configuration](#route53-configuration)
   - [S3 Bucket and IAM Configuration](#s3-bucket-and-iam-configuration)
   - [Application Configuration](#application-configuration)
   - [Maven and AWS CLI Setup](#maven-and-aws-cli-setup)
   - [Deploying to Tomcat](#deploying-to-tomcat)
   - [Load Balancer and Target Groups](#load-balancer-and-target-groups)
   - [Auto Scaling Configuration](#auto-scaling-configuration)
3. [Rearchitecting with PaaS and SaaS](#rearchitecting-with-paas-and-saas)
   - [Security Groups and Key Pair for Beanstalk](#security-groups-and-key-pair-for-beanstalk)
   - [RDS Setup](#rds-setup)
   - [Elastic Cache Setup](#elastic-cache-setup)
   - [Amazon MQ Setup](#amazon-mq-setup)
   - [MySQL Client Setup](#mysql-client-setup)
   - [Elastic Beanstalk Environment](#elastic-beanstalk-environment)
   - [CloudFront Distribution](#cloudfront-distribution)
4. [Repository Details](#repository-details)

## Project Overview
The **VProfile** project demonstrates deploying a web application on AWS, initially using EC2 instances, S3, Route53, and load balancing, followed by rearchitecting it using AWS PaaS and SaaS services like Elastic Beanstalk, RDS, Elastic Cache, and Amazon MQ. This guide provides step-by-step instructions to replicate the setup.

## Initial Deployment

### Security Groups and Key Pair
- **Create Security Groups**: Set up security groups to allow communication between Tomcat, RabbitMQ, MySQL, and Memcached instances. Ensure the security groups are configured to allow traffic as follows:
  - **Tomcat**: Allow HTTP (port 8080) and SSH (port 22) from appropriate sources.
  - **RabbitMQ**: Allow AMQ (port 5672) and management UI (port 15672) from the application instance.
  - **MySQL**: Allow MySQL (port 3306) from the application instance.
  - **Memcached**: Allow Memcached (port 11211) from the application instance.
- **Create Key Pair**: Generate a key pair for SSH access to EC2 instances.

### EC2 Instances
- Launch separate EC2 instances for:
  - **Database (DB)**: Install MySQL server. Copy the MySQL configuration and setup scripts from the GitHub repository (`https://github.com/VinitKarlekar/Vprofile-AWS_DevOps`) to configure the database.
  - **Application (APP)**: Install Tomcat 10. Copy the Tomcat configuration files and setup scripts from the GitHub repository to set up the application server.
  - **RabbitMQ (RMQ)**: Install RabbitMQ server. Copy the RabbitMQ configuration files from the GitHub repository to configure the message broker.
  - **Memcached (MemCache)**: Install Memcached. Copy the Memcached configuration files from the GitHub repository to set up the caching layer.
- For each instance, apply the appropriate security group and key pair for SSH access.

### Route53 Configuration
- Create Route53 records for each EC2 instance using their private IP addresses for internal communication.

### S3 Bucket and IAM Configuration
- **Create S3 Bucket**: Set up an S3 bucket named `vprofile-las-artifacts` for storing application artifacts.
- **Create IAM User**: Create an IAM user with full S3 bucket access and download the access key forseek for AWS CLI usage.
- **Create IAM Role**: Create an IAM role with full S3 access and attach it to the APP EC2 instance for key-based and role-based authentication.

### Application Configuration
- Rename the `application.properties` file in the repository to include instance names for Memcached, DB, and RabbitMQ.

### Maven and AWS CLI Setup
- Install the appropriate version of **Maven** for building the project.
- Install **AWS CLI** and configure it with the access key, secret key, region, and JSON output format.

### Deploying to Tomcat
- Copy the `vprofile.war` file from the `target` folder to the S3 bucket (`s3://vprofile-las-artifacts/`).
- Log into the APP instance using its public IP via Git Bash.
- Install `aws-cli --classic`.
- Copy the `vprofile.war` file from the S3 bucket to the `/tmp` folder.
- Stop the Tomcat server.
- Move `vprofile.war` from `/tmp` to `/var/lib/tomcat10/webapps/ROOT.war`.
- Start the Tomcat service.

### Load Balancer and Target Groups
- Create a target group for the application with protocol HTTP and port 8080.
- Enable **stickyness** in the target group attributes.
- Create an **Application Load Balancer** and attach it to the target group.

### Auto Scaling Configuration
- Create an **AMI** from the APP EC2 instance.
- Create a **Launch Template** using the AMI.
- Create an **Auto Scaling Group**:
  - Select the launch template.
  - Attach the group to the existing load balancer.
  - Enable Elastic Load Balancing health checks.
  - Set the scaling metric to average CPU utilization.
  - Configure notifications for scaling events.

## Rearchitecting with PaaS and SaaS

### Security Groups and Key Pair for Beanstalk
- Create a security group allowing all internal traffic for Elastic Beanstalk, RDS, Elastic Cache, and Amazon MQ.
- Create a key pair for SSH access to the Beanstalk EC2 instance.
- Update the backend security groups to allow traffic from the Beanstalk security group.

### RDS Setup
- Create a **Parameter Group** for MySQL Community Edition.
- Create a **Subnet Group** including all availability zones and subnets.
- Create an **RDS Instance**:
  - Use MySQL with the free tier template.
  - Select General Purpose (gp3) storage.
  - Use the existing VPC security group.
  - Save the RDS credentials and endpoint.

### Elastic Cache Setup
- Create a **Parameter Group** for Elastic Cache.
- Create a **Subnet Group** including all available subnets.
- Create a **Memcached Cluster**:
  - Use the "Design your own cache" and "Standard create" options.
  - Set the port for Memcached.
  - Use `t2.micro` node type.

### Amazon MQ Setup
- Create a **RabbitMQ Broker** in Amazon MQ:
  - Use a single-instance broker with `t3.micro`.
  - Configure for private access.
  - Save the broker endpoint, username, and password.

### MySQL Client Setup
- Launch an EC2 instance named `MySQL-Client`.
- Update `apt` and install MySQL client.
- Add an inbound rule to the RDS security group to allow MySQL traffic from the `MySQL-Client` security group.
- Log into the `MySQL-Client` instance and connect to the RDS endpoint using the saved credentials.
- Clone the repository: `https://github.com/hkhcoder/vprofile-project`.
- Switch to the `awsrefactor` branch.
- Run the SQL backup script: `mysql -h <RDS_endpoint> -u <username> -p < db_backup.sql`.
- Log in to MySQL to verify table creation.

### Elastic Beanstalk Environment
- Create an **IAM Role** for Elastic Beanstalk with the following policies:
  - `AWSElasticBeanstalkFullAccess`
  - `CustomPlatformForEC2Role`
  - `AWSElasticBeanstalkRoleSNS`
  - `AWSElasticBeanstalkWebTier`
- Create an **Elastic Beanstalk Environment**:
  - Use the Web Server environment with Tomcat 10.
  - Select the created service role and EC2 instance profile.
  - Choose the key pair for SSH access.
  - Select all available subnets and General Purpose (gp3) storage.
  - Configure auto-scaling with a minimum of 2 and a maximum of 4 `t2.micro` instances.
  - Enable **session stickiness** and enhanced health reporting.
  - Update the security group to allow all traffic.
- Update the `application.properties` file:
  - Replace `db01` with the RDS endpoint and password.
  - Update Memcached and RabbitMQ endpoints with their respective credentials.
- Run `mvn install` to build the `vprofile.war` file.
- Upload the `vprofile.war` file to the Elastic Beanstalk environment.

### CloudFront Distribution
- Create a **CloudFront Distribution**:
  - Select the Elastic Beanstalk load balancer as the origin.
  - Configure default settings and deploy.


This project demonstrates a complete AWS-based deployment and rearchitecture of the VProfile application, leveraging both IaaS and PaaS/SaaS solutions for scalability and efficiency.

## 📢 LinkedIn Update

I have shared an update about this project on LinkedIn!  
Check it out and feel free to connect or share your thoughts:

🔗 [View My LinkedIn Post](https://www.linkedin.com/feed/update/urn:li:activity:7326201791132389376/)

![SS1](screenshots/ss1.png)  
![SS2](screenshots/ss2.png)
![SS3](screenshots/ss3.png)
![SS4](screenshots/ss4.png)
![SS5](screenshots/ss5.png)
![SS6](screenshots/ss6.png)
![Webpage](screenshots/final.png)
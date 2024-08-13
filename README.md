Deploy Java Application on AWS 3-Tier Architecture
![image](https://github.com/user-attachments/assets/2f004d31-a8e9-40d9-94a7-6dfdd99d313f)

TABLE OF CONTENTS

    Goal
    Pre-Requisites
    Pre-Deployment
    VPC Deployment
    Maven (Build)
    3-Tier Architecture
    Application Deployment
    Post-Deployment
    Validation
    
![image](https://github.com/user-attachments/assets/91bb682f-c34b-4913-b361-f5d795a6e259)

Project Overview
Goal

The primary objective of this project is to deploy a scalable, highly available, and secure Java application using a 3-tier architecture. The application will be accessible to end-users via the public internet.
Pre-Requisites

    1.AWS Free Tier Account: Sign up for an Amazon Web Services (AWS) Free Tier account to use various AWS services for deployment.
    2.GitHub Account and Repository: Create a GitHub account and a repository to host your Java source code. Migrate the provided Java source code from the Java-Login-App repository to your own GitHub repository.
    3.SonarCloud Account: Create an account on SonarCloud for static code analysis and quality checks.
    4. JFrog Cloud Account: Create an account on JFrog Cloud to manage your artifacts.

Pre-Deployment Steps

    Create Global AMI (Amazon Machine Image):
        Install AWS CLI.
        Install CloudWatch Agent.
        Install AWS Systems Manager (SSM) Agent.

    Create Golden AMIs:
       - For Nginx:
            Install Nginx.
            Configure custom memory metrics for CloudWatch.
        - For Apache Tomcat:
            Install Apache Tomcat.
            Configure Tomcat as a systemd service.
            Install JDK 11.
            Configure custom memory metrics for CloudWatch.
        - For Apache Maven Build Tool:
            Install Apache Maven.
            Install Git.
            Install JDK 11.
            Update the Maven Home in the system PATH environment variable.

VPC Deployment

Deploy AWS infrastructure resources as per the architecture diagram.

    VPC (Network Setup):
        Create VPC networks:
            192.168.0.0/16 for Bastion Host.
            172.32.0.0/16 for application servers.
        1.Set up NAT Gateway in the public subnet and update the route table for the private subnet.
        2.Create a Transit Gateway and associate both VPCs for private communication.
        3.Create Internet Gateways for each VPC and update route tables for inbound/outbound internet access.

    Bastion Host:
        1.Deploy a Bastion Host in the public subnet with an Elastic IP (EIP).
        2.Create a security group allowing port 22 (SSH) from the public internet.

Maven (Build)

    1.Launch an EC2 instance using the Maven Golden AMI.
    2. Clone the GitHub repository to VSCode, update pom.xml with SonarCloud and JFrog deployment details, and add settings.xml with JFrog credentials.
    3. Update application.properties with JDBC connection string.
    4.Push code changes to a feature branch, create a pull request, and merge changes to the master branch.
    5. Clone the repository on the EC2 instance and build the source code using Maven with -s settings.xml.
    6. Integrate the Maven build with SonarCloud and generate an analysis dashboard using the default quality gate profile.

3-Tier Architecture

    Database (RDS):
       1. Deploy a Multi-AZ MySQL RDS instance in private subnets.
       2.Create a security group allowing port 3306 from application instances and Bastion Host.

    Tomcat (Backend):
       1. Create a private-facing Network Load Balancer and Target Group.
       2. Create a Launch Configuration:
            Use Tomcat Golden AMI.
            Deploy .war artifacts from JFrog to the webapps folder.
            Configure security groups to allow port 22 from Bastion Host and port 8080 from the private NLB.
       3. Set up an Auto Scaling Group.

    Nginx (Frontend):
        - Create a public-facing Network Load Balancer and Target Group.
        - Create a Launch Configuration:
            Use Nginx Golden AMI.
            Update proxy_pass rules in nginx.conf and reload the Nginx service.
            Configure security groups to allow port 22 from Bastion Host and port 80 from the public NLB.
        Set up an Auto Scaling Group.

Application Deployment

    1. The deployment of artifacts is handled by user data scripts during the launch of EC2 instances in the application tier.
    2. Log into the MySQL database from the application server using MySQL CLI and create the database and table schema to store user login data (as described in the README.md file in the GitHub repo).

Post-Deployment

    1. Configure a Cron job to push Tomcat application logs to an S3 bucket and rotate logs by deleting them from the server after upload.
    2. Set up CloudWatch alarms to send email notifications if database connections exceed 100.

Validation

    1. Ensure administrators can log into EC2 instances via the session manager and Bastion Host.
    2. Verify that end-users can access the application from a public internet browser.

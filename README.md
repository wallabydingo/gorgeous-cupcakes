# Gorgeous Cupcakes – Multi-Tier CloudFormation Deployment

## Overview
Gorgeous Cupcakes is a PHP and MySQL web application deployed using AWS CloudFormation.  
The project demonstrates Infrastructure as Code (IaC) through a complete multi-tier deployment on AWS. The environment consists of:

- A custom VPC  
- An EC2 web server running Apache and PHP  
- A MySQL RDS database  

The EC2 instance installs and configures the application automatically using UserData.  
Database credentials can be stored securely in AWS Systems Manager Parameter Store.

---

## Architecture

### **Network Tier (cf_vpc.yaml)**
- Creates a VPC  
- Provides one public subnet for the EC2 instance  
- Provides two private subnets for the RDS instance  
- Creates routing, Internet Gateway, and security groups  
- Exports subnet and security group IDs for use by other stacks  

### **Web Tier (cf_ec2.yaml)**
- Launches an Amazon Linux EC2 instance  
- Installs Apache, PHP, and MySQL client  
- Downloads the Gorgeous Cupcakes application from GitHub  
- Allows HTTP access through the web security group  
- Uses parameters such as the VPC stack name and key pair  

### **Database Tier (cf_rds.yaml)**
- Creates a MySQL 8.0 RDS instance  
- Deploys the database into private subnets  
- Uses template parameters for DB name, username, and password  
- Allows traffic only from the EC2 instance security group  

---

## Deployment Steps

### **Step 1 – Deploy the VPC**
1. Open AWS CloudFormation and create a new stack.  
2. Upload the `cf_vpc.yaml` template.  
3. Deploy using default settings.  
4. Record the stack name for later use.  

### **Step 2 – Deploy the EC2 Web Server**
1. Create a new stack using `cf_ec2.yaml`.  
2. Enter the required parameters, including `VPCStackName` and `KeyName`.  
3. Deploy and wait for **CREATE_COMPLETE**.  
4. Retrieve the EC2 public IPv4 address from the console.  
The EC2 instance automatically installs Apache, PHP, and the application code.

### **Step 3 – Deploy the RDS Database**
1. Create another stack using `cf_rds.yaml`.  
2. Enter values for the database name, username, and password.  
3. Wait until the RDS instance status is **Available**.  
4. Copy the RDS endpoint from the console for application configuration.

---

## Application Setup

### **Copy the database schema to EC2**
```bash
scp -i <your-key.pem> gorgeous_cupcakes_v1.sql ec2-user@<EC2-Public-IP>:~/
Import the database
mysql -h <RDS-endpoint> -u admin -p awsexample < gorgeous_cupcakes_v1.sql
Update application configuration
Edit the file:
/var/www/html/config.php
Update these fields:

RDS endpoint
Database username
Database password
Access the application
Open a browser and navigate to:
http://<EC2-Public-IP>/
Secrets Management
This project uses AWS Systems Manager Parameter Store to store sensitive values, such as the MySQL password.
Parameter name:

/gorgeouscupcakes/db-password
This keeps sensitive information out of templates and application code.
Updating Infrastructure
To update resources:
Modify the CloudFormation YAML files.
Use the Update stack option in the AWS console.
CloudFormation automatically applies the changes.
Deleting the Environment
Delete stacks in the following order to ensure proper dependency cleanup:
EC2 stack
RDS stack
VPC stack
Troubleshooting
Common issues include:
ImportValue errors caused by mismatched export names
RDS subnet configuration errors when subnet groups are incomplete
Application errors caused by incorrect database configuration
Repository Contents
cf_vpc.yaml – VPC and networking configuration
cf_ec2.yaml – EC2 server configuration
cf_rds.yaml – MySQL RDS configuration
README.md – Deployment instructions and documentation

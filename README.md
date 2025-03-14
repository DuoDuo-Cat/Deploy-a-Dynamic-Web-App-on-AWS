# Dynamic Website Hosting on AWS
## Project Overview

Shopwise is a dynamic web application hosted on AWS EC2 instances, leveraging a robust  architecture to ensure high availability, fault tolerance, and security. This DevOps project includes networking, load balancing, auto-scaling, and database management, making use of several AWS resources.
## Diagram

![Alt text](https://github.com/DuoDuo-Cat/Deploy-a-Dynamic-Web-App-on-AWS/blob/main/Reference%20Diagram.png)
## Architecture Overview

### Networking and Security

- Configured a **Virtual Private Cloud (VPC)** with both **public and private subnets** across two availability zones for high reliability.
- Deployed an **Internet Gateway** to facilitate connectivity between resources in the VPC and the public internet.
- Configured **Security Groups (SGs)** to act as network firewalls, with a total of five SGs:
  - **ALB SG** (Application Load Balancer Security Group)
  - **EICE SG** (EC2 Instance Connect Endpoint Security Group)
  - **Data Migrate SG** (Data Migration Security Group)
  - **Web Server SG** (Web Application Security Group)
  - **RDS SG** (Database Security Group)

### Public and Private Resource Deployment

- **Public Resources**:
  - **NAT Gateway**: Enables instances in private subnets to access the internet.
  - **Application Load Balancer (ALB)**: Distributes web traffic evenly across multiple EC2 instances.
- **Private Resources**:
  - **Web Servers (EC2 Instances)**: Configured in private subnets for enhanced security.
  - **Database Servers (RDS MySQL)**: Securely stored in private subnets.

### Application Hosting and Data Management

- Utilized **EC2 Instance Connect Endpoint (EICE)** for migrating SQL data to the RDS database using **Flyway**.
- Configured **Apache, PHP extensions, and MySQL** on EC2 instances for web hosting.
- Set up **Amazon Route 53 (R53) and SSL certificates** for secure domain name access.
- Created an **Auto Scaling Group (ASG)** to ensure website availability, scalability, and fault tolerance.
- Configured **Simple Notification Service (SNS)** for ASG activity alerts.
- Implemented an **HTTPS listener** on ALB to redirect HTTP traffic.
- Utilized **Amazon S3** to store web files for data migration and configuration.

## Installation and Setup

To configure the Shopwise app on an EC2 instance, the following script was used:

```bash
#!/bin/bash
# Update system packages
sudo yum update -y

# Install Apache Web Server
sudo yum install -y httpd
sudo systemctl enable httpd 
sudo systemctl start httpd

# Install PHP and required extensions
sudo dnf install -y \
php php-pdo php-openssl php-mbstring php-exif php-fileinfo php-xml php-ctype \
php-json php-tokenizer php-curl php-cli php-fpm php-mysqlnd php-bcmath php-gd \
php-cgi php-gettext php-intl php-zip

# Install MySQL 8
sudo wget https://dev.mysql.com/get/mysql80-community-release-el9-1.noarch.rpm 
sudo dnf install -y mysql80-community-release-el9-1.noarch.rpm
sudo rpm --import https://repo.mysql.com/RPM-GPG-KEY-mysql-2023
dnf repolist enabled | grep "mysql.*-community.*"
sudo dnf install -y mysql-community-server 
sudo systemctl start mysqld
sudo systemctl enable mysqld

# Enable Apache mod_rewrite
sudo sed -i '/<Directory "/var/www/html">/,/<\/Directory>/ s/AllowOverride None/AllowOverride All/' /etc/httpd/conf/httpd.conf

# Sync web files from S3
S3_BUCKET_NAME=ddcat-project-web-files 
sudo aws s3 sync s3://"$S3_BUCKET_NAME" /var/www/html

# Unzip and set up application
cd /var/www/html
sudo unzip shopwise.zip
sudo cp -R shopwise/. /var/www/html/
sudo rm -rf shopwise shopwise.zip

# Set permissions
sudo chmod -R 777 /var/www/html
sudo chmod -R 777 storage/

# Edit database credentials in .env file
sudo vi .env

# Restart Apache Server
sudo service httpd restart
```

## SQL Data Migration

To migrate SQL data from EC2 to RDS database using Flyway, the following script was used:

```bash
#!/bin/bash

S3_URI=s3://ddcat-sql-files/V1__shopwise.sql
RDS_ENDPOINT=dev-rds-db.cbk2oq0wamsk.us-east-1.rds.amazonaws.com
RDS_DB_NAME=applicationdb
RDS_DB_USERNAME= 
RDS_DB_PASSWORD=

# Update all packages
sudo yum update -y

# Download and extract Flyway
sudo wget -qO- https://download.red-gate.com/maven/release/com/redgate/flyway/flyway-commandline/11.3.4/flyway-commandline-11.3.4-linux-x64.tar.gz | tar -xvz && sudo ln -s `pwd`/flyway-11.3.4/flyway /usr/local/bin 

# Create the SQL directory for migrations
sudo mkdir sql

# Download the migration SQL script from AWS S3
sudo aws s3 cp "$S3_URI" sql/

# Run Flyway migration
flyway -url=jdbc:mysql://"$RDS_ENDPOINT":3306/"$RDS_DB_NAME" \
  -user="$RDS_DB_USERNAME" \
  -password="$RDS_DB_PASSWORD" \
  -locations=filesystem:sql \
  migrate
```

## Conclusion

This AWS DevOps project successfully deployed a highly available and secure dynamic web application using EC2, RDS, ALB, ASG, and other AWS services. The implementation ensures optimal performance, scalability, and security for end users accessing the Shopwise web application.


# Hosting a WordPress Website on AWS

This repository contains the resources and scripts used to deploy a WordPress website on Amazon Web Services (AWS). The project leverages various AWS services to ensure high availability, scalability, and security for the WordPress application.

## Architecture Overview

The WordPress website is hosted on EC2 instances within a highly available and secure architecture that includes:

- **Virtual Private Cloud (VPC)**: With public and private subnets across two Availability Zones (AZs) for fault tolerance and high availability.
- **Internet Gateway**: To allow communication between instances in the VPC and the internet.
- **Security Groups**: Acting as a virtual firewall to control inbound and outbound traffic.
- **Public Subnets**: Used for the NAT Gateway and Application Load Balancer, facilitating external access and load balancing.
- **Private Subnets**: For web servers to enhance security.
- **EC2 Instance Connect Endpoint**: For secure SSH access.
- **Application Load Balancer**: With a target group to distribute incoming web traffic across multiple EC2 instances.
- **Auto Scaling Group**: To automatically adjust the number of EC2 instances based on traffic, ensuring scalability and resilience.
- **Amazon RDS**: For a managed relational database service.
- **Amazon EFS**: For a scalable, elastic file storage system.
- **AWS Certificate Manager**: For managing SSL/TLS certificates.
- **AWS Simple Notification Service (SNS)**: For notifications related to the Auto Scaling Group activities.
- **Amazon Route 53**: For domain name registration and DNS management.

## Deployment Scripts

### WordPress Installation Script

This script is used for the initial setup of the WordPress application on an EC2 instance. It includes steps for installing Apache, PHP, MySQL, and mounting the Amazon EFS to the instance.

```bash
# create to root user
sudo su

# update the software packages on the ec2 instance 
sudo yum update -y

# create an html directory 
sudo mkdir -p /var/www/html

# environment variable
EFS_DNS_NAME=fs-064e9505819af10a4.efs.us-east-1.amazonaws.com

# mount the efs to the html directory 
sudo mount -t nfs4 -o nfsvers=4.1,rsize=1048576,wsize=1048576,hard,timeo=600,retrans=2,noresvport "$EFS_DNS_NAME":/ /var/www/html

# install the apache web server, enable it to start on boot, and then start the server immediately
sudo yum install -y httpd
sudo systemctl enable httpd 
sudo systemctl start httpd

# install php 8 along with several necessary extensions for wordpress to run
sudo dnf install -y \
php \
php-cli \
php-cgi \
php-curl \
php-mbstring \
php-gd \
php-mysqlnd \
php-gettext \
php-json \
php-xml \
php-fpm \
php-intl \
php-zip \
php-bcmath \
php-ctype \
php-fileinfo \
php-openssl \
php-pdo \
php-tokenizer

# install the mysql version 8 community repository
sudo wget https://dev.mysql.com/get/mysql80-community-release-el9-1.noarch.rpm 

# install the mysql server
sudo dnf install -y mysql80-community-release-el9-1.noarch.rpm 
sudo rpm --import https://repo.mysql.com/RPM-GPG-KEY-mysql-2023
sudo dnf repolist enabled | grep "mysql.*-community.*"
sudo dnf install -y mysql-community-server 

# start and enable the mysql server
sudo systemctl start mysqld
sudo systemctl enable mysqld

# set permissions
sudo usermod -a -G apache ec2-user
sudo chown -R ec2-user:apache /var/www
sudo chmod 2775 /var/www && find /var/www -type d -exec sudo chmod 2775 {} \;
sudo find /var/www -type f -exec sudo chmod 0664 {} \;
chown apache:apache -R /var/www/html 

# download wordpress files
wget https://wordpress.org/latest.tar.gz
tar -xzf latest.tar.gz
sudo cp -r wordpress/* /var/www/html/

# create the wp-config.php file
sudo cp /var/www/html/wp-config-sample.php /var/www/html/wp-config.php

# edit the wp-config.php file
sudo vi /var/www/html/wp-config.php

# restart the webserver
sudo service httpd restart
```

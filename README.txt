README.txt



Create VPC
Jeremy-twoge-vpc / IPv4 CIDR 10.0.0.0/20
Create subnets
Jeremy-twoge-public1 / 10.0.0.0/22 / us-east-1a
Jeremy-twoge-public2 / 10.0.4.0/22 / us-east-1b
Jeremy-twoge-private1 / 10.0.8.0/22 / us-east-1a
Jeremy-twoge-private2 / 10.0.12.0/22 / us-east-1b
Create internet gateway
Ig for jeremy-twoge-vpc
Attach to jeremy-twoge-vpc
Create NAT gateway	
Jeremy-twoge-natgw1 / Jeremy-twoge-private1 / public / eipalloc-oa7f0bba9ac024ef2
Jeremy-twoge-natgw2 / Jeremy-twoge-private2 / private
Create route table 
1 for public and 1 for private subnet
Public / add internet gateway destination
Add public subnets to rt
Private / add nat gateway destination
Add private subnets to rt
Create S3 Bucket
Jeremy-twoge-s3 / allow public / enable versionoing / 
Add policy to allow public access

Create EC2 Instance
Jeremy-twoge-ec2 / awslinux / t2 micro / Jeremy-charlie-East2 key / jeremy-twoge-vpc / jeremy-twoge-public1 / auto assign public IP / jeremy-twoge-sg / ssh, http, postgresql rules / 
Installation on EC2
Update
Sudo yum update
Sudo yum install git
Python3 
Sudo yum install python3
Sudo python -m ensurepip –upgrade
Pip3 install python-dotenv
Flask
Pip3 install flask
Pip3 install flask_sqlalchemy
SQLalchemy
Pip3 install sqlalchemy
postgreSQL
Sudo yum install postgresql postgresql-devel
Pip3 install psycopg2-binary
Creating PostgresSQL RDS
PostgreSQL / Free Tier / Single DB instance / jeremy-twoge-db / UN postgres  PA 11111111 / connect to jeremy-twoge-ec2 / jeremy-twoge-sg /  
CMD line
export DATABASE_URL=”postgresql://jeremyh:1111@localhost:5000/twogedb”
Vim .env
SQLALCHEMY_DATABASE_URI=postgressql://postgres:11111111@jeremy-twoge-db.cvyw6igek2bp.us-east-1.rds.amazonaws.com:5432/twogedb
SECRET_KEY=mysecretkey
DEBUG=True
Creating virtual environment
Python3 -m venv venv
Source venv/bin/activate
Cd twoge
Pip3 install -r requirements.txt
Launching Application 
Python3 app.py
Navigate to EC2 and open Public IPv4 address 
Create Amazon Application Load Balancer
Application Load Balancer
Jeremy-alb / ipv4 
Jeremy-twoge-vpc
AZ us-east-1a / jeremy-twoge-public1
AZ us-east-1b / jeremy-twoge-public2
Jeremy-twoge-sg
Add HTTP:80 and HTTP:5000 rule 
Target group / instances / jeremy-twoge-public / jeremy-twoge-vpc
Create Launch Template
Jeremy-twoge-launchtemplate
Amazon Linux 2 AMI
T2.micro
Jeremy-Charlie-East2 Key Pair
Jeremy-twoge-sg
Userdata- on startups


#!/bin/bash

# Update OS
sudo yum update -y

# Install Git and Python 3 if not already installed
sudo yum install git python3 python3-pip -y

# Clone Twoge repo to *root directory*
git clone https://github.com/codeplatoon-devops/twoge.git

# Navigate to the twoge directory
cd twoge

# Create a virtual environment
python3 -m venv venv

# Install required Python packages
venv/bin/pip install -r requirements.txt

# Set up environment variables
echo "SQLALCHEMY_DATABASE_URI=postgresql://postgres:11111111@jeremy-twoge-db.cvyw6igek2bp.us-east-1.rds.amazonaws.com:5432/twogedb" >> .env
echo "SECRET_KEY=mysecretkey" >> .env
echo "DEBUG=True" >> .env

# Run the app in the background
nohup venv/bin/python app.py > app.log 2>&1 &


nohup python app.py > app.log 2>&1 &


Create CloudWatch Metrics


Create EC2 autoscaling group
Create ALB, Launch Template, CloudWatch Metrics
1. Jeremy-twoge-asg / jeremy-twoge-launchtemplate
2 . Jeremy-twoge-vpc / us-east-1a(jeremy-twoge-public1) + us-east-1b(jeremy-twoge-public2)
3. Attach to jeremy-twoge-public load balancer / turn on Elastic Load Balancing Health Checks / 30 seconds
4. Capacity 2  / min 1 max 3
5. Add topic / jeremy-twoge-sns / jeremyhuegel@gmail.com / SNS IDL arn:aws:sns:us-east-1:536697237321:jeremy-twoge-sns:3940d3de-c6a6-41e3-8180-8a21c6f6ed81

# Twoge






# Twoge Infrastructure Setup

## VPC and Subnets

### Create VPC
- **Name**: `Jeremy-twoge-vpc`
- **IPv4 CIDR**: `10.0.0.0/20`

### Create Subnets
| Subnet Name            | CIDR Block    | Availability Zone |
|------------------------|---------------|-------------------|
| `Jeremy-twoge-public1` | `10.0.0.0/22` | `us-east-1a`      |
| `Jeremy-twoge-public2` | `10.0.4.0/22` | `us-east-1b`      |
| `Jeremy-twoge-private1`| `10.0.8.0/22` | `us-east-1a`      |
| `Jeremy-twoge-private2`| `10.0.12.0/22`| `us-east-1b`      |

## Networking Setup

### Create Internet Gateway
1. **Name**: `Ig for jeremy-twoge-vpc`
2. Attach to `jeremy-twoge-vpc`

### Create NAT Gateway
| NAT Gateway Name       | Subnet           | Type      | Elastic IP |
|------------------------|------------------|-----------|------------|
| `Jeremy-twoge-natgw1`  | `Jeremy-twoge-private1` | Public    | `eipalloc-oa7f0bba9ac024ef2` |
| `Jeremy-twoge-natgw2`  | `Jeremy-twoge-private2` | Private   | -          |

### Create Route Tables
1. **Public Route Table**:
   - Add internet gateway as destination.
   - Associate public subnets.
2. **Private Route Table**:
   - Add NAT gateway as destination.
   - Associate private subnets.

### Create S3 Bucket
- **Name**: `Jeremy-twoge-s3`
- Settings:
  - Allow public access
  - Enable versioning
  - Add bucket policy to allow public access
    
![S3 Bucket Policy](https://github.com/user-attachments/assets/419da1a7-5ffd-4a87-a389-76a160b438cd)




### Create EC2 Instance
- **Name**: `Jeremy-twoge-ec2`
- **AMI**: Amazon Linux
- **Instance Type**: `t2.micro`
- **Key Pair**: `Jeremy-charlie-East2`
- **Network**: `jeremy-twoge-vpc`, `jeremy-twoge-public1`
- **Security Group**: `jeremy-twoge-sg` (allow SSH, HTTP, PostgreSQL rules)


### Create PostgresSQL RDS  
- PostgreSQL / Free Tier / Single DB instance / jeremy-twoge-db / UN postgres  PA 11111111 / connect to jeremy-twoge-ec2 / jeremy-twoge-sg / 


## Installation on EC2
```bash
# Update
sudo yum update -y

# Install Git and Python3
sudo yum install git python3 -y

# Install Python dependencies
sudo python3 -m ensurepip --upgrade
pip3 install python-dotenv flask flask_sqlalchemy psycopg2-binary
Sudo yum install postgresql postgresql-devel
Pip3 install psycopg2-binary


export DATABASE_URL="postgresql://postgres:11111111@jeremy-twoge-db.cvyw6igek2bp.us-east-1.rds.amazonaws.com:5432/twogedb"
vim .env
#Add the following to .env
SQLALCHEMY_DATABASE_URI=postgresql://postgres:11111111@jeremy-twoge-db.cvyw6igek2bp.us-east-1.rds.amazonaws.com:5432/twogedb
SECRET_KEY=mysecretkey
DEBUG=True

###Virtual Evnironment setup
python3 -m venv venv
source venv/bin/activate
cd twoge
pip3 install -r requirements.txt

### Launching Application 
Python3 app.py
Navigate to EC2 and open Public IPv4 address

### Create Amazon Application Load Balancer
Application Load Balancer
Jeremy-alb / ipv4 
Jeremy-twoge-vpc
AZ us-east-1a / jeremy-twoge-public1
AZ us-east-1b / jeremy-twoge-public2
Jeremy-twoge-sg
Add HTTP:80 and HTTP:5000 rule 
Target group / instances / jeremy-twoge-public / jeremy-twoge-vpc

### Create Launch Template
Jeremy-twoge-launchtemplate
Amazon Linux 2 AMI
T2.micro
Jeremy-Charlie-East2 Key Pair
Jeremy-twoge-sg

Userdata- on startups
#!/bin/bash
sudo yum update -y
sudo yum install git python3 python3-pip -y
git clone https://github.com/codeplatoon-devops/twoge.git
cd twoge
python3 -m venv venv
venv/bin/pip install -r requirements.txt
echo "SQLALCHEMY_DATABASE_URI=postgresql://postgres:11111111@jeremy-twoge-db.cvyw6igek2bp.us-east-1.rds.amazonaws.com:5432/twogedb" >> .env
echo "SECRET_KEY=mysecretkey" >> .env
echo "DEBUG=True" >> .env
nohup venv/bin/python app.py > app.log 2>&1 &

```

### Create EC2 autoscaling group
Create ALB, Launch Template, CloudWatch Metrics
1. Jeremy-twoge-asg / jeremy-twoge-launchtemplate
2 . Jeremy-twoge-vpc / us-east-1a(jeremy-twoge-public1) + us-east-1b(jeremy-twoge-public2)
3. Attach to jeremy-twoge-public load balancer / turn on Elastic Load Balancing Health Checks / 30 seconds
4. Capacity 2  / min 1 max 3
5. Add topic / jeremy-twoge-sns / jeremyhuegel@gmail.com / SNS IDL arn:aws:sns:us-east-1:536697237321:jeremy-twoge-sns:3940d3de-c6a6-41e3-8180-8a21c6f6ed81


![Twoge-Diagram](https://github.com/user-attachments/assets/f6204b0a-d33b-4666-9592-cc58b7b70111)


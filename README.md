# VPC-Multi-Tier-Architecture-App-Server-Web-Server-DB-Server

Step 1: Creating the VPC

Navigated to AWS Management Console → VPC → Your VPCs.
Clicked Create VPC and provided the following details
VPC Name: My-VPC
IPv4 CIDR Block: 10.0.0.0/16 (provides 65,536 IPs).
Clicked Create VPC.

Step 2: Creating and Attaching an Internet Gateway
Went to VPC → Internet Gateways.
Clicked Create Internet Gateway, named it My-IGW, and created it.
Attached it to My-VPC by selecting it and clicking Attach to VPC.

Step 3: Creating Public and Private Subnets

Public Subnet:
Name: Public-Subnet
VPC: My-VPC
CIDR Block: 10.0.1.0/24
Availability Zone: us-east-1a

Private Subnet:
Name: Private-Subnet
VPC: My-VPC
CIDR Block: 10.0.2.0/24
Availability Zone: us-east-1a

Step 4: Creating and Configuring Route Tables

Public Route Table:
Created a new Route Table named Public-RT and associated it with My-VPC.
Attached Public-Subnet to Public-RT.
Added a route to the Internet Gateway:
Destination: 0.0.0.0/0
Target: My-IGW

Private Route Table:
Created another Route Table named Private-RT and associated it with My-VPC.
Attached Private-Subnet to Private-RT.
Updated Private-RT to route traffic via NAT Gateway.

Step 5: Creating Security Groups

Public Security Group (Public-SG) for Web Server:
Allow SSH (22) from My IP
Allow HTTP (80), HTTPS (443) from anywhere
Private Security Group (Private-SG) for App Server:
Allow port 8080 from Web Server.
Allow SSH (22) from Web Server.
Copy Web Server’s Security Group ID and paste in source.

Private Security Group for Database Server:
Allow MySQL (3306) only from App Server’s Security Group.
Copy App Server’s Security Group ID and paste in source.

Step 6: Launching EC2 Instances

Public EC2 for Web Server:
VPC: My-VPC
Subnet: Public-Subnet
Auto-assign Public IP: Enabled
Security Group: Public-SG
Key Pair: Used existing my-key.pem
User Data (Web Server):

#!/bin/bash
sudo yum update -y
sudo yum install httpd -y
sudo systemctl start httpd
sudo systemctl enable httpd
echo "Welcome to Web Server!" > /var/www/html/index.html

Open a browser and check with IP: http://3.222.207.119

Private EC2 for App Server:
VPC: My-VPC
Subnet: Private-Subnet
Auto-assign Public IP: Disabled
Security Group: Private-SG
User Data (App Server):
#!/bin/bash
sudo yum update -y
sudo yum install python3 -y
mkdir -p /home/ec2-user/python-server
cd /home/ec2-user/python-server
echo "<h1>Hello, Python HTTP Server is running on EC2!</h1>" > index.html
nohup python3 -m http.server 80 &

Configure RDS (Database Server)
Go to RDS → Create Database.
Choose MySQL
Set DB Instance Identifier: MultiTierDB.
Username: admin.
Password: your-password.
Choose My-VPC.
Select Private-DB-Subnet.
Security Group: Database-SG.
Click Create.

Step 7: Creating and Configuring NAT Gateway

Created a NAT Gateway in Public-Subnet and attached a new Elastic IP.

Updated Private-RT to allow internet access via NAT Gateway:

Destination: 0.0.0.0/0

Target: My-NAT-Gateway

Step 8: Testing Internet Connectivity

Checking Public EC2 Connectivity

curl ifconfig.me
ping -c 4 google.com

Check in browser with Public EC2 IP.

Connecting to Private App Server via Public EC2

Copy my-key.pem from local machine to Public EC2.

Comand to Verify the .pem file is placed in Public EC2:

ls -l

Set proper permissions:

chmod 400 my-key.pem

Connect to Private EC2:

ssh -i my-key.pem ec2-user@<Private-EC2-Private-IP>

Run commands to check internet access:

curl ifconfig.me
ping -c 4 google.com

The output should match the NAT Gateway’s Elastic IP, confirming internet access.

Step 9: Installing and Connecting MySQL in App Server

Install MySQL Client

sudo yum update -y
sudo yum install mysql -y

Verify MySQL Installation

mysql --version

Connect to RDS

mysql -h <RDS-ENDPOINT> -u <DB-USERNAME> -p

Example:

mysql -h database-1.ckhqewgcs9xu.us-east-1.rds.amazonaws.com -u admin -p

To check databases:

SHOW DATABASES;

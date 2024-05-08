
# $${\color{red} \textbf{Project} \textbf{3-tier} \ \textbf{Student} \ \textbf{App}}$$
 

### $\color{blue}{GITHUB \ REPOSITORY \ LINK:}$ 


### $\color{green}{Prerequisite:}$
- VPC 
- Subnets
- Route Table 
- Nat Gateway
- Internet Gateway
- RDS 

### $\color{blue}{Create \ VPC }$
Name: VPC-3-tier
CIDR: 192.168.0.0/16

### $\color{blue}{Create \ Subnets}$
1.Subnet-1
Name: Public-Subnet-Nginx
CIDR: 192.168.1.0/24

2.Subnet-2
Name: Private-Subnet-Tomcat
CIDR: 192.168.2.0/24

3.Subnet-3
Name:Private-Subnet-Database
CIDR: 192.168.3.0/24

### $\color{blue}{Create \ Internet \ Gateway }$
Name: IGW-3-tier
-attach igw to vpc

### $\color{blue}{Create \ Nat \ Gateway }$
Name: NAT-3-tier
-create in public subnet

### $\color{blue}{Create \ Route \ Table }$
1. RT-Public-Subnet
   - add public subnet
   - add igw
2. RT-Private-Subnet
   - add private subnets
   - add nat

### $\color{blue}{Create \ EC2 \ Instances }$
1. Nginx-Server-Public  ->create in public subnet ->allow port = 80,22
2. Tomcat-Server-Private ->create in private subnet ->allow port = 8080,22
3. Database-Server-Private ->create in private subnet ->allow port = 3306,22


### $\color{blue}{Create \ Database \ In \ RDS  }$
- Go To RDS
- Created Database
- Standard create 
- Free tier 
- DB name – database-1 
- Username – admin 
- Password – Passwd123$
- VPC –  VPC-3-tier
- Connect to Instance -> choose database instance
- Public access – no 
- A.Z. – no preference 
- Create database 
- Edit security group -> Add 3306 port


### $\color{blue}{ Connect \ To \ Nginx-Server-Public }$
- connect to instance
- change hostname
  
  ````
   sudo -i
  ````
  ````
  yum install nginx -y
  ````
- create file with name 3-tier-key.pem
  ```` 
  vim 3-tier-key.pem
  ````
- copy private key and paste it here
 
### $\color{blue}{Now \ SSH \ into \ Database \ Server }$

````
sudo -i
````
````
yum install mariadb105-server -y
````
````
systemctl start mariadb
````
````
systemctl enable mariadb
````
### $\color{blue}{Log \ in \ into \ database}$

````
mysql -h rds-endpoint   -u admin -pPasswd123$
````
Note: replace rds-endpoint with actual endpoint value
Important Commands:
````
show databases;
````
````
create database  studentapp;
````
````
use studentapp;
````

### $\color{blue}{Run \ this \ query \ to \ create \ table:}$
````
 CREATE TABLE if not exists students(student_id INT NOT NULL AUTO_INCREMENT,  
	student_name VARCHAR(100) NOT NULL,  
	student_addr VARCHAR(100) NOT NULL,   
	student_age VARCHAR(3) NOT NULL,      
	student_qual VARCHAR(20) NOT NULL,     
	student_percent VARCHAR(10) NOT NULL,   
	student_year_passed VARCHAR(10) NOT NULL,  
	PRIMARY KEY (student_id)  
);
````
````
show tables;
````
Logout from database:
````
exit
````
- back to nginx-server-public
  
### $\color{blue}{Now \ SSH \ into \ Tomcat \ Server }$
- ssh -i 3-tier-key.pem  ec2-user@ip-of-tomcat-vm
````
sudo -i
````
````
yum install java -y
````
````
mkdir /opt/tomcat
````
````
curl -O https://dlcdn.apache.org/tomcat/tomcat-8/v8.5.99/bin/apache-tomcat-8.5.99.tar.gz
````
````
tar -xvf apache-tomcat-8.5.99.tar.gz -C /opt/tomcat
````
````
cd /opt/tomcat/apache-tomcat-8.5.99/webapps
````
````
curl -O https://s3-us-west-2.amazonaws.com/studentapi-cit/student.war
````
````
cd ../lib
````
````
curl -O https://s3-us-west-2.amazonaws.com/studentapi-cit/mysql-connector.jar
````
````
yum install elinks -y
````
````
elinks
````
- using elinks package you can see output in cli use #elinks -> paste tomcat-ip:8080/student

````
exit
````
- back to nginx-server
````
vim /etc/nginx/nginx.conf
````
- :set nu
(enter below data in line 47 in between error and location)
````
location / {
proxy_pass http://private-IP-tomcat:8080/student/;
}
````
- :wq  ->save file

````
systemctl restart nginx
````

- $\color{red}{Go \ To \ Browser \ Hit \ Public-IP \ Nginx}$


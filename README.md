# $${\color{red} Project \ -3-tier \ tudent \ App \ }$$ 

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
Allow Ports security group: 
22 = SSH 
8080 = Tomcat 
3306 = Mysql / Mariadb

![instance](https://github.com/abhipraydhoble/Project-Student-App/assets/122669982/d7851745-1bfe-4f92-b7bb-18555f2dfd45)

$\color{green}{Connect \ to \ instance:}$

![connect](https://github.com/abhipraydhoble/Project-Student-App/assets/122669982/727778ca-e9ee-43c9-ab85-ff055f94d4a2)

![cli](https://github.com/abhipraydhoble/Project-Student-App/assets/122669982/0e6244e1-489c-42c1-ae89-27c8b7c37792)

- $\color{lightblue}{install \ java }$
````
yum install java-1.8* -y 
````
- $\color{lightblue}{Install \ Tomcat }$
Search tomcat 8 download  on browser

![tomcat](https://github.com/abhipraydhoble/Project-Student-App/assets/122669982/8e622609-b7df-4f26-b8e3-e787e5e16c95)

 ````
wget  https://dlcdn.apache.org/tomcat/tomcat-8/v8.5.99/bin/apache-tomcat-8.5.99.zip

unzip apache-tomcat-8.5.99.zip 
cd  apache-tomcat-8.5.99.zip 
cd bin 
[catalina.sh  -->this file is neccessary to start tomcat] 
chmod +x catalina.sh     [ give execute permission to file] 
````
### $\color{blue}{Start \ and \ Stop \ Tomcat \ using \ this \ command:}$
````
sh catalina.sh start   [ tomcat started ]
sh catalina.sh stop 
````
go to browser and public ip:8080

### $\color{blue}{SETUP \ STUDENT \ APPLICATION}$
````
yum install git -y 
git clone https://github.com/abhipraydhoble/Student-App-Project.git 
cd Student-App-Project 
````
$\color{lightblue}{Copy \ file \ from \ git \ directory \ to \ Tomcat}$

````
cp Student-App-Project/student.war apache-tomcat-8.5.93/webapps/ 
cp Student-App-Project/mysql-connector.jar apache-tomcat-8.5.93/lib/ 
````
### $\color{blue}{SETUP \ DATABASE \ IN \ RDS:}$
Go to RDS
download mariadb-server using  below command

````
dnf install mariadb105-server
systemctl start mariadb    
systemctl enable mariadb  
systemctl status mariadb
````

### $\color{blue}{Log \ in \ into \ database}$

````
mysql -h rds-endpoint   -u admin -pPasswd123$
````
Note: replace rds-endpoint with actual endpoint value
Important Commands:
````
show databases;
create database  databasename;
use databasename;
show tables;
describe tablename;

  ````
<Mariadb> Create database with name studentapp  
<Mariadb> Create database studentapp;    
<Mariadb> use studentapp;   --> Switch to newly created database   

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
Logout from database:
<Mariadb> exit

 ### $\color{blue}{ MODIFY \ context.xml:}$

```
cd apache-tomcat-8.5.93/conf
vim context.xml
````
add below line [connection string] at line 21
````
 <Resource name="jdbc/TestDB" auth="Container" type="javax.sql.DataSource"
               maxTotal="100" maxIdle="30" maxWaitMillis="10000"
               username="USERNAME" password="PASSWORD" driverClassName="com.mysql.jdbc.Driver"
               url="jdbc:mysql://DB-ENDPOINT:3306/DATABASE"/>

````
* Change  
1.Username  
2.Password   
3.DB-ENDPOINT  
4.DATABASE Name 

$\color{blue}{Start \ tomcat}$
````
cd apache-tomcat-8.5.93/bin
./catalina.sh start or  sh catalina.sh start
````

- $\color{red}{google hit}$
IP:8080/student

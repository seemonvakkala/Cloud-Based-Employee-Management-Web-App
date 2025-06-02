Git Hub : https://github.com/seemonvakkala/Cloud-Based-Employee-Management-Web-App.git   (kindly fork other repo from my GitHub account if require)

================= Project =====================
Select  Ohio region

Step 1: Create VPC   ( project-vpc )  - 10.20.0.0/16
Step 2: Create two subnets

subnet1 - 10.20.1.0/24 - public-1 (az-us-east-2a)
subnet2 - 10.20.2.0/24 - private-1 (az-us-east-2a)
Subnet3 - 10.20.3.0/24 - public-2 (az-us-east-2b) (2 public subnet because of load balancer is using)
Subnet4 - 10.20.4.0/24 - private-2 (az-us-east-2b) 

Step 3: create IGW, NAT-gateway, RT

Step 4: Create Internet Gateway  attach to VPC  -- project-IGW
Step 5: Create NAT-gateway  --  project-NAT -- subnet (public-1) -- Allocate elastic ip. Create

Step 6: Create Route table -- public-rt -- project-vpc
Step 7: Create Route table -- private-rt -- project-vpc

Step 8: Select public-rt -- edit route -- add route 0.0.0.0/0. -- target (created - IGW) 
        Subnet associations -- edit -- select public1 & public2

Step 9: Select private-rt -- edit route -- add route 0.0.0.0/0. -- target (created - nat) 
        Subnet associations -- edit -- select private-1 & private-2

++++++++++++

Security Group

Step 10: create security group -- project-sg -- project-vpc -- inbound rule - all traffic - anyway

Step 11: check the Network ACLs -- inbound rule (100 allow) & outbound rule (100 allow) + subnet association (all subnet available).

===================
Kindly select ubuntu AMI for both ec2

Step 12: Create Bastion Host (public-1 subnet) -- project-vpc -- auto-assign IP (enable) -- projects. (SG=SSH & all traffic allow)

Step 13: Create Application-server (private-1 subnet) -- project-vpc -- auto-assign IP (disable) -- projects. ((SG=SSH & all traffic allow))

+++++++++++++++++++++

Step 14: login to Bastion-host server & run following commands

# sudo apt-get update -y
Copy key pair form your PC to Bastion-host PC because we are using Application server for security level.

Terminal = scp -i projcet-tokyo.pem projcet-tokyo.pem ubuntu@3.112.194.3(private ip application server):/home/ubuntu/ 

#chmod 400 keypair-name
#ssh -I keypair-name ubuntu@private ip(application server).

..............................
Step 15: Login in application-server

#sudo apt-get update -y 
Git hub link to clone the application code : https://github.com/seemonvakkala/Cloud-Based-Employee-Management-Web-App.git
#sudo apt-get install git (if required)
# git init 
#git clone "https://github.com/seemonvakkala/Cloud-Based-Employee-Management-Web-App.git"
#ls
#cd aws-project-1.git (check files)
#sudo apt-get install apache2 -y
#sudo mv aws-project-1.git/ /var/www/html/
#cd /var/www/html/ (cross check all files & Directory should be available)

#sudo apt-get install mysql-client
For Sql-client
#sudo apt-get install mysql-client

For python and related frameworks

#sudo apt-get install python3-pip
#sudo apt-get install python3-flask
#sudo apt-get install python3-pymysql
#sudo apt-get install python3-boto3


=============================

Step 16: Create s3 bucket -- addemp--1 (allow public access)

Step 17: Create RDS instance (to store text data) 
- Before creating RDS instance - in RDS dashboard go to "subnet group" and create DB subnet group.
- subnet group Name : project - custom vpc - AZ-2a & AZ 2b
- private subnets : 10.20.4.0/24 & 10.20.2.0/24)

Now Create RDS - Standard -- mysql -- free tier -- admin -- admin123 -- custom VPC -- subnet group "project" -- don't want public access -
SG - project-sG -- (AZ us-east-2a) 
Additional configuration : database-name (employee) -- disable auto backup -- no monitoring -- no upgrades.

===============================

Step 18: create DynamoDB
Table name : employee_image_table
Primary key : empid  -- number

==============================

Step 19: login to application-server (/var/www/html) and add some changes in aws-project-1 directory

#vim config.py
Customhouse = "RDS endpoint name"
Customiser = "admin"
Custom pass = "admin123"
Custom db = "employee"
Custom bucket = "addemp--1"
Custom region = "us-east-2"
Customisable = "employee_image_table"

Step 20: #vim EmpApp.py

Save image file metadata in DynamoDB (after this line add some changes)
Region_name='us-east-2' (find region and change with current region)
Table name : 'employee_image_table'

Save & exit

===================================

Step 21: login to RDS mysql server to create table use following command

# mysql -h database-1.ctwgifrxirxf.ap-northeast-1.rds.amazonaws.com -u admin -p

Syntax : mysql -h "RDS end point name" -u -p

Asking for password and connect with MySql >
#Show databases;
We can see "employee" database is created.

#use employee;
Database changed

#CREATE TABLE employee (emp_id VARCHAR(20), first_name VARCHAR(20), last_name VARCHAR(20), primary_skills VARCHAR(20), location VARCHAR(20));

Query ok, 0 row affected

#show tables;
employee

#describe employee;

Kindly check all table field are available or not

#exit (from mysql) Bye

==================================
#sudo systemctl stop apache2 (need to stop apache2 because python & apache2 both running on port 80)
#sudo python3 EmpApp.py (to run application)

===================================
Application is running on private instance so need to add load balancer for accessing application

Step 22: create target group (select application server) Load Balancer (Application load balancer) should be in public subnet (AZ 2a & 2B) & Target load balancer

Copy LB DNS record and paste on browser you will able to see web application.

Now you can insert data in web application but "update database" not working. So for that need to attach IAM role

=======================================

Step 23: create IAM role and attached with application-server

IAM -- roles -- create role -- aWs services -- ec2 -- permission -- administrator access & EC2 full access, RDS access  -- role name (project-role) -- create role

Step 24: select Application server -- Action -- security -- modify IAM role -- role name(project-role) -- apply

Now if you click on update database button it'll work.

While click on go back button it will not work showing connection refuse.

==============================================

Step 24: login to application server and run script/application
# sudo python EmpApp.py (Do not stop this command, Create duplicate session)

Step 25: go to /var/www/html/aws-project-1/templates/

#sudo vim AddEmpOutput.html (and go to "action" option and change ip address with Load Balancer endpoint with "" commas).
#sudo vim GetEmpOutput.html (go to "button" line and after "formaction" just delete http://ipaddress)

Now in portal if back button press it will work.

Now database will update in portal/Applicatin.
Open S3 & Dynamodb so some data will available which is upload by user.

=================================================

Step 26: go SNS service & create topic.

SNS Dashboard : create topic --- standard --- topic name (event) --- rest thing as default and create topic.

Now create subscription : topic name select (event) --- protocol (email) -- email id (any).

Logan to your email and click on email link and confirm subscription.

Go to snstopic --- open(even)---edit---access policy (just delete last two lines "stringEquals - was:source....." And add following lines.

"ArnLike": {
  "aws:SourceArn": "arn:aws:s3:*:*:<bucket name>"
}

Save

Select created s3 bucket and go to properties --- Event Notification --- create event notification --- event name (any) --- select "all object create events" --- select SNS topic name (event) and save

So after this settings you will get notification if some 1 is adding any detail in portal.

===============================================





========================== optional =====================
If you have domain ready use following option otherwise no problem
-----------------------------------------------------------

Step 27: Route 53

As we discuss kindly create free domain with "freenom" website and copy the domain name and create new hosted zone in route 53.

Create hosted zone : domain name (your created domain) ---- public hosted zone --- create.

Change 4 "NS" record with freedom "name server" record.

Click on create record 
Record name : www ---- record type : A record 
Route traffic : select created load balancer
Enable "Alias" option.
Routing policy : simple routing

Create.

After some minutes kindly type your freenom domain name in browser it will work.

Done with small project

================================================================================================

Important links & commands 
    
---------------------------------------------------------
"ArnLike": {
  "aws:SourceArn": "arn:aws:s3:*:*:<bucket name>"
}
-------------------------------------------------------------------------

Git Hub url : https://github.com/zubair3337/aws-project-1.git  

App server commands


    1  pwd
    2  apt-get update -y
    3  ping google.com
    4  clear
    5  apt-get update -y
    6  apt-get insatll git -y
    7  apt-get install git -y
    8  git init
    9  git clone https://github.com/zubair3337/aws-project-1.git
   10  ls
   11  cd aws-project-1/
   12  ls
   13  clear
   14  apt-get install apache2 -y
   15  ls
   16  clear
   17  ls
   18  cd ..
   19  mv aws-project-1/ /var/www/html/
   20  ls
   21  ls /var/www/html/
   22  apt-get install mysql-client
   23  apt-get install python3-pip
   24  apt-get install python3-flask
   25  apt-get install python3-pymysql
   26  apt-get install python3-boto3
   27  clear
   28  ls
   29  cd /var/www/html/
   30  ls
   31  cd aws-project-1/
   32  ls
   33  vim config.py 
   34  vim EmpApp.py 
   35  mysql -h database-1.clgcgicuaq5n.us-east-2.rds.amazonaws.com -u admin -p
   36  clear
   37  systemctl stop apache2
   38  python EmpApp.py 
   39  python3 EmpApp.py 
   40  history

------------------------------------------------------------------------------------------------------------------------------------------------------
Note : don't stop python3 EmpApp.py (if its stop website will stop working, to perform other task open new terminal windows and complete extra task)

======================= End ===================

Delete 
- RDS server
- both EC2
- Load Balancer
- Target Group
- NAT gateway and important release elastic IP (chargeable)
- VPC and component
- S3 buket
- dynamoDB table
- SNS
- Route 53


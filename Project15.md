# AWS CLOUD SOLUTION FOR 2 COMPANY WEBSITES USING A REVERSE PROXY TECHNOLOGY
## INTRODUCTION

This project demostrates how a secure infrastructure inside AWS VPC (Virtual Private Cloud) network is built for a particular company, who uses WordPress CMS for its main business website, and a Tooling Website for their DevOps team. As part of the company’s desire for improved security and performance, a decision has been made to use a reverse proxy technology from NGINX to achieve this. The infrastructure will look like following diagram:

![](./Images/images15/proj%20arch.PNG)

### Starting Off my AWS Cloud Project

There are few requirements that must be met before i begin:

1. Create an AWS Master account. (Also known as **Root Account**)
2. Within the **Root account**, create a sub-account and name it **DevOps**. (I will need another email address to complete this)
3. Within the Root account, create an AWS **Organization Unit** (OU). Name it **Dev**. (We will launch Dev resources in there)
4. Move the **DevOps** account into the **Dev OU**.
5. Login to the newly created AWS account using the new email address.
6. Create a **free domain** name for my fictitious company at **Freenom** domain registrar [here](https://www.freenom.com/en/index.html?lang=en).

7. Create a **hosted zone** in AWS, and map it to my free domain from Freenom.

### **STEP 1: Setting Up a Sub-account And Creating A Hosted Zone**
---
- Creating a sub-account **'DevOps'** from my AWS master account in the AWS Organisation Unit console

  ![](./Images/images15/create%20sub%20acct.PNG)

- Creating an AWS **Organization Unit** (OU) named **'Dev'** within the **Root account**(I will launched Dev resources  in there) 

  ![](./Images/images15/OU%20dev.PNG)

- Moving the **DevOps account** into the **Dev OU**

  ![](./Images/images15/move%20acct%20to%20devOU.PNG)

  ![](./Images/images15/org%20structure.PNG)

- Logging in to the newly created AWS account with the new email

### **STEP 2: Setting Up a Virtual Private Network (VPC) and Security Group**
---
I'll make reference to the **architectural diagram** and ensure that my configuration is aligned with it.

1. Create a VPC
2. Create subnets as shown in the architecture
3. Create a route table and associate it with public subnets
4. Create a route table and associate it with private subnets
5. Create an Internet Gateway
6. Edit a route in public route table, and associate it with the Internet Gateway. (This is what allows a public subnet to be accessible from the Internet)
7. Create 3 Elastic IPs
8. Create a Nat Gateway and assign one of the Elastic IPs (*The other 2 will be used by Bastion hosts)
9. Create a Security Group for: 
   - Nginx Servers
   - Bastion Servers
   - Application Load Balancer
   - Webservers
   - Data Layer     

- Creating a VPC from the VPC console and also edit DNS Hostname and toggle to Enable

  ![](./Images/images15/create%20vpc.PNG)

  ![](./Images/images15/dns%20hostname%20disabled.PNG)

  ![](./Images/images15/dns%20hostname%20enable.PNG)

- Creating subnets for the public and private resources as reference in my architectural diagram. Also checking IP info link: [ipinfo.io/ips](https://ipinfo.io/ips)

  - Public-subnet-1 in my AZ-A(**eu-west-2a**) ipv4 CIDR block **(10.0.1.0/24)**

    ![](./Images/images15/pub%20sub%201.PNG)

  - Public-subnet-2 in my AZ-B (**eu-west-2b**) ipv4 CIDR block **(10.0.3.0/24)**

    ![](./Images/images15/pub%20sub%202.PNG)

  - Private-subnet-1(webserver) in my AZ-A (**eu-west-2a**) ipv4 CIDR block **(10.0.2.0/24)**

    ![](./Images/images15/priv%20sub%201.PNG)

  - Private-subnet-2(webserver) in my AZ-B (**eu-west-2b**) ipv4 CIDR block **(10.0.4.0/24)**

    ![](./Images/images15/priv%20sub%202.PNG)

  - Private-subnet-3(data layer) in my AZ-A (**eu-west-2a**) ipv4 CIDR block **(10.0.5.0/24)**  

    ![](./Images/images15/priv%20sub%203.PNG)

  - Private-subnet-4(data layer) in my AZ-B (**eu-west-2b**) ipv4 CIDR block **(10.0.6.0/24)**   

    ![](./Images/images15/priv%20sub%204.PNG)

  - All Public and Private Subnet in my two AZ has been created  

    ![](./Images/images15/all%20subnet.PNG)

- Creating a **route table** and associating it with **public subnets**  

   ![](./Images/images15/pub%20rtb.PNG)

   ![](./Images/images15/pub%20sub%20ass.PNG)

- Creating another **route table** and associating it with **private subnets** 

   ![](./Images/images15/prv%20rtb.PNG)

   ![](./Images/images15/prv%20sub%20ass.PNG)

- Creating an Internet Gateway and attached it to my VPC  

  ![](./Images/images15/igw.PNG)

  ![](./Images/images15/igw%20to%20vpc.PNG)

- Editing a route in **public route table**, and associating it with the **Internet Gateway**. (This is what allows a public subnet to be accessible from the Internet). 

  Routes tables > public rtb > Action > Edit routes > add route

  ![](./Images/images15/edit%20route%20table.PNG)

- Creating Elastic IPs

  ![](./Images/images15/Elastic%20IP.PNG)

  ![](./Images/images15/Elastic%20IP%202.PNG)

- Creating a **Nat Gateway** and assigning the Elastic IPs to it

  ![](./Images/images15/create%20natgateway.PNG)

- Edit private route table to talk to the natgateway from anywhere

  ![](./Images/images15/edit%20prv%20rtb.PNG)

- Setting up Security Group for **External Application load balancer** in a way that it will allow https traffic from any IP address since inbound traffic will be coming from the internet to my Ext-ALB.  

  ![](./Images/images15/sec%20grp%20for%20ALB.PNG)

- Setting up Security Group for **bastion server** in a way that it will only allow access from the workstations that need to SSH into the bastion servers.

  ![](./Images/images15/sec%20grp%20for%20bastion.PNG)

- Setting up Security Group for **Nginx servers** in a way that it will allow https traffic only from the **Application Load balancer** and allow **bastion servers** to ssh into it. so i will select the ALB and Bastion sec grp respectively as the source.

  ![](./Images/images15/sec%20grp%20for%20nginx.PNG)

- Setting up Security Group for **internal Application Load balancer** in a way that it will allow traffic only from **Nginx servers** so i will select the nginx sec grp as the source

  ![](./Images/images15/sec%20grp%20for%20int%20alb.PNG)

- Setting up Security Group for **Webservers** in a way that it will allow https traffic only from the **internal Application Load balancer** and ssh from bastion server so i will select the internal ALB and Bastion sec grp respectively as the source  

  ![](./Images/images15/sec%20grp%20for%20webserver.PNG)

- Setting Security Group for the **Data Layer** subnet in a way that it will allow TCP port 3306 traffic only from the **webservers.** and **bastion server** so i will select the bastion sec grp and webserver sec grp respectively as the source 

  ![](./Images/images15/sec%20grp%20for%20datalayer.PNG)

### **STEP 3: TLS Certificates From Amazon Certificate Manager (ACM)**
---
I will need **TLS certificates** to handle secured connectivity to my Application Load Balancers (ALB).
 
To create a TLS certificate, i will need to purchased and host a domain name.

- I already  got a free domain **"krisqriz.ga"** from [freenom](https://www.freenom.com/en/index.html?lang=en)

  ![](./Images/images15/free%20domain.PNG)

- Creating a **hosted zone** in the **Route 53** console and mapping it to the **domain name** acquired from **freenom**

  ![](./Images/images15/hosted%20zone%201.PNG)

  ![](./Images/images15/hosted%20zone%202.PNG)

- Navigating to **AWS ACM** to request public certificate.

  ![](./Images/images15/AWS%20cert%20man.PNG)

- Requesting a public wildcard certificate for the **domain name** i registered in **Freenom.** For wildcard domain use **(*.krisqriz.ga)**

  ![](./Images/images15/aws%20cert.PNG)

- Using **DNS** to validate the **domain name** click **certificate ID** > click **create records in Route 53** > click **create record**

  ![](./Images/images15/validate%20cert%201.PNG)

  ![](./Images/images15/validate%20cert%202.PNG)

  ![](./Images/images15/validate%20cert%203.PNG)

  ![](./Images/images15/validate%20cert%204.PNG)

  ![](./Images/images15/validate%20cert%205.PNG)

- Tag the resource  

### **STEP 4: Setting Up EFS Storage for the webservers**
---
- Create an **EFS** filesystem

  ![](./Images/images15/create%20efs%201.PNG)

  ![](./Images/images15/create%20efs%202.PNG)

- Create an **EFS mount target** per **AZ** 1&2 in my VPC **(eu-west 2a and eu-west 2b)** respectively, associate it with both **Private subnets 1** and **private subnet 2** dedicated for **webserver**

- Associate the **Security groups** created earlier for data layer.

  ![](./Images/images15/create%20efs%203.PNG)

  ![](./Images/images15/create%20efs%204.PNG)

- Create two EFS access point 1 for **wordpress** and the other for **tooling**. (Give it a name wordpress & tooling) respectively, POSIX user ID and group ID will be root user ID **"0"** and permission is **"0755"**

  ![](./Images/images15/AP%20wordpress.PNG)

  ![](./Images/images15/2%20access%20point.PNG)

### **STEP 5: Setting Up A Relational Database System**  
---
**Pre-requisite:** Create a **KMS** key from Key Management Service (KMS) to be used to encrypt the database instance.

- **Creating Key Management Service(KMS)**

  ![](./Images/images15/kms%201.PNG)

  ![](./Images/images15/kms%202.PNG)

  ![](./Images/images15/kms%203.PNG)

  ![](./Images/images15/kms%204.PNG)

  ![](./Images/images15/kms%205.PNG)  

- Creating a subnet group and add 2 private subnets (3&4 data Layer)

  ![](./Images/images15/subnet%20group.PNG)

- Create a mysql database

  using the free tier templates. However, with this i wont be able to use the encryption key

   ![](./Images/images15/rds%201.PNG)

   ![](./Images/images15/rds%202.PNG)

   ![](./Images/images15/rds%203.PNG)

### **STEP 6 Creating An AMI Out Of The EC2 Instance For Nginx And Bastion server**
---
Launched **3 EC2 Instance** based on Red Hat of the T2 micro family for **Nginx, bastion** and the one for the two **webservers**

![](./Images/images15/launch%203%20instance.PNG)

![](./Images/images15/3%20instance.PNG)

**For The Bastion Server**
---

After connecting to it through ssh on the terminal, the following commands are run to install some necessary softwares:
  
  `sudo yum install -y https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm`

  ![](./Images/images15/bastion%20inst%201.PNG)

  `sudo yum install -y dnf-utils http://rpms.remirepo.net/enterprise/remi-release-8.rpm`

  ![](./Images/images15/bastion%20inst%202.PNG)

  `sudo yum install wget vim python3 telnet htop git mysql net-tools chrony -y`

  ![](./Images/images15/bastion%20inst%203.PNG)
  ```
  systemctl start chronyd 
  
  systemctl enable chronyd
  ```
  ![](./Images/images15/bastion%20inst%204.PNG)

**For Nginx Server**
---
After connecting to EC2 Instance for the Nginx through ssh on the terminal, the following commands are run to install some necessary softwares:

`sudo yum install -y https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm`

![](./Images/images15/nginx%20ins%201.PNG)

`sudo yum install -y dnf-utils http://rpms.remirepo.net/enterprise/remi-release-8.rpm`

![](./Images/images15/nginx%20ins%202.PNG)

`sudo yum install wget vim python3 telnet htop git mysql net-tools chrony -y`
```
systemctl start chronyd 
  
systemctl enable chronyd
```
**Configure Selinux policies**
```
setsebool -P httpd_can_network_connect=1
setsebool -P httpd_can_network_connect_db=1
setsebool -P httpd_execmem=1
setsebool -P httpd_use_nfs 1
```
![](./Images/images15/nginx%20ins%203.PNG)

**Install amazon efs utils for mounting the target on the Elastic file system**
```
git clone https://github.com/aws/efs-utils

cd efs-utils

yum install -y make

yum install -y rpm-build

make rpm 

yum install -y  ./build/amazon-efs-utils*rpm
```
**setting up self-signed certificate for the nginx instance**
```
sudo mkdir /etc/ssl/private

sudo chmod 700 /etc/ssl/private

openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /etc/ssl/private/ACS.key -out /etc/ssl/certs/ACS.crt

sudo openssl dhparam -out /etc/ssl/certs/dhparam.pem 2048
```
![](./Images/images15/nginx%20ins%204.PNG)

To confirm my cert installation is successful and present in my server: `ls -l /etc/ssl/certs/`

`ls -l /etc/ssl/private/`

![](./Images/images15/nginx%20cert.PNG)

### **STEP 4: Creating An AMI Out Of The EC2 Instance For The Tooling and Wordpress Webservers**

After connecting to the EC2 instance for the tooling and wordpress site through SSH on the terminal, the following commands are run to install some necessary softwares:
```
yum install -y https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm

yum install -y dnf-utils http://rpms.remirepo.net/enterprise/remi-release-8.rpm

yum install wget vim python3 telnet htop git mysql net-tools chrony -y

systemctl start chronyd

systemctl enable chronyd
```
![](./Images/images15/webserver%20inst%201.PNG)

![](./Images/images15/webserver%20inst%202.PNG)

**Configure Selinux policies**
```
setsebool -P httpd_can_network_connect=1
setsebool -P httpd_can_network_connect_db=1
setsebool -P httpd_execmem=1
setsebool -P httpd_use_nfs 1
```
![](./Images/images15/webserver%20inst%203.PNG)

**Install amazon efs utils for mounting the target on the Elastic file system**
```
git clone https://github.com/aws/efs-utils

cd efs-utils

yum install -y make

yum install -y rpm-build

make rpm 

yum install -y  ./build/amazon-efs-utils*rpm
```
**setting up self-signed certificate for the apache webserver instance**
```
yum install -y mod_ssl

openssl req -newkey rsa:2048 -nodes -keyout /etc/pki/tls/private/ACS.key -x509 -days 365 -out /etc/pki/tls/certs/ACS.crt

# using vi editor to edit the SSL certificate file path from localhost.crt and localhost.key to ACS.crt and ACS.key respectively

vi /etc/httpd/conf.d/ssl.conf
```
![](./Images/images15/webserver%20inst%204.PNG)

![](./Images/images15/webserver%20inst%205.PNG)

![](./Images/images15/webserver%20inst%206.PNG)

**Creating an AMI out of the webserver EC2 Instance**

select instance > Action > image and templates > create image

![](./Images/images15/webserver%20ami.PNG)

**Creating an AMI out of the Bastion EC2 Instance**

![](./Images/images15/bastion%20ami.PNG)

**Creating an AMI out of the nginx EC2 Instance**

![](./Images/images15/nginx%20ami.PNG)

![](./Images/images15/3%20ami.PNG)

### **STEP 5: Configuring Target Groups**
---
**For Nginx Server**

- Selecting Instances as the target type
- Ensuring the protocol HTTPS on secure TLS port 443
- Ensuring that the health check path is **/healthstatus**

![](./Images/images15/nginx%20tg.PNG)

**For Wordpress Server**

- Selecting Instances as the target type
- Ensuring the protocol HTTPS on secure TLS port 443
- Ensuring that the health check path is **/healthstatus**

![](./Images/images15/wordpress%20tg.PNG)

**For Tooling Server**

- Selecting Instances as the target type
- Ensuring the protocol HTTPS on secure TLS port 443
- Ensuring that the health check path is **/healthstatus**

![](./Images/images15/tooling%20tg.PNG)

![](./Images/images15/Tg.PNG)

### **STEP 6: Configuring Application Load Balancer (ALB)**
---
**For External Load Balancer**

- Selecting Internet facing option
- Ensuring that it listens on **HTTPS** protocol (TCP port 443)
- Ensuring the ALB is created within the appropriate **VPC, AZ** and the right **Subnets**
- Choosing the Certificate already created from **ACM**
- Selecting **Security Group** for the external load balancer
- Selecting **Nginx** Instances as the **target group**


![](./Images/images15/ext%20ALB.PNG)

![](./Images/images15/ext%20alb%202.PNG)

**For Internal Load Balancer**

- Setting the Internal ALB option
- Ensuring that it listens on **HTTPS** protocol (TCP port 443)
- Ensuring the ALB is created within the appropriate **VPC**, **AZ** and **Subnets**
- Choosing the Certificate already created from **ACM**
- Selecting Security Group for the internal load balancer
- Selecting **webserver** Instances as the target group
- Ensuring that health check passes for the target group

![](./Images/images15/int%20ALB.PNG)

![](./Images/images15/LB.PNG)

NOTE: Based on my Internal Load balancer configuration, my default preference, all traffic is forwarded to the webserver. I will now set a rule to catch tooling request and forward to tooling target.

Select int-ALB > Listeners > view/edit rules > insert rule > Add condition > Host header > Add action > Forward to > tooling target grp > Save

![](./Images/images15/conf%20rule%201.PNG)

![](./Images/images15/conf%20rule%202.PNG)

### **STEP 7: Creating A Launch Template**
---
**For Bastion Server**

- Setting up a launch template with the **Bastion AMI**
- Ensuring the Instances are launched into the **public subnet**
- Entering the Userdata to update yum package repository and install **ansible** and **mysql**

![](./Images/images15/bastion%20template.PNG)

![](./Images/images15/bastion%20template%202.PNG)

![](./Images/images15/bastion%20template%203.PNG)

![](./Images/images15/bastion%20template%204.PNG)

**For Nginx Server**

- Setting up a launch template with the Nginx AMI
- Ensuring the Instances are launched into the public subnet
- Assigning appropriate security group
- Entering the Userdata to update yum package repository and install Nginx

![](./Images/images15/nginx%20temp%201.PNG)

![](./Images/images15/nginx%20temp%202.PNG)

![](./Images/images15/nginx%20temp%203.PNG)

**For Wordpress Server**

- Setting up a launch template with the Webserver AMI
- Ensuring the Instances are launched into the Private subnet
- Assigning appropriate security group
- Configure Userdata to update yum package repository and install wordpress and apache server

![](./Images/images15/wordpress%20temp%201.PNG)

![](./Images/images15/wordpress%20tem%202.PNG)

![](./Images/images15/wordpress%20temp%203.PNG)

**For Tooling Server**

- Setting up a launch template with the Webserver AMI
- Ensuring the Instances are launched into the Private subnet
- Assigning appropriate security group
- Configure Userdata to update yum package repository and install apache server

![](./Images/images15/tooling%20temp%201.PNG)

![](./Images/images15/tooling%20temp%202.PNG)

![](./Images/images15/tooling%20temp%203.PNG)

- All 4 Launch templates created successfully

![](./Images/images15/all%20launch%20temp.PNG)

### **STEP 8: Configuring AutoScaling Group**
---
- Selecting the right launch template
- Selecting the VPC
- Selecting both public subnets 1&2
- Enabling Application Load Balancer for the AutoScalingGroup (ASG)
- Selecting the target group you created before
- Ensuring health checks for both EC2 and ALB
- Setting the desired capacity, Minimum capacity and Maximum capacity to 2
- Setting the scale out option if CPU utilization reaches 90%
- Activating SNS topic to send scaling notifications

**ASG For Bastion Server**

![](./Images/images15/bastion%20asg%201.PNG)

![](./Images/images15/bastion%20asg%202.PNG)

![](./Images/images15/bastion%20asg%203.PNG)

![](./Images/images15/bastion%20asg%204.PNG)

![](./Images/images15/bastion%20asg%205.PNG)

**ASG For Nginx**

![](./Images/images15/nginx%20asg%201.PNG)

![](./Images/images15/nginx%20asg%202.PNG)

![](./Images/images15/nginx%20asg%203.PNG)

![](./Images/images15/nginx%20asg%204.PNG)

![](./Images/images15/nginx%20asg%205.PNG)

**NOTE:** Before i configure my **auto scaling group** for my **webserver**, Using **ssh Agent** on my **bastion server** on **mobaxterm** i will access my **RDS endpoint** and create **wordpress** and **tooling** **database**.

- **Configure MobaXterm**

![](./Images/images15/mobaxtern%20conf%20for%20ssh%20agent.PNG)

- **Using ssh agent goto to bastion server**

`ssh - A ec2-user@3.8.155.64`

![](./Images/images15/ssh%20into%20bastion.PNG)

- Goto mysql using my RDS endpoint as host, user name and password.
- And create wordpress and tooling database

![](./Images/images15/rds%20endpoint.PNG)

![](./Images/images15/create%20database.PNG)

**ASG For Wordpress**

![](./Images/images15/wordpress%20asg%201.PNG)

![](./Images/images15/wordpress%20asg%202.PNG)

![](./Images/images15/wordpress%20asg%203.PNG)

![](./Images/images15/wordpress%20asg%204.PNG)

![](./Images/images15/wordpress%20asg%205.PNG)

** ASG For Tooling Server**
 
 NOTE: same steps for wordpress ASG applies
 - Select tooling template
 - Select my vpc
 - select private subnet 1&2
 - Click Attach an existing Balancer and select tooling LB target group
 - Ensure that ELB health check type is clicked also as EC2 is picked by default.
 - Click target tracking scalling policy and input targer value to 90
 - Add notifiaction and input my SNS topic
 - Add tag. Then proceed to create auto scaling group

 NOTE: I will proceed to delete the instance (nginx, webserver and bastion) i created used to launch templates

### **STEP 9: Creating DNS Records In The Route53 For the Tooling And Wordporess site**
---
![](./Images/images15/tooling%20dns%20rec.PNG)

![](./Images/images15/dns%20rec.PNG)

### **STEP 10: Result**
---
Pasting the url http://wordpress.krisqriz.ga/ and http://www.tooling.krisqriz.ga/ will route traffic from the **External ALB** to the **nginx** will serve as my reverse proxy and then to the server for **wordpress** and **tooling** site respectively through the **internal ALB**

- **Wordpress:**

![](./Images/images15/wordpress%20login.PNG)

![](./Images/images15/wordpress%20login%202.PNG)

- **Tooling:**

![](./Images/images15/tooling%20login.PNG)

### End of Project...
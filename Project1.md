# PROJECT 1 : LAMP STACK IMPLEMENTATION

## STEP 0 : LAUNCHING A LINUX UBUNTU EC2 INSTANCE IN AWS

 -   ###  Launch and spinup an ec2 ubuntu instance in aws. created and download a new key pair of .pem file 

     ![aws instances 2](https://user-images.githubusercontent.com/106500055/172025077-dab93c4a-9529-4e24-b1b9-a4c132b2335f.PNG)
     
-  ### To connect to my ec2 linux server from my window PC : Download, install and configure MobaXterm.  
      ####   MobaXterm Configuration:
      1. On Persistent home directory, i select the local folder where my downloaded key pair of .pem file reside and click "Ok"
      
      ![image](https://user-images.githubusercontent.com/106500055/172026202-33032a9d-40d7-4989-8c27-b9ea7a664c32.png)
     
      
      2. Click session > Start local terminal > ssh -i "kriz-ec2.pem" ubuntu@ec2-18-134-94-117.eu-west-2.compute.amazonaws.com
           
            Unable to connect to host connection timeout error:
            
            ![image](https://user-images.githubusercontent.com/106500055/172027058-24d77430-c11e-4d0c-beb9-3d7b9d6dbd83.png)
            
      3. Go to ec2 console > created a new security group > Edit inbound rule to allow all network > then assigned the security group to my ec2 instance
      
      ![image](https://user-images.githubusercontent.com/106500055/172027335-6cbbc6b2-45de-41ba-8cd6-18acceb52b09.png)
      
      4. Connection was successful after: ssh -i "kriz-ec2.pem" ubuntu@ec2-18-134-94-117.eu-west-2.compute.amazonaws.com

      ![image](https://user-images.githubusercontent.com/106500055/172027567-c45b38ab-2327-44ff-9a82-430c68973523.png)
      
## STEP 1 : INSTALLING APACHE AND UPDATING THE FIREWALL  

  - ### Install Apache using Ubuntu’s package manager ‘apt’
  
     1. Update a list of packages in package manager
  
            sudo apt update   
     
     2. Run apache2 package installation 
     
            sudo apt install apache2
          
     3. Verify that apache2 is installed and running as a Service in my instance  
       
            sudo systemctl status apache2       
            
       ![Apache install and running in AWS ec2 instance](https://user-images.githubusercontent.com/106500055/172028733-8ccb408f-7534-439e-a168-8780a711694e.PNG)
       
   - ### To receive traffic by our web server we need to open TCP port 80 which is the default port that web browsers use to access web pages on the Internet

    Add a security rule to allow inbound HTTP (port 80) connections to my ec2 instance. 
   ![add rule to open http port 80](https://user-images.githubusercontent.com/106500055/172029040-e82f228b-bcf1-4cbb-b198-16758cec37d0.PNG)
   
   - ### Test how my Apache HTTP server can respond to requests from the Internet:
   
         Paste the url: http://13.40.11.79:80 on a browser 
         ran curl -s http://169.254.169.254/latest/meta-data/public-ipv4 to retrieve my public ip address
         
     ![image](https://user-images.githubusercontent.com/106500055/172029616-eef6ef54-ad65-4f8d-a53f-138d310a5e4d.png)

## STEP 2 : INSTALLING MYSQL

- Installed mysql server:  

      sudo apt install mysql-server

- log in to the MySQL console :

      sudo mysql
      
    ![login to mySql console](https://user-images.githubusercontent.com/106500055/172030087-2b820cc7-273d-432d-b01c-bd486b12664d.PNG)

- run a security script that comes pre-installed with MySQL to remove some insecure default settings and lock down access to my database system. Before running the script, 
i will set a password for the root user using **"mysql_native_password"** as default authentication method. I'll define this user’s password as **"PassWord.1"**

        ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'PassWord.1'; 
        
- Start the interactive script by running :

      sudo mysql_secure_installation

- Test to login to sql console using password

           sudo mysql -p
           
## STEP 3 : INSTALLING PHP

- Install php package, php-mysql, and libapache2-mod-php. To install these 3 packages at once, run :

      sudo apt install php libapache2-mod-php php-mysql

- To check php version insatlled run :
    
      php -v
      
   ![check php version](https://user-images.githubusercontent.com/106500055/172030614-c12f5a37-f0f7-4663-962b-eb62dbf7726c.PNG)

## STEP 4 : CREATING A VIRTUAL HOST FOR YOUR WEBSITE USING APACHE

* Set up a domain called "projectlamp"

* Create the directory for projectlamp using ‘mkdir’ command as follows:
        
        sudo mkdir /var/www/projectlamp

* Assign ownership of the directory with my current system user:
         
         sudo chown -R $USER:$USER /var/www/projectlamp

* Create and open a new configuration file in Apache’s *"sites-available directory"* using vi command-line editor
    
      sudo vi /etc/apache2/sites-available/projectlamp.conf
      
* Copy and Insert the below file in vi editor open above

         <VirtualHost *:80>
            ServerName projectlamp
            ServerAlias www.projectlamp 
            ServerAdmin webmaster@localhost
            DocumentRoot /var/www/projectlamp
            ErrorLog ${APACHE_LOG_DIR}/error.log
            CustomLog ${APACHE_LOG_DIR}/access.log combined
        </VirtualHost>

![insert to vi](https://user-images.githubusercontent.com/106500055/172031026-0e9bbfd9-bfbc-421c-b80e-40c04324bf0e.PNG)

*  Use the ls command to show the new file in the "sites-available" directory. run :
           
          sudo ls /etc/apache2/sites-available
          
 ![output of file inseted in vi](https://user-images.githubusercontent.com/106500055/172031264-a4b2eca7-23fa-4859-850c-3cebb89558d0.PNG)

* You can now use a2ensite command to enable the new virtual host: Run :
      
       sudo a2ensite projectlamp

* Use a2dissite command to disable default website that comes with Apache since custom domain name is not used
  so that Apache’s default configuration would not overwrite my virtual host.
       
          sudo a2dissite 000-default

* Check to ensure configuration file doesn’t contain syntax errors:
         
          sudo apache2ctl configtest

* Reload Apache so these changes take effect:
    
         sudo systemctl reload apache2
         
 * To test that the virtual host is working as expected, create an index.html file in the web root /var/ww/projectlamp which is currently empty

        sudo echo 'Hello LAMP from hostname' $(curl -s http://169.254.169.254/latest/meta-data/public-hostname) 
        'with public IP' $(curl -s http://169.254.169.254/latest/meta-data/public-ipv4) > /var/www/projectlamp/index.html

        input http://<Public-IP-Address>:80 on a browser to see the output
        
 ## STEP 5 : ENABLE PHP ON THE WEBSITE
 
 * With the default DirectoryIndex settings on Apache, a file named **index.html** will always take precedence 
  over an **index.php** file. So I want to edit the **/etc/apache2/mods-enabled/dir.conf** file and change the order in which the **index.php** 
  file is listed within the DirectoryIndex directive. 
    
        sudo vim /etc/apache2/mods-enabled/dir.conf
        
  * Insert the below file in vi editor open above
  
        <IfModule mod_dir.c>
            #Change this:
            #DirectoryIndex index.html index.cgi index.pl index.php index.xhtml index.htm
            #To this:
            DirectoryIndex index.php index.html index.cgi index.pl index.xhtml index.htm
        </IfModule>
        
  * reload Apache for the change to take effect
  
        sudo systemctl reload apache2

* To create a PHP script to test that PHP is correctly installed and configured on my server. I create a new file named index.php 
  inside my custom web root folder:
     
      run : vim /var/www/projectlamp/index.php

* I add the following text, which is valid PHP code, inside the file

      <?php
      phpinfo();
      
  ![insert to php](https://user-images.githubusercontent.com/106500055/172032040-e6519031-a71e-4514-991e-58b77b5b3198.PNG)

      Goto page on browser http://18.170.67.18:80 and see how it display the php page
      
  ![php display on browser](https://user-images.githubusercontent.com/106500055/172032149-4c032abf-142e-495f-b17f-7fc34ffff4dc.PNG)
  
* After checking the relevant information about my PHP server through that page. I remove the file i created as it contains sensitive information 
  about my PHP environment, and my Ubuntu server.
  
      sudo rm /var/www/projectlamp/index.php

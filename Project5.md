## PROJECT 5: CLIENT/SERVER ARCHITECTURE USING A MYSQL RELATIONAL DATABASE MANAGEMENT SYSTEM
---
### TASK :

Implement a Client Server Architecture using MySQL Database Management System (DBMS).

- Create and configure two Linux-based virtual servers (EC2 instances in AWS).

      Server A name - `mysql server`
      Server B name - `mysql client`

- On **mysql server** Linux Server install MySQL Server software.
  
  `sudo apt update`

  `sudo apt upgrade`

  `sudo apt install mysql-server`

- Check status of myql server

   `sudo systemctl status mysql.service`

   ![sql server running](https://user-images.githubusercontent.com/106500055/175837699-f1066fd9-54bb-49e8-869a-8be7ba7f1b8b.PNG)

- log in to the MySQL console :

   `sudo mysql`   

- Create a new user named **"kris"** in **mysql-server**

  `CREATE USER 'kris'@'%' IDENTIFIED WITH mysql_native_password BY 'Krispassword.1';`

  - View the newly created user

    `SELECT USER from mysql.user;`  

    ![user kris](https://user-images.githubusercontent.com/106500055/175837731-259cff16-7371-49c5-b8d1-8bd2e3acbd6e.PNG)

- Create a database named **Project_5**

- `CREATE DATABASE `Project_5`;` 

  - View the newly created database

    `SHOW DATABASES;` 

    ![show databases](https://user-images.githubusercontent.com/106500055/175837763-17459400-bfbb-479e-9dda-b9a606351358.PNG)

- Now i'll give this user **"kris"** full permission over the **Project_5 database**

  `GRANT ALL ON Project_5.* TO 'kris'@'%';`    

- On **mysql client** Linux Server install MySQL Client software

    `sudo apt update`

    `sudo apt upgrade`

    `sudo apt install mysql-client`

- Edit inbound rule for **mysql server**    

  MySQL server uses TCP port 3306 by default, so i will have to edit my ‘Inbound rules’ in ‘mysql server’ Security Groups. And allow ‘mysql client’ Private IP address. So mysql client can connect to mysql server database. 

  ![inbound rule](https://user-images.githubusercontent.com/106500055/175837647-d6618fa7-5c3f-44f5-a3bb-4f5d7612debc.PNG)

- I'll need to configure MySQL server to allow connections from remote hosts by Replacing **‘127.0.0.1’ to ‘0.0.0.0’**

1. Goto **etc** directory which holds **mysql** configuration. Then ls to see its files and folder
    
     `cd /etc/mysql`

     ![mysql conf 1](https://user-images.githubusercontent.com/106500055/175837782-d8125eea-1e35-44ec-b836-bd852ecbb239.PNG)

2. cd into **mysql.conf.d** folder and ls to see its file

    `cd mysql.conf.d`

    ![mysql conf 2](https://user-images.githubusercontent.com/106500055/175837802-f09bfc19-91f3-4322-8d84-8f357427705f.PNG)

3. Using vi editor, edit the content in **mysqld.cnf** file by replacing **127.0.0.1 with 0.0.0.0** in the Bind Address   

   `sudo vi mysqld.cnf` 

   ![127 0 0 1 edit](https://user-images.githubusercontent.com/106500055/175837846-04d86f81-f58c-420c-89eb-b2d62a29f680.PNG) 

- Connect to mysql server from mysql client instance

  - on mysql client instance

    `mysql -ukris -h 172.31.42.172 -p`   

    ![connect from client](https://user-images.githubusercontent.com/106500055/175837897-85f6563a-9f00-4bff-96bd-4713f377443c.PNG)

   I have successfully deployed a fully functional MySQL Client-Server set up. 

  ### End of Project...          

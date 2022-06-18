# PROJECT 2: LEMP STACK IMPLEMENTATION
## STEP 0 – LAUNCHING A LINUX UBUNTU EC2 INSTANCE IN AWS
---
### 1. **Launch and spinup an ec2 ubuntu instance in aws**.

![ec2 instance](https://user-images.githubusercontent.com/107736487/174419046-67d10c31-b2f8-4f33-a7e4-e5cfa3bcaa42.PNG)

### 2. **SSH to my ubuntu server from my window terminal**

      ssh -i "kriz-ec2.pem" ubuntu@ec2-18-170-42-91.eu-west-2.compute.amazonaws.com

![ssh to ubuntu](https://user-images.githubusercontent.com/107736487/174419172-ba7969eb-c36a-4be0-8ddd-6e337e0fb413.PNG)

## STEP 1 – INSTALLING THE NGINX WEB SERVER
---
- **Update 'apt' package manager**.

    `sudo apt update`
- **Install nginx using Ubuntu’s package manager ‘apt’**

    `sudo apt install nginx`

- **Verify that nginx was successfully installed and is running as a service in Ubuntu**, run:

    `sudo systemctl status nginx`  
      
    ![nginx status](https://user-images.githubusercontent.com/107736487/174419198-03f7e878-0331-4919-b440-b16396a54687.PNG)
  

- **To receive traffic to my web server i need to open TCP port 80 which is the default port that web browsers use to access web pages on the Internet
Add a security rule to allow inbound HTTP (port 80) connections to my ec2 instance**. 

  ![inbound rule](https://user-images.githubusercontent.com/107736487/174419245-5103d287-b964-44ef-ba7c-1b27b92b03ec.PNG)


- **Test how my Nginx server can respond to requests from the Internet:
Paste the url: http://18.170.42.91:80 on a browser** 

  ![nginx on browser](https://user-images.githubusercontent.com/107736487/174419320-4e00ae4d-72b6-44df-951e-28f315f903d8.PNG)

## STEP 2 — INSTALLING MYSQL
---  
- **Installed mysql server :**

  `sudo apt install mysql-server`

- **log in to the MySQL console :**

   `sudo mysql`

   ![mysql console](https://user-images.githubusercontent.com/107736487/174419383-9a274585-566c-45c8-aea6-453f6af43b12.PNG)

- **Run a security script that comes pre-installed with MySQL to remove some insecure default settings and lock down access to my database system. Before running the script, i will set a password for the root user using "mysql_native_password" as default authentication method. I'll define this user’s password as "PassWord.1"**

    `ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'PassWord.1';`   

- **Exit the MySQL shell**

    `mysql> exit`  

- **Start the interactive script by running :**

  `sudo mysql_secure_installation`

  NOTE: This will ask if you want to configure the VALIDATE PASSWORD PLUGIN. I choose to disable the validation.

  ![passwrd validation plugin](https://user-images.githubusercontent.com/107736487/174419415-04e90fae-4f2f-4e7a-a482-396904c26481.PNG)

  **I change the password for root user**

  ![change pw for root1](https://user-images.githubusercontent.com/107736487/174419439-560a6e9e-92b1-47c3-99b2-2c817a7f6de4.PNG) 

  **I enable the removal of some insecure default settings and lock down access to my database system.**

  ![removing interactive user](https://user-images.githubusercontent.com/107736487/174419457-d32828de-a983-4788-aedc-153371d8775d.PNG)

- **Test to login to sql console using password**

  `sudo mysql -p` 

  ![test to login with pwd](https://user-images.githubusercontent.com/107736487/174419483-3983a908-74f0-417d-a50a-2b1507eca0a4.PNG)

## STEP 3 – INSTALLING PHP
---
I have Nginx installed to serve my content and MySQL installed to store and manage my data. Now i can install PHP to process code and generate dynamic content for the web server.

Nginx requires an external program to handle PHP processing and act as a bridge between the PHP interpreter itself and the web server. But it requires additional configuration. I’ll need to install **php-fpm**, and **php-mysql** Core PHP packages will automatically be installed as dependencies.

- **To install these 2 packages at once, run :**

   `sudo apt install php-fpm php-mysql`

- **To check php version installed run :**

    `php -v`

   ![php version](https://user-images.githubusercontent.com/107736487/174419502-230753a2-3b2f-470e-8a63-23700371fdbb.PNG)

## STEP 4 — CONFIGURING NGINX TO USE PHP PROCESSOR
---
- **Set up a domain called "projectLEMP" as an example domain**

- **Create the root web directory for projectLEMP using ‘mkdir’ command as follows :**

  `sudo mkdir /var/www/projectLEMP`      

- **Next, I assign ownership of the directory with the $USER environment variable, which will reference your current system user :**

  `sudo chown -R $USER:$USER /var/www/projectLEMP`

- **open a new configuration file in Nginx’s **sites-available** directory using **nano** command-line editor.** 

   `sudo nano /etc/nginx/sites-available/projectLEMP`

- **Copy and Insert the below file in nano editor open above**  

   ```
   #/etc/nginx/sites-available/projectLEMP

  server {
      listen 80;
      server_name projectLEMP www.projectLEMP;
      root /var/www/projectLEMP;

      index index.html index.htm index.php;

      location / {
          try_files $uri $uri/ =404;
      }

      location ~ \.php$ {
          include snippets/fastcgi-php.conf;
          fastcgi_pass unix:/var/run/php/php8.1-fpm.sock;
      }

      location ~ /\.ht {
         deny all;
      }

    }
   ```

  ![nano editor](https://user-images.githubusercontent.com/107736487/174419528-ac2be632-8007-46ec-a5de-27fd14878e49.PNG)

- **Activate your configuration by linking to the config file from Nginx’s sites-enabled directory.
This will tell Nginx to use the configuration next time it is reloaded**

  `sudo ln -s /etc/nginx/sites-available/projectLEMP /etc/nginx/sites-enabled/`

- **Test configuration for syntax errors by typing :**

  `sudo nginx -t`  

  ![test nginx](https://user-images.githubusercontent.com/107736487/174419566-9b97d868-f578-493a-ac46-f6bc7e8fc8f8.PNG)


- **I need to disable default Nginx host that is currently configured to listen on port 80, for this i run :**

  `sudo unlink /etc/nginx/sites-enabled/default`

- **Reload Nginx to apply the changes :**

  `sudo systemctl reload nginx` 

- **My new website is now active, but the web root /var/www/projectLEMP is still empty. I'll Create an index.html file in that location so that i can test that my new server block works as expected:**   

  `sudo echo 'Hello LEMP from hostname' $(curl -s http://169.254.169.254/latest/meta-data/public-hostname) 'with public IP' $(curl -s http://169.254.169.254/latest/meta-data/public-ipv4) > /var/www/projectLEMP/index.html` 

- input http://18-170-23-247:80 on a browser to see the output 

  ![lemp is working](https://user-images.githubusercontent.com/107736487/174419612-9cf15ba8-8271-43b3-9a52-10fe24ab4a00.PNG)


## STEP 5 – TESTING PHP WITH NGINX
---
- My LEMP stack is now completely, installed, set up and fully operational. 

   I can test it to validate that Nginx can correctly handle .php files off to my PHP processor.

  For testing purpose, i create a PHP file called info.php in my document root in nano text editor:

  `sudo nano /var/www/projectLEMP/info.php`

- I paste the following lines into the new file above opened in nano editor. This is valid PHP code that will return information about my server:

  ```
  <?php
  phpinfo();
- Goto page on browser http://13.40.140.199/info.php and see how it display the php page

  ![php test](https://user-images.githubusercontent.com/107736487/174419639-618c1262-4b96-4cc8-bc08-d7cbadb573ca.PNG)

- After checking the relevant information about my PHP server through that page. I remove the file i created as it contains sensitive information about my PHP environment, and my Ubuntu server.

  `sudo rm /var/www/ec2-13-40-140-199.eu-west-2.compute.amazonaws.com/info.php`

## STEP 6 – RETRIEVING DATA FROM MYSQL DATABASE WITH PHP

- I will create a test database (DB) with simple "To do list" and configure access to it, so the Nginx website would be able to query data from the DB and display it.

  I'll need to create a new user with the **mysql_native_password** authentication method in order to be able to connect to the MySQL database from PHP.

  I'll create a database named **example_database** and a user named **example_user**

1. #### **Connect to my MySQL console using the root     account :**

   `sudo mysql -p`

   ![login to sql server](https://user-images.githubusercontent.com/107736487/174419662-8e341fe7-f939-4092-b97d-e45c18532657.PNG)

2. #### **create a new database, run the following command from my MySQL console :**

       mysql> CREATE DATABASE `example_database`;    

    ![create database in sql console](https://user-images.githubusercontent.com/107736487/174419680-7f4a1483-ee89-48bb-abca-58905b3d64a3.PNG)


3. #### **Now i can create a new user and grant him full privileges on the database i've just created.**

   The following command creates a new user named example_user, using mysql_native_password as default authentication method. i am defining this user’s password as **krislempproject**

       mysql>  CREATE USER 'example_user'@'%' IDENTIFIED WITH mysql_native_password BY 'krislempproject';

    ![create user in mysql](https://user-images.githubusercontent.com/107736487/174419797-48d44f10-4fe0-426a-9a9f-928c48e3a3eb.PNG)

4. #### **Now i need to give this user full permission over the *example_database* database :**

       mysql> GRANT ALL ON example_database.* TO 'example_user'@'%';
    ![permission to user on db](https://user-images.githubusercontent.com/107736487/174419826-05f9733b-6c4d-4b95-9fbd-06d8eba6710e.PNG)

    Exit the MySQL shell with:

       mysql> exit

5. #### **I test if the new user has the proper permissions by logging in to the MySQL console again, this time using the custom user credentials:**

       mysql -u example_user -p    

    ![signing to new db using p](https://user-images.githubusercontent.com/107736487/174419860-65d7dc4e-4268-4968-a320-a6ac3b425bc8.PNG)

6. #### **After logging in to the MySQL console as example_user, confirm that the example_user have access to the example_database database:**

       mysql> SHOW DATABASES;    

    ![user can see db](https://user-images.githubusercontent.com/107736487/174419879-1d61841c-1aa4-42a0-9172-6042910e4013.PNG)

- **Next, i’ll create a test table named todo_list. From the MySQL console, run the following statement:**

  ```
  CREATE TABLE example_database.todo_list (
     item_id INT AUTO_INCREMENT,
     content VARCHAR(255),
     PRIMARY KEY(item_id)
  );
- **I'll insert a few rows of content in my test table and a value. Run:**
  ```
  mysql> INSERT INTO example_database.todo_list (content) VALUES
                    ("My first important item"),
                    ("My second important item"),
                    ("My third important item"),
                    ("and this one more thing");

- **To confirm that the data was successfully saved to my table, then exit.**

  `mysql>  SELECT * FROM example_database.todo_list;` 

  ![insert content to by table](https://user-images.githubusercontent.com/107736487/174419911-15c2b855-4f90-4921-95b0-cc8ff19ac58c.PNG)

- **Now i can create a PHP script that will connect to MySQL and query for my content. I'll Create a new PHP file in my custom web root directory using nano editor.** 

  `nano /var/www/projectLEMP/todo_list.php`

- **I'll Copy the content below and paste in nano editor into my todo_list.php script:**

  **The PHP script connects to the MySQL database and queries for the content of the todo_list table,  and displays the results in a list.**

  ```
   <?php
   $user = "example_user";
   $password = "krislempproject";
   $database = "example_database";
   $table = "todo_list";

   try {
     $db = new PDO("mysql:host=localhost;dbname=$database", $user, $password);
     echo "<h2>TODO</h2><ol>";
     foreach($db->query("SELECT content FROM $table") as $row) {
       echo "<li>" . $row['content'] . "</li>";
     }
     echo "</ol>";
   } catch (PDOException $e) {
       print "Error!: " . $e->getMessage() . "<br/>";
       die();
   }

- **To confirm that my PHP environment is ready to connect and interact with my MySQL server.**

  **I'll access this page in my web browser by visiting the public IP address configured for my website, followed by /todo_list.php:**

  http://18.130.233.24/todo_list.php

  ![todo list result](https://user-images.githubusercontent.com/107736487/174419941-0e942ba6-5c7a-492c-a5e2-989307b0c0bc.PNG)



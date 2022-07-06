## PROJECT 6: Web Solution With WordPress
---
**NOTE:** In this Project, i'll be having a 3-Tier Setup.
1. My PC to serve as a client
2. An EC2 Redhat Linux Server as a web server (This is where i'll install WordPress)
3. An EC2 Redhat Linux server as a database (DB) server

### **Step 1 — Prepare a Web Server**
---
- Launch two EC2 instance. one will serve as **"Web Server"**, the other will serve as **Database server**

1. Create 3 volumes 10 GB each in the same AZ as my **Web Server** instance.

    ![create volume](https://user-images.githubusercontent.com/107736487/177606513-54afefe2-c48e-4d20-ba13-6a19b6de4cbd.PNG)


2. Attach all three volumes one by one to my Web Server EC2 instance

   ![attach volume](https://user-images.githubusercontent.com/107736487/177606587-3cc59973-4a8e-4dac-9765-b8d2e0068e38.PNG)

3.  Using my windows Terminal, i'll ssh to my linux web server to begin configuration (Creating partition on my disks).

- Using `lsblk` command to inspect what block devices are attached to my web server. 

   ![lsblk 1](https://user-images.githubusercontent.com/107736487/177606786-67050933-7e4f-40a2-9b25-66f64b665aa5.PNG)

- All devices in Linux reside in **/dev/** directory. Using `ls /dev/` to see all my 3 created block devices  **xvdf, xvdh, xvdg**.

- Using `df -h` command to see all mounts and free space on my web server

  ![df -h](https://user-images.githubusercontent.com/107736487/177606946-79e67367-3a27-4d54-a29d-6e5fdeb2f76c.PNG)


4. Using **gdisk** utility to create a single gpt partition on each of the 3 disks **xvdf, xvdg, xvdh**

   `sudo gdisk /dev/xvdf`
  
    ![partition create](https://user-images.githubusercontent.com/107736487/177607185-5189e76f-3955-43b4-a0a7-084a86cf60c3.PNG)

5. Install **lvm2** package 

   `sudo yum install lvm2` 
   
6. Check for available partitions.

   `sudo lvmdiskscan`

   ![lvmdiskscan](https://user-images.githubusercontent.com/107736487/177607302-0961d8e1-ace4-4a68-9b40-2b1c21032042.PNG)

7. Using **pvcreate** utility i'll mark each of my 3 disks as physical volumes to be used by LVM

   `sudo pvcreate /dev/xvdf1 /dev/xvdg1 /dev/xvdh1`

   ![pv create](https://user-images.githubusercontent.com/107736487/177607415-65845871-db85-4122-82e1-b66b60fee75b.PNG)

8. Verify that my Physical volume has been created successfully.

   `sudo pvs`

   ![pvs confirm](https://user-images.githubusercontent.com/107736487/177607496-18f19a20-8a93-4dfe-9703-76d21875c311.PNG)

9. Using **vgcreate** utility i'll add all 3 PVs to a volume group (VG). Named **webdata-vg**

   `sudo vgcreate webdata-vg /dev/xvdh1 /dev/xvdg1 /dev/xvdf1`      

10. Verify that my VG has been created successfully by running 

    `sudo vgs`

    ![vgs confirm](https://user-images.githubusercontent.com/107736487/177607587-d90aeb3d-4b34-4aa0-9f38-7812db1ac1d8.PNG)

11. Using **lvcreate** utility to create 2 logical volumes. **apps-lv** (Use half of the PV size), and **logs-lv** Use the remaining space of the PV size. 

    **NOTE:** *"apps-lv"* will be used to store data for the Website while, *"logs-lv"* will be used to store data for logs.

        sudo lvcreate -n apps-lv -L 14G webdata-vg
        sudo lvcreate -n logs-lv -L 14G webdata-vg    

12. Verify that my Logical Volume has been created successfully.

    `sudo lvs` 

    ![confirm lv create](https://user-images.githubusercontent.com/107736487/177607932-f1d778a0-28b5-4679-825e-7bc06d7a568f.PNG)

13. Verify the entire setup

        sudo vgdisplay -v #view complete setup - VG, PV, and LV

        sudo lsblk 

    ![confirm setup](https://user-images.githubusercontent.com/107736487/177608005-efc69b17-e774-4b7a-b978-b456901d7548.PNG)

14. Use mkfs.ext4 to format the logical volumes with ext4 filesystem

        sudo mkfs -t ext4 /dev/webdata-vg/apps-lv
        sudo mkfs -t ext4 /dev/webdata-vg/logs-lv                   

15. Create **/var/www/html** directory to store website files

    `sudo mkdir -p /var/www/html`

16. Create **/home/recovery/logs** to store backup of log data

    `sudo mkdir -p /home/recovery/logs`

17. Mount **/var/www/html** on apps-lv logical volume

    `sudo mount /dev/webdata-vg/apps-lv /var/www/html/`    

18. Using rsync utility to backup all the files in the log directory **/var/log** into **/home/recovery/ logs** (This is required before mounting the file system)

    `sudo rsync -av /var/log/. /home/recovery/logs/`    

19. Mount **/var/log** on **logs-lv** logical volume. (Note that all the existing data on **/var/log** will be deleted. That is why step 15 above is very
important)

    `sudo mount /dev/webdata-vg/logs-lv /var/log` 

20. Restore log files back into **/var/log** directory

    `sudo rsync -av /home/recovery/logs/. /var/log`   

21. Update **/etc/fstab** file so that the mount configuration will persist after restart of the server  

    i. run: `sudo blkid`

    ii  Copy the **UUID** of the device which i'll be updating in the **/etc/fstab** file

    iii. `sudo vi /etc/fstab`

    iv. Update **/etc/fstab** by pasting the **UUID** copied above.

    ![uuid webserver](https://user-images.githubusercontent.com/107736487/177608225-8e7bac9c-4d96-4a31-ac63-fad188de4ca6.PNG)

    - Test the configuration and reload the daemon

      `sudo mount -a`

      `sudo systemctl daemon-reload`

    - Verify my setup by running: `df -h` 

      ![verify setup df-h](https://user-images.githubusercontent.com/107736487/177608316-60e6b30b-15cb-4fb7-a27a-9dfb871aff02.PNG)

### **Step 2 — Prepare the Database Server**
---  
I'll Start the Database server and ssh into it.

All the steps above i did for the **web server,** I'll Repeat the same steps for the **Database Server**.
 
But instead of **apps-lv** i'll create **db-lv** and mount it to **/db** directory instead of **/var/www/html/**.     

- **Step 11:** Using **lvcreate** utility to create 2 logical volumes. **db-lv** and **logs-lv** 

      sudo lvcreate -n db-lv -L 14G webdata-vg
      sudo lvcreate -n logs-lv -L 14G webdata-vg  

- **Step 14:** Use **mkfs.ext4** to format the logical volumes with **ext4** filesystem

      sudo mkfs -t ext4 /dev/webdata-vg/db-lv
      sudo mkfs -t ext4 /dev/webdata-vg/logs-lv      

- **Step 15:** Create **/db** directory to store database files

    `sudo mkdir -p /db`

- **Step 16:** Create **/home/recovery/logs** to store backup of log data

    `sudo mkdir -p /home/recovery/logs`

- **Step 17:** Mount **/db** on **db-lv** logical volume

    `sudo mount /dev/webdata-vg/db-lv /db`  

- Repeat step 18 - 21 as performed above.

### **Step 3 — Install WordPress on my Web Server**
---
- Update the repository

  `sudo yum -y update`

- Install wget, Apache and it’s dependencies

  `sudo yum -y install wget httpd php php-mysqlnd php-fpm php-json`

- Start Apache
  
  `sudo systemctl enable httpd`

  `sudo systemctl start httpd`

- To install PHP and it’s dependencies

      sudo yum install https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm
      sudo yum install yum-utils http://rpms.remirepo.net/enterprise/remi-release-8.rpm
      sudo yum module list php
      sudo yum module reset php
      sudo yum module enable php:remi-7.4
      sudo yum install php php-opcache php-gd php-curl php-mysqlnd
      sudo systemctl start php-fpm
      sudo systemctl enable php-fpm
      sudo setsebool -P httpd_execmem 1  

- Restart Apache

  `sudo systemctl restart httpd`

- Download **wordpress** and copy wordpress to **var/www/html**
  ```
  mkdir wordpress
  cd   wordpress
  sudo wget http://wordpress.org/latest.tar.gz
  sudo tar xzvf latest.tar.gz
  sudo rm -rf latest.tar.gz
  sudo cp wordpress/wp-config-sample.php wordpress/wp-config.php
  sudo cp -R wordpress /var/www/html/  
  ```
- Configure SELinux Policies
  ```
  sudo chown -R apache:apache /var/www/html/wordpress

  sudo chcon -t httpd_sys_rw_content_t /var/www/html/wordpress -R

  sudo setsebool -P httpd_can_network_connect=1
  ```
### **Step 4 — Install MySQL on my DB Server**
---
    sudo yum update

    sudo yum install mysql-server`

Verify that mysql service is up and running 

 `sudo systemctl status mysqld` 
 
mysql service is not running, I'll restart the service and enable it.

 ![sql not run](https://user-images.githubusercontent.com/107736487/177608516-56dd36b7-4f78-418c-82f3-33e6531a4924.PNG) 

      sudo systemctl restart mysqld

      sudo systemctl enable mysqld

![sql running](https://user-images.githubusercontent.com/107736487/177608606-5f7aa5a5-783b-4d76-9d8b-8f7644a94d2c.PNG)

### **Step 5 — Configure DB to work with WordPress**
---
    sudo mysql
    CREATE DATABASE wordpress;
    CREATE USER `kris`@`<Web-Server-Private-IP-Address>` IDENTIFIED BY 'password';
    GRANT ALL ON wordpress.* TO 'kris'@'<Web-Server-Private-IP-Address>';
    FLUSH PRIVILEGES;
    SHOW DATABASES;
    SELECT USER FROM mysql.user;
    exit

![create db](https://user-images.githubusercontent.com/107736487/177608883-f014915b-ba46-477f-bf57-2caf94cdc7b9.PNG)


### **Step 6 — Configure WordPress to connect to remote database.**
- Edit inbound rule for my DB server

  I'll edit the inbound rule on my **DB server** and open MySQL port **3306**. I'll allow access to the **DB server** ONLY from my **Web Server’s** Private IP address.

  ![inbd rule](https://user-images.githubusercontent.com/107736487/177610074-30bbfaaa-d91e-4245-a61d-7cf36d34158e.PNG)

- Add bind-address on my DB server to enable me connect remotely

   `sudo vi /etc/my.cnf` 

   ![bind address](https://user-images.githubusercontent.com/107736487/177610204-1b747fed-64b3-4727-8443-d20363e9a024.PNG)

1. I'll Install **MySQL client** on my **web server** and test that i can connect from my **Web Server** to my **DB server** by using **mysql-client**

       sudo yum install mysql

       sudo mysql -u admin -p -h <DB-Server-Private-IP-address>

2. Verify if i can successfully execute SHOW DATABASES; command and see a list of existing databases.

    ![access db from webserver](https://user-images.githubusercontent.com/107736487/177610322-06189191-4f3c-4b5a-92cc-c950f7c674d6.PNG)

3. Change permissions and configuration so Apache could use WordPress:

4. Enable TCP port 80 in Inbound Rules configuration for my Web Server EC2 (enable from everywhere 0.0.0.0/0)

5. Using vi editor, i edit **wp-config.php** file to update my DB name, user name. password and DB host private IP address.

    - on web server goto:

       1. `cd /var/www/html/wordpress/wordpress`
       2. `sudo vi wp-config.php`

       ![wp conf edit](https://user-images.githubusercontent.com/107736487/177610675-52c514ae-76fb-4e12-bee8-9454964261e4.PNG)

6. Try to access from my browser the link to my WordPress using my **web server** public IP Address http://18.132.193.185/wordpress/  

- ## RESULT
---

![on browser](https://user-images.githubusercontent.com/107736487/177610933-53e9f0eb-425c-40b7-9940-d210f5ed6f8c.PNG)
![on browser2](https://user-images.githubusercontent.com/107736487/177611072-6837bac2-5613-42c3-9474-b406c3c15bf5.PNG)
![wordpress final1](https://user-images.githubusercontent.com/107736487/177611965-aa9b743a-0de8-4f7c-b6a0-fd3f19943123.PNG)
![wordpress dashboard](https://user-images.githubusercontent.com/107736487/177612555-7f04c4a3-934f-4edd-91a4-708fde05b833.PNG)

## End Of Project...
---

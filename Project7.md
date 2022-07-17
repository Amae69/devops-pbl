## PROJECT 7: DEVOPS TOOLING WEBSITE SOLUTION
---
### **Step 1 – Prepare NFS Server**
---
- Launch an EC2 RHEL 8 instance that will serve as my **"NFS Server"**. And create 2 volumes in the same AZ as my NFS Server, each of **8** GiB

- Using **lsblk** command to inspect the block devices that are attached to my NFS server
    
  `lsblk`

  ![lsblk](https://user-images.githubusercontent.com/107736487/178465388-ec1fb3a4-fd1f-47af-9a8c-9857a3a521d6.PNG)

- Using **gdisk** utility to create a single gpt partition on each of the 2 disks xvdb, xvdc

  `sudo gdisk /dev/xvdb`  

   ![lsblk 2](https://user-images.githubusercontent.com/107736487/178465558-34a21f4e-7fc9-4a44-971d-46f42461e076.PNG)

- Install lvm2 package

  `sudo yum install lvm2`

- Check for available partitions.

  `sudo lvmdiskscan` 

  ![lvmdiskscan](https://user-images.githubusercontent.com/107736487/178465689-be5c6e25-a7bc-4038-8ca5-a9089feb3e41.PNG)

- Using **pvcreate** utility i'll mark each of my 2 disks as physical volumes to be used by LVM

  `sudo pvcreate /dev/xvdb1 /dev/xvdc1`     

- Verify that my Physical volume has been created successfully.

  `sudo pvs`  

  ![lv confirm](https://user-images.githubusercontent.com/107736487/178465787-0ebe98c5-d99c-4d2d-b85f-c34fa0747ce6.PNG)

- Using **vgcreate** utility i'll add all 2 PVs to a volume group (VG). Named **nfs-vg**

  `sudo vgcreate nfs-vg /dev/xvdb1 /dev/xvdc1` 

- view my volume group  

  `sudo vgdisplay`

  ![vgdisplay](https://user-images.githubusercontent.com/107736487/178465924-3af291b6-7b2c-4b4b-86b8-f5aeb212892f.PNG)

- Using **lvcreate** utility to create 3 logical volumes. **lv-opt** , **lv-apps** and **lv-logs** and allocate **5 GB** each to the lv out of the **16 GB** volumes 

      sudo lvcreate -n lv-apps -L 5G nfs-vg

      sudo lvcreate -n lv-opt -L 5G nfs-vg 

      sudo lvcreate -n lv-logs -L 5G nfs-vg  

- Verify that my Logical Volume has been created successfully.

  `sudo lvs`       

  ![verify lvs](https://user-images.githubusercontent.com/107736487/178466177-581b803a-1b7d-459b-a7e7-3c545405534d.PNG)

- Use **mkfs.xfs** to format the logical volumes with **xfs** filesystem

      sudo mkfs -t xfs /dev/nfs-vg/lv-apps
      sudo mkfs -t xfs /dev/nfs-vg/lv-logs
      sudo mkfs -t xfs /dev/nfs-vg/lv-opt    

- Create mount points on **/mnt** directory for the logical volumes as follow:

  Mount **lv-apps** on **/mnt/apps** – To be used by webservers
  Mount **lv-logs** on **/mnt/logs** – To be used by webserver logs
  Mount **lv-opt** on **/mnt/opt** – To be used by Jenkins server    

  `sudo mkdir /mnt/apps /mnt/logs /mnt/opt` 

  ![mnt](https://user-images.githubusercontent.com/107736487/178466478-128096bb-54d5-44f2-bf77-6b62e27198bf.PNG)

  `sudo mount /dev/nfs-vg/lv-apps /mnt/apps`

  `sudo mount /dev/nfs-vg/lv-logs /mnt/logs`

  `sudo mount /dev/nfs-vg/lv-opt /mnt/opt`

- Update **/etc/fstab** file so that the mount configuration will persist after restart of the server

  i. run: `sudo blkid /dev/nfs-vg/*`

  ii Copy the UUID of the device which i'll be updating in the **/etc/fstab** file

  iii. `sudo vi /etc/fstab`

  iv. Update **/etc/fstab** by pasting the UUID copied above.  

  ![uuid](https://user-images.githubusercontent.com/107736487/178466574-1112cd76-b336-4c4d-b1f7-19546e01ef52.PNG)

- Test the configuration and reload the daemon

  `sudo mount -a`

  `sudo systemctl daemon-reload`

- Verify my setup by running: `df -h`  

  ![df -h](https://user-images.githubusercontent.com/107736487/178466704-5020aeb6-2255-4e11-8f83-d992b3e50f22.PNG)

- Install **NFS server**, configure it to start on reboot and make sure it is up and running

      sudo yum -y update
      sudo yum install nfs-utils -y
      sudo systemctl start nfs-server.service
      sudo systemctl enable nfs-server.service
      sudo systemctl status nfs-server.service  

    ![nfs status](https://user-images.githubusercontent.com/107736487/178466852-967337f2-40ce-47fd-a9bd-c134b52abce6.PNG)

- Set up permission that will allow my **Web servers** to read, write and execute files on **NFS**

      sudo chown -R nobody: /mnt/apps
      sudo chown -R nobody: /mnt/logs
      sudo chown -R nobody: /mnt/opt

      sudo chmod -R 777 /mnt/apps
      sudo chmod -R 777 /mnt/logs
      sudo chmod -R 777 /mnt/opt

      sudo systemctl restart nfs-server.service  

- Export the mounts for my webservers’ subnet cidr **(172.31.32.0/20)** to connect as clients.  

- Configure access to **NFS** for clients within the same subnet. using vi editor,input my subnet IPv4 cidr.

  `sudo vi /etc/exports`

      /mnt/apps <Subnet-CIDR>(rw,sync,no_all_squash,no_root_squash)
      /mnt/logs <Subnet-CIDR>(rw,sync,no_all_squash,no_root_squash)
      /mnt/opt <Subnet-CIDR>(rw,sync,no_all_squash,no_root_squash)

   ![subnet cidr](https://user-images.githubusercontent.com/107736487/178467054-b0326fc7-3997-4975-9ca9-826d9388ed7d.PNG)

  `sudo exportfs -arv`

  ![export](https://user-images.githubusercontent.com/107736487/178467132-191128ae-7451-45d8-bc64-ff09cfc73485.PNG)

- I'll check which port is used by **NFS** and open it by editing the inbound rule in my **NFS Server** Security Groups.

- **Note :** In order for my **NFS Server** to be accessible from my **client**, i must also open the following ports: **TCP 111, UDP 111, UDP 2049**

  `rpcinfo -p | grep nfs`  

  ![rpc info](https://user-images.githubusercontent.com/107736487/178467244-7913f5c3-cff2-4fbe-8a4f-7518628d88ec.PNG)
  ![inbound rule 2](https://user-images.githubusercontent.com/107736487/178467367-1c757301-e8c3-4bb2-ad4b-a136ffab5f9e.PNG)

### **STEP 2 — CONFIGURE THE DATABASE SERVER** 
---
- Launch an Ubuntu linux ec2 instance and Install **MySQL server**

  `sudo apt install mysql-server`

- Create a database and name it **tooling**

- Create a database user and name it **webaccess**

- Grant permission to webaccess user on tooling 
database to do anything only from the webservers subnet cidr 

      sudo mysql

      CREATE DATABASE tooling;
      CREATE USER 'webaccess'@'%' Identified_with_mysql_Native_password BY 'password';
      GRANT ALL ON tooling.* TO 'webaccess'@'%';
      FLUSH PRIVILEGES;

      SHOW DATABASES;
      SELECT USER FROM mysql.user;

      exit

   ![create database](https://user-images.githubusercontent.com/107736487/178467696-c5bfa98f-0717-4134-b611-61cee6db7059.PNG)

- Edit the bind address

  `sudo vi /etc/mysql/mysql.conf.d/mysqld.cnf`

- Restart mysql

  `sudo systemctl restart mysql`    

### **Step 3 — Prepare the Web Servers**
---
- Launch 2 new EC2 instance with RHEL 8 Operating System which will be my web servers and perform step 1-5 on each

1.  Install NFS client

    `sudo yum install nfs-utils nfs4-acl-tools -y`

2. Mount **/var/www/** and target the **NFS server’s** export for apps
      ```
      sudo mkdir /var/www

      sudo mount -t nfs -o rw,nosuid<NFS-Server-Private-IP-Address>:/mnt/apps /var/www      
      ```
3. Verify that NFS was mounted successfully by running `df -h`

   ![df -h wb sv](https://user-images.githubusercontent.com/107736487/178467923-c6571ccb-c314-46d4-9aa6-86634b6c085c.PNG)

4. Using vi editor,I'll add the line below on **/etc/fstab** to ensure that the changes will persist on **Web Server** after reboot.

       <NFS-Server-Private-IP-Address>:/mnt/apps /var/www nfs defaults 0 0:

    `sudo vi /etc/fstab`   

     ![fstab](https://user-images.githubusercontent.com/107736487/178468119-2ce9f898-4cd0-4781-b154-e37526c425e3.PNG)

5. Install **Remi’s repository**, **Apache** and **PHP**

   `sudo yum install httpd -y`

   `sudo dnf install https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm`

   `sudo dnf install dnf-utils http://rpms.remirepo.net/enterprise/remi-release-8.rpm`

   `sudo dnf module reset php`

   `sudo dnf module enable php:remi-7.4`

   `sudo dnf install php php-opcache php-gd php-curl php-mysqlnd`

   `sudo systemctl start php-fpm`

   `sudo systemctl enable php-fpm`

   `setsebool -P httpd_execmem 1`   

6. Verify that Apache files and directories are available on the **Web Server** in **/var/www** and also on the **NFS server** in **/mnt/apps**. If I see the same files – it means NFS is mounted correctly. 

- Apache files and directories on **web server** and **NfS server** respectively:

  ![Apache file on wbserver](https://user-images.githubusercontent.com/107736487/178468353-136a6e59-4eb6-475b-bc76-1774e554d741.PNG)
         
  ![Apache file on nfs](https://user-images.githubusercontent.com/107736487/178468442-eaad3715-4036-41bf-b5c0-d68ef7e51fcf.PNG)

- Furthermore, i created a new file `touch test.txt` from **web-server 1** and check if the **test.txt** file is present and accessible from other **Web-server 2** and **NFS-Server** 

  ![testfile wb-1](https://user-images.githubusercontent.com/107736487/178468569-07dd276c-5c06-415c-a3e8-eae16e780aac.PNG)
  ![testfile wb-2](https://user-images.githubusercontent.com/107736487/178468629-37665f03-1b71-4837-b510-1803c357c162.PNG)
  ![testfile nfs](https://user-images.githubusercontent.com/107736487/178468701-6c935121-c49e-4f9b-99b6-babcc23443f3.PNG)

- fork the tooling source code from Darey.io Github Account https://github.com/darey-io/tooling.git to my git hub account

  `sudo yum install git`

  `git clone https://github.com/darey-io/tooling.git`

  ![clone tooling](https://user-images.githubusercontent.com/107736487/178468805-94e8dd65-32ab-4296-94d6-9b09e75245cf.PNG)

- Deploy the tooling website’s code to the Webserver. Ensure that the html folder from the repository is deployed to /var/www/html 

- Go to cd **/var/www/html** directory

  - I'll move every file/folder in **tooling** folder which is present in the **/var/www/html** directory to the current directory **"html"**

    `sudo mv tooling/* .`

  - Since all the content in the **tooling** folder has been moved to the **html** directory, I'll delete the empty **tooling** folder in **html** folder  

    `sudo rm -r tooling`

     ![mv tooling](https://user-images.githubusercontent.com/107736487/178468983-30962992-365a-4a9b-b694-cbb915f7446e.PNG)
    
  - Also, I'll move every file/folder in **html** folder which is currently present in the **/var/www/html** directory to the current directory **"html"**

    `sudo mv html/* .`

  - Since all the content in the **html** folder has been moved to **var/www/html** directory, I'll delete the empty **html** folder in **var/www/html**  

    `sudo rm -r html`  

    ![mv html](https://user-images.githubusercontent.com/107736487/178469137-59d5fb49-b340-4a12-9151-9045b568169a.PNG)

- Install mysql on all my web servers

  `sudo yum install mysql`

- Using Vi editor, i'll update the website’s configuration to connect to my database (in /var/www/html/functions.php file) by inputting my Db private IP, DB user name, DB password and DB name.

  `sudo vi funtions.php`

- Apply tooling-db.sql script to my database using this command below
    
      mysql -h <databse-private-ip> -u <db-username> -p <db-pasword> < tooling-db.sql 

  `sudo mysql -h 172.31.36.58 -u webaccess -ppassword  tooling < tooling-db.sql`

  ` show databases;`

  `select * users;`

- Disable SELinux in all webserver 

  `sudo setenforce 0`

- To make this change permanent – open following config file in all webserver

   `sudo vi /etc/sysconfig/selinux` 
   
   and set SELINUX=disabled`. then restrt **httpd**.

    `sudo systemctl restart httpd`

- Open the website in my browser 
   
      http://<Web-Server-Public-IP-Address-or-Public-DNS-Name>/index.php 
      
  ensure i can login into the website.

  ![login 0](https://user-images.githubusercontent.com/107736487/178470627-ea5b7591-a4ff-4823-a090-b85ac319dcae.PNG)

  ![login 01](https://user-images.githubusercontent.com/107736487/178470690-e1c8fc94-f63b-49ba-9750-216ecbc7b038.PNG)

  ![login 1](https://user-images.githubusercontent.com/107736487/178470832-e425dce0-6d34-4d68-97ed-2732427411a6.PNG)
  
 ### **End of Project ...** 
 ---           

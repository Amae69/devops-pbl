## PROJECT 10: Load Balancer Solution With Nginx and SSL/TLS
---
**Task :**

This project consists of two parts:

1. Configure **Nginx** as a Load Balancer

2. Register a new **domain name** and configure secured connection using **SSL/TLS** certificates.

**Project architecture :**
---

![](./Images/images10/Prj10%20Arch.PNG)

**CONFIGURE NGINX AS A LOAD BALANCER**

- Create an EC2 VM based on **Ubuntu Server** 20.04 LTS and name it **Nginx LB** and open TCP port **80** for **HTTP** connections, also TCP port **443** for secured **HTTPS** connections

  ![](./Images/images10/instances.PNG)

  ![](./Images/images10/inbd%20rule.PNG)

- Update **/etc/hosts** file for local DNS with **Web Servers’** names (e.g. **Web1** and **Web2**) and their local **IP addresses**
  ```
  #Open this file on my Nginx-LB server

  sudo vi /etc/hosts

  #Add 2 records into this file with Local IP address and arbitrary name for both of my Web Servers

  <WebServer1-Private-IP-Address> Web1
  <WebServer1-Private-IP-Address> Web1
  ```

  ![](./Images/images10/etc%20host%201.PNG)

 
- Install and configure **Nginx** as a **load balancer** to point traffic to the resolvable DNS names of the **webservers**
 
  `sudo apt update`

  `sudo apt install nginx`

- I'll configure **Nginx LB** using the Web Servers’ names **(web1&web2)** I defined in **/etc/hosts**

  Open the default nginx configuration file

  `sudo vi /etc/nginx/nginx.conf`
  ```
  #insert following configuration into http section

  upstream myproject {
     server Web1 weight=5;
     server Web2 weight=5;
   }

  server {
      listen 80;
      server_name www.domain.com;
      location / {
        proxy_pass http://myproject;
      }
    }

  #comment out this line
  #       include /etc/nginx/sites-enabled/*;
  ```
  ![](./Images/images10/nginx%20config%202.PNG)

- Restart Nginx and make sure the service is up and running

  `sudo systemctl restart nginx`

  `sudo systemctl status nginx`  

### REGISTER A NEW DOMAIN NAME AND CONFIGURE SECURED CONNECTION USING **SSL/TLS** CERTIFICATES
---

- In order to get a valid **SSL** certificate, i'll need to register a new **domain name**

1. I got a free domain **"krisqriz.ga"** from **freenom** 

   ![](./Images/images10/freenom%20domain.PNG)

2. Using **Route 53** service in aws, i created a hosted zone and configure my domain name 

   ![](./Images/images10/hosted%20zone%201.PNG)

3. Create two **A** record **"krisqriz.ga"** and **"www.krisqriz.ga"** and input the **public IP** of my **Nginx-LB** server  

   ![](./Images/images10/creating%20A%20record.PNG)

**NOTE:** For the aws **Route 53** hosted zone to be connected to my free Domain name gotten from a 3rd party **"freenom"**, i need to copy the name server from the hosted zone and input it to the name server in **freenom management tool**

![](./Images/images10/name%20server%2053.PNG)

4. Freenom > My Domain > Manage Domain > management tools > Name Server > Use custom nameserver

   ![](./Images/images10/nameserver%20freenom.PNG)

- Check the status of my web services in my both web server to ensure they are still active if not, restart web service 

  `sudo systemctl restart httpd`

  `sudo systemctl status httpd`

- Check that my Web Servers can be reached from my browser using new domain name using HTTP protocol – http://krisqriz.ga

  ![](./Images/images10/login%20krisqriz.PNG)

### TO CONFIGURE SECURED CONNECTION USING SSL/TLS CERTIFICATES 
--- 

- Install **certbot** and request for an **SSL/TLS** certificate.

- Make sure **snapd** service is active and running

  `sudo systemctl status snapd`

   ![](./Images/images10/snap%20status.PNG)

- Install certbot

  `sudo snap install --classic certbot`

- Using the code below, I'll request my certificate and follow the instructions.

  `sudo ln -s /snap/bin/certbot /usr/bin/certbot`

  `sudo certbot --nginx` 

  ![](./Images/images10/certificate%20conf.PNG)

- I'll test secured access to my Web Solution by trying to reach https://krisqriz.ga  

  ![](./Images/images10/login%20https.PNG)

  ![](./Images/images10/login%20cert.PNG)

- I'll Set up periodical renewal of my **SSL/TLS** certificate

  By default, **LetsEncrypt** certificate is valid for **90 days**, so it is recommended to renew it at least every **60 days** or more frequently.

  I'll test renewal command in dry-run mode

  `sudo certbot renew --dry-run`

  ![](./Images/images10/dry%20run.PNG)

- Best pracice is to have a scheduled job that to run renew command periodically. I'll configure a **cronjob** to run the command twice a day.

  To do so, i'll edit the crontab file with the following command:

  `crontab -e`

  Add following line:

  `* */12 * * *   root /usr/bin/certbot renew > /dev/null 2>&1`

  I can always change the interval of this **cronjob** if twice a day is too often by adjusting schedule expression.  

### END OF PROJECT....    


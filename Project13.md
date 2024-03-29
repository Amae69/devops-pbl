## **ANSIBLE DYNAMIC ASSIGNMENTS (INCLUDE) AND COMMUNITY ROLES**
---
In this project i will be making use of **dynamic assignments** by using **include** module.

 **Static assignments** uses **import** Ansible module. While **Dynamic assignments** uses **include** Ansible module

### Difference between **static-assignment** and **dynamic-assignment**
---
When the **import module** is used, all statements are **pre-processed** at the time playbooks are **parsed**. Meaning, when you execute **site.yml playbook**, Ansible will process all the playbooks referenced during the time it is **parsing** the statements. This also means that, during actual **execution**, if any statement **changes**, such statements will not be considered. Hence, it is **static**.

On the other hand, when **include module** is used, all statements are **processed** only during **execution** of the **playbook**. Meaning, after the statements are parsed, any **changes** to the statements encountered during execution will be used.

### **INTRODUCING DYNAMIC ASSIGNMENT INTO MY STRUCTURE**
---
- In my `https://github.com/<your-name>/ansible-config-mgt` GitHub repository, I'll start a new **branch** and call it **dynamic-assignments**.

  `git checkout -b dynamic-assignments`

  ![](./Images/images13/new%20branch.PNG)

I'll pull down code from **main** branch to my new branch **(dynamic-assignments)**

 On **dynamic-assignments** branch run: 

 `git pull origin main`

 `git push --set-upstream origin dynamic-assignments`

- I'll create a new folder, and named it **dynamic-assignments**. Then inside this folder, i'll create a new file and name it **env-vars.yml**. I will instruct **site.yml** to include this **playbook** later. For now, I'll keep building up the structure.

   `mkdir dynamic-assignments`

   `cd dynamic-assignments`

   `touch env-vars.yml`

  ![](./Images/images13/nw%20folder%26file.PNG)

- Since I'll be using the same **Ansible** to configure **multiple environments**, and each of these environments will have certain **unique attributes**, such as **servername**, **ip-address** etc., i'll need a way to set **values** to **variables** per specific **environment**.

- For this reason, I'll now create a folder to keep each **environment’s variables** file. Therefore, create a new folder **env-vars**, then for each environment, create new **YAML files** which we will use to set **variables**.  

  `mkdir env-vars`

  `touch dev.yml stage.yml uat.yml prod.yml`

  ![](./Images/images13/env_var%20fold.PNG)

- Now I'll paste the instruction below into the **env-vars.yml** file.  
  ```
  ---
  - name: collate variables from env specific file, if it exists
    hosts: all
    tasks:
      - name: looping through list of available files
        include_vars: "{{ item }}"
        with_first_found:
          - files:
              - dev.yml
              - stage.yml
              - prod.yml
              - uat.yml
            paths:
              - "{{ playbook_dir }}/../env-vars"
        tags:
          - always
  ```
  `sudo vi dynamic-assignments/env-vars.yml`

  ![](./Images/images13/past%20code%20to%20env-vars.yml)

### **UPDATE SITE.YML WITH DYNAMIC ASSIGNMENTS**  
---  
I'll update **site.yml** file to make use of the **dynamic assignment**. 
```
---
- hosts: all
- name: Include dynamic variables 
  tasks:
  import_playbook: ../static-assignments/common.yml 
  include: ../dynamic-assignments/env-vars.yml
  tags:
    - always

-  hosts: webservers
- name: Webserver assignment
  import_playbook: ../static-assignments/webservers.yml
```
`sudo vi playbooks/site.yml`  

![](./Images/images13/update%20siteyml.PNG)

### **Community Roles/Download Mysql Ansible Role**
---
Now it is time to create a **role** for **MySQL database** – it should install the **MySQL package**, create a **database** and **configure users**.

I can browse available **community roles** here: https://galaxy.ansible.com/home

However, I'll be using a **MySQL role** developed by **geerlingguy** https://galaxy.ansible.com/geerlingguy/mysql

Inside **role** directory I'll download **MySQL** role using:

`sudo ansible-galaxy install geerlingguy.mysql` 

![](./Images/images13/sql%20role.PNG)

I'll rename the folder to **mysql**

`mv geerlingguy.mysql/ mysql`

![](./Images/images13/rename%20role.PNG)

### **Edit mysql role to work with my desired configuration setup.**
---

- I'll Read the **`README.md` file,** to know how to edit **roles configuration** to use correct **credentials** for **MySQL** required for the **tooling** website.

- I'll go to `defaults/main.yml` in **mysql role** and edit **database (tooling)**, **user (webaccess)** and **host**

  ![](./Images/images13/update%20mysql%20role.PNG)

  Now it is time to upload the changes into my GitHub:

  `git add .`

  `git commit -m 'update'`

  `git push origin dynamic-assignmnets`

  Now i'll create a **Pull** Request and merge it to **main** branch on GitHub

### **LOAD BALANCER ROLES**
---
I want to be able to choose which **Load Balancer** to use, **Nginx** or **Apache**, so i'll need to have two **roles** Nginx and Apache respectively:

Inside **role** directory I'll download **nginx** and **Apache** role using:

`sudo ansible-galaxy install geerlingguy.nginx` 

`sudo ansible-galaxy install geerlingguy.apache` 

![](./Images/images13/install%20nginx%20and%20apache%20role.PNG)

I'll rename the folder to **nginx** and **apache** respectively

`sudo mv geerlingguy.nginx/ nginx`

`sudo mv geerlingguy.apache/ apache`

![](./Images/images13/rename%20nginx%20and%20apache%20role.PNG)

### **Edit the Nginx role to work with my desired configuration**
---

I'll Read the **`README.md` file,** to know how to edit **Nginx roles configuration**.

- I'll go to `defaults/main.yml` in **Nginx role** and input my uat host: 

  **nginx role > defaults > main.yml**

   ![](./Images/images13/update%20nginx%20role%201.1.PNG)

   ![](./Images/images13/update%20nginx%20role%202%20uncomment.PNG)

- **update nginx role > task > main.yml**

  ![](./Images/images13/update%20nginx%20role%20taskmainyml%201.PNG)

  ![](./Images/images13/updt%20nginx%20role%202.PNG)

  ![](./Images/images13/updt%20ngnx%20task%20mainyml.PNG)

- **update nginx role > task > Setup-Redhat.yml**  

   ![](./Images/images13/nginx%20role%20task%20setup%20redhat.PNG)

### **Edit the Apache role to work with my desired configuration**
---
I'll Read the **`README.md` file,** to know how to edit **Apache roles configuration**

- I'll go to **defaults > main.yml** in **Apache role** and input the following:
  ```
  # webservers
  loadbalancer_name: "myapp1"
  web1: "webserver Private-public-IP"
  web2: "webserver Private-public-IP"
  ```
  ![](./Images/images13/role%20apache%20defaults%20mainyml%203.PNG)

  ![](./Images/images13/role%20apache%20defaults%20mainyml%201.PNG)

- **update Apache role > task > Setup-Redhat.yml** 

  ![](./Images/images13/apache%20role%20task%20setup%20redhat.PNG)


**Important Hints:**

Since i cannot use both **Nginx** and **Apache** load balancer, I'll need to add a **condition** to enable either one – this is where i can make use of **variables**.

- Declare a variable in **defaults/main.yml** file inside the **Nginx** and **Apache** roles. Name each variables **enable_nginx_lb** and **enable_apache_lb** respectively.

- Set both values to false like this: `enable_nginx_lb: false` and `enable_apache_lb: false`.

- Declare another variable in both roles **load_balancer_is_required** and set its value to **false** as well

  ![](./Images/images13/var%20dec%20in%20nginx.PNG)

  ![](./Images/images13/var%20dec%20in%20apache.PNG)

- Update both assignment and site.yml files respectively

  firstly i will create a file named **loadbalancers.yml** in **static assignment** folder.

  Paste the code below in **loadbalancers.yml** file
  ```
  --- 
  - hosts: lb
    roles:
      - { role: nginx, when: enable_nginx_lb and load_balancer_is_required }
      - { role: apache, when: enable_apache_lb and load_balancer_is_required }
  ```
  ![](./Images/images13/updt%20loadbalanceryml%20file.PNG)

  Paste the code below in **site.yml file**  
  ```
  - name: Loadbalancers assignment
    hosts: lb
      - import_playbook: ../static-assignments/loadbalancers.yml
        when: load_balancer_is_required
  ```      
   ![](./Images/images13/updt%20siteyml%20lb.PNG)

- Now i'll make use of **env-vars\uat.yml** file to define which **loadbalancer** to use in **UAT** environment by setting respective environmental variable to **true**.

  I will activate **load balancer**, and enable **nginx** by setting these in the respective environment’s **env-vars** file.
  ```
  enable_nginx_lb: true
  load_balancer_is_required: true
  ```
  ![](./Images/images13/env-vars%20uatyml.PNG)

- Now i'll run the ansible playbook

  `ansible-playbook -i inventory/uat.yml playbooks/site.yml`

  ![](./Images/images13/ansible%20playbook.PNG)

### END OF PROJECT...
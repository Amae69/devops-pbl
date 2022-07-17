## Continous Integration Pipeline For Tooling Website
---
**Task**

I'll enhance the architecture prepared in **Project 8** (https://github.com/Amae69/devops-pbl/blob/d41ce9abb64ab44f07c89c07781261243e6821b3/Project8.md) by adding a **Jenkins server**, 

I'll configure a job to automatically deploy source codes changes from **Git** to **NFS server**.

**Project Architecture:**

 ![](./Images/images9/Proj-8%20Archi.PNG)

### Step 1 – Install Jenkins server
---
- I create an AWS EC2 server based on **Ubuntu Server 20.04 LTS** and name it **"Jenkins"**

- Install **JDK**

  `sudo apt update`

  `sudo apt install default-jdk-headless`

- Install Jenkins 
  ```
  wget -q -O - https://pkg.jenkins.io/debian-stable/jenkins.io.key | sudo apt-key add -
  sudo sh -c 'echo deb https://pkg.jenkins.io/debian-stable binary/ > \
    /etc/apt/sources.list.d/jenkins.list'
  
  sudo apt update
  
  sudo apt-get install jenkins
  ```







## **MIGRATION TO THE CLOUD WITH CONTAINERIZATION - DOCKER**
---

### **INTRODUCTION**
---
In this project, the frontend and the backend(MySQL) of tooling application is built and containerized using DOCKER of which its image is pushed to Docker registry. And further in the project, the php-todo application is also built into a container and pushed into the AWS Elastic Container Registry using a CI/CD tool known as Jenkins and Docker Compose is also implemented.

The following outlines the steps:

### **STEP 0: Install Docker and prepare for migration to the Cloud**

First, i need to install **Docker Engine**, which is a **client-server** application that contains:

- A server with a long-running daemon process dockerd.
- APIs that specify interfaces that programs can use to talk to and instruct the Docker daemon.
- A command-line interface (CLI) client docker.
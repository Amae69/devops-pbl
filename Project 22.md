## **DEPLOYING APPLICATIONS INTO KUBERNETES CLUSTER**
---
### **INTRODUCTION**

This project demonstrates how containerised applications are deployed as pods in Kubernetes and how to access the application from the browser.

### **Prerequisite :**
I ensure that i have [AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html) and [kubectl](https://docs.aws.amazon.com/eks/latest/userguide/install-kubectl.html) install and configure in a directory in my Local machine which i will need to create and manage an Amazon EKS cluster.

![](./Images/image22/confirm%20aws%20cli%20and%20kubctl.PNG)

### **Step 0: Create cluster using AWS EKS Doc Or [GitHub repo](https://github.com/weaveworks/eksctl)**
---

- Using this [Doc](https://docs.aws.amazon.com/eks/latest/userguide/service_IAM_role.html#create-service-role) create Amazon EKS cluster role

![](./Images/image22/cluster%20role%20create.PNG)

- Create EKS on Amazon EKS console Using [Amazon EKS Doc](https://docs.aws.amazon.com/eks/latest/userguide/getting-started-console.html)

![](./Images/image22/Eks%20cluster.PNG)

- I provision compute capacity for my cluster by adding a [managed node group](https://docs.aws.amazon.com/eks/latest/userguide/getting-started-console.html). Firstly i create a node IAM role and attach the required Amazon EKS IAM managed policy to it as described on doc above.

![](./Images/image22/doc%20on%20creating%20node%20group.PNG)

![](./Images/image22/node%20grp%201.PNG)

![](./Images/image22/node%20grp%202.PNG)

My node group was design to create two **ec2 instance** max to serve as my node

![](./Images/image22/cluster%20with%20node.PNG)

![](./Images/image22/Node%20ec2.PNG)

### **Step 1: Configure my computer to communicate with my newly created EKS cluster**
---
I'll Create or update a **kubeconfig** file for my cluster. By running the code below. The settings in this file enable the **kubectl CLI** installed in my Local PC to communicate with my cluster. 

Run: `aws eks update-kubeconfig --region region-code --name my-cluster`

Replace **region-code** with my **EKS cluster region**. Also, Replace **my-cluster** with the name of my cluster.

On my PC, Run: `kubectl get nodes` to confirm the configuration.

![](./Images/image22/update%20kubeconfig.PNG)

### **Step 2: Creating A Pod For The Nginx Application**
---
- I create a folder named **k8s-prj22** then CD into it. There i will create and write all my **manifest file**.

- Create a file named **nginx-pod.yaml** and write a **manifest code** shown below in it to create an **nginx pod**

```
---
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
  labels:
    app: nginx-pod
spec:
  containers:
    - image: nginx:latest
      name: nginx-pod
      ports:
        - containerPort: 80
          protocol: TCP
```          
Run: `kubectl apply -f nginx-pod.yaml` to create the nginx pod

Run: `kubectl get pods`

Run: `kubectl describe pod nginx-pod`

![](./Images/image22/create%20nginx%20pod.PNG)

### **Step 3: ACCESSING THE APP FROM THE BROWSER**
---
Now I have a running Pod with **Nginx** container, I'll need to access it from the browser. But all I have is a running Pod that has its own IP address which cannot be accessed through the browser. 

To achieve this, i'll need another Kubernetes object called **Service** to accept our request and pass it on to the Pod.

A **service** is an object that accepts requests on behalf of the Pods and forwards it to the Pod’s IP address. Run: `kubectl get pod nginx-pod  -o wide`,to see the Pod’s **IP address**. But there is no way to reach it directly from the outside world.

![](./Images/image22/pod%20ip%20address.PNG)

- Create a file named **nginx-service.yaml** copy and paste **manifest code** shown below into it create a **service object**
```
---
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  selector:
    app: nginx-pod
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
```      

- Creating service for the nginx pod by applying the manifest file. 

  Run: `kubectl apply -f nginx-service.yaml`

  Run: `kubectl get service nginx-service -o wide`

![](./Images/image22/create%20nginx%20service.PNG)

- Since the type of **service** created for the Nginx pod is a **ClusterIP** which cannot be accessed **externally**, we can do **port-forwarding** in order to bind the **machine's port** to the **ClusterIP service port**, i.e, tunnelling traffic through the machine's port number to the port number of the nginx-service: `kubectl port-forward svc/nginx-service 8089:80`

![](./Images/image22/port%20forwarding.PNG)

- Accessing the Nginx application from the browser: `http://localhost:8089`

![](./Images/image22/access%20app%20on%20browser.PNG)

- We can also access the **Nginx app** on browser using **NodePort** which is a type of **service** that exposes the service on a **static port** on the **node’s IP** address and they range from **30000-32767** by default.

- Editing the **nginx-service.yml** manifest file to expose the Nginx service in order to be accessible to the browser by adding **NodePort** as a type of **service:**

![](./Images/image22/nodeport%20service.PNG)

To access the service, i must:

- Allow the **inbound traffic** in my **EC2’s Security Group** to the NodePort range **30000-32767**

- Get the **public IP** address of the **node** the Pod is running on, append the **nodeport** and access the app through the browser.

- Accessing the **nginx** application from the browser with the value of the nodeport **30080** which is a port on the node in which the Pod is scheduled to run: http://3.8.122.85:30080/

![](./Images/image22/nodeport%20browser%20acc.PNG)

### **STEP 4: Creating A Replica Set**
---
- The **replicaSet** object helps to maintain a stable set of **Pod replicas** running at any given time to achieve availability in case one or two pods dies.

- Deleting the nginx-pod: `kubectl delete pod nginx-pod`

- Creating the replicaSet manifest file and applying it: `kubectl apply -f rs.yaml`

Replica set manifest file below named **rs.yaml**
```
---
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: nginx-rs
  labels:
    app: nginx-pod
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx-pod
  template:
    metadata:
      labels:
        app: nginx-pod
    spec:
      containers:
        - name: nginx-pod
          image: nginx:latext
          ports:
            - containerPort: 80
              protocol: TCP
```
- Inspecting the set up
```
kubectl get pods

kubectl get rs -o wide  
```        
![](./Images/image22/create%20replicaset.PNG)

**Note:** Deleting one of the **pods** will cause another one to be scheduled and set to run.

#### **Scale ReplicaSet up and down:**
- Two ways pods can be scaled are: **Imperative** and **Declarative**

- Imperative method is by running a command on the CLI: `kubectl scale rs nginx-rs --replicas=5`

![](./Images/image22/scale%20replicaset.PNG)

- Declarative method is done by editing the **rs.yaml manifest file** and changing to the desired number of replicas and applying the update.

### **USING AWS LOAD BALANCER TO ACCESS MY SERVICE IN KUBERNETES.**
---
Note: I'll only be able to test this using **AWS EKS**. In the next project, I'll update my **Terraform code** to build an **EKS cluster**.

I have previously accessed the **Nginx service** through **ClusterIP**, and **NodeIP**, but there is another service type – **Loadbalancer**. This type of service does not only create a Service object in K8s, but also provisions a real external Load Balancer (e.g. Elastic Load Balancer – ELB in AWS)

To get started I'll update my **service manifest file** and use the **LoadBalancer type**. Also, ensure that the selector references the **Pods** in the **replica set**.

```
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  type: LoadBalancer
  selector:
    tier: frontend
  ports:
    - protocol: TCP
      port: 80 # This is the port the Loadbalancer is listening at
      targetPort: 80 # This is the port the container is listening at
```
- Apply the configuration: `kubectl apply -f nginx-service.yaml`

- Get the newly created service: `kubectl get service nginx-service`      

![](./Images/image22/lb%20service.PNG)

Check: ELB resource will be created in my **AWS console**

![](./Images/image22/lb%20console.PNG)

**NOTE:** A Kubernetes component in the control plane called **Cloud-controller-manager** is responsible for triggering this action. It connects to your specific cloud provider’s (AWS) APIs and create resources such as Load balancers.

- Get the output of the entire **yaml** for the **service**. You will some additional information about this service in which you did not define them in the **yaml manifest**. Kubernetes did this for me.

- Run: `kubectl get service nginx-service -o yaml`

Output will look like this below.

```
apiVersion: v1
kind: Service
metadata:
  annotations:
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"v1","kind":"Service","metadata":{"annotations":{},"name":"nginx-service","namespace":"default"},"spec":{"ports":[{"port":80,"protocol":"TCP","targetPort":80}],"selector":{"app":"nginx-pod"},"type":"LoadBalancer"}}
  creationTimestamp: "2021-06-18T16:24:21Z"
  finalizers:
  - service.kubernetes.io/load-balancer-cleanup
  name: nginx-service
  namespace: default
  resourceVersion: "21824260"
  selfLink: /api/v1/namespaces/default/services/nginx-service
  uid: c12145d6-a8b5-491d-95ff-8e2c6296b46c
spec:
  clusterIP: 10.100.153.44
  externalTrafficPolicy: Cluster
  ports:
  - nodePort: 31388
    port: 80
    protocol: TCP
    targetPort: 80
  selector:
    tier: frontend
  sessionAffinity: None
  type: LoadBalancer
status:
  loadBalancer:
    ingress:
    - hostname: ac12145d6a8b5491d95ff8e2c6296b46-588706163.eu-central-1.elb.amazonaws.com
```    

![](./Images/image22/output%20of%20yaml%20lb.PNG)

1. A **clusterIP** key is updated in the manifest and assigned an **IP address**. Even though i have specified a **Loadbalancer** service type, internally it still requires a **clusterIP** to route the external traffic through.

2. In the ports section, **nodePort** is still used. This is because Kubernetes still needs to use a dedicated port on the **worker node** to route the traffic through. Ensure that port range 30000-32767 is opened in my inbound Security Group configuration.

To access the Nginx service, Copy and paste the **load balancer’s** dns to the browser.

![](./Images/image22/lb%20dns%20on%20brows.PNG)

### **STEP 4: Creating Deployment**
---

A **Deployment** is another layer above **ReplicaSets** and **Pods**. It manages the deployment of **ReplicaSets** and allows for easy updating of a **ReplicaSet** as well as the ability to roll back to a previous version of **deployment**. It is declarative and can be used for rolling updates of **micro-services**, ensuring there is no downtime.

It is highly recommended to use **Deplyments** to manage **replica sets** rather than using **replica sets** directly.

To get started:

1. Delete the ReplicaSet: `kubectl delete rs nginx-rs`

2. Create a **deployment.yaml** manifest file
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    tier: frontend
spec:
  replicas: 3
  selector:
    matchLabels:
      tier: frontend
  template:
    metadata:
      labels:
        tier: frontend
    spec:
      containers:
      - name: nginx
        image: nginx:latest
        ports:
        - containerPort: 80
```
3. create the deployment: `kubectl apply -f deployment.yaml`

4. Inspecting the setup:
```
kubectl get deploy

kubectl get rs

kubectl get pod
```
![](./Images/image22/create%20deployment.PNG)

5. Exec into one of the Pod’s container to run Linux commands: `kubectl exec -it nginx-deployment-5c9bdf6884-znzfw bash`

List the files and folders in the Nginx directory `ls -ltr /etc/nginx/`

![](./Images/image22/exec%20into%20pod.PNG)

### **Self Side Task:**
---

- Build the Tooling app Dockerfile and push it to Dockerhub registry.

- Write a Pod and a Service manifests, ensure that you can access the Tooling app’s frontend using port-forwarding feature.

### **Task Implementation**
---
- I install [Docker Engine](https://docs.docker.com/engine/install/), on my PC



### **PERSISTING DATA FOR PODS**
---
Deployments are **stateless** by design. Hence, any data stored inside the **Pod’s container** does not **persist** when the Pod dies.

If you were to update the content of the **index.html** file inside the **container**, and the Pod dies, that content will not be lost since a new Pod will replace the dead one.

Let us try that:

1. Scale the Pods down to 1 replica : `kubectl scale deploy nginx-deployment --replicas=1`

![](./Images/image22/scale%20deployment.PNG)

2. Exec into the running container : `kubectl exec -it nginx-deployment-5c9bdf6884-4zlh8 bash`

3. Install **vim** so that i can edit the file
```
apt-get update
apt-get install vim
```
4. Update the content of the file and add the code below `/usr/share/nginx/html/index.html`
```
<!DOCTYPE html>
<html>
<head>
<title>Welcome to DAREY.IO!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to DAREY.IO!</h1>
<p>I love experiencing Kubernetes</p>

<p>Learning by doing is absolutely the best strategy at 
<a href="https://darey.io/">www.darey.io</a>.<br/>
for skills acquisition
<a href="https://darey.io/">www.darey.io</a>.</p>

<p><em>Thank you for learning from DAREY.IO</em></p>
</body>
</html>
```
Run : `vim /usr/share/nginx/html/index.html` and input code above

Check the browser: 

![]()



## **PERSISTING DATA IN KUBERNETES**
---
### **INTRODUCTION**

The pods created in Kubernetes are **ephemeral**, they don't run for long. When a pod dies, any data that is not part of the container image will be lost when the container is restarted because Kubernetes is best at managing stateless applications which means it does not manage data persistence. To ensure data persistent, the PersistentVolume resource is implemented to acheive this.

The following outlines the steps:

### **STEP 1: Setting Up AWS Elastic Kubernetes Service With [EKSCTL](https://docs.aws.amazon.com/eks/latest/userguide/eksctl.html)**

- Installing or updating **eksctl** on my Local **window PC** using this installation on [github](https://github.com/weaveworks/eksctl/blob/main/README.md#installation)

- Using Gitbash:
```
# for ARM systems, set ARCH to: `arm64`, `armv6` or `armv7`
ARCH=amd64
PLATFORM=windows_$ARCH

curl -sLO "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$PLATFORM.zip"

# (Optional) Verify checksum
curl -sL "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_checksums.txt" | grep $PLATFORM | sha256sum --check

unzip eksctl_$PLATFORM.zip -d $HOME/bin

rm eksctl_$PLATFORM.zip
```

![](./Images/image23/install%20eksctl.PNG)

![](./Images/image23/ls%20bin.PNG)

- Testing the installation was successful with the following command:`$ eksctl version`

![](./Images/image23/eksctl%20version.PNG)

### **Creating EKS Cluster using [eksctl](https://docs.aws.amazon.com/eks/latest/userguide/getting-started-eksctl.html)**
---

- Setting up EKS cluster with a single commandline:
```
eksctl create cluster \
  --name kz-prj23 \
  --version 1.24 \
  --region eu-west-2 \
  --nodegroup-name kz-worker-nodes \
  --node-type t3.medium \
  --nodes 2
```  
![](./Images/image23/create%20cluster%20using%20eksctl.PNG)

![](./Images/image23/cluster%20on%20console.PNG)

### **STEP 2: Creating Persistent Volume Manually For The Nginx Application**
---
- Creating a deployment manifest file for the Nginx application and applying it:

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
`kubectl apply -f deployment.yaml`

`kubectl get deploy nginx-deployment`

![](./Images/image23/create%20deployment.PNG)

- Verifying that the pod is running:  `kubectl get pod`

- Exec into the pod and navigating to the nginx configuration file:  `kubectl exec <pod name> -it bash`

![](./Images/image23/exec%20into%20pod.PNG)

![](./Images/image23/exec%20into%20pod%202.PNG)

- When creating a **volume** it must exists in the same **region** and **availability zone** as the EC2 instance **(worker Node)** running the **pod**. To confirm which node is running the pod: `kubectl get po nginx-deployment-5d6cf97577-2rgmt -o wide`

![](./Images/image23/get%20pod%20owide.PNG)

**NOTE:** There are some restrictions when using an **awsElasticBlockStore** volume:

- The nodes on which pods are running must be AWS EC2 instances

- Those instances need to be in the same region and availability zone as the EBS volume.

- EBS only supports a single EC2 instance mounting a volume.

To check the **AZ** where the **node** is running: `kubectl describe node ip-192-168-81-220.eu-west-2.compute.internal`

![](./Images/image23/describe%20node.PNG)

- Creating a **volume** in the **Elastic Block Storage** section in **AWS** in the same **AZ** as the **node** running the **nginx pod** which will be used to mount volume into the Nginx pod.

![](./Images/image23/create%20volume%20from%20console.PNG)

- Copy volume ID: 

![](./Images/image23/volume%20id.PNG)

- Then update the deployment configuration with the volume spec and volume mount:

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    tier: frontend
spec:
  replicas: 1
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
        volumeMounts:
        - name: nginx-volume
          mountPath: /usr/share/nginx/
      volumes:
      - name: nginx-volume
        awsElasticBlockStore:
          volumeID: "vol-01631231d20e3b67c"
          fsType: ext4
```          
Apply the new configuration and check the pod. As you can see, the old pod is being terminated while the updated one is up and running: `kubectl apply -f deployment.yaml`
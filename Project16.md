## **AUTOMATING INFRASTRUCTURE WITH IAC USING TERRAFORM PART 1**
---

INTRODUCTION
---
This project demonstrates how the AWS infrastructure for 2 websites that was built manually in project 15 is automated with the use of Terraform.

- Infrastructure Architectural Diagram:

  ![](./Images/images16/tooling_project_15.png)

The following outlines the steps taken:

### **STEP 0: Setting Up AWS CLI And S3 Buckets**
---
After creating an IAM user with AdministrativeAccess permissions in AWS and acquiring the access key and secret access key, 

![](./Images/images16/Iam%20user.PNG)

![](./Images/images16/iam%20user%20policy.PNG)

the following step was taken:

- Creating S3 bucket in AWS for storing Terraform state file and naming it **kris-dev-terraform-bucket**

![](./Images/images16/s3%20bucket%201.PNG)

![](./Images/images16/s3%20bucket%202.PNG)

- Find Doc: [Boto3](https://boto3.amazonaws.com/v1/documentation/api/latest/guide/quickstart.html)

- Running the following command on my terminal to install **boto3 (AWS SDK for Python)** including the AWS Common Runtime (CRT) which boto3 uses to incorporate features not otherwise available in the AWS SDK for python: 

  `pip install boto3[crt]`

  ![](./Images/images16/install%20boto3.PNG)

- Downloading and running the AWS CLI MSI installer for Windows from the terminal: `C:\> msiexec.exe /i https://awscli.amazonaws.com/AWSCLIV2.msi`  

    see: [AWS CLI documentation](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html)

- To confirm installation Start menu > cmd. Then type `aws --version`

  ![](./Images/images16/aws%20version.PNG)

- Configuring access file with the **Access key** and **secret access key**: `aws configure`

  ![](./Images/images16/aws%20configure.PNG)

After configuring authentication and installed **boto3**, I'll ensure i can programmatically access my **AWS account** by running following commands in **>python:**  

- Testing AWS CLI by running the following commands on my terminal which prints my newly created buckets:
  ```
  import boto3
  s3 = boto3.resource('s3')
  for bucket in s3.buckets.all():
      print(bucket.name)
  ```    
  ![](./Images/images16/test%20aws%20cli.PNG)

- Using powershell admin mode, I Install [Terraform](https://docs.chocolatey.org/en-us/choco/setup#:~:text=First%2C%20ensure%20that%20you%20are,for%20the%20command%20to%20complete.) on my windows machine using [Chocolatey](https://docs.chocolatey.org/en-us/choco/setup#:~:text=First%2C%20ensure%20that%20you%20are,for%20the%20command%20to%20complete.) package management system.

  `choco install terraform`

  ![](./Images/images16/inst%20terraform.PNG)

- Confirm terraform has been installed successfully

  On powershell run: `terraform help`

  ![](./Images/images16/confirm%20terra.PNG)

### **STEP 1: Creating VPC Resource**
---
- Creating a folder called **PBL**
- Creating a file in the PBL folder and naming it **main.tf**
- Entering the following configuration which adds **AWS** as a **provider** and a resource to create a **VPC** in the **main.tf** file:
  ```
  provider "aws" {
  region = "eu-west-2"
  }

  # Create VPC
  resource "aws_vpc" "main" {
  cidr_block                     = "172.16.0.0/16"
  enable_dns_support             = "true"
  enable_dns_hostnames           = "true"
  enable_classiclink             = "false"
  enable_classiclink_dns_support = "false"
  }
  ```
- Running the following command which downloads the necessary plugins for Terraform to work:

  `terraform init`

  ![](./Images/images16/terraform%20init.PNG)

  **Note** A new directory has been created: **.terraform\....** This is where Terraform keeps **plugins.** Generally, it is safe to delete this folder. It just means that you must execute terraform init again, to download them.

- Check to see what **terraform** intends to create before we tell it to go ahead and create the **aws_vpc** resource the following command is run: 

  `terraform plan`  

  ![](./Images/images16/teraform%20plan.PNG)

 - Proceed to execute the plan: 
 
   `terraform apply`

   ![](./Images/images16/terraform%20apply%201.PNG)
   ![](./Images/images16/terraform%20apply%202.PNG)

- A new file **terraform.tfstate** is created as a result of the above command which Terraform uses to keeps itself up to date with the exact state of the infrastructure and **terraform.tfstate.lock.info** file which Terraform uses to track who is running its code against the infrastructure at any point in time.

### **STEP 2: Creating Subnet Resources**
---
According to the architectural design, we require **6 subnets**:

- 2 public
- 2 private for **webservers**
- 2 private for **data layer**

Let us create the first **2 public subnets**. Therefore declare 2 resource blocks, one for each of the subnets.

**Note:** We are using the **vpc_id argument** to interpolate the value of the VPC id by setting it to **aws_vpc.main.id**. This way, Terraform knows inside which VPC to create the subnet.

Add below configuration to the **main.tf** file:
```
# Create public subnets1
    resource "aws_subnet" "public1" {
    vpc_id                     = aws_vpc.main.id
    cidr_block                 = "172.16.0.0/24"
    map_public_ip_on_launch    = true
    availability_zone          = "eu-west-2a"

}

# Create public subnet2
    resource "aws_subnet" "public2" {
    vpc_id                     = aws_vpc.main.id
    cidr_block                 = "172.16.1.0/24"
    map_public_ip_on_launch    = true
    availability_zone          = "eu-west-2b"
}
```
- Run: `terraform plan` and `terraform apply`

  ![](./Images/images16/terra%20plan.PNG)

  ![](./Images/images16/terra%20apply.PNG)

- Checked my aws console to confirm if the VPC and the subnets has been created  

  ![](./Images/images16/vpc%20on%20console.PNG)

  ![](./Images/images16/subnet%20on%20console.PNG)

**NOTE:** So far we have declared multiple resource blocks for each subnet in the code. Also, Both the **availability_zone** and **cidr_block** arguments are **hard coded**, which is not the best practice. So, we need to optimize this by introducing a **count** argument. 

- Firstly I destroyed what have been created so far by running 

  `terraform destroy`  

  ![](./Images/images16/terra%20dest.PNG)
  ![](./Images/images16/terra%20dest%202.PNG)

### **STEP 3: Refactoring The Codes**
---
1. Inorder to make the work dynamic, hard coded values are removed by introducing **variables**

- Starting with the **Provider block** Declaring a variable named **region** and giving it a default value, and updating the provider section by referring to the declared variable. 
  ```
  variable "region" {
        default = "eu-west-2"
    }

    provider "aws" {
        region = var.region
    }
    ```
- Do the same to **cidr** value in the **vpc** block, and all the other arguments.  
  ```
  variable "region" {
        default = "eu-west-2"
    }

    variable "vpc_cidr" {
        default = "172.16.0.0/16"
    }

    variable "enable_dns_support" {
        default = "true"
    }

    variable "enable_dns_hostnames" {
        default ="true" 
    }

    variable "enable_classiclink" {
        default = "false"
    }

    variable "enable_classiclink_dns_support" {
        default = "false"
    }

    provider "aws" {
    region = var.region
    }

    # Create VPC
    resource "aws_vpc" "main" {
    cidr_block                     = var.vpc_cidr
    enable_dns_support             = var.enable_dns_support 
    enable_dns_hostnames           = var.enable_dns_hostnames
    enable_classiclink             = var.enable_classiclink
    enable_classiclink_dns_support = var.enable_classiclink_dns_support

    }  
    ```
2. Fixing multiple resource blocks by introducing some interesting concepts; **Loops & Data sources**

- Fetching Availability zones from AWS, and replacing the hard coded value in the subnet’s availability_zone section with the use of **Data Sources**. 
  ```
   # Get list of availability zones
    data "aws_availability_zones" "available" {
        state = "available"
    }
  ```
- Introducing a **count** argument in the subnet block to make use of the new data resource:

  ```
  # Create public subnet1
    resource "aws_subnet" "public" { 
        count                   = 2
        vpc_id                  = aws_vpc.main.id
        cidr_block              = "172.16.1.0/24"
        map_public_ip_on_launch = true
        availability_zone       = data.aws_availability_zones.available.names[count.index]

    }
  ```
- The count tells terraform to invoke a loop to create 2 subnets. and the data resource will return a list object that contains a list of AZs.

- But if Terraform is being run with this configuration, it may succeed for the first time, but by the time it goes into the second loop, it will fail because the **cidr_block** still has to be hard coded because the same cidr_block cannot be created twice within the same VPC.

- To make the cidr block dynamic, a function **cidrsubnet()** is introduced which accepts **3** parameters.  
  ```
   # Create public subnet1
    resource "aws_subnet" "public" { 
        count                   = 2
        vpc_id                  = aws_vpc.main.id
        cidr_block              = cidrsubnet(var.vpc_cidr, 4 , count.index)
        map_public_ip_on_launch = true
        availability_zone       = data.aws_availability_zones.available.names[count.index]

    }
  ```
- Introuducing length() function, which basically determines the length of a given list and passing it to data.aws_availability_zones.available.names

- But since this returns the value of 3 instead of 2 that is preffered, the variable to store the desired number of public subnets is declared and it is set to the default value.  
  ```
  variable "preferred_number_of_public_subnets" {
    default = 2
  }
  ```
- Updating the count argument with a condition of which Terraform checks first if there is a desired number of subnets, Otherwise, it will use the data returned by the lenght function.
  ```
  # Create public subnets
  resource "aws_subnet" "public" {
    count  = var.preferred_number_of_public_subnets == null ? length(data.aws_availability_zones.available.names) : var.preferred_number_of_public_subnets   
    vpc_id = aws_vpc.main.id
    cidr_block              = cidrsubnet(var.vpc_cidr, 4 , count.index)
    map_public_ip_on_launch = true
    availability_zone       = data.aws_availability_zones.available.names[count.index]
  
  }
  ```
  ![](./Images/images16/refactor%20code%201.PNG)

  ![](./Images/images16/refactor%20code%202.PNG)

### **STEP 4: Introducing Variables.tf And terraform.tfvars**
---
- To make the code a lot more readable and better structured instead of having a long list of variables in **main.tf** file, the variable declarations is moved to a separate file and a file for non-default values for each of the variables is created.

- Creating a new file and naming it **variables.tf**

- Moving all the variable declarations into the new file.

- Creating another file and naming it **terraform.tfvars**

- Setting values for each of the variables:

```
region = "eu-central-1"

vpc_cidr = "172.16.0.0/16" 

enable_dns_support = "true" 

enable_dns_hostnames = "true"  

enable_classiclink = "false" 

enable_classiclink_dns_support = "false" 

preferred_number_of_public_subnets = 2
```
- main.tf

![](./Images/images16/maintf%20ref.PNG)

- variables.tf

![](./Images/images16/variabletf%20ref.PNG)

- terraform.tfvars

![](./Images/images16/terraformtfvars%20ref.PNG)

### **STEP 5: Executing Terraform Apply**
---
- Run `terraform fmt` to formate the code

- Run `terraform plan`

- Run `terraform apply --auto-approve`

![](./Images/images16/tf%20apply%201.PNG)

![](./Images/images16/tf%20apply%202.PNG)

- Check AWS console to confirm my resources are created using **IAC** tool: **Terraform**

  - VPC
    
    ![](./Images/images16/vpc%20ref.PNG)

  - subnet

    ![](./Images/images16/subnet%20ref.PNG) 

### **END OF PROJECT ...**     
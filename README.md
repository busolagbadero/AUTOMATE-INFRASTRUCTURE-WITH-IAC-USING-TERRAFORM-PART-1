# AUTOMATE-INFRASTRUCTURE-WITH-IAC-USING-TERRAFORM-PART-1

Now that we have manually created the AWS infrastructure for two websites, it's time to implement Infrastructure as Code (IaC) using Terraform to automate the process.

![bday](https://user-images.githubusercontent.com/94229949/230887934-55a45f8a-b988-4ce2-a09e-4c1314885b43.png)


To simplify the authentication process, I accessed the Ubuntu WSL server and utilized the command "aws configure" to log in to my AWS account.

Create an S3 bucket for the purpose of storing the Terraform state file.

![sunday60](https://user-images.githubusercontent.com/94229949/230889413-3d485b96-f58e-4047-a443-ee3c85e73482.png)


To begin, establish a directory named "PBL" and within it, create a file labeled "main.tf." Inside the "main.tf" file, insert code that declares AWS as a provider, as well as a resource to produce a VPC. The provider block notifies Terraform that our objective is to create infrastructure within AWS. To proceed, we must acquire the essential plugins for Terraform to operate, which are utilized by providers and provisioners. At this stage, our "main.tf" file solely contains a provider. As a result, Terraform will download the AWS provider plugin. The command "terraform init" can be used to achieve this, as demonstrated below.


![sunday61](https://user-images.githubusercontent.com/94229949/230896454-dee78c8b-1f2b-4e02-ad24-ece5c572c715.png)


Execute the command "terraform plan" to view the actions Terraform intends to perform. During this process, a new file named "terraform.tfstate" will be generated. This file serves as a reference point for Terraform to determine the current state of the infrastructure, including what already exists, what must be created, or destroyed based on the entirety of the Terraform code that is being developed.

Additionally, you may have observed that another file is produced during the planning and application stages. However, this file is immediately deleted. The file is referred to as "terraform.tfstate.lock.info" and is employed by Terraform to track who is utilizing the code against the infrastructure at any given moment. This feature is crucial for teams working on the same Terraform repository concurrently. The lock function prevents a user from executing Terraform configuration against the same infrastructure when another user is doing the same, effectively avoiding duplicates and conflicts.

In line with our architectural plan, we necessitate six subnets consisting of two public subnets, two private subnets for webservers, and two private subnets for the data layer. To create these subnets, include the following configuration in the "main.tf" file:


# Create public subnets1
    resource "aws_subnet" "public1" {
    vpc_id                     = aws_vpc.main.id
    cidr_block                 = "172.16.0.0/24"
    map_public_ip_on_launch    = true
    availability_zone          = "us-east-1a"

}

# Create public subnet2
    resource "aws_subnet" "public2" {
    vpc_id                     = aws_vpc.main.id
    cidr_block                 = "172.16.1.0/24"
    map_public_ip_on_launch    = true
    availability_zone          = "us-east-1b"
}


To create two subnets, we need to declare two resource blocks - one for each subnet.

To specify the VPC in which the subnets will be created, we use the vpc_id argument and interpolate the value of the VPC id by setting it to aws_vpc.main.id. This allows Terraform to know which VPC to use for subnet creation.

Observations:

Hard coded values: It is important to note that the availability_zone and cidr_block arguments are hard coded, which is not a good practice. We should always strive to make our work dynamic and avoid hard coding.


Multiple Resource Blocks: In the current code, we have declared multiple resource blocks for each subnet. This is not efficient because if we wanted to create 10 subnets, our code would be cumbersome. To optimize our code, we need to introduce a count argument and create a single resource block that can dynamically create resources.


To improve our code, we can refactor it by creating separate files named variables.tf and terraform.tfvars.


## The variables.tf file will contain the following content:

    variable "region" {
      default = "us-east-1"
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

    variable "preferred_number_of_public_subnets" {
      default = null
    }

## The terraform.tfvars file will contain the content below:


    region = "us-east-1"

    vpc_cidr = "172.16.0.0/16" 

    enable_dns_support = "true" 

    enable_dns_hostnames = "true"  

    enable_classiclink = "false" 

    enable_classiclink_dns_support = "false" 

    preferred_number_of_public_subnets = 2


## The main.tf will have the content below:


 #Get list of availability zones
  
      data "aws_availability_zones" "available" {
      
      state = "available"
     }

      provider "aws" {
  
   
     region = var.region
 
     }

## Create VPC

    resource "aws_vpc" "main" {
  
    cidr_block                     = var.vpc_cidr
  
    enable_dns_support             = var.enable_dns_support
  
    enable_dns_hostnames           = var.enable_dns_support
  
    enable_classiclink             = var.enable_classiclink
  
    enable_classiclink_dns_support = var.enable_classiclink
  
    }

# Create public subnets
    resource "aws_subnet" "public" {
    count  = var.preferred_number_of_public_subnets == null ? length(data.aws_availability_zones.available.names) : var.preferred_number_of_public_subnets   
    vpc_id = aws_vpc.main.id
    cidr_block              = cidrsubnet(var.vpc_cidr, 4 , count.index)
    map_public_ip_on_launch = true
    availability_zone       = data.aws_availability_zones.available.names[count.index]
    }


The main purpose of creating these files is to avoid hard coding values and promote reusability.

The variables.tf file contains all the variable declarations from the main.tf file.

The terraform.tfvars file sets values for each of the variables.

The structure of the PBL folder will be like:


![te-11](https://user-images.githubusercontent.com/94229949/231288127-477699c2-5542-4f08-8703-3d7a5ade7c14.png)


Run terraform plan to ensure all configuration is not showing any error and run terraform apply.



![te-15](https://user-images.githubusercontent.com/94229949/231288297-514b7163-89b5-4948-98a8-82cc13c66409.png)


![te-16](https://user-images.githubusercontent.com/94229949/231288332-5e4c64f5-612a-47a6-9c4e-994f0b9e3008.png)


To confirm if the VPC and Subnets were created, go to your AWS account to confirm.


![bbq1](https://user-images.githubusercontent.com/94229949/231289805-3a562453-d265-4d7a-895f-30a3209bdb6d.png)

The above shows 2 VPC's with one of them been the default VPC that comes with the AWS and the other one is the one we created with vpc-cidr = 172.16.0.0/16

Below is the sunbets created also:


![bbq2](https://user-images.githubusercontent.com/94229949/231289968-ff4c0302-254d-4e18-b3c1-6d88e7f2a483.png)


To destroy the infrastructure, just run the command terraform destroy.

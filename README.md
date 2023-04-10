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

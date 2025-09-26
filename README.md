# General Set Up Instructions
## Task 1
### Create a new SSH keypair in your Linux environment.
```
ssh-keygen -t rsa -b 4096
```
When prompted, give your key a name. For example: "terraform_key".  
### Add the public key to the included cloud-init config file.
Open the scripts/cloud-config.yaml.tpl file. 
```
sudo nano scripts/cloud-config.yaml.tpl
```
Change the public_key placeholder to the the contents of the public key file ```cat <public-key-name>``` <br>
Add the following code to the bottom of the file.
```
packages:
    - nginx
    - nmap
runcmd:
    - [ systemctl, enable, --now, nginx ]
```
## Task 2
Make the following changes in main.tf:
```
resource "aws_vpc" "web" {
    cidr_block = "10.0.0.0/16"
        enable_dns_support = true
        enable_dns_hostnames = true
}
```
```
resource "aws_subnet" "web" {
    # set availability zone
    availability_zone = "us-west-2a"
    # add public ip on launch
    map_public_ip_on_launch = true

}
```
```
resource "aws_internet_gateway" "web-gw" {
    # add vpc
    vpc_id = aws.vpc.web.id
}
```
```
resource "aws_route_table" "web" {
    # add vpc
    vpc_id = aws.vpc.web.id
}
```
```
resource "aws_route" "default_route" {
    # add gateway id
    gateway_id = aws_internet_gateway.web-gw.id
}
```
```
resource "aws_route_table_association" "web" {
    # add subnet id
    subnet_id = aws_subnet.web.id
}
```
```
resource "aws_Security_group" "web" {
    # add vpc id
    vpc_id = aws_vpc.web.id
}
```
```
resource "aws_vpc_security_group_ingress_rule" "web-ssh" {
    # allow ssh anywhere
    cidr_ipv4 = "0.0.0.0/0"
    from_port = 22
    to_port = 22
    ip_protocol = "tcp"
}
```
```
resource "aws_vpc_security_group_ingress_rule" "web-http" {
    # allow http anywhere
    cidr_ipv4 = "0.0.0.0/0"
    from_port = 80
    to_port = 80
    ip_protocol = "tcp"
}
```
```
resource "aws_instance" "web" {
    # set instance type
    ami = data.aws_ami.debian.id
    instance_type = "t3.micro"
    # add user data for cloud-config file in scripts directory
    user_data = file("~/4640-w4-lab-start-w25/scripts/cloud-config.yaml")
    # add vpc security group
    vpc_security_group_ids = [aws_security_group.web.id]
}
```
For each of the tag arguments, prepend the local project name variable. For example: ```Name = ${local.project_name}-Web``` in the ```resource "aws_subnet" "web"``` block
### Commands used to initialize terraform
Initialize the repository
```terraform init```
Format main.tf
```terraform fmt```
Validate files
```terraform validate```
Make a plan
```terraform plan -out lab4```
Apply the plan
```terraform apply lab4```
## Task 3
### Connect to the web user 
```ssh -i <path-to-ssh-key> web@<public-ip or public-dns>```

## To destroy infrastructure
```terraform destroy```
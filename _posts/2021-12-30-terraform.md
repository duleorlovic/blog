---
layout: post
---

Terraform provides Configuration Management on an infrastructure level, not on the level of software of your machines.

# Install

Download binary https://www.terraform.io/downloads
Example https://github.com/wardviaene/terraform-course

# Usage

Commands

```
# download provider plugins
terraform init
# Terraform has created a lock file .terraform.lock.hcl to record the provider
# selections it made above. Include this file in your version control repository
git init .
echo .terraform/ > .gitignore

# check the changes live (if there are unudefined vars it will asks for vars)
terraform plan
# you can save to a file and apply them
terraform plan -out changes.terraform && terraform apply changes.terraform

# push the changes
terraform apply
terraform apply -auto-approve # accept yes

# correct the format
terraform fmt
# validate syntax
terraform validate

# inspect current state from terraform.tfstate
terraform show

# to show all resources from current state
terraform state list
# to rename resource
terraform state mv aws_instance.example aws_instance.production

# create visual representation of a configuration or execution plan
terraform graph

# find resource ID and import to the state as ADDRESS. Only for importing the
# state still need to write resource definitions since next apply will remove it
# todo: https://learn.hashicorp.com/tutorials/terraform/state-import
terraform import ADDRESS ID
terraform import aws_instance.example i-0dba323asd123

# show defined outputs
terraform output
terraform output resource-name

# refresh remote state
terraform refresh

# configure remote state storage
terraform remote

# to remove
terraform destroy

# manually mark resource as tainted, it will be destructed and recreated at the
# next apply
terraform taint
terraform untaint
```

main.tf for `terraform` block (for main setting and providers)
https://registry.terraform.io/ and for resources

```
# main.tf
terraform {
  required_providers {
    # https://www.terraform.io/language/providers/requirements
    docker = {
      source  = "kreuzwerker/docker"
      version = "~> 2.13.0"
    }
  }
}
```

```
# resource.tf
# provider block use local name of the provider
provider "docker" {}

# resource block has two strings before block: resource type and resource name
# prefix for resource type match the provider name. resource id is type.name
resource "docker_image" "nginx" {
  name         = "nginx:latest"
  keep_locally = false
}

# access the container on http://localhost:8000/
resource "docker_container" "nginx" {
  image = docker_image.nginx.latest
  name = "tutorial"
  ports {
    internal = 80
    external = 8000
  }
}
```

variable block
https://learn.hashicorp.com/tutorials/terraform/variables
```
# variables.tf or vars.tf
variable "myvar" {
  type = "string"
  default = "hello"
}

# array
variable "mylist" {{
  description = "A list of zones"
  type = list(string)
  default = [ "abc", "qwe" ]
}

variable "mymap" {
  type = map(string)
  default = {
    mykey = "my value"
  }
}

# tuple is like a list with a different type
[0, "string", false]

# you can override variable as attribute:
terraform apply -var "myvar=NewName" -var "RDS_PASSWORD=$PASSWORD"_

# you can test inside `terraform console`
to create:
tomap({"k"="v", "k2"="v2"}) # returns { "k"="v", "k2"="v2" }
tolist("a", "b") # returns array ["a", "b"]

var.myvar
"${var.myvar}" # interpolation
var.mymap["mykey"]
values(mymap) # ["my value"]
lookup(var.mymap, "mykey", "default")
var.mylist[0]
element(var.mylist, 0)
index(mylist, elem) # find index of element in a mylist
slice(var.mylist, 0, 2)
merge(map1, map2) # merge two arrays
coalesce(string1, string2) # returns first non empty value
format("server-%03d", count.index + 1)  returns server-001 server-002
join(delim, mylist), join(",", var.AMIS) # ami-1,ami-2
replace("aaab", "a", "c") # "cccb"
split(",", "a,b,c") # ["a", "b", "c"]
substring("abcd", offset, length)
timestamp()
uuid()
```

Math `${2+3*4}`

Count
```
count.FIELD
$[count.index]
```

Conditionals
```
count = "${var.env == "prod" ? 2 : 1 }"
```

For loops
```
[for s in ["a", 1]: upper(s)]
{for k,v in { "a"=1 }: k => v}
```

Nested block does no have equal sign and we can repeat them
```
  ingress {
  }
  ingress {
  }
```
For each loop uses `dynamic` and `content` keywords and `.key` and `.value`
attributes of the variable
```
  dynamic "ingress" {
    for_each = [22, 443]
    content {
      from_port   = ingress.value
      to_port     = ingress.value
      protocol    = "tcp"
    }
  }
```

Project structure
```
staging/
production/
modules/
```
to differentiate between accounts you can use different profiles from aws
configure.

secrets are usually stored in `terraform.tfvars` (autoloaded, other name can be
loaded like `-var-file="testing.tfvars"`) which should be git ignored
```
echo */terraform.tfvars >> .gitignore
```
and contains all keys in format `NAME = "value"`
```
AWS_ACCESS_KEY = "AKIA..."
AWS_SECRET_KEY = "6JX..."
AWS_REGION = "us-east-1"
```
and those variables should be declared in `vars.tf` (note that they can be found
in tfstate so keep tfstate in secure place also if you use secrets)
```
echo */terraform.tfstate >> .gitignore
echo */terraform.tfstate.backup >> .gitignore
```
You can `export TF_VAR_DB_PASSWORD=asd1234` instead of using `terraform.tfvars`
```
# vars.tf
# without value or default will be asked
# variable "AWS_ACCESS_KEY" {}
# variable "AWS_SECRET_KEY" {}
variable "AWS_REGION" {
  default = "us-east-1"
}

variable "AMIS" {
  type = map
  default = {
    # find ami on https://cloud-images.ubuntu.com/locator/ec2/ search example
    # us-east-1 hvm
    # https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/finding-an-ami.html#finding-quick-start-ami
    us-east-1 = "ami-0b0ea68c435eb488d"
  }
}

variable "DB_PASSWORD" {
  description = "RDS posgres user password"
  sensitive   = true
}
```

aws provider

```
# provider.tf
provider "aws" {
  # comment so current `aws configure` will be used
  access_key = vars.AWS_ACCESS_KEY
  secret_key = var.AWS_SECRET_KEY
  region     = var.AWS_REGION
}
```

# Provision software

Use file uploads with `file` provisioner and `remote-exec` to run the script
```
provisioner "file" {
  source = "app.conf"
  destination = "/etc/myapp.conf"
  connection {
    type = ssh
    user = var.instance_username
    password = var.instance_password
  }
}
```

For aws we use ssh keypairs for which we need resource `aws_key_pair`

```
# keys.tf
# key is generated with ssh-keygen -f mykey
# https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/key_pair
resource "aws_key_pair" "mykeypair" {
  # existing keys can be imported with: terraform import aws_key_pair.deployer deployer-key
  # https://console.aws.amazon.com/ec2/v2/home?region=us-east-1#KeyPairs:
  # mykeypair will be destroyed when we run terraform destroy
  # Find IP address from aws console or using output
  # ssh -i mykey ubuntu@3.84.117.126
  # ssh -i mykey ubuntu@$(cat terraform.tfstate | jq -r '.resources[].instances[].attributes.public_ip | select( . != null )')
  # ssh -i mykey ubuntu@$(aws ec2 describe-instances --query "Reservations[*].Instances[*].PublicIpAddress" --output=text)
  key_name = "mykeypair"
  public_key = file(var.PATH_TO_PUBLIC_KEY)
}

# resource.tf
resource "aws_instance" "example" {
  ami           = lookup(var.AMIS, var.AWS_REGION)
  instance_type = "t2.micro"
  key_name = aws_key_pair.mykeypair.key_name

  provisioner "file" {
    source = "script.sh"
    destination = "/tmp/script.sh"
  }
  provisioner "remote-exec" {
    inline = [
      "chmod +x /tmp/script.sh",
      "sudo /tmp/script.sh"
    ]
  }
  # another way to output info is using local-exec provisioner (it is performed
  # only first time resource is created)
  provisioner "local-exec" {
    command = "echo ${aws_instance.example.private_ip} >> private_ips.txt"
  }
  connection {
    user = var.INSTANCE_USERNAME
    private_key = file(var.PATH_TO_PRIVATE_KEY)
    host = self.public_ip
  }
}
output "ip" {
  description = "Public IP address for EC2 instance"
  value = aws_instance.example.public_ip
}
output "private_ip" {
  value = aws_instance.example.private_ip
}
```
Add to vars
```
# vars.tf
variable "PATH_TO_PUBLIC_KEY" {
  default = "mykey.pub"
}

variable "PATH_TO_PRIVATE_KEY" {
  default = "mykey"
}

variable "INSTANCE_USERNAME" {
  default = "ubuntu"
}
```
```
# script.sh
#!/bin/bash
apt-get update
apt-get -y install nginx
```

## State

`terraform.tfstate` is where it keeps track of remote state. there is also
`terraform.tfstate.backup` for previous state.
You can keep `terraform.tfstate` in git repo so you can see state changes.
For example when you remove the instance from aws console, `terraform apply`
will make a changes to meet the correct remote state.
But is is not adviceable since another person can change the state so you loose
the sync since you do not use same lock (also it contains sensitive information)
For single user repo you can add both `terraform.tfstate` and
`.terraform.lock.hcl` to the repo. Lock is important so we use same versions of
the providers and modules.

```
# Find all resource names
cat terraform.tfstate | jq '.resources[].name'
# find public_ip
cat terraform.tfstate | jq '.resources[].instances[].attributes.public_ip | select( . != null )'
```

You can save state remote using a backend functionality in terraform
https://www.terraform.io/language/settings/backends/remote

Credentials are different then those secrets for provision.
```
# backend.tf
terraform {
  backend "s3" {
    bucket = "duleorlovic-test"
    key = "tfstate"
    region = "eu-central-1"
  }
}
```
and run `terraform init` to check backend.
Inside bucket properties you can enable "Versions" to see old states.

## Packer

https://learn.hashicorp.com/packer you can create a custom image
todo: https://learn.hashicorp.com/collections/terraform/provision

# Other providers

https://registry.terraform.io/browse/providers
Any company that opens API can be an provider: Datadog (monitoring), Github,
mailgun, DNSSimple

Providers are similar.

# Template provider templatefile

For creating customized configuration files, template based on variables from
resource attributes (for example public ip address).
Use case is cloud init config (user-data on AWS).

```
# templates/init.tpl
echo "hello world ${myip}"
```
Instead of separate resource
```
# deprecated
data "template_file" "my-template" {
  template = file("templates/init.tpl")

  vars {
    myip = aws_instance.database1.private_ip
  }
}

# usage
resource "aws_instance" "web" {
  user_data = data.template_file.my-template.rendered
}
```
but now we use `templatefile(file, map)` function
```
locals {
  web_vars = {
    my_ip = aws_instance.database1.private_ip
  }
}

resource "aws_instance" "web" {
  user_data = templatefile("templates/init.tpl", local.web_vars)
  # or inline
  user_data = templatefile("templates/init.tpl", {
    my_ip = aws_instance.database1.private_ip
  })
}
```

# Modules

You can organize files inside folders (locally) or remote on github.
```
module "module-example" {
  source = "github.com/duleorlovic/terraform-module-example"
  # locally
  source = "./module-example"

  # override or add additional arguments
  region = "us-east-1"
}
```
For example
```
# module-example/vars.tf
variable "region" {} # the input parameter
variable "ip-range" {}

# module-example/cluster.tf
resource "aws_instance" "instance-1" {
}

# module-example/output.tf
output "aws-cluster" {
  value = aws_instance.instance-1.public_ip
}
```
To use output of module you can reference `module.module-name.output`
```
output "some-output" {
  value = module.module-example.aws-cluster
}
```
You have access to `path.cwd` `path.module` and `path.root`.
To download you need to run
```
terraform get
ls -l .terraform/modules
```

todo: https://learn.hashicorp.com/tutorials/terraform/module

# Terraform Cloud

quick start
```
terraform login
git clone https://github.com/hashicorp/tfc-getting-started.git
cd tfc-getting-started/
scripts/setup.sh
```
todo: https://learn.hashicorp.com/collections/terraform/cloud-get-started

# AWS CLI

https://aws.amazon.com/cli/

If you export keys, cli will use that
https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-getting-started.html


To see or update `~/.aws/credentials` and `~/.aws/config` you can run
```
aws configure
```

```
aws ec2 describe-instances
```

# AWS provider

main defines version of providers
```
# main.tf
# https://registry.terraform.io/providers/hashicorp/aws/latest/docs
terraform {
  required_providers {
    aws = {
      source = "hashicorp/aws"
      version = "~> 3.0"
    }
  }
}
```

provider configures specific provider
```
# provider.tf
provider "aws" {
  region = "us-east-1"
}
```

VPC
```
# vpc.tf
# https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/vpc
resource "aws_vpc" "main" {
  cidr_block           = "10.0.0.0/16"
  instance_tenancy     = "default"
  enable_dns_support   = "true"
  enable_dns_hostnames = "true"
  enable_classiclink   = "false"
  tags = {
    Name = "main"
  }
}
```
Subnet
```
# subnet.tf
# https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/subnet
resource "aws_subnet" "main-public-1" {
  vpc_id                  = aws_vpc.main.id
  cidr_block              = "10.0.1.0/24"
  # this will actually differentiate public and private subnets
  map_public_ip_on_launch = "true"
  availability_zone       = "eu-west-1a"

  tags = {
    Name = "main-public-1"
  }
}

resource "aws_subnet" "main-public-2" {
  vpc_id                  = aws_vpc.main.id
  cidr_block              = "10.0.2.0/24"
  map_public_ip_on_launch = "true"
  availability_zone       = "eu-west-1b"

  tags = {
    Name = "main-public-2"
  }
}

resource "aws_subnet" "main-private-1" {
  vpc_id                  = aws_vpc.main.id
  cidr_block              = "10.0.4.0/24"
  map_public_ip_on_launch = "false"
  availability_zone       = "eu-west-1a"

  tags = {
    Name = "main-private-1"
  }
}

```
internet gateway
```
# https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/internet_gateway
resource "aws_internet_gateway" "main-gw" {
  vpc_id = aws_vpc.main.id

  tags = {
    Name = "main"
  }
}
```

route tables
```
# https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/route_table
resource "aws_route_table" "main-public" {
  vpc_id = aws_vpc.main.id
  route {
    cidr_block = "0.0.0.0/0"
    gateway_id = aws_internet_gateway.main-gw.id
  }

  tags = {
    Name = "main-public-1"
  }
}

```
route table associations with public subnets
```
# https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/route_table_association
# this will actually differentiate public and private subnets
resource "aws_route_table_association" "main-public-1-a" {
  subnet_id      = aws_subnet.main-public-1.id
  route_table_id = aws_route_table.main-public.id
}

resource "aws_route_table_association" "main-public-2-a" {
  subnet_id      = aws_subnet.main-public-2.id
  route_table_id = aws_route_table.main-public.id
}
```
to enable internet for private instances we use `nat_gateway` resource type (it
needs elastic ip)
```
# nat.tf
# elastic ip is needed for nat gw
resource "aws_eip" "nat" {
  vpc = true
}

resource "aws_nat_gateway" "nat-gw" {
  allocation_id = aws_eip.nat.id
  subnet_id     = aws_subnet.main-public-1.id
  depends_on    = [aws_internet_gateway.main-gw]
}

# VPC setup for NAT
resource "aws_route_table" "main-private" {
  vpc_id = aws_vpc.main.id
  route {
    cidr_block     = "0.0.0.0/0"
    nat_gateway_id = aws_nat_gateway.nat-gw.id
  }

  tags = {
    Name = "main-private-1"
  }
}

# route associations private
resource "aws_route_table_association" "main-private-1-a" {
  subnet_id      = aws_subnet.main-private-1.id
  route_table_id = aws_route_table.main-private.id
}
```

EC2 instance needs security group
```
# securitygroup.tf
# https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/security_group
resource "aws_security_group" "allow-ssh" {
  vpc_id      = aws_vpc.main.id
  name        = "allow-ssh"
  description = "security group that allows ssh and all egress traffic"
  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

  ingress {
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }
  tags = {
    Name = "allow-ssh"
  }
}

# I use this so I can ping any instances in my VPC
resource "aws_security_group" "allow-ping" {
  vpc_id      = aws_vpc.main.id
  name        = "allow-ping"
  description = "security group that allows ping to it"
  ingress {
    from_port   = 8
    to_port     = 0
    protocol    = "icmp"
    cidr_blocks = ["0.0.0.0/0"]
  }
  tags = {
    Name = "allow-ping"
  }
}
```


instance resource EC2
```
# instance.tf
# https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/instance
resource "aws_instance" "example" {
  ami           = var.AMIS[var.AWS_REGION]
  # t2.micro automatically adds 8GB EBS storage Elastic block storage
  instance_type = "t2.micro"

  # the VPC subnet, if not defined, default VPC is used, note that security
  # group needs to be in the same vpc as instance
  subnet_id = aws_subnet.main-public-1.id

  # the security group, if not defined, default security group is used
  vpc_security_group_ids = [aws_security_group.allow-ssh.id, aws_security_group.allow-ping.id]

  # the public SSH key
  key_name = aws_key_pair.mykeypair.key_name

  tags = {
    Name = "ExampleAppServerInstance"
  }
}

output "ip" {
  description = "Public IP address for EC2 instance"
  value = aws_instance.example.public_ip
}
```

ebs_volume
```
# instance.tf
resource "aws_ebs_volume" "ebs-volume-1" {
  availability_zone = "eu-west-1a"
  size              = 20
  type              = "gp2"
  tags = {
    Name = "extra volume data"
  }
}

resource "aws_volume_attachment" "ebs-volume-1-attachment" {
  device_name = "/dev/xvdh"
  volume_id   = aws_ebs_volume.ebs-volume-1.id
  instance_id = aws_instance.example.id
}
```
user_data is used to do script at launch - creation of instance (not reboot): to
install extra software, join a cluster, mount volumes
Use as string or templates
```
# instance.tf
resource "aws_instance" "example" {
  ami           = var.AMIS[var.AWS_REGION]
  instance_type = "t2.micro"

  # the public SSH key
  key_name = aws_key_pair.mykeypair.key_name

  # user data
  user_data = data.template_cloudinit_config.cloudinit-example.rendered
}

resource "aws_ebs_volume" "ebs-volume-1" {
  availability_zone = "eu-west-1a"
  size              = 20
  type              = "gp2"
  tags = {
    Name = "extra volume data"
  }
}

resource "aws_volume_attachment" "ebs-volume-1-attachment" {
  device_name  = var.INSTANCE_DEVICE_NAME
  volume_id    = aws_ebs_volume.ebs-volume-1.id
  instance_id  = aws_instance.example.id
  skip_destroy = true                            # skip destroy to avoid issues with terraform destroy
}
```
data for scripts
```
# cloudinit.tf
data "template_file" "init-script" {
  template = file("scripts/init.cfg")
  vars = {
    REGION = var.AWS_REGION
  }
}

data "template_file" "shell-script" {
  template = file("scripts/volumes.sh")
  vars = {
    DEVICE = var.INSTANCE_DEVICE_NAME
  }
}

data "template_cloudinit_config" "cloudinit-example" {
  gzip          = false
  base64_encode = false

  part {
    filename     = "init.cfg"
    content_type = "text/cloud-config"
    content      = data.template_file.init-script.rendered
  }

  part {
    content_type = "text/x-shellscript"
    content      = data.template_file.shell-script.rendered
  }
}
```
scripts
```
# scripts/init.cfg
#cloud-config

repo_update: true
repo_upgrade: all

packages:
  - lvm2

output:
  all: '| tee -a /var/log/cloud-init-output.log'
```
```
# scripts/volumes.sh
#!/bin/bash

set -ex 

vgchange -ay

DEVICE_FS=`blkid -o value -s TYPE ${DEVICE} || echo ""`
if [ "`echo -n $DEVICE_FS`" == "" ] ; then 
  # wait for the device to be attached
  DEVICENAME=`echo "${DEVICE}" | awk -F '/' '{print $3}'`
  DEVICEEXISTS=''
  while [[ -z $DEVICEEXISTS ]]; do
    echo "checking $DEVICENAME"
    DEVICEEXISTS=`lsblk |grep "$DEVICENAME" |wc -l`
    if [[ $DEVICEEXISTS != "1" ]]; then
      sleep 15
    fi
  done
  # make sure the device file in /dev/ exists
  count=0
  until [[ -e ${DEVICE} || "$count" == "60" ]]; do
   sleep 5
   count=$(expr $count + 1)
  done
  pvcreate ${DEVICE}
  vgcreate data ${DEVICE}
  lvcreate --name volume1 -l 100%FREE data
  mkfs.ext4 /dev/data/volume1
fi
mkdir -p /data
echo '/dev/data/volume1 /data ext4 defaults 0 0' >> /etc/fstab
mount /data

# install docker
curl https://get.docker.com | bash
```

static public ip and static private ip
```
# instance.tf
resource "aws_instance" "example" {
  ami           = var.AMIS[var.AWS_REGION]
  instance_type = "t2.micro"
  subnet_id = aws_subnet.main-public-1.id
  private_id = "10.0.1.4" # within the range of main-public-1
}

# elastic ip address is free when it is assigned to instance, otherwise it is
# not free
resource "aws_eip" "example-eip" {
  instance = aws_instance.example.id
  vpc = true
}

output "public-ip" {
  value = aws_eip.example-eip.public_ip
}
```
route53 can be used so we use hostname instead of ip address
```
# reute53.tf
resource "aws_route53_zone" "newtech-academy" {
  name = "newtech.academy"
}

resource "aws_route53_record" "server1-record" {
  zone_id = aws_route53_zone.newtech-academy.zone_id
  name    = "server1.newtech.academy"
  type    = "A"
  ttl     = "300"
  records = [aws_eip.example-eip.public_ip]
}

resource "aws_route53_record" "www-record" {
  zone_id = aws_route53_zone.newtech-academy.zone_id
  name    = "www.newtech.academy"
  type    = "A"
  ttl     = "300"
  records = ["104.236.247.8"]
}

resource "aws_route53_record" "mail1-record" {
  zone_id = aws_route53_zone.newtech-academy.zone_id
  name    = "newtech.academy"
  type    = "MX"
  ttl     = "300"
  records = [
    "1 aspmx.l.google.com.",
    "5 alt1.aspmx.l.google.com.",
    "5 alt2.aspmx.l.google.com.",
    "10 aspmx2.googlemail.com.",
    "10 aspmx3.googlemail.com.",
  ]
}

output "ns-servers" {
  description = "Use this nameservers on your registar"
  value = aws_route53_zone.newtech-academy.name_servers
}
output "check-server1" {
  value = "host server1.newtech.academy ${aws_route53_zone.newtech-academy.name_servers[0]}"
}
```

RDS relation database service: managed database with replication (high
availability), automated snapshots (backups) and security updates, instance
replacement (vertical scaling, for example more memory/cpu)
You need to define: subnet group (to specify in what subnets db will be in),
parameter group (change settings in db since we do not have ssh access to
instances) and security group (firewall to enable incomming traffic)
https://github.com/wardviaene/terraform-course/tree/master/demo-12
```
# rds.tf
resource "aws_db_subnet_group" "mariadb-subnet" {
  name        = "mariadb-subnet"
  description = "RDS subnet group"
  subnet_ids  = [aws_subnet.main-private-1.id, aws_subnet.main-private-2.id]
}

resource "aws_db_parameter_group" "mariadb-parameters" {
  name        = "mariadb-parameters"
  family      = "mariadb10.4"
  description = "MariaDB parameter group"

  parameter {
    name  = "max_allowed_packet"
    value = "16777216"
  }
}

resource "aws_db_instance" "mariadb" {
  allocated_storage       = 100 # 100 GB of storage, gives us more IOPS than a lower number
  engine                  = "mariadb"
  engine_version          = "10.4.13"
  instance_class          = "db.t2.small" # use micro if you want to use the free tier
  identifier              = "mariadb"
  name                    = "mariadb"
  username                = "root"           # username
  password                = var.RDS_PASSWORD # password
  db_subnet_group_name    = aws_db_subnet_group.mariadb-subnet.name
  parameter_group_name    = aws_db_parameter_group.mariadb-parameters.name
  multi_az                = "false" # set to true to have high availability: 2 instances synchronized with each other
  vpc_security_group_ids  = [aws_security_group.allow-mariadb.id]
  storage_type            = "gp2"
  backup_retention_period = 30                                          # how long you’re going to keep your backups
  availability_zone       = aws_subnet.main-private-1.availability_zone # prefered AZ
  skip_final_snapshot     = true                                        # skip final snapshot when doing terraform destroy
  tags = {
    Name = "mariadb-instance"
  }
}

```
```
# securitygroup.tf
resource "aws_security_group" "example-instance" {
  vpc_id      = aws_vpc.main.id
  name        = "allow-ssh"
  description = "security group that allows ssh and all egress traffic"
  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

  ingress {
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }
  tags = {
    Name = "example-instance"
  }
}

resource "aws_security_group" "allow-mariadb" {
  vpc_id      = aws_vpc.main.id
  name        = "allow-mariadb"
  description = "allow-mariadb"
  ingress {
    from_port       = 3306
    to_port         = 3306
    protocol        = "tcp"
    security_groups = [aws_security_group.example-instance.id] # allowing access from our example instance
  }
  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
    self        = true
  }
  tags = {
    Name = "allow-mariadb"
  }
}
```

IAM identity and access management: user can have groups and roles. user can
login using login/password or token and MFA, and access key and secret key.
You can use predefined policy and attach to group or you add gruop inline policy
```
# iam.tf
# group definition
resource "aws_iam_group" "administrators" {
  name = "administrators"
}

resource "aws_iam_policy_attachment" "administrators-attach" {
  name       = "administrators-attach"
  groups     = [aws_iam_group.administrators.name]
  policy_arn = "arn:aws:iam::aws:policy/AdministratorAccess"
  # or inline
  policy = <<EOF
}

# resource "aws_iam_group_policy" "my_policy" {
#   name   = "my_administrator_policy"
#   group  = aws_iam_group.administrators.id
#   policy = <<EOF
#   {
#     "Version": "2012-10-17",
#     "Statement": [
#       {
#         "Effect": "Allow",
#         "Action": "*",
#         "Resource": "*"
#       }
#     ]
#   }
#   EOF
# }

# user
resource "aws_iam_user" "admin1" {
  name = "admin1"
}

resource "aws_iam_user" "admin2" {
  name = "admin2"
}

resource "aws_iam_group_membership" "administrators-users" {
  name = "administrators-users"
  users = [
    aws_iam_user.admin1.name,
    aws_iam_user.admin2.name,
  ]
  group = aws_iam_group.administrators.name
}

output "warning" {
  value = "WARNING: make sure you're not using the AdministratorAccess policy for other users/groups/roles. If this is the case, don't run terraform destroy, but manually unlink the created resources"
}

```
iam roles
Roles are used to add temporary access that they normally would't have.
Roles can be attached to EC2 instances, from that instance user can obtain
access credentials, using those credentials user or service can assume the role.

```
# s3.tf
resource "aws_s3_bucket" "b" {
  bucket = "mybucket-c29df1"
  acl    = "private"

  tags = {
    Name = "mybucket-c29df1"
  }
}
```
```
# instance.tf
resource "aws_instance" "example" {
  ami           = var.AMIS[var.AWS_REGION]
  instance_type = "t2.micro"

  # the VPC subnet
  subnet_id = aws_subnet.main-public-1.id

  # the security group
  vpc_security_group_ids = [aws_security_group.example-instance.id]

  # the public SSH key
  key_name = aws_key_pair.mykeypair.key_name

  # role:
  iam_instance_profile = aws_iam_instance_profile.s3-mybucket-role-instanceprofile.name
}

```

```
# iam.tf
resource "aws_iam_role" "s3-mybucket-role" {
  name               = "s3-mybucket-role"
  assume_role_policy = <<EOF
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Action": "sts:AssumeRole",
      "Principal": {
        "Service": "ec2.amazonaws.com"
      },
      "Effect": "Allow",
      "Sid": ""
    }
  ]
}
EOF

}

resource "aws_iam_instance_profile" "s3-mybucket-role-instanceprofile" {
  name = "s3-mybucket-role"
  role = aws_iam_role.s3-mybucket-role.name
}

resource "aws_iam_role_policy" "s3-mybucket-role-policy" {
  name = "s3-mybucket-role-policy"
  role = aws_iam_role.s3-mybucket-role.id
  policy = <<EOF
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
              "s3:*"
            ],
            "Resource": [
              "arn:aws:s3:::mybucket-c29df1",
              "arn:aws:s3:::mybucket-c29df1/*"
            ]
        }
    ]
}
EOF

}

```

autoscalling needs: launch configuration (properties of the instance to be
launched ami id, security group) or launch template () and autoscalling group
(scaling properties like min instances, health check).
https://github.com/wardviaene/terraform-course/tree/master/demo-15

launch configuration and autoscalling group
```
# autoscaling.tf
# https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/launch_configuration
resource "aws_launch_configuration" "movebase-launchconfig" {
  name_prefix     = "movebase-launchconfig"
  image_id        = var.AMIS[var.AWS_REGION]
  instance_type   = "t2.micro"
  key_name        = aws_key_pair.movebase-key-pair.key_name
  security_groups = [aws_security_group.movebase-allow-ssh-and-all-egress.id]
}

# https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/autoscaling_group
resource "aws_autoscaling_group" "movebase-autoscaling-group" {
  name                      = "movebase-autoscaling-group"
  vpc_zone_identifier       = [aws_subnet.movebase-subnet-public-1.id, aws_subnet.movebase-subnet-public-2.id]
  launch_configuration      = aws_launch_configuration.movebase-launchconfig.name
  min_size                  = 1
  max_size                  = 2
  health_check_grace_period = 300
  health_check_type         = "EC2"
  force_delete              = true

  tag {
    key                 = "Name"
    value               = "movebase-ec2-instance"
    propagate_at_launch = true
  }
}

```
policy and alarm
autoscalling policy is triggered based on threshold (cloudwatch alarm)
```
# autoscalingpolicy.tf
# https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/autoscaling_policy
resource "aws_autoscaling_policy" "movebase-autoscaling-policy-cpu" {
  name                   = "movebase-autoscaling-policy-cpu"
  autoscaling_group_name = aws_autoscaling_group.movebase-autoscaling-group.name
  adjustment_type        = "ChangeInCapacity"
  scaling_adjustment     = "1"
  cooldown               = "300"
  policy_type            = "SimpleScaling"
}

# https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/cloudwatch_metric_alarm
resource "aws_cloudwatch_metric_alarm" "movebase-cloudwatch-metric-alarm-cpu" {
  alarm_name          = "movebase-cloudwatch-metric-alarm-cpu"
  alarm_description   = "movebase-cloudwatch-metric-alarm-cpu"
  comparison_operator = "GreaterThanOrEqualToThreshold"
  evaluation_periods  = "2"
  metric_name         = "CPUUtilization"
  namespace           = "AWS/EC2"
  period              = "120"
  statistic           = "Average"
  threshold           = "30"

  dimensions = {
    "AutoScalingGroupName" = aws_autoscaling_group.movebase-autoscaling-group.name
  }

  actions_enabled = true
  alarm_actions   = [aws_autoscaling_policy.movebase-autoscaling-policy-cpu.arn]
}

# scale down alarm
resource "aws_autoscaling_policy" "movebase-autoscaling-policy-cpu-scaledown" {
  name                   = "movebase-autoscaling-policy-cpu-scaledown"
  autoscaling_group_name = aws_autoscaling_group.movebase-autoscaling-group.name
  adjustment_type        = "ChangeInCapacity"
  scaling_adjustment     = "-1"
  cooldown               = "300"
  policy_type            = "SimpleScaling"
}

resource "aws_cloudwatch_metric_alarm" "movebase-cloudwatch-metric-alarm-cpu-scaledown" {
  alarm_name          = "movebase-cloudwatch-metric-alarm-cpu-scaledown"
  alarm_description   = "movebase-cloudwatch-metric-alarm-cpu-scaledown"
  comparison_operator = "LessThanOrEqualToThreshold"
  evaluation_periods  = "2"
  metric_name         = "CPUUtilization"
  namespace           = "AWS/EC2"
  period              = "120"
  statistic           = "Average"
  threshold           = "5"

  dimensions = {
    "AutoScalingGroupName" = aws_autoscaling_group.movebase-autoscaling-group.name
  }

  actions_enabled = true
  alarm_actions   = [aws_autoscaling_policy.movebase-autoscaling-policy-cpu-scaledown.arn]
}

```
notification
```
# sns.tf
# https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/sns_topic
resource "aws_sns_topic" "movebase-sns-topic" {
  name         = "movebase-sns-topic"
}

# https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/sns_topic_subscription
resource "aws_sns_topic_subscription" "user_updates_sqs_target" {
  topic_arn = aws_sns_topic.movebase-sns-topic.arn
  protocol = "email"
  endpoint = var.NOTIFICATION_EMAIL
}

resource "aws_autoscaling_notification" "example-notify" {
  group_names = [aws_autoscaling_group.movebase-autoscaling-group.name]
  topic_arn     = aws_sns_topic.movebase-sns-topic.arn
  notifications  = [
    "autoscaling:EC2_INSTANCE_LAUNCH",
    "autoscaling:EC2_INSTANCE_TERMINATE",
    "autoscaling:EC2_INSTANCE_LAUNCH_ERROR"
  ]
}
```

on ubuntu you can simulate the load
```
sudo apt install stress
stress --cpu 2 --timeout 300
```

ELB elastic load balancer distributes incomming traffic, scales when you receive
more trafic, it healthcheck all instances (and stop sending traffic to them)
It also used as SSL terminator and manage ssl certificate).
It can be spread over multiple availability zones.
Classic load balancer routes on network level: port 80 to port 8080
ALB routes on application level: /api to different instances
https://github.com/wardviaene/terraform-course/tree/master/demo-16
```
# elb.tf
# https://docs.aws.amazon.com/elasticloadbalancing/latest/application/introduction.html#application-load-balancer-components
# Each target group routes requests to one or more registered targets, such as
# EC2 instances, using the protocol and port number that you specify in
# listener. You can register a target with multiple target groups. You can
# configure health checks on a per target group basis. Health checks are
# performed on all targets registered to a target group that is specified in a
# listener rule for your load balancer.
# old https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/elb classic
# new https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/lb which can be application, network, gateway
# todo: tutorial https://medium.com/cognitoiq/terraform-and-aws-application-load-balancers-62a6f8592bcf
# todo: long tutorial https://hiveit.co.uk/techshop/terraform-aws-vpc-example/
resource "aws_lb" "movebase-lb" {
  name            = "movebase-lb"
  security_groups = [aws_security_group.movebase-allow-ssh-and-all-egress.id, aws_security_group.movebase-allow-80.id]
  subnets         = [aws_subnet.movebase-subnet-public-1.id, aws_subnet.movebase-subnet-public-2.id]
  tags = {
    Name = "movebase-lb"
  }
}

# https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/lb_listener
# https://docs.aws.amazon.com/elasticloadbalancing/latest/application/load-balancer-listeners.html
resource "aws_lb_listener" "movebase-lb-listener-80" {
  load_balancer_arn = aws_lb.movebase-lb.arn
  port              = "80"
  protocol          = "HTTP" # fpr ALB: HTTP or HTTPS for NLB: TCP, TLS, UDP and TCP_UDP

  default_action {
    type             = "forward" # forward, redirect, fixed-response, authenticate-cognito and authenticate-oidc
    target_group_arn = aws_lb_target_group.movebase-lb-target-group.arn
  }
}

# https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/lb_target_group
resource "aws_lb_target_group" "movebase-lb-target-group" {
  name             = "movebase-lb-target-group"
  port             = 80
  protocol         = "HTTP"
  vpc_id           = aws_vpc.movebase-vpc.id
  tags = {
    Name = "movebase-lb-target-group"
  }
  # https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/lb_target_group#health_check
  health_check {
    healthy_threshold   = 2
    unhealthy_threshold = 2
    timeout             = 3
    interval            = 30
  }
}

# to connect to autoscaling, we define a link in autoscalling.tf
# "aws_lb_target_group_attachment" is used to connect with specific instance

# autoscalling.tf
# we could also use inline load_balancers inside "aws_autoscaling_group"
# https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/autoscaling_attachment
resource "aws_autoscaling_attachment" "movebase-autoscaling-group-attachment" {
  autoscaling_group_name = aws_autoscaling_group.movebase-autoscaling-group.name
  alb_target_group_arn   = aws_lb_target_group.movebase-lb-target-group.arn
}
```

ECS ec2 container services - cluster for docker containers. You need to start
autoscalling group with custom AMI (which contains ECS agent)
```
# ecs.tf
# cluster
resource "aws_ecs_cluster" "example-cluster" {
  name = "example-cluster"
}

resource "aws_launch_configuration" "ecs-example-launchconfig" {
  name_prefix          = "ecs-launchconfig"
  image_id             = var.ECS_AMIS[var.AWS_REGION]
  instance_type        = var.ECS_INSTANCE_TYPE
  key_name             = aws_key_pair.mykeypair.key_name
  iam_instance_profile = aws_iam_instance_profile.ecs-ec2-role.id
  security_groups      = [aws_security_group.ecs-securitygroup.id]
  user_data            = "#!/bin/bash\necho 'ECS_CLUSTER=example-cluster' > /etc/ecs/ecs.config\nstart ecs"
  lifecycle {
    create_before_destroy = true
  }
}

resource "aws_autoscaling_group" "ecs-example-autoscaling" {
  name                 = "ecs-example-autoscaling"
  vpc_zone_identifier  = [aws_subnet.main-public-1.id, aws_subnet.main-public-2.id]
  launch_configuration = aws_launch_configuration.ecs-example-launchconfig.name
  min_size             = 1
  max_size             = 1
  tag {
    key                 = "Name"
    value               = "ecs-ec2-container"
    propagate_at_launch = true
  }
}

```

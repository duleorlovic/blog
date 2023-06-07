---
layout: post
---

Terraform provides Configuration Management on an infrastructure level, not on
the level of software on your machines.

# Tutorials

Tutorial excellent https://cloudcasts.io/course/terraform
Udemy TODO: https://www.udemy.com/course/learn-devops-infrastructure-automation-with-terraform/ https://github.com/wardviaene/terraform-course

# Install

Download binary https://www.terraform.io/downloads

```
brew tap hashicorp/tap
brew install hashicorp/tap/terraform
terraform -install-autocomplete
```

# Usage

Commands

```
# download provider plugins
terraform init
# Terraform has created a lock file .terraform.lock.hcl to record the provider
# selections it made above. Include this file in your version control repository
git init .
cat >> .gitignore << 'HERE_DOC'
.terraform/
terraform.tfvars
terraform.tfstate
terraform.tfstate.backup
HERE_DOC

# check the changes live
terraform plan
# you can save to a binary file and apply them
terraform plan -out changes.terraform && terraform apply changes.terraform
# see plan changes
terraform plan -out /tmp/tfplan && terraform show -json /tmp/tfplan | jq -r ".resource_changes[0].change.before.data , .resource_changes[0].change.after.data"

# push the changes
terraform apply
terraform apply -auto-approve # accept yes
# I suggest to save the state in git
terraform apply -auto-approve && git add .

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
# this will clear the state file but you can restore using git or
# terraform.tfstate.backup
terraform destroy
# destroy specific resource
terraform destroy --target aws_instance.demo_vm_1

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

# Variable_block

https://learn.hashicorp.com/tutorials/terraform/variables

```
# variables.tf
# if variable does not have value or default, it will be asked
variable "AWS_ACCESS_KEY" {}

variable "mystring" {
  type = string
  default = "hello"
}

variable "mynumber" {
  type = number
  default = 1.42
}

variable "mybool" {
  type = bool
  default = true
}

# list is array
variable "mylist" {
  description = "A list of zones"
  type = list(string)
  default = [ "abc", "qwe" ]
}

variable "mymap" {
  type = map(string)
  default = {
    mykey = "my value"
    # also colon is possible
    mykey: "my value"
  }
}
variable "AMIS" {
  type = map
  default = {
    # find ami on https://cloud-images.ubuntu.com/locator/ec2/ search example us-east-1 hvm
    # https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/finding-an-ami.html#finding-quick-start-ami
    us-east-1 = "ami-0b0ea68c435eb488d"
  }
}

variable "myset" {
  type = set
  default = [1, 2]
}

variable "myobject" {
  type = object({name=string, age=number})
  default = {
    name = "Joe"
    age = 42
  }
}

variable "mytuple" {
  type = type([number, string, bool])
  default = [0, "string", false]
}
```
Use
```
# tuple is like a list with a different type
[0, "string", false]

# identifiers can contain letters, digits, underscore, hyphens, but first char
# must not be a digit. If key in a map is not valid identifier, it has to be
# quoted

# you can override variable as attribute:
terraform apply -var "myvar=NewName" -var "RDS_PASSWORD=$PASSWORD"_

# you can see variables inside `terraform console` but you can not define
# variable inside console. Also you can not see outputs, but you can use
# directly their values
aws_s3_bucket.my-bucket.arn

# you can play with functions:
tomap({"k"="v", "k2"="v2"}) # returns { "k"="v", "k2"="v2" }
tolist("a", "b") # returns array ["a", "b"]

# string
var.myvar
"${var.myvar}" # interpolation inside string
# string manipulation
coalesce(string1, string2) # returns first non empty value
format("server-%03d", count.index + 1)  returns server-001 server-002
join(delim, mylist), join(",", var.AMIS) # ami-1,ami-2
replace("aaab", "a", "c") # "cccb"
split(",", "a,b,c") # ["a", "b", "c"]
substring("abcd", offset, length)

# map
var.mymap["mykey"]
var.mymap.mykey
keys(mymap) # ["mykey"]
values(mymap) # ["my value"]
lookup(var.mymap, "mykey", "default")
merge(map1, map2) # merge two hashes

# list
var.mylist[0] # same as element(var.mylist, 0)
var.mylist[*].arn # splat expression `*` instead of index returns list or arns
index(mylist, elem) # find index of element in a mylist
slice(var.mylist, 0, 2)

# access to one property in the list of maps
[ { a = "a" }, { a = "c" }][*].a
# [ "a", "c" ]

timestamp()
uuid()
```

You can also reference values of resources `aws_s3_bucket.name`

Secrets are usually stored in `terraform.tfvars` (autoloaded is
`terraform.tfvars` and any file `*.auto.tfvars`, other name can be
loaded like `-var-file="testing.tfvars"`) which should be git ignored
since it contains all keys in format `NAME = "value"`
```
AWS_ACCESS_KEY = "AKIA..."
AWS_SECRET_KEY = "6JX..."
AWS_REGION = "us-east-1"
```
You can use env to define variables when you export with prefix TF_VAR_ env
variable `export TF_VAR_DB_PASSWORD=asd1234`.
So use it instead of using `terraform.tfvars`

To output variable which was marked as sensitive you can
* `t output DB_PASSWORD` explicitly show that value
* `grep --after-context=10 outputs terraform.tfstate` grep state
* decode plan that is saved in tmp folder
  ```
  t plan -target=vault_generic_secret.foobar -out=/tmp/tfplan
  t show -json /tmp/tfplan  | jq -r ".resource_changes[0].change.before.data , .resource_changes[0].change.after.data"
  ```
* mark as non sensitive
  ```
    output "mysecret" {
      value = nonsensitive(var.mysecret)
    }
  ```


Math `${2+3*4}`

Count object has `.index` attribute and can be used for iterations
```
resource "aws_iam_user" "example" {
  count = length(var.mylist)
  # name = "neo.${count.index}"
  name = var.mylist[count.index]
}
```
and resulting resource is actually array of resources so we have to use `[]`
```
output "first_user_arn" {
  value = aws_iam_user.example[0].arn
}
output "all_users_arn" {
  value = aws_iam_user.example[*].arn
}
```
Count can not be used to loop over inline block.
When you remove from the list, other elements will be moved by one position ie
lot of resources changes instead of one single resource deletion
This is not the case with `for_each`
https://www.terraform.io/language/meta-arguments/for_each
Usually create variable map with numbers and you can use `each.key` and
`each.value`
```
variable "public_subnet_numbers" {
  type = map(number)
  description = "Map of AZ to a number that should be used for public subnet, used in for_each"
  default = {
    us-east-1a = 1
    us-east-1b = 2
  }
}
resource "aws_subnet" "public" {
  for_each = var.private_subnet_numbers
  cidr_block = each.value
}

output "keys" {
  value = keys(aws_subnet.public)
}
# or in console: keys(aws_subnet.public)
# [ "us-east-1a", "us-east-1b"...
output "all_arns" {
  value = values(aws_subnet.public)[*].arn
}
```
output of for_each on resource is a map, keys are keys in for_each, values are
resources.
Removing one element from map will result in removing one resource (no shifting
down other resources).


Conditionals can be defined in 3 ways, using count paramentar, string directives
and for_each

```
# ternary syntax
count = "${var.env == "prod" ? 2 : 1 }"
```
To conditionally make resource use `count` and star `*` for output
```
variable "enable_eip" {
  description = "If set to true, enable eip"
  type        = bool
}

resource "aws_eip" "ec2_eip" {
  # ternary syntax
  count = var.enable_eip ? 1 : 0
}

output "ec2_eip_public_ip" {
  value = aws_eip.ec2_eip.*.public_ip
  # or
  value = var.enable_eip ? aws_eip.ec2_eip[0].public_ip : null
}
```
If else can be implemented in a similar way (`count = var.enable_eip ? 0 : 1`)
When you need to pick the value of one that has been defined you can use
`one(concat())`
```
output "neo_cloudwatch_policy_arn" {
  value = one(concat(
    aws_iam_user_policy_attachment.full_access[*].policy_arn,
    aws_iam_user_policy_attachment.read_only[*].policy_arn
  ))
}
```

String directives can have two forms: for directive (loop) and if directive
(conditional)

```
# %{ for <ITEM> in <COLLECTION> }<BODY>%{ endfor }
output "for_directive" {
  value = "%{ for name in var.mylist }${name}, %{ endfor }"
}
# for_directive = "neo, trinity, morpheus, "

# for directive with index
# %{ for <INDEX>, <ITEM> in <COLLECTION> }<BODY>%{ endfor }
output "for_directive_index" {
  value = "%{ for i, name in var.names }(${i}) ${name}, %{ endfor }"
}
# for_directive_index = "(0) neo, (1) trinity, (2) morpheus, "
```
`if` string directive for condition to remove last `, ` and put `.`
```
output "for_directive_index_if_else_strip" {
  value = <<EOF
%{~ for i, name in var.names ~}
${name}%{ if i < length(var.names) - 1 }, %{ else }.%{ endif }
%{~ endfor ~}
EOF
}
# for_directive_index_if_else_strip = "neo, trinity, morpheus."
```


For expression (for loops). Return type depends if we wrap with [] or {}
(requires `=>` sumbol).  https://www.terraform.io/language/expressions/for
```
[for s in ["a", 1]: upper(s)]
{for k,v in { "a"=1 } : upper(k) => v}

# you can split map based on if condition:
variable "users" {
  type = map(object({
    is_admin = bool
  }))
}
locals {
  admin_users = {
    for name, user in var.users : name => user
    if user.is_admin
  }
}

# to group results, ie merge values to array, add three dots ...
locals {
  users_by_role = {
    for name, user in var.users : user.role => name...
  }
}
```

Nested block does no have equal sign and we can repeat them
```
  nested_block {
  }
  nested_block {
  }
```
For nested block we can use for_each loop using `dynamic` and `content` keywords
and `.key` and `.value` attributes of the `nested_block` variable.
Iteration with for_each uses set or map (list can be converted
with`for_each = toset(var.mylist)`)
```
  dynamic "nested_block" {
    for_each = [22, 443]
    content {
      from_port   = nested_block.value
      to_port     = nested_block.value
      protocol    = "tcp"
    }
  }
```
To create multiple subnets you can use https://www.terraform.io/language/functions/cidrsubnets

```
  cidr_block = cidrsubnet(aws_vpc.vpc.cidr_block, 4, each.value)

# X.X.xxxx0000.0000
cirdsubnets("10.1.0.0/16",4)
# ["10.1.0.0/20"]
cirdsubnets("10.1.0.0/16",4, 4)
# ["10.1.0.0/20", "10.1.16.0/20"]
```

# Data

Use data from provider, like search for ami
https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/instance
```
# ami.tf
data "aws_ami" "ami" {
  # https://cloud-images.ubuntu.com/locator/ec2/
  # https://awscli.amazonaws.com/v2/documentation/api/latest/reference/ec2/describe-images.html
  # in case of error in creating an instance, try to find ami-123 on web
  # https://us-east-1.console.aws.amazon.com/ec2/v2/home?region=us-east-1#Images:visibility=public-images
  # and create instance from web console to see minimum requirements
  # or with cli aws ec2 describe-images --image-ids ami-0a24ce26f4e187f9a

  most_recent = true
  filter {
    name = "name"
    values = ["ubuntu/images/hvm-ssd/ubuntu-jammy-22.04-amd64-server-*"]
  }

  owners = ["099720109477"] # Canonical official

  # https://stackoverflow.com/questions/28168420/choose-a-free-tier-amazon-machine-image-ami-using-ec2-command-line-tools
  # aws ec2 describe-images --owner amazon --filter "Name=description,Values=*Ubuntu*" "Name=owner-alias,Values=amazon" "Name=architecture,Values=x86_64" "Name=image-type,Values=machine" "Name=root-device-name,Values=/dev/sda1" "Name=root-device-type,Values=ebs" "Name=virtualization-type,Values=hvm"
  # filter {
  #   name = "root-device-type"
  #   values = ["ebs"]
  # }
  # filter {
  #   name = "description"
  #   values = ["*Ubuntu*"]
  # }
  # filter {
  #   name = "owner-alias"
  #   values = ["amazon"]
  # }
  # filter {
  #   name = "architecture"
  #   values = ["x86_64"]
  # }
}

output "ami_id" {
  value = data.aws_ami.ami.id
}

```

# Random


```
# https://registry.terraform.io/providers/hashicorp/random/latest/docs/resources/shuffle
resource "random_shuffle" "subnet_id" {
  input = var.subnet_ids
  result_count = 1
}

# usage
  subnet_id = random_shuffle.subnet_id.result[0]
```

# Project structure

```
staging/
production/
modules/
```
to differentiate between accounts you can use different profiles `aws configure`
Each folder has it's own `terraform.tfvars` which is autoloaded.

You can run from root using script and -chdir
```
# run.sh
#!/usr/bin/env bash
 
TF_ENV=$1
 
DIR="$( cd "$( dirname "${BASH_SOURCE[0]}" )" >/dev/null 2>&1 && pwd )"
 
# Always run from the location of this script
cd $DIR
 
if [ $# -gt 0 ]; then
    if [ "$2" == "init" ]; then
        terraform -chdir=./$TF_ENV init -backend-config=../backend-$TF_ENV.tf
    else
        terraform -chdir=./$TF_ENV $2
    fi
fi
 
# Head back to original location to avoid surprises
cd -
```

Also you can create separate folders in each env so for example you can update
ec2 instances without need to take care rds resources (for example if rds is
updated on aws you have to update in terraform, but ec2 team does not need to
think about rds at all).
To reference resources from other folder you can use data and filter, for
example to find vpc from ec2 folder
```
# variables.tf
data "aws_vpc" "vpc" {
  tags = {
    Name        = "cloudcasts-${var.infra_env}-vpc"
    Project     = "cloudcasts.io"
    Environment = var.infra_env
    ManagedBy   = "terraform"
  }
}

data "aws_subnet_ids" "public_subnets" {
  vpc_id = data.aws_vpc.vpc.id

  tags = {
    Name        = "cloudcasts-${var.infra_env}-vpc"
    Project     = "cloudcasts.io"
    Environment = var.infra_env
    ManagedBy   = "terraform"
    Role        = "public"
  }
}
```

## Workspaces

You can organize different environment you can use Workspaces
https://www.terraform.io/language/state/workspaces

When state is local file, it will create new `terraform.tfstate.d/dev/` folder
On S3 it prepends state file with workspace name.
When you change the folder, it will change the workspace
```
terraform workspace list
terraform workspace new dev
terraform workspace show
```
use like variable
```
# variables.tf
locals {
  env = terraform.workspace
}

# main.tf
  env = local.env
```

# SSH key pair

For aws we use ssh keypairs for which we need resource `aws_key_pair`

```
# variables.tf
variable "PATH_TO_PUBLIC_KEY" {
  default = "my_key.pub"
}
variable "PATH_TO_PRIVATE_KEY" {
  default = "my_key"
}

# key_pairs.tf
# https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/key_pair
# key is generated with: ssh-keygen -f my_key
resource "aws_key_pair" "mykeypair" {
  # existing keys can be imported with: terraform import aws_key_pair.deployer deployer-key
  # https://console.aws.amazon.com/ec2/v2/home?region=us-east-1#KeyPairs:
  # mykeypair will be destroyed when we run terraform destroy
  # Find IP address from aws console or using output
  # ssh -i my_key ubuntu@3.84.117.126
  # ssh -i my_key ubuntu@$(cat terraform.tfstate | jq -r '.resources[].instances[].attributes.public_ip | select( . != null )')
  # ssh -i my_key ubuntu@$(aws ec2 describe-instances --query "Reservations[*].Instances[*].PublicIpAddress" --output=text)
  key_name = "mykeypair"
  public_key = file(var.PATH_TO_PUBLIC_KEY)
}

# resource.tf
resource "aws_instance" "example" {
  ami           = lookup(var.AMIS, var.AWS_REGION)
  instance_type = "t2.micro"
  key_name = aws_key_pair.mykeypair.key_name
```

For error
```
│ Error: error importing EC2 Key Pair (my_key): InvalidParameterValue: Value for parameter PublicKeyMaterial is invalid. Length exceeds maximum of 2048.
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

```
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
Add to var
```

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
will make a changes to meet the correct remote state. Also if you remove tfstate
file , it will try to create all new (duplicated) resources since it lots ids.
For single user repo you can add both `terraform.tfstate` and
`.terraform.lock.hcl` to the repo. Lock is important so we use same versions of
the providers and modules.

```
# Find all resource names
cat terraform.tfstate | jq '.resources[].name'
# find public_ip
cat terraform.tfstate | jq '.resources[].instances[].attributes.public_ip | select( . != null )'
```

But for multiuser project, another person can change the state so you loose the
sync since you do not use same lock. You can save state remote using a backend
functionality in terraform
https://www.terraform.io/language/settings/backends/remote

Credentials are different then those secrets for provision.
```
# backend.tf
terraform {
  backend "s3" {
    bucket = "trk-tfstates"
    key = "myapp/terraform.tfstate"
    profile = "2022trk"
    region = "eu-central-1"
  }
}
```
and run `terraform init` to set up the backend.
Inside bucket properties you should enable "Versions" to see old states.
To can enable state locking, you should create a dynamodb table with `LockID`
partition key and if you name the table `tfstate-lock` you can use like
```
    dynamodb_table = "tfstate-lock"
```
https://www.terraform.io/language/settings/backends/s3#dynamodb-state-locking
```

```

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
https://www.terraform.io/language/functions/templatefile
```
# modules/ec2/user_data_httpd_server.tftpl
#!/bin/bash
sudo apt update -y
sudo apt install -y apache2
echo "Hello World from hostname=$(hostname -f) subnet_id=${subnet_id}" > /var/www/html/index.html
```
use `templatefile(file, map)` function
```
resource "aws_instance" "web" {
  user_data = templatefile("${path.module}/user_data_httpd_server.tftpl", {
    subnet_id = random_shuffle.subnet_id.result[0]
  })
}
```

# Locals and tags

Since we can not use variables inside other variables, we can use `local` which
is defined with pluralized version `locals`. We will use it to set the name and
other important information that can be defined as tags
```
locals {
  common_tags = {
    Name = "${var.environment}-web-server"
    Environment = var.environment
    ManagedBy = "terraform"
    SourceUrl = "https://github.com/duleorlovic/tf-aws-s3-static-website-bucket"
    TfstateUrl = "@air:terraform_modules/tf-aws-s3-static-website-bucket/examples/create_static_site/terraform.tfstate"
  }

resource "aws_vpc" "vpc" {
  tags = merge(local.common_tags, {
    Name = "${var.env}-vpc"
  })
}
```

if you want to provide ability to override any tag you can use merge ., override
```
variable "override_tags" {
  type = map(string)
  description = "Override or add new tags"
  default = {}
}

resource "aws_vpc" "vpc" {
  tags = merge(merge(local.common_tags, {
    Name = "${var.env}-vpc"
  }),
    var.override_tags
  )
}
```

# Lifecycle

https://www.terraform.io/language/meta-arguments/lifecycle
You can ignore certain changes (for example tags) and create before destroy
```
 lifecycle {
    ignore_changes = [
      # Ignore changes to tags, e.g. because a management agent
      # updates these based on some ruleset managed elsewhere.
      tags,
    ]
    create_before_destroy = true
    prevent_destroy = true # for example not to release public ip
  }
```

To prevent downtime you can create new resources, apply, remove unused
resources, and apply again.
Many resources has immutable parameters so terraform will remove them and create
another (instead of updating) so try to use `create_before_destroy` strategy.

# Modules

Done https://learn.hashicorp.com/collections/terraform/modules
Todo https://www.terraform.io/language/modules/develop/composition
TODO: https://blog.gruntwork.io/how-to-create-reusable-infrastructure-with-terraform-modules-25526d65f73d

You can organize files inside folders (locally) or remote on github.
To use it you need to define `source`. You can define `version` and other meta
arguments like `count`, `depends_on`
```
module "module-example" {
  source = "github.com/duleorlovic/terraform-module-example"
  # locally
  source = "./module-example"

  # override or add additional arguments
  region = "us-east-1"
}
```
Example module has at least three files: main, outputs.tf (variables that other
modules can use) and variables.tf (input variables defined inside `module`):
```
# module-example/variables.tf
variable "region" {} # the input parameter
variable "ip-range" {}

# module-example/cluster.tf
resource "aws_instance" "instance-1" {
}

# module-example/outputs.tf
output "aws-cluster" {
  value = aws_instance.instance-1.public_ip
}
```
To use output of module you can reference `module.module-name.output-name`
```
output "some-output" {
  value = module.module-example.aws-cluster
}
```
You have access to `path.cwd`, `path.module` and `path.root`.
To download you need to run get or init
```
terraform get
ls -l .terraform/modules
```

# Move

https://learn.hashicorp.com/tutorials/terraform/move-config
I think you can access module resources directly (no need to output)
```
moved {
  from = aws_instance.example
  to = module.ec2_instance.aws_instance.example
}
```
I tried to move a list or resources (created with for_each), but no success, it
recreated them (maybe to try `from = aws_instance.example.us-east-1a` or `from =
values(aws_instance.example)[0]`

# Terraform Cloud

quick start
```
terraform login
git clone https://github.com/hashicorp/tfc-getting-started.git
cd tfc-getting-started/
scripts/setup.sh
```
todo: https://learn.hashicorp.com/collections/terraform/cloud-get-started

# VIM

https://github.com/hashicorp/terraform-ls/blob/main/docs/USAGE.md#cocnvim
Add json to `:CocConfig` and install language server
```
brew install hashicorp/tap/terraform-ls
```

https://github.com/evilmartians/terraforming-rails/tree/master/tools/lint_env

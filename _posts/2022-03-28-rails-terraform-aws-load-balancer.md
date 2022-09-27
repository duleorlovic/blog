---
layout: post
---

# Introduction

We are using Terraform to provision Loadbalancer and other resources on AWS.
First you need to learn [Terraform
Fundamentals](https://github.com/duleorlovic/terraform-fundamentals)

For AWS, you can start with basic tutorial
https://learn.hashicorp.com/collections/terraform/aws-get-started and than
invest in advanced tutorials
https://learn.hashicorp.com/collections/terraform/aws

We need to ignore terraform internal files, .env and keys

```
# ignore `.terraform` folders
cat >> .gitignore << HERE_DOC
# ignore terraform folder
.terraform/
.env
# ignore keys in any folder
mykey
mykey.pub
HERE_DOC
```

Create separate folder for staging and production (it is recommended to have
same setup on staging as on production), or you can create folder for each setup
```
# mkdir terraform_staging
# mkdir terraform_production
mkdir terraform_one_instance
mkdir terraform_load_balancer
```

Just make sure you `cd` to that folder when you are using `terraform` cli
```
cd terraform_one_instance
```

First file to look at is `terraform_one_instance/main.tf` where we define
provider config with credentials.

# AWS CLI

To authenticate you can use several methods
https://registry.terraform.io/providers/hashicorp/aws/latest/docs#authentication-and-configuration
One way is to use default config and AWS CLI
https://aws.amazon.com/cli/
https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-getting-started.html

If you export keys, cli will use that env variables.
To see or update `~/.aws/credentials` and `~/.aws/config` you can run
```
aws configure
```
To list current instances
```
aws ec2 describe-instances
```

But I like to use provider configuration since we can see that we need
those variables. Anything that could be changed I put inside `var.tf`
For secrets I use `.env` file. You can load those keys from .env to bash ENV:
```
env
```

So for AWS keys we define
```
# .env
# Load env into current shell with:
# set -o allexport; source .env; set +o allexport
TF_VAR_AWS_ACCESS_KEY=AKIA...
```
and variable declaration (TF_VAR_{name} is used as {name} variable)
```
# var.tf
variable "AWS_ACCESS_KEY" {}
```
and use in aws provider configuration
```
# main.tf
provider "aws" {
  # comment if you want current `aws configure` to be used
  access_key = var.AWS_ACCESS_KEY
}
```

Inside `main.tf` we have`terraform` block that defines version of providers
and `provider` block that configures specific provider
```
# main.tf
terraform {
  # this block is optional
  required_providers {
    # https://registry.terraform.io/providers/hashicorp/aws/latest/docs
    aws = {
      source  = "hashicorp/aws"
      version = "~> 3.0"
    }
  }
}

provider "aws" {
  # comment if you want current `aws configure` to be used
  access_key = var.AWS_ACCESS_KEY
  secret_key = var.AWS_SECRET_KEY
  region     = var.AWS_REGION
}
```

Maybe the best way is to avoid env variables and use aws profile
`aws configure --profile 2022trk`
```
# main.tf
provider "aws" {
  profile = "2022trk"
}
```

# File names

We use AWS resources https://registry.terraform.io/providers/hashicorp/aws/latest/docs

Common resources are:
* `vpc.tf` vpc, subnet (a, b), internet gateway and route table, and association
  between route table and subnets
* `security_group.tf` for ssh, http, ping, mysql access
* `key.tf` to import ssh key

You should organize your modules based on privilege and volatility
https://learn.hashicorp.com/tutorials/terraform/pattern-module-creation?in=terraform/modules#network-module
* security: IAM
* routing: Route53, RouteTable, HostedZone
* Network: VPC, NAT gateway, security group
* web: LoadBalancer, AutoScalingGroup, S3
* app: LoadBalancer, AutoScalingGroup
* database: RDS

# AMI

name "ubuntu/images/hvm-ssd/ubuntu-focal-20.04-amd64-server-*"

# VPC

Find all availability zones
```
aws --profile=2022trk ec2 describe-availability-zones --query 'AvailabilityZones[].ZoneName'
```

# One instance

For one instance we can use `instance.tf`
* to enable internet for private instances we use `nat_gateway` resource type
  (it needs elastic ip)
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
# not free, so better is to use `instance` and assign on create. otherwise you
# can assign ip to instance in association aws_eip_association
# https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/eip
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
# https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/security_group
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

TODO https://medium.com/@ajays871/rails-6-deployment-using-terraform-docker-and-aws-codepipeline-a5fb15ede5eb
https://nts.strzibny.name/hybrid-docker-compose-rails/

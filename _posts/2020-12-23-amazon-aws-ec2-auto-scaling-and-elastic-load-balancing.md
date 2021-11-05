---
layout: post
---

# VPC

Video
components of vpc https://youtu.be/LX5lHYGFcnA?t=2921

VPC are limited to a region but stretch across Availability Zones AZs
Subnets can be Private or Public and are limited to a single AZ
Subnet is associated with route table, so when EC2 instance inside it wants to
communicate to internet outband, route table should contain IGW along with
default local entry (private subnet can not communicate with internet)
```
10.0.0.0/16  Local
0.0.0.0/0    IGW (Internet Gateway)
```
EC2 instance can use Elastic IP (static public IP address) to be able to get
inbound internet connections.
AWS managed Network Address Transation (NAT-GW) gateway enables EC2 instances in
private subnet to connect to internet outband. So here is route table for
private subnet (NET-GW is in public subnet so it can connect to internet through
IGW).
```
10.0.0.0/16  Local
0.0.0.0/0    NAT-GW
```

5 IPs are un-usable reserved .0 (network) .1 (router) .2 .3 .255 (broadcast) http://jodies.de/ipcalc?host=192.168.0.5&mask1=26&mask2=

VPC Endpoints is used to connect to Amazon S3 using Amazon private networks (not
going to internet).
VPC Interface Endpoints is creating elastic interface (with IP address) so you
can use them to connect to external services using private network.

Create on https://console.aws.amazon.com/vpc/home?region=us-east-1#vpcs:
Default aws is 172.31.0.0/16
Create 10.0.1.0/24 (251 IP addresses)
Difference between N ACLs nackles and Securuty Groups
https://youtu.be/LX5lHYGFcnA?t=9070
Security groups define only Deny

# EC2

https://docs.aws.amazon.com/autoscaling/ec2/userguide/GettingStartedTutorial.html

Mount EBS volume
https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ebs-using-volumes.html

# EFS

https://aws.amazon.com/getting-started/tutorials/create-network-file-system/

Create EFS https://console.aws.amazon.com/efs/home?region=us-east-1#/get-started
but use some new nfs-security-group (later you can see on Network tab) and allow
incoming connections for NFS 2049 from your instances.

Click on *Attach* to see command to mount using efs mount helper
Video https://www.youtube.com/watch?v=4jy2FILK5R8
Install https://github.com/aws/efs-utils
```
sudo apt-get update
sudo apt-get -y install git binutils
git clone https://github.com/aws/efs-utils
cd efs-utils
./build-deb.sh
sudo apt-get -y install ./build/amazon-efs-utils*deb
```
Mount
```
sudo mount -t efs fs-b95a6d4c:/ efs
```
When it timeouts that means that nfs-security-group should allow input rule for
NFS type (port 2049) for source that EC2 belongs.

Permanently mount automatically on reboot
```
# /etc/fstab
fs-b95a6d4c:/ /home/ubuntu/efs efs defaults,_netdev,tls 0 0
```

# Elastic load balancer

Classic looks at IP address and port (OSI Layer 4)
https://docs.aws.amazon.com/elasticloadbalancing/latest/classic/ssl-server-cert.html

ALB Application load balancers looks at url (OSI Layer 7)
https://docs.aws.amazon.com/elasticloadbalancing/latest/application/introduction.html


https://github.com/lserman/capistrano-elbas
```
# list instances
cap production elbas:ssh
```
It could be that new instances are started from old image and that is preventing
capistrano to deploy code to all instances. You can go to auto scaling group and
set maximum capacity 1 and than deploy.

Static ip address on load balancer
It can be done on Network Load Balancer
https://aws.amazon.com/premiumsupport/knowledge-center/elb-attach-elastic-ip-to-public-nlb/
https://aws.amazon.com/elasticloadbalancing/faqs/
Q: How does Network Load Balancer compare to what I get with the TCP listener on a Classic Load Balancer?

A: Network Load Balancer preserves the source IP of the client which in the Classic Load Balancer is not preserved. Customers can use proxy protocol with Classic Load Balancer to get the source IP. Network Load Balancer automatically provides a static IP per Availability Zone to the load balancer and also enables assigning an Elastic IP to the load balancer per Availability Zone. This is not supported with Classic Load Balancer.


# Auto scaling groups ASG

https://www.youtube.com/watch?v=-hFAWk6hyZA AWS Autoscaling | Autoscaling and Load Balancing in AWS | AWS Training | Edureka

https://www.youtube.com/watch?v=_Hu9WWHfSMk 
What are AWS Load Balancer, Auto Scaling and Route 53 | AWS Tutorial | Edureka | AWS Rewind - 4

A launch configuration includes:
* AMI (amazon machine image, it is bootable copy of snapshoty: only a copy) +
  instance type (t2.micro)
* EC2 user data
* EBS volumes
* Security groups
* SSH Key Pair

Scaling policies using ClodWatch alarms or using EC2 managed rules: average CPU
usage, number of requests on the ELB per instance, average network in, or using
custom metric (number or connected users, using PutMetric API from our app to
CloudWatch)

# SSL

When we use ELB (ALB/NLB) and eable listener (HTTPS/TLS) on port 443 than we
have to use certificates on load balancer, ie we need to copy paste or use AWS
Cert Manager ACM to keep certs.

There is some post
https://autoize.com/automating-lets-encrypt-https-behind-a-load-balancer/ to use
passthrough so ssl is terminated on server (lsyncd and restart services when we
update certs) but that is for Digital Ocean .  also depends on http-01 or dns-01
challenge (lexicon
https://id-rsa.pub/post/certbot-auto-dns-validation-with-lexicon/ ) ?  Renewing
should be on one instance (LB should forward check path to it).

API to upload to iam https://docs.aws.amazon.com/elasticloadbalancing/latest/classic/elb-update-ssl-cert.html#us-update-lb-SSLcert-cli
import acm Amazon Certificate Manager https://medium.com/@iamjasonchild/custom-ssl-certificate-with-letsencrypt-acm-route53-powered-by-certbot-e457614df6b8
import to iam https://gist.github.com/chrisjm/32a782317e377d52cc95fda8777e8dfe
script to generate and upload cert https://gist.github.com/mikob/a89fd8c5f85e0a00d557

https://docs.aws.amazon.com/acm/latest/userguide/import-certificate-api-cli.html
https://docs.aws.amazon.com/elasticloadbalancing/latest/classic/elb-add-or-delete-listeners.html
```
sudo snap install --classic aws-cli
```

Import (upload) certificate to ACM we need  AWSCertificateManagerFullAccess
permissions

```
# list certificates
AWS_CONFIG_FILE=~/efs/.dns_keys aws acm list-certificates

# uploading certificate
cd /etc/letsencrypt/live/asd.movebase.link
sudo su
AWS_CONFIG_FILE=/home/ubuntu/efs/.dns_keys aws acm import-certificate --certificate fileb://cert.pem --certificate-chain fileb://chain.pem --private-key fileb://privkey.pem
# this commands returns ARN which we have to use to set up ELB certificate
{
    "CertificateArn": "arn:aws:acm:us-east-1:219232999684:certificate/369b84d6-4527-49ed-8fc1-27004561f4da"
}
```

Set certificate on ELB (we need AmazonEC2FullAccess)
https://docs.aws.amazon.com/elasticloadbalancing/latest/classic/elb-update-ssl-cert.html#us-update-lb-SSLcert-cli
```
aws elb set-load-balancer-listener-ssl-certificate --load-balancer-name my-load-balancer --load-balancer-port 443 --ssl-certificate-id arn:aws:acm:region:123456789012:certificate/12345678-1234-1234-1234-123456789012
```
for Network load balancers use elbv2 instead of elb
```
AWS_CONFIG_FILE=/home/ubuntu/efs/.dule_keys aws elbv2 add-listener-certificates --listener-arn arn:aws:elasticloadbalancing:us-east-1:219232999684:listener/net/elb-trk/0b0c954a93bd6917/7cdcf7185ab7a1ec --certificates CertificateArn=arn:aws:acm:us-east-1:219232999684:certificate/369b84d6-4527-49ed-8fc1-27004561f4df

# I got error when I try to set IsDefault=true
An error occurred (ValidationError) when calling the AddListenerCertificates operation: You cannot set the isDefault parameter for a certificate.
```


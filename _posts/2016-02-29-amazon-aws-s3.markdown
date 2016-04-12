---
layout: post
title: Amazon AWS S3
tags: s3
---

# Create IAM user and attach policy

I usually one user for testing (keys that I export in my basrc), one user for
all my apps, and one for some projects which other devs could have access.
For test IAM user I attach
[policy](http://blogs.aws.amazon.com/security/post/Tx3VRSWZ6B3SHAV/Writing-IAM-Policies-How-to-grant-access-to-an-Amazon-S3-bucket)
for *Programmatic read and write permissions* with name for example
**full_access_to_video_uploading_demo** . Keep in mind that underscore _ is not
equal hyphen - .
Carrierwave and paperclip need something more than put, get and delete, so I
added `s3:*` below `"s3:DeleteObject"`.

~~~
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "s3:ListBucket"
            ],
            "Resource": [
                "arn:aws:s3:::duleorlovic-test",
                "arn:aws:s3:::duleorlovic-test-eu-central-1",
                "arn:aws:s3:::duleorlovic-test-us-east-1"
            ]
        },
        {
            "Effect": "Allow",
            "Action": [
                "s3:*"
            ],
            "Resource": [
                "arn:aws:s3:::duleorlovic-test/*",
                "arn:aws:s3:::duleorlovic-test-eu-central-1/*",
                "arn:aws:s3:::duleorlovic-test-us-east-1/*"
            ]
        }
    ]
}
~~~


# Mark bucket public

You can deploy frontend code to S3
[guide](https://docs.aws.amazon.com/AmazonS3/latest/dev/website-hosting-custom-domain-walkthrough.html)
you can make it public. Edit bucket policy to

~~~
{
  "Version":"2012-10-17",
  "Statement":[{
  "Sid":"AddPerm",
        "Effect":"Allow",
    "Principal": "*",
      "Action":["s3:GetObject"],
      "Resource":["arn:aws:s3:::menucards-staging/*"
      ]
    }
  ]
}
~~~

# Add CNAME so your domain name point to your bucket

You bucket needs to have the SAME NAME AS YOUR DOMAIN `some.domain.com` or you
need to use Amazon DNS Route 53.
For default region *US Standard* you can add CNAME record with value
`some.domain.com.s3.amazonaws.com` on your domain provider page.
If bucket is from different region than you need to use full CNAME
**bucket-name**.s3-website[-.]**region**.amazonaws.com .  List of regions you
can find [website
endpoints](http://docs.aws.amazon.com/AmazonS3/latest/dev/WebsiteEndpoints.html)
For frankfurt would be `some.domain.com.s3-website.eu-central-1.amazonaws.com`
You should find your value in Properties of you bucket -> Endpoint.
Your bucket-name can not be different than your domain name!
[link](http://docs.aws.amazon.com/AmazonS3/latest/dev/VirtualHosting.html)

# Access files

Files can we accessed in two ways. One is to go on properties and find link
<https://s3.eu-central-1.amazonaws.com/duleorlovic-test-eu-central-1/icon-menu.png>
In this way (bucketname subfolder) you can access any file with both http and
https protocol. You can also access the file when you enable website hosting.

# Https and website

Another way to access files is to use **Endpoint** 
<http://duleorlovic-test-eu-central-1.s3-website.eu-central-1.amazonaws.com/icon-menu.png>
or short without region, just bucket.s3.amazonaws.com:
<http://duleorlovic-test-eu-central-1.s3.amazonaws.com/icon-menu.png>

But you can access your public site on S3 Endpont only with http (non SSL)
protocol.

For HTTPS you need Cloud Front and Route 53 service
[link](http://knightlab.northwestern.edu/2015/05/21/implementing-ssl-on-amazon-s3-static-websites/)
or <https://www.cloudflare.com/> . You can set up naked domain, just add `CNAME`
with name `@` (or you `domain.com`) and value `Endpoint`.

# Region problems

* [gulp aws-publish]({% post_url 2016-03-02-gulp-tasks %}) for non us-east-1
  (for example eu-central-1, southeast-1) problems arise for bucket name with
  dot. Single world like
  <http://duleorlovic-test-southeast-1.s3-website-ap-southeast-1.amazonaws.com/>
  works fine. Also multiple words on us-east-1 region works fine. Solution is to
  use param `var publisher = awspublish.create({region:
  process.env.AWS_REGION...` and `export AWS_REGION=ap-southeast-1`. It seems
  that region can be us-east-1 in other cases (for example it works for single
  word southeast and region us-east-1)

* ng-s3upload than if your bucket name contains only one word (for example
  <https://duleorlovic-test-us-east-1.s3.amazonaws.com/>) than you can upload
  files with ng-s3upload but if it contains multiple
  (<https://assets.test.trk.in.rs.s3.amazonaws.com/>) than there is
  `ng-s3upload.js:135 OPTIONS https://assets.test.trk.in.rs.s3.amazonaws.com/
  net::ERR_INSECURE_RESPONSE` error.


# CORS

Sometime you need to upload files [directly on S3](
{% post_url 2015-12-24-ionic-direct-aws-s3-upload %}) which means that our
javascript code needs to send data but browser prevents with this error

~~~
XMLHttpRequest cannot load
https://duleorlovic-test-southeast-1.s3.amazonaws.com/. Response to preflight
request doesn't pass access control check: No 'Access-Control-Allow-Origin'
header is present on the requested resource. Origin 'http://localhost:9000' is
therefore not allowed access. The response had HTTP status code 403.
~~~

We need to go Bucket -> Properties -> Permissions -> Add Cors configuration and
add `<AllowedMethod>POST</AllowedMethod>` and `<AllowedHeader>*</AllowedHeader>`
to the provided example:

~~~
<CORSConfiguration>
    <CORSRule>
        <AllowedOrigin>*</AllowedOrigin>
        <AllowedMethod>GET</AllowedMethod>
        <AllowedMethod>POST</AllowedMethod>
        <MaxAgeSeconds>3000</MaxAgeSeconds>
        <AllowedHeader>*</AllowedHeader>
        <AllowedHeader>Authorization</AllowedHeader>
    </CORSRule>
</CORSConfiguration>
~~~


# Tools

Firefor plugin [S3Fox](https://www.youtube.com/watch?v=L1cqzEYYUB0) firefox
organizer is great since you can access specific buckets.

To check your DNS you can use linux command `host`:

*  host frankfurt.trk.in.rs

# aws-sdk

You can use aws-sdk for simple uploading
<https://github.com/duleorlovic/securiPi/blob/master/s3.rb>

# Elastic Beanstalk

Using EB client you need to add permissions for AWSElasticBeanstalkFullAccess
for AWS keys in your env

~~~
sudo pip install awsebcli
eb init
eb create myapp-env
eb setenv SECRET_KEY_BASE=`rake secret`
eb printenv
eb logs
eb deploy
eb open
~~~

Rails sqlite database can't be shared between instances (event when we just
deploy the code). Adding database is one click. Just use proper env var names:

~~~
# config/database.yml
default: &default
  adapter: postgresql
  encoding: unicode
  database: <%= ENV['RDS_DB_NAME'] %>
  username: <%= ENV['RDS_USERNAME'] %>
  password: <%= ENV['RDS_PASSWORD'] %>
  host: <%= ENV['RDS_HOSTNAME'] %>
  port: <%= ENV['RDS_PORT'] %>
~~~

# Admin access

You can create users and attach AdministratorAccess (Resource *) and create
password so they can login in on app url like: `https://123123.signin.aws.amazon.com/console`.

# Errors

* `NoSuchKey` 404 Not Found error is when you remove `index.html` and enable
  static website hosting with index document `index.html`

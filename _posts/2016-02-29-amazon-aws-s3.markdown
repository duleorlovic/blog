---
layout: post
title: Amazon AWS S3
tags: s3
---

# Create IAM user and attach inline policy

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

# Website hosting

You can use your `some.domain.com` to point to your bucket
[guide](https://docs.aws.amazon.com/AmazonS3/latest/dev/website-hosting-custom-domain-walkthrough.html)
This is simple as adding one CNAME record.

Your bucket needs to have the SAME NAME AS YOUR DOMAIN `some.domain.com` or you
need to use Amazon DNS Route 53. Also if you want to serve from root domain
`domain.com` than you must use DNS Route 53.
For default region *US Standard (us-east-1)* you can add CNAME record with value
`some.domain.com.s3.amazonaws.com` on your domain provider page.

But if bucket is from different region than from CNAME you need to use full
**Endpoint** which you can find in Properties of you bucket -> Endpoint.
for CNAME

**bucket-name**.s3-website[-.]**region**.amazonaws.com

List of regions you
can find on [website
endpoints](http://docs.aws.amazon.com/AmazonS3/latest/dev/WebsiteEndpoints.html).
For frankfurt would be `some.domain.com.s3-website.eu-central-1.amazonaws.com`
For southeast region is `ap-southeast-1`.
Your bucket-name can not be different than your domain name!
[virtual hosting](http://docs.aws.amazon.com/AmazonS3/latest/dev/VirtualHosting.html),
[regions and endpoints](http://docs.aws.amazon.com/general/latest/gr/rande.html)

## Mark bucket public

You can deploy frontend code to S3 but you need to make it public. Edit bucket
policy to

~~~
{
  "Version":"2012-10-17",
  "Statement":[{
  "Sid":"AddPerm",
        "Effect":"Allow",
    "Principal": "*",
      "Action":["s3:GetObject"],
      "Resource":["arn:aws:s3:::projektor.trk.in.rs/*"
      ]
    }
  ]
}
~~~

## SSL on Amazon Cloud front

You can request certificate [AWS Certificate
Manager](https://console.aws.amazon.com/acm/). It will send email to the
`administrator@trk.in.rs` with the link to approve. Administrator of domain can
just click on the link to approve it.

Once you have approved certificate you can use it on [cloud
front](https://console.aws.amazon.com/cloudfront/) distribution. Some
[notes](http://docs.aws.amazon.com/gettingstarted/latest/swh/getting-started-create-cfdist.html).

* it could 15 mins for distribution to become deployed
* when selecting *Origin Domain Name* do not select option from dropdown since
  it does not contain region (it will work only for us-east-1 buckets). You need
  to use bucket Endpoint here
* update your DNS record so CNAME is something like
  d1ub4fmsp3scvp.cloudfront.net

* default is [24
hours](http://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/Expiration.html)
for cache to expire... you can try to invalidate some files manually

# Access files

Files can we accessed in three ways.

1. **bucket_name as path**.
   *s3`[-.]region`.amazonaws.com/`bucket-name`/`file-name`*. Go to the properties
   of the file and find *link*, for example:
   <https://s3.eu-central-1.amazonaws.com/duleorlovic-test-eu-central-1/icon-menu.png>.
   You will notice that for us-east-1 region, link does not contain region part,
   only <https://s3.amazonaws.com/duleorlovic-test-us-east-1/favicon.ico>
1. **bucket_name as subdomain**.`bucket-name`.s3.amazonaws.com/`filename`. This
   is shorter since this url does not contain region
   <https://duleorlovic-test-eu-central-1.s3.amazonaws.com/icon-menu.png>
1. enable **website hosting** so you can use *Endpoint* and append fileName
   <http://duleorlovic-test-eu-central-1.s3-website.eu-central-1.amazonaws.com/icon-menu.png>
  * note that url contains `website-` so website needs to be enabled
  * note that this way you can not access using SSL https protocol
  * for HTTPS you need Cloud Front and Route 53 service
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
    </CORSRule>
</CORSConfiguration>
~~~

This error also occurs when we mismatch bucket_name or region. To test if bucket
is working, follow the link from the error.


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
eb status
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

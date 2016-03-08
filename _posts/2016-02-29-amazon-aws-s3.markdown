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
Another is to use **Endpoint** (you need to enable website hosting)
<http://duleorlovic-test-eu-central-1.s3-website.eu-central-1.amazonaws.com/icon-menu.png>

# Tools

Firefor plugin [S3Fox](https://www.youtube.com/watch?v=L1cqzEYYUB0) firefox
organizer is great since you can access specific buckets.

To check your DNS you can use linux command `host`:

*  host frankfurt.trk.in.rs


# Errors

* `NoSuchKey` 404 Not Found error is when you remove `index.html` and enable
  static website hosting with index document `index.html`
* 

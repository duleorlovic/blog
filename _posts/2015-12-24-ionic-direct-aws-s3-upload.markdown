---
title: Ionic & Rails Direct S3 image upload
layout: post
tags: ionic aws s3 angular
---

This is interesting topic since we need to delve with angular and aws specifics.

# AWS Bucket

* create a bucket in USA **vide-uploading-demo**
* create user and download credentials
* go to the user page and create inline
  [policy](http://blogs.aws.amazon.com/security/post/Tx3VRSWZ6B3SHAV/Writing-IAM-Policies-How-to-grant-access-to-an-Amazon-S3-bucket)
  for *Programmatic read and write permissions*
  with name for example **full_access_to_video_uploading_demo** . Keep in mind that
  underscore _ is not equal hyphen - .

Carrierwave and paperclip need something more than put, get and delete, so I added `s3:*` below `"s3:DeleteObject"`.

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
                "arn:aws:s3:::video-uploading-demo"
            ]
        },
        {
            "Effect": "Allow",
            "Action": [
                "s3:PutObject",
                "s3:GetObject",
                "s3:DeleteObject",
                "s3:*"
            ],
            "Resource": [
                "arn:aws:s3:::video-uploading-demo/*"
            ]
        }
    ]
}
~~~

# Record image

Read docs on
[cordova-plugin-camera](https://github.com/apache/cordova-plugin-camera) rather
than on [ngCordova camera](http://ngcordova.com/docs/plugins/camera/).


~~~
bower install ngCordova --save-dev
sed -i '/<\/head>/i \
    <!-- plugins -->
    <script src="lib/ngCordova/dist/ng-cordova.js"></script>\
' www/index.html
ionic plugin add cordova-plugin-camera
~~~

Example app [Angluarjs-Ionic-Schedule-App](https://github.com/sirius2013/Angluarjs-Ionic-Schedule-App)


Android emulator needs SD card before using camera, so add some MB in
`tools/android avd`.

~~~
cordova plugin add cordova-plugin-file-transfer
~~~

[AWS S3](http://aws.amazon.com/articles/1434) conditions can prevent user
uploading on different location.

starts-with should be set for all files...

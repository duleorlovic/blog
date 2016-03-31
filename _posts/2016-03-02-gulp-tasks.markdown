---
layout: post
title: Gulp tasks
tags: gulp
---

# Deploy to S3

You can deploy frontend code on any static web host, like AWS S3. You just need
to add *deploy* task that is using [gulp-awspublish](https://www.npmjs.com/package/gulp-awspublish) `npm install --save-dev gulp-awspublish` .

It has some problems when bucket is from eu-central-1 and contains dots, so read
[aws s3]({% post_url 2016-02-29-amazon-aws-s3 %})

~~~
// gulp/build.js
var awspublish = require('gulp-awspublish');

gulp.task('deploy', ['build'], function() {
  console.log("AWS_BUCKET_NAME=" + process.env.AWS_BUCKET_NAME);
  console.log("AWS_REGION=" + process.env.AWS_REGION);
  console.log("AWS_ACCESS_KEY_ID=" + process.env.AWS_ACCESS_KEY_ID);
  console.log("AWS_SECRET_ACCESS_KEY=" + process.env.AWS_SECRET_ACCESS_KEY);
  if (!process.env.AWS_BUCKET_NAME || !process.env.AWS_ACCESS_KEY_ID || !process.env.AWS_SECRET_ACCESS_KEY) {
    console.error('You must provide bucket name, access key id and secret access key to deploy.');
    process.exit(1);
  }
  
  // create a new publisher using S3 options
  // http://docs.aws.amazon.com/AWSJavaScriptSDK/latest/AWS/S3.html#constructor-property
  var publisher = awspublish.create({
    region: process.env.AWS_REGION,
    params: {
      Bucket: process.env.AWS_BUCKET_NAME
    },
    accessKeyId: process.env.AWS_ACCESS_KEY_ID,
    secretAccessKey: process.env.AWS_SECRET_ACCESS_KEY
  });

  // define custom headers for assets
  var headers = {
    'Cache-Control': 'max-age=0, s-maxage=86400'
  };

  return gulp.src(['dist/**/*'])
    .pipe(publisher.publish(headers))
    .pipe(publisher.sync())
    .pipe(awspublish.reporter());
});
~~~

# Gulp api production or local

To change which api should be used, I use param `--api` like

~~~
gulp serve --api production
gulp serve --api localhost:3000
gulp deploy --api production
~~~

You need to stop eventual gulp serve for localhost, when you want to deploy for
production.

~~~
// gulp/script.js
var replace = require('gulp-replace');
var argv = require('yargs').argv;
var gulpif = require('gulp-if');
var serverUrl = false; // default is what is defined in constants.coffee
if (argv.api == 'production') {
  serverUrl = 'PRODUCTION_SERVER_URL';
} else if (argv.api) {
  serverUrl = '"http://' + argv.api + '"';
}
// add below function buildScripts() like
    .pipe(gulpif(!!serverUrl,replace(/API_URL: \w*SERVER_URL/g, 'API_URL: ' + serverUrl)))
~~~

~~~
# src/app/index.constants.coffee
# do not rename this constants since they are used in gulp scripts task
PRODUCTION_SERVER_URL = 'https://myapp.herokuapp.com'
STAGING_SERVER_URL = 'https://myapp-staging.herokuapp.com'
LOCAL_SERVER_URL = 'http://localhost:3000'
angular.module('myappAngular')
  .constant 'CONFIG',
    API_URL: STAGING_SERVER_URL + '/api/v1'
~~~


---
title: Ionic & Rails: Direct S3 image upload
layout: post
tags: ionic aws angular
---

This is interesting topic since we need to delve with angular and aws specifics.

## Record image

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

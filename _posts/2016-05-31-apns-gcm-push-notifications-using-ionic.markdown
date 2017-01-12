---
layout: page
title: APNS GCM Push Notification using Ionic
---

# APNS

## Compile for iPhone

You can download `vmdk` file and just use with your virtual box
[video](https://www.youtube.com/watch?v=wI3ng69kTD0)
Virtualbox machine settings -> system ->

* Chipset should be PIIX3.
* Enable EFI, but some tutorials says that should be unchecked.

<https://randosity.wordpress.com/2010/06/21/running-mac-os-x-in-virtualbox>

You cannot sign in to iCloud because there was a problem verifying the identity
of this Mac. Try restarting your Mac and signing in again


# GCM

You need to register a project on <https://console.developers.google.com> and
enable "Google Cloud Messaging". Create API key and save as GOOGLE_API_KEY .

You will also need project number, top righ "Project Settings" and go to the
[settings](
https://console.developers.google.com/iam-admin/settings/project?project=cybernetic-tide-90121)
It is not Project ID, but it is just 12 numbers like 123123. Save it as
GOOGLE_PROJECT_NUMBER

You can install Chrome extension to register a client [google chrome example gcm
notifications](https://github.com/GoogleChrome/chrome-app-samples/tree/master/samples/gcm-notifications)

[pushwoosh
extension](https://chrome.google.com/webstore/detail/pushwoosh-gcm-notificatio/ndeeiaalfemhaahglpildclkgilgnfce/related?hl=en)
can register but it can not receive messages.

There is nice explanation how to create notification on your site
<https://developers.google.com/web/updates/2015/03/push-notifications-on-the-open-web?hl=en>


# Server side in ruby

~~~
cat >> Gemfile <<HERE_DOC
# for notifications Google Cloud Messaging
gem 'gcm'
HERE_DOC
bundle

sed -i config/secrets.yml -e '/^test:/i \
  # google cloud messaging\
  google_api_key: <%= ENV["GOOGLE_API_KEY"] %>'

cat >> config/initializers/google-api.rb <<HERE_DOC
MY_GCM = GCM.new Rails.application.secrets.google_api_key
HERE_DOC

rails g model NotificationToken token kind:integer notifiable:references{polymorphic} # user:references
# make sure kind has default 0, since we can not write that in generator
last_migration

# app/models/concerns/notifiable.rb
module Notifiable
  extend ActiveSupport::Concern
  included do
    has_many :notification_tokens, as: :notifiable
  end
  class_methods do
    def send_messages(message = "Hi All from myapp", collapse_key = nil, additional_data = nil)
      tokens = all.map do |user|
        user.notification_tokens.map(&:token)
      end.flatten
      send_to_tokens(tokens, message, collapse_key, additional_data)
    end

    def send_to_tokens(tokens, message, collapse_key, additional_data)
      data = { message: message }.merge additional_data
      data = data.merge collapse_key: collapse_key
      response = MY_GCM.send(tokens, data: data)
      XceednetMailer.internal_notification('push message', tokens: tokens, data: data, response_body: response[:body]).deliver_now if Rails.env.development?
      logger.info response[:body]
      response[:body]
    end
  end

  def send_message(message = "Hi from myapp", collapse_key = nil, additional_data = nil)
    tokens = notification_tokens.map(&:token)
    self.class.send_to_tokens(tokens, message, collapse_key, additional_data)
  end
end

# config/routes.rb
+      resources :notification_tokens

# app/controllers/notification_tokens_controller.rb
+ module Api
+   module V1
+     class NotificationTokensController < ApplicationController
+       prepend_before_action :authenticate_current_user
+       def index
+         @notification_tokens = current_user.notification_tokens
+         render json: @notification_tokens
+       end
+ 
+       def create
+         token = notification_token_params[:token]
+         @notification_token = current_user.notification_tokens
+                                           .where(token: token)
+                                           .first_or_initialize
+         if @notification_token.save
+           render json: @notification_token
+         else
+           render json: @notification_token.errors, status: :bad_request
+         end
+       end
+ 
+       private
+ 
+       def notification_token_params
+         params.require(:notification_token)
+               .permit(:token)
+       end
+     end
+   end
+ end
~~~

# Errors

`NotRegisterd` gcm error response is when token is invalid. When user updates
the app token become invalid.

# Client

Old plugin [PushPlugin](https://github.com/phonegap-build/PushPlugin) has been
replaced with new
[phonegap-plugin-push](https://github.com/phonegap/phonegap-plugin-push).
You can use ngcordova wrappers, but I found it is easier to use directly.

GCM notifications arrives with very small delay on real devices, but on
emulator it could be 10 seconds.
Emulator should be build with Google API support (this is important!).

App has 3 states: not loaded (killed, or back button pressed), background and
foreground. In every state notification will be shown in android status bar.
Clicking on it will load the app again (if it was killed than it will start).
Before, when app is foreground than no notification will appear in android
status bar.
[android.forceShow](https://github.com/phonegap/phonegap-plugin-push/blob/master/docs/API.md#pushnotificationinitoptions)
is true than callback is called only when user click on notification (for me
sometimes it does use callback at all, for foreground and backgroun case), if
false (default) callback is called immediately.

Anyway, when you receive message you can trigger reload for some of your data
based on collapse_key. If collapse key is set than any new message with the same
collapse_key will overwrite old one. There is limit of 4 different collapse_key
at one time.
Even I do not use collapse key, new message will overwrite old one
[stackoverflow](http://stackoverflow.com/questions/27713624/gcm-non-collapsible-message-always-replaces-the-previous-message)

# Phonegap plugin push

[Install](https://github.com/phonegap/phonegap-plugin-push/blob/master/docs/INSTALLATION.md)
all required packages:

~~~
cordova plugin add phonegap-plugin-push --variable SENDER_ID=$GOOGLE_PROJECT_NUMBER
ionic build
android update sdk --no-ui --all --filter "extra-android-m2repository"
~~~

# PushPlugin

~~~
cordova plugin add https://github.com/phonegap-build/PushPlugin.git
ionic state save

# edit www/index.html to add `script-src * 'unsafe-inline' 'unsafe-eval';` to
# the Content-Security-Policy meta

# add `GCM_SENDER_ID: '123123'` to the www/js/app/constants.js

# add `messageService.register();` to  $rootScope.$on('auth:login-success in
# www/js/app.run.js to register new token on user login (validation-success
# is too much, we need just on login)

cat >> www/js/services/message.service.js << HERE_DOC
angular.module('starter')
.service('messageService', function(notifyService, $http, CONFIG, $cordovaPush, $rootScope, $ionicPlatform) {
  this.register = function() {
    $ionicPlatform.ready(function() {
      var androidConfig = {
        senderID: CONFIG.GCM_SENDER_ID,
      }
      notifyService.log({messageService_register: androidConfig});
      $cordovaPush.register(androidConfig).then(function(result) {
        notifyService.log({cordovaPush_register: 'success', result: result});
        // Success
      }, function(err) {
        notifyService.log({cordovaPush_register: 'error'});
        // Error
      });
    });
  };

  $rootScope.$on('$cordovaPush:notificationReceived', function(event, notification) {
    notifyService.log({cordovaPush_notificationReceived: notification.event, notification: notification});
    switch(notification.event) {
      case 'registered':
        if (notification.regid.length > 0 ) {
          saveToken(notification.regid);
          // alert('registration ID = ' + notification.regid);
        }
        break;

      case 'message':
        notifyService.toast('message = ' + notification.message);
        switch(notification.collapse_key) {
         case 'new_order':
           $rootScope.$broadcast('new_order', notification.payload);
           break;
         case 'do_not_collapse':
           break
      }
        break;

      case 'error':
        alert('GCM error = ' + notification.msg);
        break;

      default:
        alert('An unknown GCM event has occurred');
        break;
    }
  });

  // WARNING: dangerous to unregister (results in loss of tokenID)
 //  $cordovaPush.unregister(options).then(function(result) {
 //    // Success!
 //  }, function(err) {
 //    // Error
 //  })

  function saveToken(token) {
    var data = { notification_token: {token: token}};
    $http.post(CONFIG.API_URL + '/notification_tokens', data).then( function(response) {
      notifyService.log({messageService_saveToken: 'sucess', response_data: response.data});
    },
    function (error) {
      notifyService.log({messageService_saveToken: 'error', error: error});
    });
  };
});
HERE_DOC

# add where you want to update data, something like
# +  $scope.$on('new_order', function(event, args) {
# +    notifyService.log({orderController_new_order: args});
# +    getOrders();
~~~

This is usefull notification service since `$log.debug object` does not work.

~~~
// www/js/services/notify.service.js
angular.module('starter')
.service('notifyService', function($cordovaToast, $log) {
  this.toast = function(message) {
    var text = message;
    if (typeof message === 'object')
    {
      text = JSON.stringify(message);
    }
    if (window.plugins && window.plugins.toast)
    {
      $cordovaToast.show(text, 'long', 'center');
    }
    else
    {
      alert(text);
    }
    $log.debug(text);
  };

  this.alert = function(message) {
    // alert stays always as opposite to toast which hides automatically
    var text = message;
    if (typeof message === 'object')
    {
      text = JSON.stringify(message);
    }
    alert(text);
    $log.debug(text);
  };

  this.log = function(message) {
    var text = message;
    if (typeof message === 'object')
    {
      text = JSON.stringify(message);
    }
    $log.debug(text);
  }
});
~~~
# Android Emulator

I found that genymotion works fine. Just install [google play
services](https://github.com/codepath/android_guides/wiki/Genymotion-2.0-Emulators-with-Google-Play-support)
It works for me on Android 5.1. After drag and drop those two files, you need to
logs in and open Google+ which will trigger update of Google play services.
You can use same login on all emulators.
If genymotion emulator dissapears, run `adb kill-server` to clean connections.

For first message you need to wait minute or two. But than it works instantly.

# Service workers

Another cool is service workers <https://developers.google.com/web/fundamentals>


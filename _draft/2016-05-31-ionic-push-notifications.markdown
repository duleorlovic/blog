---
title: Ionic Push Notification
---

# Server

~~~
cat >> Gemfile <<HERE_DOC
# for notifications Google Cloud Messaging
gem 'gcm'
HERE_DOC

sed -i config/secrets.yml -e '/^test:/i \
  # google cloud messaging\
  google_api_key: <%= ENV["GOOGLE_API_KEY"] %>\
  '

cat >> config/initializers/google-api.rb <<HERE_DOC
MY_GCM = GCM.new Rails.application.secrets.google_api_key
HERE_DOC

rails g model NotificationToken token kind:integer user:references
# make sure kind has default 0

# app/models/user.rb
+  has_many :notification_tokens, dependent: destroy
+
+  def self.send_messages(message = "Hi from myapp", collapse_key = 'do_not_collapse')
+    options = { data: { message: message }, collapse_key: collapse_key }
+    tokens = all.map do |user|
+      user.notification_tokens.map(&:token)
+    end.flatten
+    logger.debug user_send_messages_tokens: tokens
+    response = MY_GCM.send(tokens, options)
+    logger.debug response_body: response[:body]
+    response[:body]
+  end
+
+  def send_message(message = "Hi from myapp", collapse_key = 'do_not_collapse')
+    options = { data: { message: message }, collapse_key: collapse_key }
+    response = MY_GCM.send(notification_tokens.map(&:token), options)
+    logger.debug response[:body]
+    response[:body]
+  end

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

# Ionic PushPlugin

Following <http://ngcordova.com/docs/plugins/pushNotifications> you can easilly 
and [PushPlugin](https://github.com/phonegap-build/PushPlugin) feature.
It is very small delay on real devices, but on emulator is slow. Emulator should
be build with Google API support (this is important!).

When app is not loaded (killed, or back button pressed) or when it is runing in
background, notification will be shown in android status bar. Click on it to
load the app again (if it was killed than it will start).
When app is foreground than no notification in android status bar.
Anyway, when you receive message you can trigger reload some of your data based
on collapse_key.

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

* new version is https://github.com/phonegap/phonegap-plugin-push


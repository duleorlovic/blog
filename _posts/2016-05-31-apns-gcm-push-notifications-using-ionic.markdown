---
layout: post
---

# RPush

https://github.com/rpush/rpush

You can test sending notification from firebase
https://console.firebase.google.com/u/1/project/movebase/notification
or using curl as in this example
https://github.com/firebase/quickstart-js/tree/master/messaging#curl

Register with npm package inside webpacker
https://firebase.google.com/docs/web/setup#add-sdks-initialize


If rpush does not work (ie queued: 123 and not queued: 0), you can restart every day
```
0 0 * * * su - ubuntu -c 'cd /home/ubuntu/myapp/current && if ! /home/ubuntu/.rbenv/bin/rbenv exec bundle exec rpush status | grep -q "queued: 0"; then echo `date` restarting because queued is not zero | sudo tee -a /home/ubuntu/myapp/current/log/rpush.log; sudo monit restart rpush;echo restarted; fi' >> /home/ubuntu/myapp/current/log/crontab.log;

```


# Old docs

# APNS

[Overview](https://developer.apple.com/library/content/documentation/NetworkingInternet/Conceptual/RemoteNotificationsPG/APNSOverview.html#//apple_ref/doc/uid/TP40008194-CH8-SW1)
explain how it works. Your server acts as *provider* which stores tokens.
Firebase call tokens as IID Token (Instance ID Token).

> The token itself is opaque and persistent, changing only when a device’s data
> and settings are erased
> Each app instance receives its unique token when it registers with APNs.
> Device tokens change when the user updates the operating system and when a
> device’s data and settings are erased. As a result, apps should always request
> the current device token at launch time.

> If APNs attempts to deliver a notification and the destination device is
> offline, APNs stores the notification for a limited period of time and
> delivers it when the device becomes available again. This component stores
> only the most recent notification per device and per app. If a device is
> offline, sending a new notification causes the previous notification to be
> discarded. If a device remains offline for a long time, all stored
> notifications are discarded.

Maximum size is 4KB. Payload is something like: `{ "aps" : { "alert" : "My
message", "badge": 5, "sound": "default" }, "my_custom_data": "my custom data"
}`

> Silent notifications are not meant as a way to keep your app awake in the
> background, nor are they meant for high priority updates. APNs treats silent
> notifications as low priority and may throttle their delivery altogether if
> the total number becomes excessive. The actual limits are dynamic and can
> change based on conditions, but try not to send more than a few notifications
> per hour.

Silent notification are configured with `{"aps" : { "content-available" : 1 } }`
and not alert, badge or sound keys.

You can add customer actions using `category` in notification, so user responds
before booting whole app.

Feedback service should be used once a day to get tokens that are not longer
active
[docs](https://developer.apple.com/library/content/documentation/NetworkingInternet/Conceptual/RemoteNotificationsPG/BinaryProviderAPI.html#//apple_ref/doc/uid/TP40008194-CH13-SW12)

## How to make invalid apn token

Feedback service will show tokens that are failed after you last checked. This
will not show some invalid tokens that you have generated like: `123123`. You
need to delete your app manually. Also the token would not be failed if you just
delete your app, you need to send message to actually try to use token. After
that you will find it in feedback service.  When you read feedback service than
you need to try again to send to that token to find in in feedback service.

It could be that you use mobile app with different cert than on server. Those
tokens will not be invalidated, just messages will not be delivered.

Running MAC in virtualbox is ok when you buy macOS, There are some youtube video
[video](https://www.youtube.com/watch?v=wI3ng69kTD0) with links to download
`vmdk` file and just use with your virtual box but that is not fair. Than you
will get message like:

> You cannot sign in to iCloud because there was a problem verifying the
> identity of this Mac. Try restarting your Mac and signing in again

# How to generate Push Notification cert

How to generate Push Notification cert

* developer.apple.com - account
* Certificates, Identifiers, … App Ids
* Push notifications -  DEV for testing, production for app store and test
flight -> generate
* Keychain -> Certificate assistant -> generate request for certificate with
authority
* Upload that file to push notification screen on generate Dev/Prod SSL client
form
* Follow steps to generate cert file
* Import that file as a login keychain (Note that it has to have keychain linked
to certificate to be able to export it as a .p12 file)
* Export certificate as a .p12 file
* Terminal, navigate to root folder of .p12

  ~~~
  openssl pkcs12 -in yourcert.p12 -out yourcert.pem -nodes -clcerts
  ~~~

# iPhone tips

* to close any app use double click on home button and swipe up
* sometimes you need to restart Settings app (geard icon) to be able to set up
variables for specific app

# FCM Firebase cloud messaging

Upgrade <https://developers.google.com/cloud-messaging/faq> Note that GCM
(developers.google.com/cloud-messaging) is deprecated so use FCM instead.
For server side in ruby you need to register a project on
<https://console.developers.google.com> and enable "Firebase Cloud Messaging".
Create API key and save as GOOGLE_API_KEY.
Instead of google console you can create a project in firebase console
https://console.firebase.google.com/

Best way to start is video https://www.youtube.com/watch?v=BsCBCudx58g
More info for server https://firebase.google.com/docs/cloud-messaging/server and
error codes https://firebase.google.com/docs/reference/fcm/rest/v1/FcmError

There are two formats of payloads: notification (with eventual data) and data
https://firebase.google.com/docs/cloud-messaging/concept-options#top_of_page
There are fields that needs to be send
https://firebase.google.com/docs/reference/fcm/rest/v1/projects.messages#Notification
* data message, does not contain notification and can trigger onMessageReceived
  when app is both in foreground, background, killed
* notification message, set the `notification` key. It can include optional
  `data` payload which can be consumed by client app. For general notificaiton
  you can use title and body, and for android, you can add other fields.
  https://firebase.google.com/docs/reference/fcm/rest/v1/projects.messages#androidnotification
  ```
  token/topic/condition: "registration_id/weather/foo in topics",

  notification: {
    title: 'Message title',
    body: 'message body',
    icon: 'myicon', // this can override default lancher icon from manifest
    color: '#FFFFFF',
    sound: 'default', // can be my_sound which is located in /res/raw
    tag: 'post_123', // replace existing notification with same tag in drawer
    click_action: 'intent'. // If specified, an activity with a matching intent
                            // filter is launched when a user clicks on the notification. 
    body_loc_key: 'my_key', // key used to localize message
    body_loc_args: [],      // agrs used for localization keys
    title_loc_key: ''.      // same as body
    title_loc_args: [],     // same as body
    channel_id: '',  // Android 8 API 26, notifications are separated by
    channels https://developer.android.com/guide/topics/ui/notifiers/notifications#ManageChannels
  }

  android: {
    collapse_key: 'new_post' // max 4 different keys, only last from the same
                                key will be send on resume
    priority: 'normal', // normal or high (consumes more battery)
    restricted_package_name: 'com.trk.web',
    ttl: '600' //time to live
    data: {
    }
    notification: {
    }
  }
  webpush: {
  }
  apns: {
  }
  ```

For Rails we use https://github.com/spacialdb/fcm gem which use [Legacy http
server
protocol](https://firebase.google.com/docs/cloud-messaging/http-server-ref)
Send message to maximum 1000 tokens [source](https://github.com/spacialdb/fcm#usage)
First argument to `fcm.send` is what is actual `registration_ids` (if it is a
single string than it is converted to array) and it is merged to second options
param. It can contain
* `collapse_key` is used to avoid sending too many same messages when the device
  comes back online, max is 4 different keys
* `priority` normal or high
* `content_available` and `mutable_content` iOS specifics
* `time_to_live` seconds how long it should be on server if device is offline,
  default is 4 weeks

* `data` custom key value that can be used by the app
* `notification` object that sets visible parts of the message. not all keys are
  the same for android and iOS, you can see two tables on
  https://firebase.google.com/docs/cloud-messaging/http-server-ref#notification-payload-support

  ```
  notification: {
    title: 'hello' // title is not visible on iOS phones and tables
    body: 'Body text'
    sound: 'default` // file name from /res/raw (android) or Library/Sounds (ios)
    click_action: // corresponds to category (ios) or activity intent (android)
                  // when app is not in foreground, it will open the app
    body_loc_key: // for localisation purpose
    body_loc_args:
    title_loc_key:
    title_loc_args:

    (ios only)
    badge: 'new',  // if 0 than badge is removed

    (android only)
    android_channel_id: 'channel_id' //
    icon: 'file_name' // file name is from drawable resource
    tag: 'post_123', // it replaces notification with a same tag in drawer but
                      // only for when app is not in foreground
    color: '#ffffff' // icon color
  }
  ```

To receive data and notification in android app you can follow https://www.youtube.com/watch?v=we8eGwIED8Y

```
# config/initializers/fcm.rb
MY_FCM = FCM.new Rails.application.secrets.firebase_cloud_messaging_server_key

# app/services/firebase_cloud_messaging.rb

```

# Firebase web

Create project on firebase https://console.firebase.google.com and on Settings
-> Tab Cloud Messaging -> Web Configuration -> Web Push certificates -> create
key pair. Copy public key as `GOOGLE_VAPID_KEY`

Examples can be found on https://github.com/firebase/quickstart-js
Messaging example you need to update
[YOUR_PUBLIC_VAPID_KEY_HERE](https://github.com/firebase/quickstart-js/blob/master/messaging/index.html#L89)

~~~
npm install -g firebase-tools
firebase login
firebase use --add
firebase serve -p 8001
~~~


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
https://developers.google.com/web/fundamentals/push-notifications/

Docs for [response](https://developers.google.com/cloud-messaging/http#response)

> if the value of failure and canonical_ids is 0, it's not necessary to parse
> the remainder of the response

If there are failures than iterate one by one and if message is some of the:
"Unavailable', 'NotRegistered', 'InvalidRegistration' than you can remove it.
If you notice `registration_id` than you should update it token

# How to make invalid gcm token

You need to clear all your app data.

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
      ApplicationMailer.internal_notification('push message', tokens: tokens, data: data, response_body: response[:body]).deliver_now if Rails.env.development?
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

# Service workers

Another cool thing is service workers
<https://developers.google.com/web/fundamentals>

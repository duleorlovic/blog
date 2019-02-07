---
layout: post
---

# Native login

Since mobile can use sdk to login user, it should be done native (android
activity).
After Firebase auth you should pass the token to the webview (do not pass
`user.getUid()` since that wont change)
https://firebase.google.com/docs/auth/android/manage-users#get_a_users_profile

That accessToken can be validated in ruby
https://medium.com/@igorkhomenko/how-to-validate-firebase-id-token-in-ruby-23f4f54c89ab

There is a firebase-ruby gem but only wrapper for Real time database REST API
https://github.com/oscardelben/firebase-ruby
https://firebase.google.com/docs/database/rest/start

# Turbolinks

https://github.com/turbolinks/turbolinks-android
You just need to use `<com.basecamp.turbolinks.TurbolinksView` and override
`visitProposedToLocationWithAction`. Look at demo app.

# Notifications

Option to use email of mobile app notifications.

# Android Emulator

I found that genymotion works fine. Just install [google play
services](https://github.com/codepath/android_guides/wiki/Genymotion-2.0-Emulators-with-Google-Play-support) after drag and drop those two files, you need to
logs in and open Google+ which will trigger update of Google play services.
You can use same login on all emulators.
If genymotion emulator dissapears, run `adb kill-server` to clean connections.

For first message you need to wait minute or two. But than it works instantly.

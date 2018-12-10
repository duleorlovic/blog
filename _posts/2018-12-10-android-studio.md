---
layout: post
---

# Google Sign in

To connect your app with Firebase you can use Gradle or Assistant.
For using Gradle, you need to copy SHA and paste to firebase console. To get
`Debug signing certificate SHA-1 (optional)` from the Android Studio, you can go
to right toolbox `Gradle` and double click on `root -> Tasks -> android ->
signinReport`. Another way to get sha is running command

```
keytool --exportcert -list -v -alias androiddebugkey -keystore $ANDROID_SDK_ROOT/.android/debug.keystore
Enter keystore password:
# type: android
```

Using assistant, you will grand permission to update your firebase settings, so
you do not need to paste.

There is a collection of apps for quick start
https://github.com/firebase/quickstart-android

`google-services.json` file should be copied to `app/` folder.

# Keyboard shortcuts

Comment with `Ctrl + /` uncomment with `Ctrl + shift + /`

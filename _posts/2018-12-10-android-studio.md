---
layout: post
---

# Install Firebase

`google-services.json` file should be copied to `app/` folder.

There is a collection of apps for quick start
https://github.com/firebase/quickstart-android

When I Import existing project the subfolders, I got error
```
Gradle sync failed: Could not find com.google.gms:google-services:4.2.0.
```

so I need to add this line to gradle https://stackoverflow.com/questions/53706565/error-could-not-find-com-google-gmsgoogle-services4-2-0
```
maven { url 'https://dl.bintray.com/android/android-tools' }
```


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


# Keyboard shortcuts

Comment with `Ctrl + /` uncomment with `Ctrl + shift + /`

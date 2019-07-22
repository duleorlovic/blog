---
layout: post
---

# Install Firebase

There is a collection of apps for quick start
https://github.com/firebase/quickstart-android

`google-services.json` file should be copied to `app/` folder, but you can use
different google services json for different build type
https://developers.google.com/android/guides/google-services-plugin#adding_the_json_file

```
app/google-services.json
app/src/release/google-services.json
```

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

You can register different firebase project with the same package name, but you
need to use different signing key (it is not possible to have same SHA on two
different projects for the same package name). So you need to use another key
(you can keep same package name `com.myapp.app`).

```
keytool --exportcert -list -v -alias key0 -keystore app/myapp_debug.jks
# type the password: myapp123456
# in output find SHA1: 87:16:52:08:C7:B1:BE:BC:73:BE:9E:37:64:67:CB:9D:76:0C:2B:01
```

Add dependencies

```
# build.gradle
buildscript {
  dependencies {
    // Add this line
    classpath 'com.google.gms:google-services:4.0.1'
  }
}

# app/build.gradle
dependencies {
  // Add this line
  implementation 'com.google.firebase:firebase-core:16.0.1'
}

// Add to the bottom of the file
apply plugin: 'com.google.gms.google-services'
```

# Signing config keys

https://coderwall.com/p/zrdsmq/signing-configs-with-gradle-android
Instead of default certificate located at `$HOME/.android/debug.keystore` (which
is different for each user ie sdk installation) you can create your own keys.
That way new developer do not need to add SHA to firebase project, since he can
imediatelly build and run the app in emulator or device.

https://developer.android.com/studio/publish/app-signing
There are `signing` and `upload` key.

You can sign in manually from command line or using gradle https://developer.android.com/studio/publish/app-signing#gradle-sign
Generate `.jks` file from Android studio
~~~
# app/build.gradle
android {
+    signingConfigs {
+      debug {
+          // You need to specify either an absolute path or include the
+          // keystore file in the same directory as the build.gradle file.
+          storeFile file("mykey.jks")
+          storePassword "asdf"
+          keyAlias "key0"
+          keyPassword "asdf"
+      }
+    }
+
     buildTypes {
         release {
             minifyEnabled false
             proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
         }
+        debug {
+            signingConfig signingConfigs.debug
+
+        }
     }
~~~

Release to App Store https://www.youtube.com/watch?v=AWawL5HFn64

# Keyboard shortcuts

* Comment with `Ctrl + /` uncomment with `Ctrl + shift + /`
* `Alt + Enter` when you want to import something that can not be resolved (when
  you paste some code snipets))
* fix code indent with `Ctrl + Alt + Super + L`

# Webview

https://developer.android.com/guide/webapps/webview#java
Start new app with blank intent.
In `app/serv/main/AndroidManifest.xml` add permission for internet and also for
http traffic https://stackoverflow.com/questions/45940861/android-8-cleartext-http-traffic-not-permitted/50834600#50834600

```
<manifest...
    <uses-permission android:name="android.permission.INTERNET"></uses-permission>
    <application
        ...
        android:usesCleartextTraffic="true"
```

In java you need to enable javascript in webview, set user agent so you can
customize the response.

https://github.com/duleorlovic/android-webview-example/commit/3097878534c019264f58050612a2d5564267b63d


```
public class MainActivity extends AppCompatActivity {

    private WebView myWebView;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        myWebView = (WebView) findViewById(R.id.webview);
//        setContentView(myWebView);
        WebSettings webSettings = myWebView.getSettings();
        webSettings.setJavaScriptEnabled(true);
        webSettings.setUserAgentString("myapp");
        myWebView.setWebViewClient(new MyWebViewClient());
        myWebView.addJavascriptInterface(new WebAppInterface(this), "Android");

        myWebView.loadUrl("http://192.168.5.4:3001/");
    }

    @Override
    public boolean onKeyDown(int keyCode, KeyEvent event) {
        // Check if the key event was the Back button and if there's history
        if ((keyCode == KeyEvent.KEYCODE_BACK) && myWebView.canGoBack()) {
            myWebView.goBack();
            return true;
        }
        // If it wasn't the Back key or there's no web page history, bubble up to the default
        // system behavior (probably exit the activity)
        return super.onKeyDown(keyCode, event);
    }
}

public class WebAppInterface {
    Context mContext;

    /** Instantiate the interface and set the context */
    WebAppInterface(Context c) {
        mContext = c;
    }

    /** Show a toast from the web page */
    @JavascriptInterface
    public void showToast(String toast) {
        Toast.makeText(mContext, toast, Toast.LENGTH_SHORT).show();
    }
}
```

* to change launcher icon and other icons you can go to File -> New -> Image
  Asset and than select "Lanucher icons" and add svg image file. Repeat proccess
  but select "Notification icons". You can see `git status` that existing files
  will be updated, for example `app/src/main/res/mipmap-hdpi/ic_launcher.png`
  insteaad of mipmap-hdpi, there is mipmap-mdpi/xhdmpi/xxhdpi/xxxhdpi density
  and ic_launcher_foreground.png/ic_launcher_round.png
  There is `app/src/main/res/values/ic_launcher_background.xml` which is some
  `vector` but do not know how to use it.
  For notification icon path is `app/src/main/res/drawable/ic_notificaiton.png`

# Const for server Url

When you build `release` (Build -> Select build variant -> release), by default
`serverUrl` is targeting production server `https://myapp.herokuapp.com`.
When you want to target local Rails server than choose `debug` and update
`app/src/main/java/com/myapp/app/Const.java`, by default is
`http://192.168.5.4:3001`.

```
package com.myapp.app;

import android.content.Context;
import android.content.pm.ApplicationInfo;

public class Const {
   // Use for example
   // String mUrl = Const.getServerUrl(getApplicationContext());
   //
    public static String getServerUrl(Context context) {
        boolean isDebuggable = (0 != (context.getApplicationInfo().flags
                & ApplicationInfo.FLAG_DEBUGGABLE));
        String mUrl;
        if (isDebuggable) {
            mUrl = "http://192.168.5.4:3001";
        } else {
            mUrl = "https://myaapp.herokuapp.com";
        }
    return(mUrl);

    }
}
```

# Release new version on Play store

Increment `android { defaultConfig { versionCode X }}}` in `app/build.gradle`.
(Module: app). Than go to Build -> Generate Signed Bundle or APK -> Apk ->
Choose
`app/myaapp.jks` and add passwords from gradle config. Upload
`app/release/app-release.apk` to Google Play console , Release Management -> App
preleases -> Production Track -> Manage -> Create Release or Edit Release
https://play.google.com/apps/publish/

Note that Playstore is somehow cached on Emulator, so install from real device
when you want to checound latest release.

# Errors

When I Import existing project the subfolders, I got error
```
Gradle sync failed: Could not find com.google.gms:google-services:4.2.0.
```

so I need to add this line to gradle https://stackoverflow.com/questions/53706565/error-could-not-find-com-google-gmsgoogle-services4-2-0
```
maven { url 'https://dl.bintray.com/android/android-tools' }
```

For error
```
  emulator: ERROR: x86 emulation currently requires hardware acceleration!
Please ensure KVM is properly installed and usable.
CPU acceleration status: This user doesn't have permissions to use KVM (/dev/kvm)
```
fix is `sudo chown orlovic -R /dev/kvm`

# Tips

* add loader before page is shown https://android.jlelse.eu/loading-splash-screen-for-webview-in-android-studio-ef68ec05720a
* if you have multiple andoird icons

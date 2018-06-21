Hat eigentlich nicht mit Node zu tun...
Auﬂer das es ein Node Programm (cordova) benutzt zum erstellen

Anleitung von: https://zolmeister.com/2014/01/how-to-turn-webapp-into-native-android.html

===Step 1 - Hello World application===
# go into your project
cd myapp
# create a mobileapp folder for your app
cordova create mobileapp com.whatever.appname AppName
cd mobileapp
# add android
cordova platform add android
# compile the app
cordova build
# now, plug in your testing device, and let's run our test app on it
cordova run android
# if you don't have a device handy you can use an emulator (much slower)
## cordova emulate android

# install plugins for alerts and network information
# used to alert the user if they are not connected to the internet
cordova plugin add https://git-wip-us.apache.org/repos/asf/cordova-plugin-dialogs.git
cordova plugin add https://git-wip-us.apache.org/repos/asf/cordova-plugin-network-information.git

===Step 2 - Portrait mode only===
Now let's edit our android manifest to force the app to stay in portrait mode, edit:

platforms/android/AndroidManifest.xml
and add this config to it (on <activity>):

android:screenOrientation="portrait"

===Step 3 - Content===
Finally we get to adding our website. Edit your index.html to look similar to mine:

www/index.html
<!doctype html>
<html lang="en">
<head>

  <title>Zoggle</title>
  <script type="text/javascript" charset="utf-8" src="cordova.js"></script>
  <script>
  document.addEventListener("deviceready", onDeviceReady, false);
  function onDeviceReady() {
    //navigator.splashscreen.hide();
    if (navigator.network.connection.type == Connection.NONE) {
      networkError()
    } else {
      loadApp()
    }
  }

  function loadApp() {
    navigator.app.loadUrl("http://zoggle.zolmeister.com")
  }

  function networkError() {
    navigator.notification.alert('Zoggle requires an internet connection')
    var $net = document.createElement('div')
    $net.innerHTML = 'Zoggle requires an internet connection'
    document.body.appendChild($net)
  }
  </script>

  <style>
  body {
    padding: 15px;
    background: #23252e;
    color: #01ced3;
    text-align: center;
  }
  div {
    font-size: 20px;
  }
  </style>
</head>
<body>
</body>
</html>

===Step 3 - Icons===
Lastly we need to add icons for our application.
You will find all icons here:

platforms/android/res
Just replace them with your icon (of the correct size).
And that's it. Now lets look into compiling for release on the app store.

===Step 4 - Publishing!===
Now we can generate a release version of the APK

cordova build --release
Your APK file should be located here:

platforms/android/ant-build/MyApp-release-unsigned.apk
To submit it to the app store, we need to sign it (cryptographically). This page details how to do that: http://developer.android.com/tools/publishing/app-signing.html but the short version is this:

# generate a keystore
keytool -genkey -v -keystore my-release-key.keystore -alias alias_name -keyalg RSA -keysize 2048 -validity 10000
# sign the apk
jarsigner -verbose -sigalg SHA1withRSA -digestalg SHA1 -keystore my-release-key.keystore MyApp-release-unsigned.apk alias_name
# zip-align the apk
zipalign -v 4 MyApp-release-unsigned.apk MyApp.apk
And that's it! You can now upload that APK to Google play and publish your application.


===Bonus - Splash Screen===
I created a splash screen for Zoggle, but the game loaded so quickly that it became unnecessary. However it was a bit tricky, so I'll go ahead and explain the process.

First install the cordova plugin

cordova plugin add https://git-wip-us.apache.org/repos/asf/cordova-plugin-splashscreen.git
Then edit your main activity Java file. Mine was here:

platforms/android/src/com/zolmeister/zoggle/Zoggle.java
And add this line:

super.setIntegerProperty("splashscreen", R.drawable.splash);
Then, in the app code above (www/index.html), uncomment the following line:

navigator.splashscreen.hide();
which will hide the splashscreen once the app loads.

===Creating a splash screen image===
Splash screen images are not regular images, instead they are 9-patch images. This allows them to stretch to meet different screen sizes. Here is a great video introduction to how they work:

http://www.youtube.com/watch?v=MQRA9nwxU7g The Android SDK tools come with a tool called draw9patch, which you can read about here: http://developer.android.com/tools/help/draw9patch.html

It's not the easiest to use, but it works. Simply run it, open your image, and zoom in on an edge of your image. The 1 pixel transparent border on your image will be the guide which lets the device know how to stretch the image.

Here is mine: 

The black lines mean the the content there WILL BE STRETCHED (so long to figure this out...). This means if you don't want your logo to be distorted, draw the sections around the outside of your logo. Lastly, make sure your image ends in '.9.png' to let the device know that it is a 9-patch image. Then simply put it inside the folder next to your icon:

platforms/android/res/drawable/splash.9.png
===Done! Now go check out Zoggle!===
---
layout: post
title: unsigned.apk 自簽成 signed.apk
date: 2017-04-26
tags:  unsigned android signed
---

建立一個 meteor android hello 的 APK 來測試

```
# id
uid=0(root) gid=0(root) groups=0(root)
# mkdir /root/build
# meteor --allow-superuser create hello
# docker run --rm -it -v /root/hello:/app -v /root/build:/build echochio/meteor-android-build
# ls -l /root/build/android/
total 476
drwxr-xr-x 13 root root   4096 Apr 25 13:29 project
-rw-r--r--  1 root root    231 Apr 25 13:29 README
-rw-r--r--  1 root root 479145 Apr 25 13:29 release-unsigned.apk
```

ps : 要先安裝 java jdk

再來 產生 key
```
# keytool -genkey -v -keystore key/my-release-key.keystore -alias alias_name -keyalg RSA -keysize 2048 -validity 100000
Enter keystore password:
Re-enter new password:
What is your first and last name?
  [Unknown]:  chio
What is the name of your organizational unit?
  [Unknown]:  chio
What is the name of your organization?
  [Unknown]:  chio
What is the name of your City or Locality?
  [Unknown]:  chio
What is the name of your State or Province?
  [Unknown]:  chio
What is the two-letter country code for this unit?
  [Unknown]:  tw
Is CN=chio, OU=chio, O=chio, L=chio, ST=chio, C=tw correct?
  [no]:  y

Generating 2,048 bit RSA key pair and self-signed certificate (SHA256withRSA) with a validity of 100,000 days
        for: CN=chio, OU=chio, O=chio, L=chio, ST=chio, C=tw
Enter key password for <alias_name>
        (RETURN if same as keystore password):
[Storing key/my-release-key.keystore]
```

再將 key 加入 unsigned.apk
```
 # jarsigner -verbose -keystore key/my-release-key.keystore unsigned.apk alias_name
Enter Passphrase for keystore:
 updating: META-INF/MANIFEST.MF
   adding: META-INF/ALIAS_NA.SF
   adding: META-INF/ALIAS_NA.RSA
  signing: AndroidManifest.xml
  signing: assets/cdvasset.manifest
  signing: assets/www/application/d2ac207fb19c704a4887e76224708a851e370591.js
  signing: assets/www/application/head.html
  signing: assets/www/application/index.html
  signing: assets/www/application/program.json
  signing: assets/www/cordova-js-src/android/nativeapiprovider.js
  signing: assets/www/cordova-js-src/android/promptbasednativeapi.js
  signing: assets/www/cordova-js-src/exec.js
  signing: assets/www/cordova-js-src/platform.js
  signing: assets/www/cordova-js-src/plugin/android/app.js
  signing: assets/www/cordova.js
  signing: assets/www/cordova_plugins.js
  signing: assets/www/plugins/cordova-plugin-meteor-webapp/www/webapp_local_server.js
  signing: assets/www/plugins/cordova-plugin-splashscreen/www/splashscreen.js
  signing: assets/www/plugins/cordova-plugin-statusbar/www/statusbar.js
  signing: classes.dex
  signing: res/drawable-land-hdpi-v4/screen.png
  signing: res/drawable-land-mdpi-v4/screen.png
  signing: res/drawable-land-xhdpi-v4/screen.png
  signing: res/drawable-port-hdpi-v4/screen.png
  signing: res/drawable-port-mdpi-v4/screen.png
  signing: res/drawable-port-xhdpi-v4/screen.png
  signing: res/mipmap-hdpi-v4/icon.png
  signing: res/mipmap-mdpi-v4/icon.png
  signing: res/mipmap-xhdpi-v4/icon.png
  signing: res/mipmap-xxhdpi-v4/icon.png
  signing: res/mipmap-xxxhdpi-v4/icon.png
  signing: res/xml/config.xml
  signing: resources.arsc
jar signed.
```

檢查 檔案 (unsigned -> signed)
```
# jarsigner -verify unsigned.apk
jar verified.
```

改檔名可發行了 .....

```
 # mv unsigned.apk hello.apk
```

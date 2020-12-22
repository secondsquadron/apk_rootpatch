# How to patch android apps which detect rooted system

I've run into apps which checked for being rooted or have a custom ROM.
As Magisk was not always an option I decided to patch these apps.


## Guide for retrieving, decompiling, modifying and recompiling an app

- **Get it from the phone**



    adb shell pm list packages | grep <app>
    adb shell pm path <app.package.name>
    adb pull data/app/<app.package.name>-<someid>==/base.apk


- **First install framework, then decompile and test if it builds**\
 ('-f -r and aapt2' used as for me apk does not build by default)


    apktool if <app.package.name>
    apktool d -f -r --use-aapt2 <app.package.name>.apk
    apktool b -f -r --use-aapt2 <app.package.name>

- **Modify the code**

>    Search for root checker libraries in _smali_classes_ directory.
    Usually they use some popular library like _rootbeer_. Check the consts as most probably they contain the blacklisted super user apps' package names. They also test super user by su binary, test-keys method, or some other way, take care of them as well. Overwriting them with some non-existent package name, file path is enough.
    After you have edited the code just build the app.

- **Create keystore first then sign**


    keytool -genkey -v -keystore my-release-key.keystore -alias android_rel -keyalg RSA -keysize 2048 -validity 10000
    jarsigner -verbose -sigalg SHA1withRSA -digestalg SHA1 -keystore my-release-key.keystore <app>/<app.package.name>/dist/<app.package.name>.apk android_rel
    zipalign -v 4  <app>/<app.package.name>/dist/<app.package.name>.apk <app.package.name>_signed.apk

- **Upload and install or install directly**


    adb push <app>/<app.package.name>_signed.apk /sdcard/
    adb -d install -r <app.package.name>_signed.apk

- **Check for diffs**


    diff -q -r <app.package.name>.apk <app.package.name>.apk_
    diff -Nau <files> > file.patch
    diff -Naur <app.package.name> <app.package.name>_

---
layout: page
title: CyberThreat18 CTF challenge write-up - "Binary A"
permalink: /post/cyberthreat18-ctf-ba/
excerpt: Write-up of one of the CTF challenges from CyberThreat18, specifically we will be pulling apart an Android application, patching out some of the code behind the app, and putting it back together so we can run the patched version.
---

## Introduction
I recently attended a new cyber security conference in London called [CyberThreat18](https://www.cyberthreat2018.com/) hosted by the National Cyber Security Centre and SANS Institute.

Over the two-day period, the event included a Capture The Flag (CTF) competition, broken into four sessions, in which teams and individuals raced to crack the challenges and collect the most points.

This is a write-up of one of the challenges called "Binary challenge A", and the methods used here were taken from [an excellent two-part blog post](https://pen-testing.sans.org/blog/2015/06/30/modifying-android-apps-a-sec575-hands-on-exercise-part-1) series by [@edskoudis](https://twitter.com/edskoudis) on the SANS Penetration Testing blog.

## Challenge description
We're given a link to download a zip file which contains the challenge assets; a single Android application package file (APK) named `ncscpin.apk`.

Installing the application into a [Genymotion](https://www.genymotion.com/) virtual Android device (helpfully, Genymotion starts an instance of ADB server with the virtual device available, so we can just do `adb install ncscpin.apk` and we're good to go) we end up with an app called NCSCPin. Firing it up, we are presented with a 5-digit pin entry dialogue. Entering a random guess such as `12345` gives us a toast notification informing us that the pin was incorrect. Moreover, it doesn't seem like you can attempt to enter the PIN twice without closing the application and killing it from it's suspended mode.

![NCSCPin app with invalid PIN entered](/images/ct18-ctf-ba-1.png)

Clearly brute-forcing it is out of the question, so it's time to get creative.

## Pulling apart the APK
Using the excellent [Apktool](https://ibotpeaches.github.io/Apktool/) we can pull apart the APK and have the application's dex bytecode decompiled into smali files.

```
$ apktool d -r ncscpin.apk
I: Using Apktool 2.3.1 on ncscpin.apk
I: Copying raw resources...
I: Baksmaling classes.dex...
I: Copying assets and libs...
I: Copying unknown files...
I: Copying original files...
$ tree -L 1 ncscpin
ncscpin
├── AndroidManifest.xml
├── apktool.yml
├── lib
├── original
├── res
├── resources.arsc
└── smali

4 directories, 3 files
```

Note the use of the `-r` flag to avoid decoding the package's resources - we don't need to modify these, and you can sometimes hit issues trying to rebuild the APK if the resources have been decoded. By not decoding them, they will simply be copied in their original form when rebuilding the package.

Apktool has created a few directories for us here, and the one we're interested in is `smali`, which is where it has placed the decompiled application code.

## Patching the app

Of interest to us in particular are the two `MainActivity` files in `smali/com/example/mainuser/ncscpin` - these are essentially the equivalent of an entrypoint for an Android application.

In `MainActivity.smali` we can see a virtual method defined called `isPinCorrect` - sounds interesting!

```
.method public native isPinCorrect(Ljava/lang/String;)Z
.end method
```

In `MainActivity$1.smali` we can see where this function is called from and a conditional jump on the value it returns:

```
invoke-virtual {v0, v1}, Lcom/example/mainuser/ncscpin/MainActivity;->isPinCorrect(Ljava/lang/String;)Z

move-result v0

if-eqz v0, :cond_0
```

By changing the conditional jump instruction from `if-eqz` to `if-nez` we can essentially reverse the PIN validation so that entering anything _other_ than the "correct" PIN (which we don't know) will be treated as a _correct_ attempt.

## Rebuilding, signing, installing

Now that we've made our desired change, we can rebuild the package, again using Apktool, which places the rebuild package in the `dist` folder under our top level extraction directory.

```
$ apktool b ncscpin
I: Using Apktool 2.3.1
I: Checking whether sources has changed...
I: Smaling smali folder into classes.dex...
I: Checking whether resources has changed...
I: Copying raw resources...
I: Copying libs... (/lib)
I: Building apk file...
I: Copying unknown files/dir...
$ tree ncscpin/dist/
ncscpin/dist/
└── ncscpin.apk

0 directories, 1 file
```

Unfortunately, we can't just deploy this new APK file as it currently stands - attempting to do so gives us the following error

```
$ adb install ncscpin/dist/ncscpin.apk
adb: failed to install ncscpin/dist/ncscpin.apk: Failure [INSTALL_PARSE_FAILED_NO_CERTIFICATES: Failed to collect certificates from /data/app/vmdl1069109607.tmp/base.apk: Attempt to get length of null array]
```

We need to sign our APK file, and in order to do that we need to generate a key and certificate for ourselves. We'll do the key and certificate generation and signing using the Java `keytool` and `jarsigner` tools respectively.

```
$ keytool -v -genkey -keystore ncscpin.keystore \
>            -alias NCSCPin \
>            -keyalg RSA \
>            -keysize 1024 \
>            -sigalg SHA1withRSA \
>            -validity 356
Enter keystore password:  
Re-enter new password:
What is your first and last name?
  [Unknown]:  Chris Moore
What is the name of your organizational unit?
  [Unknown]:  
What is the name of your organization?
  [Unknown]:  
What is the name of your City or Locality?
  [Unknown]:  
What is the name of your State or Province?
  [Unknown]:  
What is the two-letter country code for this unit?
  [Unknown]:  
Is CN=Chris Moore, OU=Unknown, O=Unknown, L=Unknown, ST=Unknown, C=Unknown correct?
  [no]:  yes

Generating 1,024 bit RSA key pair and self-signed certificate (SHA1withRSA) with a validity of 356 days
        for: CN=Chris Moore, OU=Unknown, O=Unknown, L=Unknown, ST=Unknown, C=Unknown
[Storing ncscpin.keystore]
```

Now we can use this keystore to sign our APK file

```
$ jarsigner -keystore ncscpin.keystore \
>           -sigalg SHA1withRSA \
>           -digestalg SHA1 NCSCPin \
>           ncscpin/dist/ncscpin.apk
Enter Passphrase for keystore:
jar signed.

Warning:
The signer's certificate is self-signed.
No -tsa or -tsacert is provided and this jar is not timestamped. Without a timestamp, users may not be able to validate this jar after the signer certificate's expiration date (2019-02-21).
```

Now that we've signed our APK file, we can install it into our virtual Android device. Note that we have to uninstall the existing application first, as we haven't gone to the effort of giving the application a valid update path.

```
$ adb uninstall com.example.mainuser.ncscpin
Success
$ adb install ncscpin/dist/ncscpin.apk
Success
```

Finally, we can fire up our modified version of the application inside the virtual Android device, enter our dummy PIN of `12345` and we get a toast notification containing the flag of `HooksArentJustForPirates`!

![Modified version of the application revealing the flag](/images/ct18-ctf-ba-2.png)

## Closing remarks
A massive thank you goes to the team at [Helical Levity](https://twitter.com/HelicalLevity) for putting together the pre-conference challenges and this CTF, as well as to [James Lyne](https://twitter.com/jameslyne) and the folks at [SANS EMEA](https://twitter.com/SANSEMEA) and the [National Cyber Security Centre](https://twitter.com/ncsc) for making this awesome conference happen.

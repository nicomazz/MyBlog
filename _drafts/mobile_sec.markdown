---
layout: single
title:  Mobile security for busy people
date:   2019-11-13 23:29:25  +0100
categories: security 
comments: true

---

At the beginning of December 2019 I'll have an interview for mobile apps
security engineer at Facebook, and I think I don't know a lot about it.
I'll write here what I learn.

## OWASP Mobile top 10

An important source is the OWASP Mobile top 10, but is rather boring to read, so I dropped it after
the first 2 vulnerabilities, that are:

1. Improper platform usage: It consists in the misusage of the platform api, with some form of insecure
coding

2. Insecure data storage: An adversary can access all the data having physical access to the phone due to
bad storage standards.

Then I found this wonderfull book: [Mobile Security Testing Guide](https://mobile-security.gitbook.io/).
This post can be seen as a short summary of the main points of it.te

## Android platform Overview

### Java compilation for android

The usual compilation for java is:

```
Java source code => Java bytecode => Java VM
```

For Android apps, the java bytecode is compiled with **DEX** compiler, to produce dalvik
bytecode, interpreted by the Dalvik VM
```
Java source code => Java bytecode => Dalvik bytecode => Java VM
```

In the past the code was translated in machine code just-in-time (JIT). With [ART](https://source.android.com/devices/tech/dalvik) (Android Run time) the process is done AOT (Ahead-of-time), improving app performance.

### Android Users and groups

In Android each app runs with a separate Linux user. There are several predefined users.
> User id < 10.000 are reserved for the system. User app are in the between 10000 and 99999.
If an app has a specific permission, its user id is added to the permission group. e.g.:
```bash
$ id
uid=10188(u0_a188) gid=10188(u0_a188) groups=10188(u0_a188),3003(inet)...
```
Here the app belongs to the group **3003**, so has `android.permission.INTERNET` permission.
Relations between group IDs and permissions are [here](http://androidxref.com/7.1.1_r6/xref/frameworks/base/data/etc/platform.xml)

### The App sandbox

Each app has access to `/data/[package-name]`, with permissions similar to:

```
drwx------  6 u0_a120             u0_a120             4096 2017-01-19 12:54 com.android.chrome
```

If two apps are signed with the same certificate and has the same sharedUserId, such as

```xml
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
  package="com.android.nfc"
  android:sharedUserId="android.uid.nfc">
```

then they can also access each other's data directory.

### App bundles

Bundles bundle only the resources necessary for the target architecture and system (eg. you don't have the arm compiled libraries if you are x86_64)

To create an APK from a bundle:

```bash
$ bundletool build-apks --bundle=/MyApp/my_app.aab --output=/MyApp/my_app.apks
```

### Android Manifest

It describes the app structure, and it is the resource you want to look at when you start whatever you want to start.
Here several things are available:

- The Activity that is lunched when you click on the app, containing: `<action android:name="android.intent.action.MAIN" />`
- Package name
- Target version
- All the services
  - If a service has `android:exported="true"` then it is accessible also from other apps (true by default!)
- Broadcast receivers
- All the Activities
- The permissions requested (The most sensible have to be requested at runtime)
  - There are several prodection levels: Normal (Internet), Dangerous (RECORD_AUDIO), Signature (ACCESS_MOCK_LOCATION), SystemOrSignature (ACCESS_DOWNLOAD_MANAGER)

[Here](https://developer.android.com/guide/topics/manifest/manifest-intro.html) The full documentation.

### Broadcast receiver

An Intent can be sent everywhere. If it has to be received only within the sending app, it's better to use `LocalBroadcastManager`.

## Tools

### Root

Obtain it with Magisk. The emulator is already rooted.

> IMPORTANT: the emulator is typically x86 (because also your computer is probably x86_64), while ~all the android devices outside are arm. These are two different architectures. Sometimes an apk doesn't have the libraries needed to work in both architectures, so you can't open the app with the emulator, per example. There are amd emulators running on x86, but they are incredibly slow.

There are so many tools. The one that I prefer are:

- Frida: it allows dynamic instrumentation of running android apps. It both works with or without root, but with root is easier and less tedious to setup. You can scan the heap searching for specific objects, or modify the implementation of real java methods.

## Storing data

The most used techniques are:

- Shared Preferences. Small key-values pairs. If the database is opened with  `MODE_WORLD_READABLE` then all the other apps (so users) have the permissions to read it. However, those modes are now deprecated.
- SQLite Databases: By default it is unencrypted (even if it is in the private app directory). But it can also be encrypted.
- Firebase Database: Many people don't change the security settings. [Firebase scanner](https://github.com/shivsahni/FireBaseScanner) can be used to find the details for the firebase connection.
- Realm Databases: can be encrypted or not.
- Internal Storage
- External Storage: The permission `WRITE_EXTERNAL_STORAGE` can be an eloquent indicator this is used

### Common sources for encrpytion keys

- Data somewhere and a function that combine those data (e.g. xor)
- `res/values/strings.xml`
- Build configurations
- SharedPreferences. Even if they are plaintext, there are alternatives such as [secure-preferences](https://github.com/scottyab/secure-preferences). However, they aren't bullet proof, but it's more obfuscation. An attacker can change those values, so it is necessary to implement some form of input validation when those are read back.
- Android KeyStore:
  - Sometimes the key is handled in a Trusted Execution Environment (TEE) or Secure Element (SE).
  - if the implementation is software-based, all the keys are accessible from `/data/misc/keystore/` with root permissions

### Where to check

So, the places where to check are:

- sql databases: `/data/data/<package-name>/databases`
- interesting pref values: `/data/data/<package-name>/shared_prefs`
- Realm databases `/data/data/<package-name>/files/`. I fthere is something, then [Realm Browser](https://github.com/realm/realm-browser-osx)(only ios sadly) can be used.

### Content provider

They can be queried as if they were a real database, with a custom implementation. They can define custom paths and the permissions in each of them. with `drozer` we can automatically find all the providers vulnerable for things such as SQLi

```bash
dz> run scanner.provider.injection -a com.mwr.example.sieve
```

They can be queried directly with adb (eg. `$ adb shell content query --uri content://com.owaspomtg.vulnapp.provider.CredentialProvider/credentials`), and modifiyed with drozer.


### Backup options

- `adb backup` creates a full backup app's data directory (USB debugging has to be enabled). 
  - The `allowBackup` flag has to be enabled in the manifest. (it is by default)
  - `dd if=mybackup.ab bs=24 skip=1|openssl zlib -d > mybackup.tar` to have a tar from the .ab file
- Online backup
- Backup API: key/values backups (max 5MB) or auto backups (max 25MB using user's Google drive account)

### Avoid screenshot for the task manager

Using this, it is impossible to capture screenshots.

```java
getWindow().setFlags(WindowManager.LayoutParams.FLAG_SECURE,
                WindowManager.LayoutParams.FLAG_SECURE);
```

###  Sensitive data in memory

If you have to deal with secret keys, or buffers with secret information, it is better not to use `StringBuffers`, because after the use the data will stick somewhere, and it is hard to delete them.

In general, it is impossible to clear immutable types: as a consequence they will remain around until the GC will clear them. It is better to use primitive types, and then clear them soon after the usage.

Another interesting thi bong is that, even if the secret data array is cleared, it is possible that the bytecode optimizer doesn't really do the clear stage, because the array is never used afterwards.

To gather the memory dump it iis possible to use `objection` or `Fridump`. More info on how to do a memory dump [here](https://mobile-security.gitbook.io/mobile-security-testing-guide/android-testing-guide/0x05c-reverse-engineering-and-tampering#memory-dump).
However, it is also possible to use Android Studio to dump and analize the Java Heap.

Further analysis, also with a SQL-like language can be done with Eclipse Memory Analyzer Tool (MAT). eg:
```sql
SELECT * FROM byte[] b WHERE toString(b).matches(".*1\.2\.840\.113549\.1\.1\.1.*")
```

## Android Cryptographic APIs

### Random number generator

`SecureRandom` should be used without any arguments in the constructor. If the default java random with a number in the construction is used, then it is possible to guess the numbers.

..There are many other things that can be useful


## Android Network API

### todo

## Android Platform API

### Permissions

Apps must explicitly request access to resources and data that are outside their sandbox trought specific permissions such as INTERNET, RECORD_AUDIO, ACCESS_MOCK_LOCATION, ACCESS_DOWNLOAD_MANAGER.

- It is possible to give permissions to activities, services and broadcast receivers. This restricts which apps can use the components. If they don't have the needed permissions, a `SecurityException` is thrown.
- For content providers, there is `android:readPermission` and `android:writePermission`. ContentProviders also extends the default permission model, allowing specific permissions for each URI.

#### Custom permissions

It is possible to create custom permissions: an example can be a new permission needed to lunch a specific activity.

```xml
<permission android:name="com.example.myapp.permission.START_MAIN_ACTIVITY"
        android:label="Start Activity in myapp"
        android:description="Allow the app to launch the activity of myapp app, any app you grant this permission will be able to launch main activity by myapp app."
        android:protectionLevel="normal" />

<activity android:name="TEST_ACTIVITY"
    android:permission="com.example.myapp.permission.START_MAIN_ACTIVITY">
    <intent-filter>
        <action android:name="android.intent.action.MAIN" />
        <category android:name="android.intent.category.LAUNCHER"/>
     </intent-filter>
</activity>
```

It is possible to use the Android Asset Packaging tool (aapt) to examine the permissions needed from a package: `adb shell pm list packages -f | grep -i <keyword>`, or, with a more detailed result: `adb shell dumpsys package sg.vp.owasp_mobile.omtg_android | grep permission`.
With Drozer, it is possible to list all the info (and permissions): `dz> run app.package.info -a com.android.mms.service`.
It is possible to enforce the permissions also at runtime, but dynamic instrumentation might overpass the check very easily.

### Injection flaws

Functionalities can be exposed to

- Other apps, through IPC mechanisms
- the user, through the UI

An example of library that can be used to validate things in the ui (such as inputs) s [saripaar](https://github.com/ragunathjawahar/android-saripaar)

### Fragment injection

The intent to start a `PreferenceActivity` receives the name and the paramenters of the fragment to show inside as text and bundle respectively, and reflection is used to instantiate the fragment.
   In this way an attacker can load an arbitrary class in the context of another application (that exports the vulnerable activity). Things are fixed if targetSdkVersion is more or equal than 19. With target < 19 the issue can be fixed with `isValidFragment`.

### URL schemes

With drozer is possible to scan all the URI available for each app.
```
dz> run scanner.activity.browsable -a com.google.android.apps.messaging
```

### Instant app

To discover if an app has an instant component, search for `dist:module dist:instant="true"` in the manifest.
To enable instant app support things have to be done. Refer to the dynamic analysis of the main guide. The `ia` executable is used.

### Functionality exposure through IPC

There are the standard IPC techqniques, such as using shared files or network sockets, or mobile specific things, that offer performance improvements.

If there is an intent-filter, otherwise specified, the exported flag is set to true, and the app component will be accessible form other apps.

For an exported service, we can look at the `handleMessage(Message msg)` method.


## things to research more

- [ ] signature only permissions
- [ ] where an app can read and write, and how to modify those paths
- [ ] What is a Binder exactly?
- [ ] Something more about Zygote?
- [ ] check the content provider class
- [x] when an activity can be launched from outside the appliction that defines it?
   The exported property of the activity must be set to true. If the activity uses intent filters, and this value is set to false, then an exception is thrown.
- [x] What is an intent filter
- [ ] check better how the signing is handled (seems important from the message)
- [ ] singleTask (or, tasks in general)
- [ ] `android:process=":remote"`
-------------------------
## Fist interview passed. new things to research, other then the ones listed above
- [ ] What is OAuth2?

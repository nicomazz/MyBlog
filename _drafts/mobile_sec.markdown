---
layout: single
title:  Mobile security study
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


### Backup MSTG-STORAGE-8
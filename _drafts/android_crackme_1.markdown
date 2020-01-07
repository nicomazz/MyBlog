---
layout: single
title: Android Crackme 1 - use Frida to hook functions 
date:   2019-12-10 01:14:12 +0100
categories: programming 
comments: true
---

The crackme apk can be found here:
https://github.com/OWASP/owasp-mstg/tree/master/Crackmes.



Let's start by the solution:
```js
// $ frida -U -l frida.js -f owasp.mstg.uncrackable1 --no-pause
Java.perform(function() {
   var RootDetector = Java.use('sg.vantagepoint.a.c');
   RootDetector.a.implementation = function(s) { return false; } 
   RootDetector.b.implementation = function(s) { return false; } 
   RootDetector.c.implementation = function(s) { return false; }

   var Decryptor = Java.use('sg.vantagepoint.a.a');
   console.log(Decryptor);
   Decryptor.a.implementation = function(aa, bb) {
      var ret = this.a(aa, bb);
      printByteArray(ret);
      return ret;
   }
});


function printByteArray(byteArray){
   var buffer = Java.array('byte', byteArray);
   var result = '';
   for (var i = 0; i < buffer.length; ++i) {
      result += (String.fromCharCode(buffer[i]));
   }
   console.log(result);
}
```
Now you can insert a random string, press "Verify", and you will see the
solution!

The easiest way to get started is the following
```bash
$ wget https://github.com/OWASP/owasp-mstg/raw/master/Crackmes/Android/Level_01/UnCrackable-Level1.apk
$ adb install UnCrackable-Level1.apk
# install the decompiler:
$ git clone https://github.com/b-mueller/apkx && cd apkx && sudo ./install.sh
# Extract the source code:
$ apkx UnCrackable-Level1.apk
```

Now you can find the source code in `UnCrackable-Level1/src`.

You can run the above script with `frida -U -l frida.js -f owasp.mstg.uncrackable1 --no-pause`.
To find the name of the package, I used `frida-ps -U | grep uncr` after having
started the app. You should see something like `17630  owasp.mstg.uncrackable1`

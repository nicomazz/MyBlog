---
layout: single
title:  How to download videos from Kaltura (and double their speed) when it's not allowed
date:   2018-11-07 22:26:01 +0000
categories: programming 
toc: true
comments: true
---
1. Open Google Chrome
2. Press F12 (to open the developer console)
3. Go on the page with the video
4. Go on the "Network" tab on the developer console
5. Once the page has been completely loaded, and the video started, search on the network tab for `.m3u8` files
6. On the right go on the "Preview" tab
7. You should find something like
```json
jQuery235252({
   "entryId":"...",
   "duration":5466,
   "baseUrl":"",
   "flavors":[
      {
         "url":"...",
         "ext":"mp4",
         "bitrate":583,
         "width":960,
         "height":540,
         "audioLanguage":null,
         "audioLanguageName":null,
         "audioLabel":null,
         "audioCodec":null,
         "defaultAudio":false,
         "frameRate":15
      },
      {
         "url":"AN_URL",
         "ext":"mp4",
         "bitrate":843,
         "width":1280,
         "height":720,
         "audioLanguage":null,
         "audioLanguageName":null,
         "audioLabel":null,
         "audioCodec":null,
         "defaultAudio":false,
         "frameRate":15
      }
   ]
})
```
8. Download the file at the url of your favourite flavour
9. Open it or download it with VLC.
---
layout: single
title:  Open-source startrails
date:   2020-04-12 16:16:01 -0000
categories: photography 
toc: true
comments: true
classes: wide
header:
  image: https://img2.juzaphoto.com/002/shared_files/uploads_hr/2447230_large13482.jpg
  teaser: https://img2.juzaphoto.com/002/shared_files/uploads_hr/2447230_large13482.jpg
gallery:
  - url: https://img2.juzaphoto.com/002/shared_files/uploads_hr/2447230_large13482.jpg>
    image_path: https://img2.juzaphoto.com/002/shared_files/uploads_hr/2447230_large13482.jpg
    alt: "placeholder image 1"
    title: "Startrails in asiago"
  - url: https://img2.juzaphoto.com/001/shared_files/uploads/899102_l.jpg>
    image_path: https://img2.juzaphoto.com/001/shared_files/uploads/899102_l.jpg
    alt: "Croatia startrails"
    title: "Croatia startrails"
  - url: https://img2.juzaphoto.com/001/shared_files/uploads/900317_l.jpg
    image_path: https://img2.juzaphoto.com/001/shared_files/uploads/900317_l.jpg
    alt: "Croatia startrails"
    title: "Croatia startrails"
  - url: https://img2.juzaphoto.com/001/shared_files/uploads_hr/905541_large80129.jpg
    image_path: https://img2.juzaphoto.com/001/shared_files/uploads_hr/905541_large80129.jpg
    alt: "Croatia startrails"
    title: "Croatia startrails"

---

## Ingredients

- Digital camera
- Lens (a wide-angle is better)
- Tripod
- Remote controller. There are several types of this:
  - IR based: this is not ok. However, if your camera has an IR receiver, and you have a phone with an IR transmitter, you can use it! There are several apps for that. Sadly, putting an IR transmitter on phones was the trend only from 2015 to 2017.
  - Mechanical only, with cable and shutter lock: This is fine
  - Digital one: This is the best one. It's probably more expensive (around 15 euro?)

The following image shows my setup. I currently use a Nikon D610, a Samyang 14mm 2.8f, the cheapest tripod on Amazon (my real one is not with me now), and a digital remote.


![]({{"/assets/images/startrails_setup.jpg" | absolute_url}})


## Correct value for different variables

> TL;DR Aperture 2 stop closed wrt maximum, 1-minute exposition for each shot, zero delays between shots, and point the North star, then merge with gimp or starstax.

There are several variables to tweak for the perfect picture. I arrived at the following conclusions after years of trials:

- **Pointing position**: If you point to the polar star, you will have concentric circles. If you point opposite from it, you will have stripes. I usually use [this app](https://play.google.com/store/apps/details?id=com.google.android.stardroid) to be sure to point the correct star
- **Shutter speed**: You can calculate the maximum shutter speed to avoid stripes [here](). However, we do want stripes. The correct mode to set on the camera is **BULB**, that means it will keep the shutter open as long as the button (or the remote) is kept press. Shooting the entire picture in only one frame is not viable for the following reasons:
  - *Hot pixels* **(Important!)**: with time, the sensor of your camera heats up. The more the heat, the more the number of pixels that burns. Once a group of pixels is burnt, you will have a group of red, green, or blue pixels without any other information. You can decrease this effect by shooting several pictures (e.g. instead of a 2h exposure, 120 shots of 1 minute). In general, a FULL-FRAME camera will have less noise than an APS-C
  - //todo put hot pixel image
  - *Camera software limitations*: With most cameras, the shutter is automatically closed after ~1h.
  - *Excessive background light*: even if it seems dark, light pollution is a big problem. If you try to keep the shutter opened for more than 10 minutes, the stars will not have contrast with the background anymore
- **Aperture**: It depends a lot by the lens, but in general, keeping the aperture opened at maximum (minimum f value) it's never a good idea. The definition curve of most lenses is terrible at full aperture. Usually, I keep my 14mm at 4f, instead of 2.8f. I would say to close at least 2 stops from the maximum aperture
- **Image format**: This is tricky. RAW sounds to be the best solution, but with cheap cameras (like my old D3100) the frame buffer is minimal. This means that after you shot 5 pictures, you have to wait for several seconds for them to be transferred from the internal camera memory to the SD card. For this reason, the only solution for the D3100 was to shot in JPEG. If you have a better camera, go for RAW.
- **Dark frame**: After the last picture, put the lens cap on, and shot another picture with the same settings as the precedents. Why? Because in this way you capture the sensor noise, and you will be able to subtract it to the final image, removing most hot pixels, and having better clarity.

{% include gallery gallery_layout="half" caption="Some of my startrails" %}
## Post-processing

There are two paths: the open-source one, and the expensive one. I used to follow the latter, but even the former gives good results.

1. I import my pictures in [Darktable](https://www.darktable.org/), and apply very basic corrections. It is important that you apply the same settings to all the pictures. IT is the right moment to remove aeroplane traces. If you did a dark frame, apply the same corrections to it as well.
2. I export everything in a folder.
3. I use GIMP to stick all of them on top of the others. The main idea is to overlap them and keep only the "lightest" color. In the case of the sky, the result of the overlap will be the superpositions of the stars. Unfortunately, from version 2.10 of GIMP, the support for python has been dropped, so you have to install an older version (If you are an arch user, you can install [python2-gimp](https://aur.archlinux.org/packages/python2-gimp/) from AUR). The script I use is [this](https://github.com/themaninthesuitcase/gimp-startrail-compositor).
4. If you don't like GIMP, [StarstaX](https://markus-enzweiler.de/software/starstax/) is a good alternative, but it only supports Mac OS and Windows.
5. The foreground might be a little bit burnt. I usually shot another picture at a faster shutter speed for the foreground, and overlap the two in GIMP.





## GIMP/Darktable vs Photoshop/Lightroom thoughts

The only things that I'm missing in gimp are parametric adjustments layers, that are available in photoshop.

Regarding Darktable (open source) vs Lightroom (very expensive), the former wins under every aspect except one: simplicity. For the average user, darktable is too challenging to use and requires knowledge of nontrivial graphics notions (that I don't have).


## More photos
- [My juza profile](www.juzaphoto.com/p/Nicomazz)

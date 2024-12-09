---
layout: post
title: "Remove Audible DRM with ffmpeg"
categories: amazon drm audible media audiobook
published: true
---
Audible `.aax` files contain DRM that prevent them being played on many devices. 
Open audio filetypes like `.m4a` (and it's bookmarkable version `.m4b`) can be played on almost any digital audio device.  
A straight conversion between filetypes with ffmpeg is easy - for example

```
ffmpeg -i breakfast_of_champions.aax -c copy breakfast_of_champions.m4b
```
although for Audible it will be necessary to authorise ffmpeg with our Audible encryption key, which can be passed in with the `activation_bytes` option.

Activation bytes are stored as an 8 digit hex string, and can be obtained by either 

- logging into Amazon and authorising with a "new device": as detailed in [this repository](https://github.com/inAudible-NG/audible-activator)
- extracting the string stored in one of our `.aax` files: a plugin for [rainbow crack](http://project-rainbowcrack.com/) can be found in [this repository](https://github.com/inAudible-NG/tables)

With our activation bytes (lets say that our string is `01ad80b5`)
```
ffmpeg -activation_bytes 01ad80b5 -i breakfast_of_campions.aax -c copy breakfast_of_champions.m4b
```


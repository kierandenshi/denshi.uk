---
layout: post
title: "Remove Audible DRM with ffmpeg"
categories: amazon drm audible media audiobook
published: true
---
Audible `.aax` files contain DRM that prevent them being played on older devices (e.g. classic iPod). 
An open bookmarkable audio filetype like `.m4b` can be played on almost any digital audio device.  

A straight conversion between filetypes with ffmpeg is easy - for example
```
ffmpeg -i 1984.aax -c copy 1984.m4b
```
although for Audible it will be necessary to authorise ffmpeg with our Audible encryption key, which can be passed in with the `activation_bytes` option.

Activation bytes are stored as an 8 digit hex string, and can be obtained by either 

- logging into Amazon and authorising with a "new device": as detailed in [this repository](https://github.com/inAudible-NG/audible-activator)
- extracting the string stored in one of our `.aax` files: a plugin for [rainbow crack](http://project-rainbowcrack.com/) can be found in [this repository](https://github.com/inAudible-NG/tables)

With our activation bytes (lets say that our string is `01ad80b5`)
```
ffmpeg -activation_bytes 01ad80b5 -i 1984.aax -c copy 1984.m4b
```

Any audio player should be able to playback m4b files, and iOS users can also use the built in "Books" app. 
Beyond simple convenience, removing DRM also makes it possible to free oneself from Amazon without losing access to one's hundred or so legally purchased **Audible.com** audiobooks.  


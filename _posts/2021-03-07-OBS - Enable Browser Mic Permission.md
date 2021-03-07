---
layout: post
author: Cody Merritt Anhorn
title: OBS - Enable Browser Mic Permission
categories: [blog]
tags: [HTML, OBS]
---

Quick article showing how to enable the **Mic Permission** for the Browser Source of OBS, Open Broadcaster Software. 

# How To

Its easy, you just have to include a startup flag. The flag, <code>--use-fake-ui-for-media-stream</code>, auto accepts the permission of browser sources.

> Use at your own risk with Third-Party Browser Pages, since this does auto accept the permissions, you could accidentally give permission to your camera or mic through OBS.

The easiest way is to update the shortcut to include the flag, like below.

~~~ bash
"C:\Program Files\obs-studio\bin\64bit\obs64.exe" --use-fake-ui-for-media-stream
~~~

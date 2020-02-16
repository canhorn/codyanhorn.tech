---
layout: post
author: Cody Merritt Anhorn
title: Blender 2.80+ - Including Animations in GLTF Export
---

One thing I found lacking in the world is how to correctly export Animations from Blender using the GLTF Export Plugin. Since a few flags need to be in place for this to work correctly I though I would share my findings to help remind me later.

## How To

The first thing is to split the actions into discrete NLA Tracks you want to export. Below I will have some example screenshots of the settings I use to get my exported animations working. The second is to make sure that NLA Strips is checked in the Export settings for the Blender GLFT Export Plugin, checkout the screenshot below for where the flag is located in the plugin configuration.



## Screenshots

***Step One***

![This is an image shows the expected NLA Tracks that should be configured for a clean export.](/image/Posts/Blender/2020-02-16/StepOne.png)
This is an image shows the expected NLA Tracks that should be configured for a clean export.

***Step Two*** 

![This is an image show the flag location for enabling the export of NLA Strips during Export.](/image/Posts/Blender/2020-02-16/StepTwo.png)
This is an image show the flag location for enabling the export of NLA Strips during Export.

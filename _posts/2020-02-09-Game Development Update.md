---
layout: post
author: Cody Merritt Anhorn
title: Game Development 2020.02.09
categories: [blog, game_development]
tags: [updates]
---

The past week I worked on getting a Desktop client created and learning Blender. Time was spend creating and rigging the base Fire Immortal model, this will allow for easier creation of animations. 

## Desktop Client

The client uses .NET core WebWindow to display a locally deployed Blazor Server. When the client starts up it will also startup an instance of the Game Client Server, the server has configuration that points to the Core Server and Asset Server. This allows for an exe application to be created that will allow for distributions to be easily created that expect an exe application.  

## Blender Animations

Using Blender I was able to rig the Fire Immortal mesh that was created in prior weeks. A skeleton and the model was broken up into parts, that could then be individually bound to the skeleton. This will allows for animations to be created, below is an Idle animation example.  

## Bugs fixed

### Missing CSS in Docker image

I found that the CSS in the Docker image for the Client Server was not correctly setup, it turned out that the that to generate the CSS was not correctly set. It also had a bug in the path that was used to move the generated CSS file from the NodeJS stage of the image build into the source code build stage, that would then be used in the runtime stage.


## Some Screenshots 
![This is a gif of an model showing an Idle Animation.](/image/Posts/GameDevelopment/2020-02-09/Idle_Animation.gif)

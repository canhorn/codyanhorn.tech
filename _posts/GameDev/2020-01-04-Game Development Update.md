---
layout: post
author: Cody Merritt Anhorn
title: Game Development 2020.01.04
---

Worked on getting a VoxEdit export into the game. I plan on using VoxEdit to create my Voxel art models. Since VoxEdit comes with a rigger and animation engine it should help to stream line the process a bit.

The process was pretty straight forward. Was just adding the model to the asset server and creating a Asset and using it on an Client Entity. I did have to fix a single bug and added a few features to the client as well.

## Added Features

### Client Entity - Rotation, Scaling, and Scaling Determinant

The Client Entity now supports Rotation, Scaling, and Scaling Determinant values. These can be filled in the configuration for a client entity. The scaling needs to be greater than one.

Example Configuration:
~~~json
{
    "id": "ED22B145-1DE8-4C23-A928-0887D24BADB8",
    "name": "Home",
    "position": { "X": 0.0, "Y": -0.6, "Z": 35.0 },
    "rotation": { "X": 0.0, "Y": 2.0, "Z": 0.0 },
    "scaling": { "X": 1.0, "Y": 1.0, "Z": 1.0 },
    "scalingDeterminant": 0.2,
    "assetId": "11B335CD-8D63-4001-ADDC-A866EEE1E416",
    "properties": {
        "dense": true
    }
}
~~~

### Expanded the JS Exposed services

I needed more services to create an Automation bot for the game, future blog post on that. I exposed the Command and Query services, it was only the event service before.
This will allow for external services to access send commands into the platform, also enabling the ability to query for exposed on as a query event.

## Bugs fixed

### GLTF Loader

The bug that was fixed was in the GLTF Loader, it had an issue with the way it was wrapping the meshes loaded for the mesh. It was using the root mesh that was a wrapper around the actual mesh put in place from the loading process. The fix was to use the second mesh in the list, and the loading worked as normal.

### 'hitEntity' Undefined 

This was caused by not checking to make sure a hitEntity was found in the Companion system. It was an easy fix just checked for valid hitEntity, and if not valid just exit event handler.

## Some Screenshots 
![This image shows the Home VoxEdit export.](/image/Posts/GameDevelopment/2020-01-04/Home.png)
![This is a picture of the test world.](/image/Posts/GameDevelopment/2020-01-04/TheWorld.png)

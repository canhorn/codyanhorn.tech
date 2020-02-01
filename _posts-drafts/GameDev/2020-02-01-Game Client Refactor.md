---
layout: post
author: Cody Merritt Anhorn
title: Game Client Refactor 2020.02.01
---

This post will only details a few things done, but a whole lot of work was done at a solution level for the Client, <a href="blog/tdb" title="Post detailing the Game Client Solution Refactor.">Link to Blog Post</a>.

For bugs we were able to fix an issue with the rotation movement logic, so it will correctly orientation the agents to there target direction.

## Added Features

### Game Client Solution Refactor

This was a huge change to the solution structure of the Game Client, and the finer details will be covered in this <a href="blog/tdb" title="Post detailing the Game Client Solution Refactor.">blog POST TBD</a>.

### Added New Immortal Model

We added some new models for the Immortals of the world, they are currently not animated.

### Client Agents - Rotation and Scaling Determinant

The Client Agents now supports Scaling Determinant on the server, and Rotation on the Client. The Scaling determinate can be filled in the configuration for a client entity. The scaling needs to be greater than one. The rotation is only used on the client so when starting all entities will be facing the same direction, but could be moved onto the server if needed.

## Bugs fixed

### Fixed Agent Rotation

When calculating the rotation angle it was not normalizing the forward direction and right direction. Meaning that the vectors were not getting calculated in a consistent manner. Normalizing them means to remove the magnitude or in this case the location of the entity at that location relative to the direction we want to find out. It layman's terms, we needed to set the values of the vectors from 0-1 so the math used to get the rotation direction it correctly figure out.


## Some Screenshots 
![This is an image of the new Immortal models added to the world.](/image/Posts/GameDevelopment/2020-02-01/RoamingImmortals.png)

---
layout: post
author: Cody Merritt Anhorn
title: Game Development 2020.03.27
categories: [blog, game_development]
tags: [updates]
---



## Demo Update

As before the demo did not have a lot of work done on it, but the Editor updates should be coming to and end. Then we will be focusing on the demo more in the coming weeks.

## Editor Updates

To help make the editor feel more editor like, I added an undo/redo feature to the platform. The action are tracked on the server and the Undo/Redo service layer will keep track of it internal state, using event the platform is able to track MediatR IRequest events. Using this pattern makes it so action can be played back in a consistent manner. This makes it very easy to undo quite a few action, since these are saved in state on the server, so they persist between page requests. It might need an enhancement to the state size to make sure the actions do not take up large areas of memory.

I will be working getting the transform work done for the Client entity.
Today I will be working on Rotation and Scale for the Editor.

All properties that I deem editable should now be fully editable. Custom property data was also added as editable fields to the Selected Entity section. Custom properties are a combination of platform used properties, these are supplied by the zone server as property metadata. The property metadata also keeps track of the property data type, so for example the densityBox should be in the for of a Vector3 and this will be provided by the server that this is needed. Properties that do not have metadata will default to a string, and can be anything. It is up to the script or platform logic to take this into account when using the custom properties.

In the future I will want to expand on the property editing to encompass all the properties on an Entity, since this is only for Client Entities it does not include tags.

As part of the property work it was also getting the Bounding Box dictating the DensityBox to display. The editor will take care of displaying the DensityBox as a wire-frame box around the entity. This will probably need to be expanded on to be more dynamic.


## Platform Updates
### Id Simplification 

### Entity Consolidation

## Bugs fixed
### Animation's not stopping correctly

While working through the Id simplification updates I noticed that the animation events that pick up start/stop were not using the correct event data structure. They were using an 'any' in TypeScript, meaning that it would accept anything. This should of been been using the DataType of the event and was not, so it was not failing to compile but the id check was not causing errors, it was just not correctly validating that the id matched. The fix was to make sure that the correct id was getting checked, and was easier to see when I had to update all location that used id to clientId as stated in the <a href="#id-simplification">Id Simplification</a> section.

## Future Work

While writing this article it has come to my attention that most of the feature of the server are being intertwined with the Editor/Client code. I might go about separating these areas into extensions that are just namespaces on the Editor and Client that dictate the project from the Zone server.

## Screenshots 

![Moving selected entity around the game world.](/image/Posts/GameDevelopment/2020-03-27/Entity_Movement.gif)
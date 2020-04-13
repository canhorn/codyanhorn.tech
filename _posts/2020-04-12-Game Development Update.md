---
layout: post
author: Cody Merritt Anhorn
title: Game Development 2020.04.12
categories: [blog, game_development]
tags: [updates]
---

This update we had some awesome feature added to the Editor, Undo/Redo being one of them. Another being the ability to edit most properties on the Selected Client Entity, with two way communication of property state between the in-world and the wrapping editor. As part of a larger platform effort the Id's on the client were simplified and the common entities were consolidated into a simplified base abstraction.

## Demo Update

As before the demo did not have a lot of work done on it, but the Editor updates should be coming to and end. Then we will be focusing on the demo more in the coming weeks.

## Editor Updates

To help make the editor feel more editor like, I added an undo/redo feature to the platform. The action are tracked on the server and the Undo/Redo service layer will keep track of it internal state, using event the platform is able to track MediatR IRequest events. Using this pattern makes it so action can be played back in a consistent manner. This makes it very easy to undo quite a few action, since these are saved in state on the server, so they persist between page requests. It might need an enhancement to the state size to make sure the actions do not take up large areas of memory.

As part of the larger platform update I took the time to implement an encapsulated data structure for Position, Rotation, and Scaling. This makes it easier to attach transformation logic to different types of entities. Rotation and Scaling were added to the editor, for in editor and live in the game world. 

Custom property data was added as editable fields to the Selected Entity display area. Custom properties are a combination of platform used properties, platform properties are supplied by the zone server as property metadata. The property metadata also keeps track of the property data type, for example the densityBox should be in the form of a Vector3. These details will be provided by the server, properties that do not have metadata will default to a string. It is up to the script or platform logic to take this into account when using the custom properties.

In the future I will want to expand on the property editing to encompass all the properties on an Entity, since this is only for Client Entities it does not include tags currently.

As part of the property work a display for the Bounding Box dictating the DensityBox in the game world. The editor will take care of displaying the DensityBox as a wire-frame box around the entity. 


## Platform Updates
### Id Simplification 

To make it easier to reason about on the client what an Id pertains to I went and updated all client related entities to be 'clientId'. This will make it easier for inter module communication for id comparison of entities that should only be used on the client side. This helps in client side only events, since the event will be tags with 'clientId' and not just 'id' making it easier to know that which Id should be check on an entity. 

Entities also have a consistent usage of 'globalId', which correlates to their platform id. The globalId helps to track different events from the server that relate to the platform related event. 

The last entity that was simplified is the 'entityId', which correlates to their Zone server id. This id is specific to events that are published from the server, and not related to the client inter-communication.

### Entity Consolidation

To help create a simplified model layer between all different entity types of Player, Agent, and Client Entity. Most of the work came from creating the base classes/abstraction that each entity should be inherited from. 

## Bugs fixed
### Animation's not stopping correctly

While working through the Id simplification updates I noticed that the animation events that pick up start/stop were not using the correct event data structure. They were using an 'any' in TypeScript, meaning that it would accept anything. This should of been been using the DataType of the event and was not, so it was not failing to compile but the id check was not causing errors, it was just not correctly validating that the id matched. The fix was to make sure that the correct id was getting checked, and was easier to see when I had to update all location that used id to clientId as stated in the <a href="#id-simplification">Id Simplification</a> section.

## Future Work

While writing this article it has come to my attention that most of the feature of the server are being intertwined with the Editor/Client code. I might go about separating these areas into extensions that are just namespaces on the Editor and Client that dictate the project from the Zone server.

## Screenshots 

***New Editor Layout***
![Displayed is the New Editor Layout with a Selected Client Entity.](/image/Posts/GameDevelopment/2020-04-12/New_Editor_Layout.png)
***Undo/Redo Feature***
![Undo/Redo of world editing.](/image/Posts/GameDevelopment/2020-04-12/Undo_Redo.gif)
***Client Entity Property Edit***
<div style="width: 100%">
    <img style="width: 50%" title="View of selected Client Entity property edit." src="/image/Posts/GameDevelopment/2020-04-12/Selected_Client_Entity_Property_Edit.png" />
</div>
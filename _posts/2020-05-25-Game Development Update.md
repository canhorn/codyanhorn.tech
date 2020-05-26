---
layout: post
author: Cody Merritt Anhorn
title: Game Development 2020.05.25
categories: [blog, game_development]
tags: [updates]
---

We got some exiting stuff done in the Demo over the past month, we were able to get some GUI per player, stating their captured Immortals, the LeaderBoard GUI was also implemented, and a day/night cycle lighting. We also had quite a few Platform updates and the Zone had a load of Unit Testing done to get the coverage up.

Since the last update was over a month ago, it might turn out to be a long article.

## What is all this for?

I don't think I have gone over why I am making this game, or better why I am in Game Development. Most of comes back to keep my skills sharp, and I find that making a game is a great way to keep up-to-date on tech, since I can try all types of things on this platform. An online game gives me a reason to work in all areas of software, be that Identity Management, Operations, Website development, Desktop Client development, Scaling application, and I can go on. 

With this platform I am able to branch out into other technology, in the case of mobile development I can create a wrapper for my game or look at what a game would look and function on a mobile device. 

But over all other reason, I want to look at making a platform that could be used entirely on the web to create a fully functioning game. With the movement of creating Web based Coding environments this would make it easier to create a game development platform that just needs a server to be deployed with the services I am creating. Checkout my [game studio website](https://ehzgames.studio) for more details on the [Game Development Platform](https://ehzgames.studio/game-development-platform.html) that I am developing.

## Demo Update

The Demo had some major updates, the display of the game world to the player. We added the UI a list of Captured Immortals, showing their Name. Since the players state will get reset on login the player can then start capturing new Immortals when they come back to the game.

<a href="/image/Posts/GameDevelopment/2020-05-25/Caught_Immortals_GUI.png" 
    target="_blank"
    title="A picture of the current players Caught Immortals GUI.">
    ![A picture of the current players Caught Immortals GUI.](/image/Posts/GameDevelopment/2020-05-25/Caught_Immortals_GUI.png)
</a>
**A picture of the current players Caught Immortals GUI.**

Another major UI update was showing the LeaderBoard. The Game Server will now keep track of the players and their capture count, this can then be show to all players in the game the current score of the other players. This UI can be show by pressing and holding the "t" key, it will display the currently active players and their score! 

<a href="/image/Posts/GameDevelopment/2020-05-25/LeaderBoard_GUI.gif" 
    target="_blank"
    title="A clip showing the LeaderBoard GUI before and after the t key is pressed.">
    ![A clip showing the LeaderBoard GUI before and after the t key is pressed.](/image/Posts/GameDevelopment/2020-05-25/LeaderBoard_GUI.gif)
</a>
**A clip showing the LeaderBoard GUI before and after the t key is pressed.**

A lesser update was to the Lighting, I moved out the day/night cycle that was already in place to a module that can be attached to the light. This will be updated in the future, but it was updated to allow for settings to be imported into the module. Currently lighting is not a setting supplied by the server, so this will be added in the future to allow for the server to be the authority for the settings.

## Platform Updates

### Better Cross Service Commit Tracking

I decided to update the way that I track changes over multiple services, it was making the crafted commit message the same across all the service and adding a UUID to the message.

**Format of Message**
~~~
[Commit Subject]
[Commit Description]

[UUID]
~~~

**Example of Message**
~~~
Server Core Refactor
Moved the Server Core related functionality out of the Base Zone Server project.
This will make it easier to encapsulate the logic for just the Server Core related functionality.
Created a standalone Server.Core.Connection library that can be moved into the Core Service solution in the future.
Added enough unit testing to cover any gaps that existed before the move.

01C996D2-7B50-448B-80CC-4821B4AD00A6
~~~

### Hot-Reloading

Big updates to the platform were done to enable Hot Reloading for the GUI, ServerModules and Map. These requires changes to the Editor, Client and Zone with these platform services updated to use Event triggers in other platform services. The Editor can show realtime changes to the ClientEntities or GUI or ServerModules, the Client was also updated with most of the changes as well. 

### Editor Updates

The two notable updates to the Editor were the addition of the Asset Selector and the Zone Settings Reload. The asset selector displays a list of the server configured assets, these are models and such. The current selector does not allow for context settings, like only displaying Model typed assets in the select. This makes it easier to get the Id for the asset to put in the settings for the ClientEntity.

I also took the time to make sure that when different settings are changed that the Zone server settings are reloaded. This makes the backend state more consistent to changes, one issue that was happening before was if the selected client entity changed it would not get updated in all used areas. Having a way to trigger state changes to the Zone and in turn the Editor State makes it so areas are consistent.

### Client

Most of the Client updates, out side of the platform related changes, was to the TypeScript target version and exposing debugging statistics. The TypeScript was updated to allow for more current features, I wont dig into the specifics but the update allows for more compatibility with libraries and syntax usage. A common abstraction for exposing debugging statistics was added to the client, this makes it easier to debug, since I can just add some details to generic abstraction. 

I can then use the debugging abstraction to get details about the events/commands/queries publish on the client, and it can also be used to keep tabs on debugging object. This was exposed in a way that it can be read from the Dev Console and by other tools, in the future I will probably expose a window that can be opened for a client to see details about the currently connected client. Another use case is to allow for a details about the connected player to be access from the Editor.

### Zone

The zone had quite a few of updates over the past month, performance improvements, Unit Testing, fixed Code coverage for branches, hot reloading of files propagated to clients, splitting up Logging GUI for platform reuse, Map Mesh refactoring and Server Core refactor.

### Zone - Behavior Tree Improvements

The Behavior Tree processing was updated to use a Queue, this caused multiple improvements to the overall processing of the registered Behavior Trees. The first was that a single behavior tree processing does not affect the others, as before it would wait for one to finish before the next would process. The way it was originally done was to allow for the behavior trees to better predict other that are being processed, I chose to drop that requirement for being able to trigger as many as possible. 

The behavior trees are now responsible, during their processing, to enqueue themselves up for their next processing cycle, they also include a timestamp to make sure they are not trigger consecutively very quickly. After the improvements were done I found that it increase the behavior trees that could be processed went up substantially. Also the improvements made the code a lot simpler, by removing the complexity that was used to keep track of the behavior tree and the actors that needed to be processed.

### Zone - Map Mesh Refactor

The Map Mesh was updated to its own project, allow for better System separation of concerns to other platform systems. Since this system was pretty old to begin with I went and updated most of the System layout into something that could work better with the patterns I have in place now. The first was to move the Map settings into a separate folder, allowing for better hot reloading on changes. Next was to implement client action that can be picked up by the Client or the Editor. 

The Map settings were also exposed to the Editor through the Explorer from a node, this is then shown in a Text Editor on the Editor Service. Any changes to the Map will trigger an event to all clients that get picked up and cause a reload of the map from the settings that come along in the event. Their was cleanup done to the client around the MapMeshMaterial, this is what controls the texture blending on the terrain from the setting of the server. With the refactor also exposed all of the settings that the Map will use, before it was a small subset of settings that just controlled the size and shape of the map, exposing the material settings so the different layers of the map can be controlled from the Editor.

### Zone - GUI Messaging Split

This involved splitting out the CombatSystemLog GUI into ao SystemLog module, this takes SystemLogging in general out of the context to the Combat System and into a platform provided System. The new ClientAction created for MessageFromSystem, can now be used by any System using by just publishing an event.

### Zone - Server Core Refactor

This was a simple update to the structure of the projects, I moved the core namespace in the base Zone Game project out into the core project. This project can be better segregated from the reset of the services in the application, since most of the Model and Events were already moved out of the Core platform logic this move was relatively easy. As part of the move Unit Test were also added to cover the missing coverage for the project.


## Bugs fixed

### Updated Data Properties of IObjectEntity to use ConcurrentDictionary

This change was done to add more stability to the platform communication, when the platform was publishing change events it would get a serialization exception. This exception was caused by the changing of the Data/RawData objects while the entity was getting serialized for transit. The fix was simple enough, since I already built an wrapper layer using extension around the Set/Get/Populate of the entity data. 

The solution to the exception was to change all references of <code>Dictionary<string, object></code> to <code>ConcurrentDictionary<string, object></code>, this did require the setting of properties to be updated, but with the already stated abstraction was an easy update.

### Agent Movement after Map Created/Rebuild

This was a problem introduced after the Map Hot Reloading and before the Behavior Tree Queue processing. The initial fix was a combination of multiple changes to the platform, re-register the agents on a map change, monitor fore stale agent paths, and also to remove the serialization of the path. The main issue was probably the accidental serialization of the path state, this was fixed by making the accessor of path a method, so it wont get serialized. I said it probably fixed it, since the Monitor put in place probably masks the issue now. The most likely fixed was probably the updates to the Behavior Tree processing to be queued based. With these changes the behavior tree being process would be better resilient to exceptions.

### Fixed Lighting on Tree Asset Script

This was a relatively easy fix to make, the script was setting specularColor, emissiveColor and ambientColor to be managed by the framework and not set explicitly in the script.

### Fixed TimerWrapper Validation Exceptions

Fixed an issue with TimerWrapper, when an exception is thrown from the Validation run it could would cause the whole timer to stop functioning since it would not ever set the Timers internal state to allow for future running.

## Screenshots 

<a href="/image/Posts/GameDevelopment/2020-05-25/Day_Night_Cycle.gif" 
    target="_blank"
    title="A clip showing the Day/Night Cycle Lighting in the game world.">
    ![A clip showing the Day/Night Cycle Lighting in the game world.](/image/Posts/GameDevelopment/2020-05-25/Day_Night_Cycle.gif)
</a>
**A clip showing the Day/Night Cycle Lighting in the game world.**
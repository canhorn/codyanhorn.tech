---
layout: post
author: Cody Merritt Anhorn
title: Game Development 2020.03.03
---

Since the last update on 2020.02.09, their was been a lot of work across the board. Added a way to get details about the internals of the running game client externally, I made some changes to the way the terminal works in the Editor, added Entity management to the editor as well. More details provided below.

## Work on Demo

I have spend the majority of the time working on the AI, using behavior tree's, to get them following their owner. A lot of the work done in the different areas were to support this, by either adding features to the Editor to support easier manipulation of the entries on the server, to displaying details externally to see the state of the Agent/Actor on the client. Below is a great clip of the Immortal following their owner.

## Game Client Update

Added a bootstrapping abstraction to the client startup of the client. This will allow for external systems to know when the system is ready to receive commands.

Added a new Tracking Service, this allows for anything internally to the game client to register objects that can later be queried from the console externally to the internals. Will be mainly used to gain access to internal state for external tools.

Made some updated to the client pages to start the game when it picks up that a user is logged in, with help from the bootstrapping state prior. With that I simplified the navigation menu so it only contains what is related to the game.

## Game Zone Updates

Their is now an event setup to propagate state changes for entities, so when an entity is updated it will publish an event that other systems can listen on and react accordingly. This was wired up on the client with updates to properties of entries that have changed. WHen an entity is stopped it will now trigger an publish event to notify clients that it has stopped. With all of this also added a way to disable Application Insights, since even it was not successfully setup it would still flood the logs with statements that it was logging.

The the future I want to refactor the Startup.cs in the Zone Server main project. So I put some TODO and comments in the code to remind me of this. It probably wont be done as part of Demo work, but is something that should be done eventually. Since I want to be able to generate this file from a configuration file, in turn dynamically generating the whole zone server. 

## Game Editor Updates

***--*** Update all dependent libraries to 2.1, so I can use C# 8 features. 

***--*** Refactored the project a little bit to use the standard library I have for Identity.

***--*** Added hooks into the Zone Terminal so it can send and receive responses from the Zone server.

***-- --*** Also includes a quick Command drop-down for hard-coded commands to be fired.

***--*** Added Entity Management to Zone Editor page, so you can now see and send quick commands to a specific entity.

## Bugs fixed

### Fixed issues with Entity Moving stopping all entities

Their was a slight but in the handler for stopping entities, in that it would always stop the entity on an event. The fix was to make it so the event received was actually for this entity to stop.

### Fixed issues with bad meshes when loading from GLTF models.

It was not so much an issue with the mesh, but with the code I used to tie back the parent to the mesh. I removed the setting of mesh's children parent to the root. this turned out to cause the relative animation data to be based off the root, and not the real root of the mesh.

### Behavior Tree Interpreter Not Found Exception

There was an issue with the Behavior Tree not correctly advancing the ShapeQueue past the children of a Traversal Node. In certain scenarios it the state of the Actors behavior tree could get in a state where the active node is pop from the queue and then a traversal would try to advance past its last child. Since that child was already popped from teh queue it would clear the queue. 

The fix was to change up this advance past the children to keep another list in the state that would keep track of queues order and the call to advance the queue pass the current node's children to just get the next node in the list past the last child.

## Some Screenshots 
![This is a click of the Immortal following its Owner Player.](/image/Posts/GameDevelopment/2020-03-03/Following_Owner.gif)

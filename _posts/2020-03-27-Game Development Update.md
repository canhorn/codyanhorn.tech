---
layout: post
author: Cody Merritt Anhorn
title: Game Development 2020.03.27
categories: [blog, game_development]
tags: [updates]
---

Nothing was really done for the Demo specially, most of the work was done for the platform and a lesser extent to the Editor. Where most of the work was done was with the platform to make a really nice editor to help with the creation of the map that will be used by in the Demo.

## Work on Editor 

The platform editor now has a new Game Editor section for a Zone, with a view into the game using the same platform at the Client. The editor also allows for selection from in game and in real time pushed up to the wrapping page. The Editor is still pretty rough and will have more work in the coming week, with the work to supporting the new <a href="#entity-transform-consolidation">Transform</a> work. The editor was also updated with Azure DevOps Pipeline's allowing for the server to be deployed.

## Platform Updates

The platform does have a lot of work done across many areas, checkout the areas below for more details.

### .NET Core 3.1

Finally took the time to update all services of the platform, this was every service that were either 2.2 or 3.1. This was a nice update, since this helped with the build times and runtime performance seems to be quite better as well. The update required a few updates to the applications, mainly around the SignalR JSON protocol checkout out the <a href="#made-json-handling-consistent">Made JSON Handling Consistent</a> section below for more details.

### Azure DevOps Pipeline Addition

The application that did not have a Azure DevOps pipeline configuration were updated to include one, the Azure DevOps build were replaced by these the in repo configuration build files.

### Updated Project Structure

Some of the projects were still root projects, I took the time to move around the files so they are now solution based. This allowed me to add a test project, and the Dockerfile file's were updated to work with this new structure.

### Made JSON Handling Consistent

Ok, this was a huge problem, not so much that it was that the SignalR internal JSON protocol was changed to the new System.Text.Json project/namespace released in 3.0. It was that this namespace does not have feature parity with Newtonsoft.Json, I do like that this has a performance increase in parsing time. 

The main problem I ran into is that the current version of System.Text.Json does not include delegation of fields. Since I use another provided data structure, the Vector3, and since we are building a Game we kind of need to use that everywhere. This is probably not a big issue, since the release of .NET 5.0 will bring fields to the JSON parser, I hope it is still in draft. But after that is dropped, we will go through and update everything to 5.0 and move to just System.Text.Json. I think that will be great and can wait to get a huge performance improvement to the platform.

### Entity Transform Consolidation

This was done at the platform layer of the all the applications. The Transform data structure is really nice, it holds all the standard details about an Entity. This information holds the Position, Rotation, and Scale of the entity. By having this information in a standard format, and having it across all entity types, makes it easier to create an Editor, hey look we are building one of those. 
 
### Exposing Services

I took some time an implemented a way to expose the Command, Query, and Event services to external sources. This was done to allow for the Editor to hook into the internal state of the game. Allowing for the editor to use the same client that a player would use. But doing this will also allow for the editor send commands to the game for debugging purposes. Like editing the properties in an editor and seeing how it affects the entity, in real time. 

## Bugs fixed

### Player Position not getting saved

Found out during the fixing of the JSON handling, making it consistent, that the player details were not getting sent correctly to the Player server. Check out the <a href="#made-json-handling-consistent">Made JSON Handling Consistent</a> section from the Platform Updates to get more details on what was done to make the JSON correctly sent around the platform.

## Some Screenshots 

![Moving selected entity around the game world.](/image/Posts/GameDevelopment/2020-03-27/Entity_Movement.gif)
![New Game Editor view with Editor Window, Client Entity List, and Client Entity Selected.](/image/Posts/GameDevelopment/2020-03-27/Game_Editor.png)
![Show off the selection of Client Entities in the Game Editor.](/image/Posts/GameDevelopment/2020-03-27/Game_Editor_Window.png)


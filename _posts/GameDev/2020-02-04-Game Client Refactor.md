---
layout: post
author: Cody Merritt Anhorn
title: Game Client Refactor
categories: [blog, game_development]
tags: [updates, refactor]
---

In an effort to make my Game Client more modular and also update the hosting server, I decided it was time to refactor my current solution into three main areas: Clients, Servers, and Shared. This allowed me to better structure the areas of the Client and setup for future enhancements to the Game Development Platform. 

## Background of Old Client

The old client was built on top of the ASP.NET Core SPA services, using the ReactJS Framework for the client side code. This client has been around for a while, and was structured in a way that would not require much of work to move to a new hosting platform. The client was split into a few different area as well, the Engine, Client, Game, and Core. The client is also written in TypeScript using BabylonJS as the rendering engine for the game.

### Core 

This has general abstraction around different patterns, its built heavily on Dependency Injection, Command and Query Responsibility Segregation, and a logging abstraction. This also has a few other features that help with the configuration, internalization, monitoring, throttling, etc...

### Engine 

This area of the client is where all the low level abstraction around BabylonJS was housed. The systems in place help to control the life cycle of the game, from startup to shutdown. It also has a built in GUI abstraction that can be fully utilized using just configuration in the form of JSON. The Engine also contains a Particle system abstraction it too utilizing configuration based on JSON.

### Client

This area ties directly to the Zone Server, have all the authentication and connection management to the Core and Zone servers. The Client also keeps track of all Entities of the game, and delegating the event from the server out to the rest of the Game Engine. One of the central areas of the Client is the Scene Orchestrator, this keeps track of the current rendered assets. There is light logic for Administration, but most of this logic has been moved out into the Editor/Dashboard and the main hooks here are to allow for client side commands to be pushed up to the different servers.

### Game 

This section currently only has the starting "Game" of the client. This can be expanded on to allow for more client side based games, that do not have/need multiplayer.

## The New Client

The new client, as stated in the opening of the post, has three main areas; Clients, Servers, and Shared. This should make it easier to understand the structure and help to encapsulate the features better into areas of function. The front end game engine is still heavy TypeScript but the ReactJS code has been remove completely. It has been replaced with a Blazor Server backend/frontend, this will be used to manage the startup life cycle of the TypeScript part of the Client. 

The client now has an Asset Server, the asset server will host all frontend related files. This will also be where the TypeScript client will be hosted, the way I can have a Content Delivery Network front loading the Asset Server to all clients. The Client Server is built in a way that it can be used in both a publicly hosted, browser accessed client but also using the new WebWindow to access a locally hosted Blazor Server. 

Structuring a Desktop client with a locally hosted Blazor server, the same that can be hosted on a remote server, allows me to manage and enhance one project updating both Deployment scenarios. The Clients should be able to also use WebViews of Android and IOS to connect up to the Host Blazor Server scenario. Or if in the future if I choose to create a fully Native solution that could also be done.

## Screenshots 
![This is an image of the new solution structure.](/image/Posts/GameDevelopment/2020-02-04/NewProjectStructure.png)

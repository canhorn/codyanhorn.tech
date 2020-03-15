---
layout: post
author: Cody Merritt Anhorn
title: Game Development 2020.03.14
categories: [blog, game_development]
tags: [updates]
---

This past week has been focused on Entity management and consistency between the Client and Zone. For one we made some platform updates to the way Movement works, enhanced the Follow Owner behavior script, and general fixes around entity management.

## Work on Demo

We finished up the finer points of the Follow Owner Behavior tree. It will now correctly follow the actor/agent's owner and keep in sync with the owner as they move around the map. Made wide sweeping changes to the description/summary details for all behavior scripts. 

With the cleanup I started using the "see" tag provided by the summary comments of C#. Using the see tag I am able to give easy access to the actually representation on Data or input that is provided to the script from the runner services. It also give the ability to see this information in intellisense. That is not really applicable to the Zone scripts, but it can be used in other places of the code for more details.

## Platform Updates

### Made Speed Formula Consistent Between Client and Zone

To help with managing Speed between the Client and Zone I went add added a state property to Entities. This state property will hold speed details, that can be used on both the Zone server and on the client. Using this allowed for fine tuning of the speed for the entities and make sure that the movement on the Zone server is relatively in sync.

### Cleanup of Entity Repositories

There were a few entity wrappers, around Agent and Player mainly, that would just proxy to the Entity Repositories. I removed the update and delete from these wrappers, since they did not add any value and so you can just call them directly. Also I recently added an EntityUpdateCommand and that should be used, since it triggers other events on run.

### Cleanup of Player Entity Global Update

This was spread around multiple areas of code, I added a hook into the Update Properties, event that is published during Entity Updates. It will just listen for these events being published and check if it was a player triggering a Global Update command.

### General Cleanup Items

Removed old combat system IScriptServices, not used since we now have a newer script runner. The script runner has an abstraction that it uses, and the data is passed in as a general dictionary/map to the script. This was missed during the initial creation of the script runner and so this dead code was removed.

## Bugs fixed

### Fixed Issue with PopulateData on Entities From SignalR

Enhancements were made to the PopulateData to support the new .NET System provided JSON library. This is to help with the object that come from other services through the SignalR connections, since now by default SignalR uses the "faster" System provided library. It sill supports the Newtonsoft types, since those are still used when parsing files.

### Fixed Issues with Concurrent Selector Interpreter

The Concurrent Selector Interpreter has an issue were it did not take into account the new advance queue to token/node logic. So that was updated to support that, and it I also enable the reset property for this Interpreter.

### Fixed Deployment Issue with Client

The client had the deployment issue of not being able to build the solution, it was missing the Clients folder in the Dockerfile building. The clients was added, but the build Dockerfile was not updated, so it was a simple fix.

## Some Screenshots 
![This shows off smoother movement of an Agent.](/image/Posts/GameDevelopment/2020-03-14/Smooth_Movement.gif)

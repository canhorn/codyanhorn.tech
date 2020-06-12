---
layout: post
author: Cody Merritt Anhorn
title: Game Development 2020.06.11
categories: [blog, game_development]
tags: [updates]
---


This updates a whole lot of work done to the Demo, I think the main game mechanics are done. All that is left is some more aesthetics to the game world. I also fixed a few bugs and made a performance improvement to the way behavior trees are reported.

## Demo Update

The Player entity a new state object was created that is used to keep track of capture Immortals and time tracking for the game rules pertaining to the escape of the captured Immortals. With these changes it allows for tracking the last Capture time, using this was are now able to give feedback to the Player when they have 10 and 5 seconds left, with triggering a skill on the player when their captured Immortals escape.

We now display feedback that the user has 10/5 seconds left to capture another Immortal, when this timer runs out a skill is triggered. This skill, called EscapeOfCaptures, will show a message to the Player that they have lost all of their captures, also giving the captured Immortals time to escape by freezing the player in their location for a few seconds.

With this update I also made sure that the LeaderBoard was correctly sorting the player list by highest first, based on captured Immortals. The LeaderBoard will also stay in sync with the escape of captures for a Player, so when a player looses all of their captures it will updates every players local LeaderBoard.

### Zone Updates

A big update that was done to the Zone service was to move the Skill namespace and logic out of the base Combat System. By doing this I was able to better encapsulate the logic specifically for the Skill combat subsystem, it is a plugin under the Combat System. This allowed me to better manage the Testing that was done for this plugin, instead of having to test all of the Combat System it helped to create a smaller subset of testing. I took the time to refactor the Skill logic into something more modern, with consistent formatting and file layout structured the same as the rest of the projects in the Zone service.

I took the time to update the skill effect SkillEffectScriptResponse, I made a huge cross cutting change. The change was to make it easier to pass state between scripts, since they might require data from one script to another. I also added a static build method, New, that can be used to better structure the responses from Skill Effect scripts. 

Since the main purpose of the response is to trigger client action's, I found that I needed a way to send a Client Skill Action to a single connection, usually the Player/Caster of the skill. To accommodate this I spilt up the ClientSkillAction into two logical types, one to send a client action to all connected players and the other to send a client action to a specific player connection. Since the effect response allows for multiple client actions this makes the system even more versatile to different use cases. The New method on the Skill SkillEffectScriptResponse will return a default initialized object, and the response structure was updated to function like a builder, so I am able to generate the result without having to make sure the script creates a valid response object.

Some odds and ends that were done to the platform are as follows:

**-** Added missing error messaging for the combat system.

**-** Added better client side logging for ServerModules.

**-** Added better logging for Server Scripts.

## Performance Improvements 

### Agent Behavior Tree Reporting

There was some overhead to serializing the data of the Reporting Item's that were added to an Agent's Behavior Tree Tick Report. The serialization was done at the write of the report items, this turned into a huge performance boost. 

## Bugs fixed

### Game Fix - Fixed CompanionsCaught state for Players

When the player was being created it was reusing a static state object, this was causing a "shared" state between players. The method to get the new Game State for a player was updated to make sure they are getting a newly created state object just for the player.

## Future Work

The focus for the next few weeks will be to add assets to the game in the form of models, animations and sound. I hope to figure out how to deploy the game platform in a cost effect way, with this game demo published on it.

## Game Media


<a href="/image/Posts/GameDevelopment/2020-06-11/Ten_Second_GUI.png" 
    target="_blank"
    title="Image of the 10 seconds left GUI.">
    ![Image of the 10 seconds left GUI.](/image/Posts/GameDevelopment/2020-06-11/Ten_Second_GUI.png)
</a>
**Image of the 10 seconds left GUI.**

<a href="/image/Posts/GameDevelopment/2020-06-11/Five_Second_GUI.png" 
    target="_blank"
    title="Image of the 5 seconds left GUI.">
    ![Image of the 5 seconds left GUI.](/image/Posts/GameDevelopment/2020-06-11/Five_Second_GUI.png)
</a>
**Image of the 5 seconds left GUI.**

<a href="/image/Posts/GameDevelopment/2020-06-11/Escaped_Second_GUI.png" 
    target="_blank"
    title="Image of the Immortals escaped GUI.">
![Image of the Immortals escaped GUI.](/image/Posts/GameDevelopment/2020-06-11/Escaped_Second_GUI.png)
</a>
**Image of the Immortals escaped GUI.**



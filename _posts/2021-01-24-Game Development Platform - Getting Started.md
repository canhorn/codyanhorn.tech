---
layout: post
author: Cody Merritt Anhorn
title: Game Development Platform - Getting Started
categories: [blog]
tags: [Release]
---

After you join the Game Development Platform, you might be lost on where to start, this article will give you an overview of the Platform and where you can get started.

## Platform Overview

This will be a quick overview, you can checkout more details on the <a href="https://ehzgames.studio/game-development-platform.html" target="_blank">Game Development Platform</a> page.

The platform is a collection of services that that help manage the different parts of your game for you, it includes a client, asset server, editor, game server, agent, player and identity management. Most of your time will be spent in either the Game Client, testing your game, or the Editor where you will be creating your game. The Editor includes a live in-game view

You can checkout the Game Development Platform page for details on each one of these services, but the main point is that you don't have to manage these your self. You can access these service directly or indirectly in your games, as the platform is built out more of the capability's will be exposed. 

<a href="/image/Posts/2021-01-24/GameDevPlatform.png" 
    target="_blank"
    title="Shows the services of the Game Platform and how they interconnect.">
    ![The Game Development Platform Overview.](/image/Posts/2021-01-24/GameDevPlatform.png)
</a>
**The services of the Game Platform and how they interconnect.**

If you have any feedback you can submit it on the <a href="https://cloud.sandbox.ehzgames.studio/feedback" target="blank">Platform Feedback Form</a>.

## Account Page

After logging in you will see a page, like the image below, that gives you an overview of your account. Here are also a few quick links to the services provided to you, in the list is the **Game Client Link** and **Editor Link**. Using these links you gain access to your Game and the Editor for it, the Game is public to anybody with an Identity Account. The Editor is only accessible by you as the Owner of the Game Platform, or an Infrastructure Administrator for management and feedback debugging.

<a href="/image/Posts/2021-01-24/Account_Page.png" 
    target="_blank"
    title="Shows the Account Details page and elements on it.">
    ![The Users Account Page.](/image/Posts/2021-01-24/Account_Page.png)
</a>
**The Users Account Page including links to the Game Client and Editor.**

## Game Client

The Game Client is a public facing game window to the platform provided to you, by giving out this URL you give users that are also part of the platform access to your game world. 

Some built in controls for movement are the WASD keys, for Forward/Left/Down/Right movement in the game world. As the Editor is built out you will be given more control over what a player can do in the world.

<a href="/image/Posts/2021-01-24/Game_Client.png" 
    target="_blank"
    title="Current UX of the Game Client for a Game Platform.">
    ![The Game Client Home Page.](/image/Posts/2021-01-24/Game_Client.png)
</a>
**The Game Client, the image is of an customized Map.**

## Editor 

The Editor is where a Platform Owner can change the different aspects of their game, the editor is locked to the owner for editing.  The Editor service currently has a Live, Entity and File editors, that can be used to change different aspects of the game from.

<a href="/image/Posts/2021-01-24/Editor_Home.png" 
    target="_blank"
    title="The Current Home page of Editor as a Logged in User.">
    ![The Game Client Home Page.](/image/Posts/2021-01-24/Editor_Home.png)
</a>
**The Editor home page when an authorized owner is logged in.**

### Live/Entity Editor

The Live Editor shows an in world representation of your games current state. The user will be logged into the game world and they can move around it as they edit the different settings/configuration of their game world.

Included is a list of Client Entities, a Client Entity is a static resource in the game world. Using the Entity Editor you can change the properties of the currently selected Client Entity. You can change the currently selected entity using the Entity List blase selected from the Blade selection box. 

<a href="/image/Posts/2021-01-24/Live_Editor.png" 
    target="_blank"
    title="The UI for the Live Editor with the Entity List and Editor Blades open.">
    ![The Game Client Home Page.](/image/Posts/2021-01-24/Live_Editor.png)
</a>
**Here is the UI for the Live Editor with the Entity List and Editor Blades open.**

### File Editor

The Editor currently supports Add/Edit/Delete of files on the Zone server, with a file editor that has light syntax highlighting of the files content.

<a href="/image/Posts/2021-01-24/File_Editor.png" 
    target="_blank"
    title="The UI for the File Editor including the open File Explorer Blade.">
    ![The Game Client Home Page.](/image/Posts/2021-01-24/File_Editor.png)
</a>
**Here is the UI for the File Editor including the File Explorer Blade open a few layers deep.**


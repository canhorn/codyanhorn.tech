---
layout: post
author: Cody Merritt Anhorn
title: Azure DevOps - External App Integration
categories: [blog]
tags: [.NET, C#, Azure, DevOps]
---

This article will give a hight level overview of the Azure DevOps Rest API, including Authentication all in a Blazor Application.

## Why

I have been getting into using the Azure DevOps Boards feature, this gives me an easy cloud solution to management my work. But it was starting to get a little tedious to add new Work Items, and I also had movements where I just wanted to quickly open up the application on my phone and input some features for further flushing out.

So I decided to see how hard it was for me to create a application that would allow for me to create new Work Items and list of some of the top ones. Well a few things had to be done, one was for authentication, how to structure the API Request, and how to host the application.

## Application Setup

The application has 3 main areas, the Authentication, Pages, and Service Requests. The authentication is OAuth, I found this the easiest to use, since I would not have to worry about refreshing tokens and it could be expanded to support more users. The other added benefit is that you can login and access any of your personal Boards, not just mine.

The Pages, or should I say page is the home page, this home page has a few components on it. The configuration component, these are settings specific to you and are saved in local storage of your browser. These settings will show up initially when they are not currently configured, and will disappear after they are all filled. 

The next component on the Page is a Work Item form, it has two fields; a Title input field, it will fill the "System.Title" field for your work Item. And the second field is for the Description, this will fill the "System.Description" field on the Work Item, with a submit button.

The last component is a list of the Work Item based on your configuration, it will display them in a table, using the service it is able to make the complex navigation of your Board to Work Item.

The last thing for the application setup is the AppSettings, this includes details about how the application should Authenticate. It includes setting for the Authentication, Token, and Profile URL, with Client configuration for the App Id, App Secret, Scope, and Callback. These settings setup the Authentication/Authorization of the application to the Azure DevOps APIs.

## Example Project

You can also checkout the source of the project here at <a href="https://github.com/canhorn/EventHorizon.DevOps.QuickBoard" target="_blank">canhorn/EventHorizon.DevOps.QuickBoard</a> Github repository. This should serve as a good base for getting started with an external Azure DevOps integration.

If you have any question hit me up on Twitter or check me out on Twitch, details below.
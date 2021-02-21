---
layout: post
author: Cody Merritt Anhorn
title: .NET - My Workflow Pattern
categories: [blog]
tags: [.NET, C#, MediatR, Hangfire]
---

This article will go over how I personally use MediatR, Hangfire, and my custom Observer libraries to create a Workflow Pattern for running background process or realtime if I choose.

## Details

When creating the <a href="https://" target="_blank">Game Development Platform</a> I created a pattern I could use to trigger a command, and get feedback on the command. The pattern, I call the Workflow Pattern, takes in a command, runs logic pertaining to that command and provides feedback to the initiator. 

The reason it is called a Workflow is because it these workflows are a collection of validation steps, making sure that the workflow can be processed. And a collection of commands/processing that needs to take place during the lifecycle of the workflow, encapsulating validation and processing into a single command workflow helps to group the logic for success or a rollback on failure. 

The workflows can be triggered from MediatR, and the framework will pickup the workflow passing it to Hangfire. Hangfire takes the process and queues it up for processing, when it is picked up by either the main or a background process it will run the command handler. At the end it can either tell the workflow that it is done and it will get propagated to the web/main application, that can then be displayed to the initiator.

<a href="/image/Posts/2021-02-21/Background-Workflow.png" 
    target="_blank"
    title="The Background Workflow, mapping out the path of a Create Platform User request.">
    ![Image of a Background Workflow](/image/Posts/2021-02-21/Background-Workflow.png)
</a>
**The Background Workflow, mapping out the path of a Create Platform User request.**

Another feature of the workflow is that they are just commands, so if I need an immediate response from the handler it can be ran out of thw workflow framework. The workflow has a process in place that will monitor events from Hangfire in the case the background job was not correctly handled, and display that state for the job. Each workflow gets a unique job id that also helps with tracking, and if a job needs to be reran or debugged it can be easily traced through the workflow. The notification is built on the back of the custom Observer patterns I built out to help with my Blazor components, by using interfaces and a registration process any process an listen for changes.

<a href="/image/Posts/2021-02-21/Blocking-Workflow.png" 
    target="_blank"
    title="The Blocking Workflow, mapping out the path of a User Registration request.">
    ![Image of a Blocking Workflow](/image/Posts/2021-02-21/Blocking-Workflow.png)
</a>
**The Blocking Workflow, mapping out the path of a User Registration request.**

The observer pattern I created allows for better integration into the async/await nature of MediatR. I can add an Observer Interface, then implement the Handler, Register the Component with the Observer framework, then any triggers of the observer interface will be propagated to all my observers. This allows for me easily create responsive user interfaces by allowing for workflows to be queued up in the background and the user can still navigate the site, getting notified when the workflow has finished processing.

## References

<a href="https://dotnet.microsoft.com/download" 
    target="_blank"
    title="The .NET 5 framework website.">
    .NET 5
</a> - The .NET 5 framework website.

<a href="https://github.com/jbogard/MediatR" 
    target="_blank"
    title="The MediatR Github project.">
    MediatR
</a> - The MediatR Github project.

<a href="https://www.hangfire.io/" 
    target="_blank"
    title="The Hangfire Background Processing Website.">
    HangFire
</a> - The Hangfire Background Processing Website.

<a href="https://www.nuget.org/packages/EventHorizon.Observer/" 
    target="_blank"
    title="The Observer Library I created to facilitate async request callbacks.">
    The EventHorizon Observer Library
</a> - The Observer Library I created to facilitate async request callbacks.

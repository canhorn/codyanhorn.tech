---
layout: post
author: Cody Merritt Anhorn
title: Blazor - WebAssembly Interop
categories: [blog, blazor]
tags: [.NET, C#, Blazor, WASM]
---

Since I really love any reason to use Blazor I though I would look into using it in my new Game Client. This article is a quick one going over my findings.

## Why

As part of my research to create a new Game Client, I wanted to see if I could reuse the work I have already done on my other services using C#. Since my Zone Server, this is the main game server, is using C#/.NET Core for all of its logic I want to see if I can use the already created libraries on the Client. My current client is build in mostly TypeScript, it works just fine but having the to not duplicate the logic on both the client and the server would make it easier to develop features in the future. This will make it so I can create create scripts for the client the same way I create scripts for my server, using CSX.

<a href="/image/Posts/Blazor/2020-06-18/Ran_Performance_Tests.png" 
    target="_blank"
    title="Sample of Test Runs.">
    ![Sample of Test Runs.](/image/Posts/Blazor/2020-06-18/Ran_Performance_Tests.png)
</a>
**Sample of Test Runs.**


# Example Project

As part of my research I created a Blazor WASM library, EventHorizon.Blazor.Interop, this contains a slim abstraction around the methods I will be using to interface with my rendering library, BabylonJS.

You can clone <a href="https://github.com/canhorn/EventHorizon.Blazor.Interop" target="_blank">EventHorizon.Blazor.Interop</a>, this GitHub project has a working sample project using the library in the myriad of tests I created, I will probably update this library with updates and more performance tests in the future.

# Use Libraries

Below are some libraries I got inspiration from  developing this for this library, it also includes a test or two for each.

[BlazorJsFastDataExchanger](https://github.com/Lupusa87/BlazorJsFastDataExchanger)

[MediatR](https://github.com/jbogard/MediatR)

[Mono.WebAssembly.Interop](https://www.nuget.org/packages/Mono.WebAssembly.Interop)
---
layout: post
author: Cody Merritt Anhorn
title: Blazor - The Big Three - Desktop/Mobile/Website
categories: [blog, blazor]
tags: [.NET, C#, Blazor, Desktop, Xamarin, WebAssembly]
---

I have been working with Blazor for a while now, since before Blazor Server went General Availability, and with that I have played around with a many different ways to create Applications. Blazor Server is currently the most stable of the technology today, so this article might be out of date in the future, so be warned. I will try and add new articles as the technology changes in the future.

Checkout the <a href="https://github.com/canhorn/EventHorizon.Blazor.Chat" target="_blank">Sample Project Here</a>.

## The Big Three

In this article I will go over creating a Desktop (Blazor WASM), a Mobile (Mobile Blazor Bindings), and a Website (Blazor Server) all with Blazor as the framework used to create the UX in some for or fashion. I wont go over the full details on how to get these up and running from start to finish, but just how Blazor was used in each application type.

The application I choose to show off these three application types is a Chat messaging application. By using SignalR as the service of choice the applications are able to be started up and interface with each other. The other underlying thing to point out is that all the applications inherit from the same model, that uses the .NET SignalR Client to connect up to the SignalR server.

This model is in another project, so all project can reference the project and use the same abstraction. Each application still needs to supply its own UX for the Model but using Blazor you can inherit/bind on the same simplified abstraction. The application just needs to setup the Dependency Injection for a settings object that the Initialization of the component can use to connect to the SignalR server.

## Desktop - Blazor WebAssembly

The Desktop application is using the <a href="https://github.com/SteveSandersonMS/WebWindow">WebWindow</a> project from Steve Sanderson. Using that Experimental library I was able to create a Blazor WebAssembly application that is built on cross platform desktop like technology. The UX is just a simple razor page, that inherits from the Model project stated earlier in this article.

## Mobile - Mobile Blazor Bindings

The Mobile application uses <a href="https://github.com/xamarin/MobileBlazorBindings">Mobile Blazor Bindings</a> to help abstract away the Xamarin.Forms elements into a more familiar ASP.NET Razor syntax. By using the Model project I am again able to use the same underlying logic to send and receive messages using SignalR, and just focus on the UX for the Mobile Application.

## Website - Blazor Server

Blazor Server is more mature of the other two application with being production ready per Microsoft, you can checkout more details about <a href="https://docs.microsoft.com/en-us/aspnet/core/blazor/?view=aspnetcore-3.1">ASP.NET Core Blazor here</a>. As state in the prior application type using the Model project I am able to just focus on the UX, which looks exactly the same as the WebAssembly version and uses the same model too. If I was going to take this further I would create a Web base Razor Class Library that both the Blazor Server and WebAssembly applications could use.

## Demo

***Desktop/Mobile/Website Cross Messaging Demo***
![Showing how a SignalR Server can be used to create a Chat application between three platform types.](/image/Posts/Blazor/2020-05-12/Desktop-Mobile-Website.gif)
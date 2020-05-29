---
layout: post
author: Cody Merritt Anhorn
title: Blazor Workshop - A Mobile Client Fork
categories: [blog, blazor]
tags: [.NET, C#, Blazor]
---

For the past week I took the time to fork the <a href="https://github.com/csharpfritz/blazor-workshop" target="_blank">csharpfritz/blazor-workshop</a> GitHub project and run through the sessions but creating them in the <a href="https://github.com/xamarin/MobileBlazorBindings" target="_blank">Experimental Mobile Blazor Bindings</a> project. The project is quite simple so it was a good project to build a Mobile client in. 

Checkout the fork here: <a href="https://github.com/canhorn/blazor-workshop/tree/canhorn/mobile" target="_blank">canhorn/blazor-workshop</a>

## My Impressions Sofar

The client was built in a way that focused on a client UX, so the UX was not exactly the same as the session. The features of selection of a Pizza, customizing with toppings, adding it to your order, and submitting that order as where the fork is currently at (End of Session #2).

All in all it was not that complicated to get the UX for the process in place, the development is a little bit clunky since it needs a service proxy, ngrok in my case, to make the API calls for pizzas,  toppings, and order submitting. The Picker Control was not provided by the binding library yet, so the project includes an example what the Picker control might look like implemented. The UI could have been replaced with something like a stacked view of checkboxes, or something similar to other online services. I though showing an example of a custom control might come in handy for others in the future.

I might be working on this more over the coming weeks, and will look at including write-ups as I work through more of the workshop sessions.

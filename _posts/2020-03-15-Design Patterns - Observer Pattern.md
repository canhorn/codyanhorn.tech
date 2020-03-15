---
layout: post
author: Cody Merritt Anhorn
title: Design Patterns - Observer Pattern
categories: [blog, design_pattern]
tags: [Design Pattern, Observer]
---

With my start into Blazor I found that using just a my normal patterns, they work great in the context of event driven designs, but I needed a way to keep tabs on state changes across my over arching application state. Enter the Observer Pattern, the observer pattern is actually a really straight forward pattern. You have something like the list of entities on a game server, and want to notify "observers" of changes to those entities.

## Problem Scenario

I am going to propose a problem and we are going to see how we can solve that problem with the Observer pattern. We want to share the state of a Count across browser sessions/instances, using Blazor Server as the backing of this example. So the Count will be the state we want to observe and when it changes we will notify observers of this change.

## Project Example

Below is an example Observer Pattern and the the necessary interop files.

Here is project:
<a href="https://github.com/canhorn/EventHorizon.Blazor.Pattern.Observer" target="_blank">EventHorizon.Blazor.Pattern.Observer</a>


***Index.razor***
~~~ html
@page "/"

<ViewCount />
<IncrementCountButton />
<ViewCount />
~~~

***IncrementCountButton.razor***
~~~ html
<div>
    <button @onclick="HandleIncrementCount">Increment Count</button>
</div>

@code {
    [Inject]
    public CountStateActions CountStateActions { get; set; }

    public async Task HandleIncrementCount()
    {
        await CountStateActions.Increment();
    }
}
~~~

***ViewCount.razor***
~~~ html

@Count

~~~

***ViewCount.razor.cs***
~~~ csharp
using EventHorizon.Blazor.Pattern.Observer.Count;
using EventHorizon.Blazor.Pattern.Observer.Count.Observers;
using EventHorizon.Observer.Register;
using System;
using MediatR;
using Microsoft.AspNetCore.Components;
using System.Threading.Tasks;
using EventHorizon.Observer.Unregister;

// The CountStateChangedObserver make this class an observer of the CountState
public class ViewCountModel : ComponentBase, IDisposable, CountStateChangedObserver
{
    [Inject]
    public IMediator Mediator { get; set; }
    [Inject]
    public CountState CountState { get; set; }

    protected int Count { get; private set; }

    protected override async Task OnInitializedAsync()
    {
        Count = CountState.Count;
        // Register this observer, so it can be triggered
        await Mediator.Send(
            new RegisterObserverCommand(this)
        );
    }

    // Typed Handler for CountState Changes
    public async Task Handle(
        CountStateChangedObserverArgs args
    )
    {
        Count = CountState.Count;

        await InvokeAsync(StateHasChanged);
    }

    public void Dispose()
    {
        // When this Component is disposed, no longer need to observer state changes.
        Mediator.Send(
            new UnregisterObserverCommand(this)
        ).ConfigureAwait(false).GetAwaiter().GetResult();
    }
}
~~~

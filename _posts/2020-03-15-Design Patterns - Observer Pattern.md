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
        CountStateActions.Increment();
    }
}
~~~

***ViewCount.razor***
~~~ html

<div>
    @Count
</div>

~~~

***ViewCount.razor.cs***
~~~ csharp

public ViewCountModel : ComponentBase, IDisposable, CountStateChangedObserver
{
    [Inject]
    public CountState CountState { get; set; }
    [Inject]
    public IMediator Mediator { get; set; }

    public int Count { get; set; }

    
    protected override async Task OnInitialized()
    {
        await Mediator.Send(
            new RegisterObserverCommand(this)
        );
        HandleCountChanged();
    }

    // This is will handle any changes to the CountState
    // What is nice about C# and its typing allows multiple Observers to be handle.
    // The Args parameter is what dictates which handler will get called.
    public Task Handle(
        CountStateChangedObserverArgs args
    )
    {
        HandleCountChanged();
        InvokeAsync(StateChange);
        return Task.Completed;
    }

    public void Dispose()
    {
        Mediator.Send(
            new UnregisterObserverCommand(
                this
            )
        ).ConfigureAwait(false).GetAwaiter().GetResult();
    }

    private void HandleZoneStateChange()
    {
        Count = CountState.Count;
    }
}
~~~

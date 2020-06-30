---
layout: post
author: Cody Merritt Anhorn
title: Let Us Create - Blazor - OBS Remote
categories: [blog, blazor, let_us_create]
tags: [.NET, C#, Blazor, CSS, HTML]
---

Do you use OBS and want a fancy Blazor Server website to change scenes? (Or control most any part of your OBS setup...) Follow this Post to learn how!

## Blazor

Blazor is great, their is not much it cant do, and that goes for connecting out to OBS and changing your scenes! In this article I will be going over creating a simple application, using the default template for Blazor Server. By the end of this post you should be able to control an instance of OBS from a hosted Blazor Server application. 

## OBS

OBS, Open Broadcaster Software, is a nice piece of software I personally use during my [Twitch Streams](https://twitch.tv/canhorn). This software is used to help manage the capture your games, or in my case programming activity, also giving the user/streamer the ability to change to different screen layouts for hight quality interaction with your viewers. 

## Pre Requisites

This setup will assume you already have OBS installed with the WebSocket plugin installed and .NET Core 3.1+, here is a few links you can use to get start.

**-** [OBS Website](https://obsproject.com/)

**-** [WebSockets API for OBS Studio.](https://github.com/Palakis/obs-websocket/releases/tag/4.8.0)

**-** [.NET Core](https://dotnet.microsoft.com/download/dotnet-core)

Install these per the defaults, if you have any problems hit me up at my [Twitter](https://twitter.com/CodyAnhorn), other social contact links below. 

## Creating Your Blazor Site

To get right to it I find it better to start with a template, this way you will have all the plumbing setup for you and you only have to focus on the UX.

~~~ bash
dotnet new blazorserver -o GenericCompany.Blazor.ObsRemote
~~~

## Add 'OBS.WebSocket.NET' NuGet Package

~~~ bash
dotnet add package OBS.WebSocket.NET --version 4.7.2
~~~

## Create the OBSRemotePage.razor

Create a new file <code>~/EventHorizon.Blazor.ObsRemote/Pages/OBSRemote.razor</code> in whatever way you prefer. 

You can use this content to get started:

~~~ html
@page "/obs-remote"

<h1>OBS Remote</h1>

<h2>Connect to OBS</h2>
<div class="form-group">
    <label for="obsUrl">OBS WebSocket URL</label>
    <input id="obsUrl" class="form-control" @bind-value="Url" placeholder="OBS WebSocket URL" />
</div>
<div class="form-group">
    <label for="obsPassword">OBS WebSocket Password (optional)</label>
    <input id="obsPassword" class="form-control" @bind-value="Password" placeholder="OBS WebSocket Password (optional)" />
</div>
<div>
    <button class="btn btn-outline-primary" @onclick="HandleStartConnection">Start Connection</button>
</div>

@code {
    public string Url { get; set; } = string.Empty;
    public string Password { get; set; } = string.Empty;

    public void HandleStartConnection()
    {
    }

    protected override void OnInitialized()
    {
    }
}
~~~

## Update NavMenu.razor

Lets add the OBS Remote Page navigation link to the <code>~/EventHorizon.Blazor.ObsRemote/Shared/NavMenu.razor</code>.

Add this snippet someplace in the <code>class="nav flex-column"</code> div:
~~~ html
<li class="nav-item px-3">
    <NavLink class="nav-link" href="obs-remote">
        <span class="oi oi oi-video" aria-hidden="true"></span> OBS Remote
    </NavLink>
</li>
~~~

Here is an example of what your <code>NavMenu.razor</code> can look like:
~~~ html
<div class="top-row pl-4 navbar navbar-dark">
    <a class="navbar-brand" href="">EventHorizon.Blazor.ObsRemote</a>
    <button class="navbar-toggler" @onclick="ToggleNavMenu">
        <span class="navbar-toggler-icon"></span>
    </button>
</div>

<div class="@NavMenuCssClass" @onclick="ToggleNavMenu">
    <ul class="nav flex-column">
        <li class="nav-item px-3">
            <NavLink class="nav-link" href="" Match="NavLinkMatch.All">
                <span class="oi oi-home" aria-hidden="true"></span> Home
            </NavLink>
        </li>
        <li class="nav-item px-3">
            <NavLink class="nav-link" href="counter">
                <span class="oi oi-plus" aria-hidden="true"></span> Counter
            </NavLink>
        </li>
        <li class="nav-item px-3">
            <NavLink class="nav-link" href="fetchdata">
                <span class="oi oi-list-rich" aria-hidden="true"></span> Fetch data
            </NavLink>
        </li>
        <li class="nav-item px-3">
            <NavLink class="nav-link" href="obs-remote">
                <span class="oi oi oi-video" aria-hidden="true"></span> OBS Remote
            </NavLink>
        </li>
    </ul>
</div>

@code {
    private bool collapseNavMenu = true;

    private string NavMenuCssClass => collapseNavMenu ? "collapse" : null;

    private void ToggleNavMenu()
    {
        collapseNavMenu = !collapseNavMenu;
    }
}
~~~

## Connect to OBS with ObsWebSocket

Here we will add a variable that will hold the OBS Connection, if the connection connected, and show feedback to the user when it is connected.

~~~ html
@page "/obs-remote"

<h1>OBS Remote</h1>

@if(!IsConnected) 
{
    <h2>Connect to OBS</h2>
    <div class="form-group">
        <label for="obsUrl">OBS WebSocket URL</label>
        <input id="obsUrl" class="form-control" @bind-value="Url" placeholder="OBS WebSocket URL" />
    </div>
    <div class="form-group">
        <label for="obsPassword">OBS WebSocket Password (optional)</label>
        <input id="obsPassword" class="form-control" @bind-value="Password" placeholder="OBS WebSocket Password (optional)" />
    </div>
    <div>
        <button class="btn btn-outline-primary" @onclick="HandleStartConnection">Start Connection</button>
    </div>
}
else 
{
    <div>Connected to OBS</div>
}

@code {
    public bool IsConnected { get; set; } = false;
    public string Url { get; set; } = string.Empty;
    public string Password { get; set; } = string.Empty;
    
    private OBS.WebSocket.NET.ObsWebSocket _obs;

    public void HandleStartConnection()
    {
        _obs.Connect(Url, Password);
    }

    protected override void OnInitialized()
    {
        _obs = new OBS.WebSocket.NET.ObsWebSocket();
        _obs.Connected += HandleConnected;
    }

    private void HandleConnected(
        object _,
        EventArgs __
    )
    {
        IsConnected = true;
    }
}
~~~

## Create a Display to see Your Scenes

Here we will create a simple table to show the scenes, if the scene is active, and a button to set the scene to active.

> Make sure OBS has at least two scenes.


~~~ html
@page "/obs-remote"

<h1>OBS Remote</h1>

@if (!IsConnected)
{
    <h2>Connect to OBS</h2>
    <div class="form-group">
        <label for="obsUrl">OBS WebSocket URL</label>
        <input id="obsUrl" class="form-control" @bind-value="Url" placeholder="OBS WebSocket URL" />
    </div>
    <div class="form-group">
        <label for="obsPassword">OBS WebSocket Password (optional)</label>
        <input id="obsPassword" class="form-control" @bind-value="Password" placeholder="OBS WebSocket Password (optional)" />
    </div>
    <div>
        <button class="btn btn-outline-primary" @onclick="HandleStartConnection">Start Connection</button>
    </div>
}
else
{
    <h2>Scenes</h2>
    <div>
        <button class="btn btn-outline-primary" @onclick="HandleGetScenes">Get Scenes</button>
    </div>
    @if (!string.IsNullOrEmpty(CurrentScene))
    {
        <div>Current Scene is "@CurrentScene"</div>
    }
    <hr />
    <div>
        <table class="table table-borderless">
            <tbody>
                @foreach (var scene in Scenes)
                {
                    <tr>
                        <th scope="col">
                            @scene.Name
                            @if (scene.Name == CurrentScene)
                            {
                                <span>(Current Scene)</span>
                            }
                            else
                            {
                                <button class="btn btn-outline-secondary"
                                        @onclick="() => HandleSetToCurrent(scene.Name)">
                                    Activate
                                </button>
                            }
                        </th>
                    </tr>
                }
            </tbody>
        </table>
    </div>
}

@code {
    public bool IsConnected { get; set; } = false;

    protected override void OnInitialized()
    {
        _obs = new OBS.WebSocket.NET.ObsWebSocket();
        _obs.Connected += HandleConnected;
        _obs.SceneChanged += HandleSceneChanged;
    }

    #region Scene Details
    public string CurrentScene { get; set; } = string.Empty;
    public IList<OBS.WebSocket.NET.Types.OBSScene> Scenes { get; set; } = new List<OBS.WebSocket.NET.Types.OBSScene>();

    public void HandleGetScenes()
    {
        var sceneDetails = _obs.Api.GetSceneList();
        CurrentScene = sceneDetails.CurrentScene;
        Scenes = sceneDetails.Scenes;
    }

    public void HandleSetToCurrent(
        string sceneName)
    {
        _obs.Api.SetCurrentScene(sceneName);
    }

    private void HandleSceneChanged(
        OBS.WebSocket.NET.ObsWebSocket _,
        string newSceneName
    )
    {
        CurrentScene = newSceneName;
        InvokeAsync(StateHasChanged);
    }
    #endregion

    #region Start Connection
    public string Url { get; set; } = "ws://localhost:4444";
    public string Password { get; set; } = string.Empty;

    private OBS.WebSocket.NET.ObsWebSocket _obs;

    public void HandleStartConnection()
    {
        _obs.Connect(Url, Password);
    }

    private void HandleConnected(
        object _,
        EventArgs __
    )
    {
        IsConnected = true;
    }
    #endregion
}
~~~

## Source Code
 
Here is a working example project: <a href="https://github.com/canhorn\EventHorizon.Blazor.ObsRemote" target="_blank">EventHorizon.Blazor.ObsRemote</a>

## Publish the Application

Do you want a self contained application? Well run this command and take the generated artifact folder where ever you want. From just a run of the exe the application will startup and then you can navigate a browser to <a href="http://localhost:5000" target="_blank">here</a> you can access the OBS Remote.

~~~ bash
dotnet publish -c Release -r win-x64 --self-contained true
~~~

You can find the published folder here:
<code>~\EventHorizon.Blazor.ObsRemote\bin\Release\netcoreapp3.1\win-x64\publish</code>
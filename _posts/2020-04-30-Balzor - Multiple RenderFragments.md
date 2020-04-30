---
layout: post
author: Cody Merritt Anhorn
title: Blazor - Multiple RenderFragments
categories: [blog, blazor]
tags: [.NET, C#, Blazor]
---

In Blazor using using a single RenderFragment will usually be used to dictate where the component's children will be displayed. But Blazor also allows for the usage of multiple RenderFragments, when starting out how to dictate this can be a little obscure, this article should help with that.

## How it works

It is actually pretty straight forward, you just include multiple RenderFragment properties in your component. Make sure to include the <code>[Parameter]</code> attributes, so it can be passed in from the parent components. The nuance of including multiple RenderFragments comes in the way you can pass them into the component, most ways have you pass in parameters through attributes of the Tag, with a RenderFragment you can include children Attributes/Parameters in the form of Tags. 

The Tag names correlate to the RenderFragment Field name the he component, these are also referenced as top-level items by Blazor. The Blazor framework is smart enough to know how a RenderFragment should be defined and can help you with errors during compile/usage of the Component.

## Ways to use Multiple RenderFragment

When creating a Modal having the user dictate the Header, Body, and Footer as distinct areas makes it easer to get the usage of the different areas at a glance. To also helps to name the areas in the component, by using the names of Label and Content you can get the general idea of where the content will be displayed. 

Another is the usage in Custom areas of a component, in a Text Editor component I was able to dictate an extension to the Menu Bar. RenderFragments help to make it easier to add extra functionality to a component, by giving an extension point to a component.

## Example 

You can checkout <a href="https://github.com/canhorn/EventHorizon.Blazor.MultipleRenderFragments" target="_blank">EventHorizon.Blazor.MultipleRenderFragments</a>, this GitHub project is a working example of the code below. 

***Index.razor***
~~~ html
<h1>Multiple Render Fragments</h1>

<button class="btn btn-primary" type="button" @onclick="OpenModal">
    Open
</button>

<DetailsArea Title="Game World Details" IsOpen="IsOpen">
    <Header>
        Game World Details
    </Header>
    <Body>
        <div style="font-weight: bold;">Players: </div>
        <div>1,000,000,000</div>
    </Body>
    <SubContent>
        <button class="btn btn-secondary" @onclick="CloseModal">Close</button>
    </SubContent>
</DetailsArea>


@code {
    protected bool IsOpen = false;

    protected void OpenModal()
    {
        IsOpen = true;
    }
    protected void CloseModal()
    {
        IsOpen = false;
    }
}
~~~

***DetailsArea.razor***
~~~ html
@if (!IsOpen) 
{
    <div>@Title (Is Closed)</div>
}
<div class="details-area" style="@(IsOpen ? "display: block;" : "display: none;")">
    <h1>
        @Header
    </h1>
    <div>
        @Body
    </div>
    <div>
        @SubContent
    </div>
</div>


@code {
    [Parameter]
    public bool IsOpen { get; set; }
    [Parameter]
    public string Title { get; set; }

    [Parameter]
    public RenderFragment Header { get; set; }
    [Parameter]
    public RenderFragment Body { get; set; }
    [Parameter]
    public RenderFragment SubContent { get; set; }
}
~~~
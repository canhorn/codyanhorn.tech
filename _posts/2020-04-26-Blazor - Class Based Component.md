---
layout: post
author: Cody Merritt Anhorn
title: Blazor - A Class/Code Only Based Component
categories: [blog, blazor]
tags: [.NET, C#, Blazor]
---

One powerful feature of Blazor is the ability to create Code-Only components. Using the <code>BuildRenderTree</code> method from the ComponentBase class you can, in code, create the output for the page.

## Ways to Create Components

We have a two main ways to create components, you can create a RenderFragment or in a ComponentBase use the <code>BuildRenderTree</code> method. Using the RenderFragment delegate method gives you a builder, that will be used to create the components, checkout the example below under Index.razor, this has an example where it creates a component. By using the <code>BuildRenderTree</code> method on the ComponentBase you can create the whole component from C#. Checkout the comments in the code below for more details.

Another aspect is that these components can also be used just like a razor file component. Below I have an example of using the code generated, and as using the component as a Tag on the page, both are completely valid Razor syntax.

## Example 

You can checkout <a href="https://github.com/canhorn/EventHorizon.Blazor.ManualRender" target="_blank">EventHorizon.Blazor.ManualRender</a>, this GitHub project is a working example of the code below. 

***Index.razor (RenderFragment Builder)***
~~~ html
<h1>Build a Component</h1>

<!-- Using the Code only Component as a Tag -->
<div>Displayed</div>
<div>
    <GameAgentDetails Name="Jeff" Role="Tree" Function="Being Pretty" />
</div>
<hr />
<button type="button" @onclick="RenderComponent">
    Create Game Agent
</button>
<div>Built in Code</div>
<!-- Using the Code generated Component -->
<div>
    @CustomRender
</div>
@code {
    private RenderFragment CustomRender { get; set; }
    
    // This is our "dynamically" created component.
    private RenderFragment CreateComponent() => builder =>
    {
        // This Component is built right in Code
        builder.OpenComponent(0, typeof(GameAgentDetails));
        // Here we add some attributes to the opened GameAgentDetails Component
        builder.AddAttribute(1, "Name", "Steve");
        builder.AddAttribute(2, "Role", "NPC");
        builder.AddAttribute(3, "Function", "Being Awesome");
        builder.CloseComponent();
    };    
    
    private void RenderComponent()
    {
        CustomRender = CreateComponent();
    }
}
~~~

***GameAgentDetails.cs***
~~~ csharp
using Microsoft.AspNetCore.Components;
using Microsoft.AspNetCore.Components.Rendering;

/// This Component is completely generated in C#, without the use of any .razor file.
public class GameAgentDetails : ComponentBase
{
    [Parameter]
    public string Name { get; set; }
    [Parameter]
    public string Role { get; set; }
    [Parameter]
    public string Function { get; set; }

    // This is the BuildRenderTree, we can use this to build from scratch the HTML that will be placed on the page.
    protected override void BuildRenderTree(RenderTreeBuilder builder)
    {
        var seq = 0;
        // We open a div
        builder.OpenElement(seq, "div");
        // Adding a arbitrary class attribute with a custom value.
        builder.AddAttribute(++seq, "class", "agent-details__name");
        // Then add some content to that the div, in this case the passed in Name parameter.
        builder.AddContent(++seq, Name);
        // We then close the currently open element.
        builder.CloseElement();
        
        builder.OpenElement(++seq, "div");
        builder.AddAttribute(++seq, "class", "agent-details__role");
        builder.AddContent(++seq, Role);
        builder.CloseElement();

        builder.OpenElement(++seq, "div");
        builder.AddAttribute(++seq, "class", "agent-details__function");
        builder.AddContent(++seq, Function);
        builder.CloseElement();
    }
}
~~~
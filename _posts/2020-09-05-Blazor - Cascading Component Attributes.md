---
layout: post
author: Cody Merritt Anhorn
title: Blazor - Cascading Component Attributes
categories: [blog, blazor]
tags: [.NET Core, C#]
---

When using Blazor you will sometimes need to give a Button some of your own flare, with that comes the problem of mapping all the attributes to another tag from your component. Below I show an out of the box way to cascade the attributes from your wrapper component to a tag, in this case a button.

## Example

Checkout a live Demo here <a href="https://lively-mud-0597d4e10.azurestaticapps.net/" target="_blank">EventHorizon.Blazor.UX</a>.

You can also checkout the source of the whole site in the <a href="https://github.com/canhorn/EventHorizon.Blazor.UX" target="_blank">canhorn/EventHorizon.Blazor.UX</a> Github repository. 

**Button.razor**<br />
<span style="font-size: 0.8rem">
    A cascaded attributes example component. Show how you can expand on the class with your own and any that are also passed in.
</span>
~~~ html
<button class="my-awesome-button @@class" @attributes="Attributes">@ChildContent</button>

@code {
    [Parameter(CaptureUnmatchedValues = true)]
    public IDictionary<string, object> Attributes { get; set; }
    [Parameter]
    public string @class { get; set; }
    [Parameter]
    public RenderFragment ChildContent { get; set; }
}
~~~

**Example Usage of Button**
~~~ html
<Button class="--font-size-huge" disabled>Huge Disabled Button</Button>
~~~

**Generated HTML**
~~~ html
<button class="my-awesome-button --font-size-huge" disabled>Huge Disabled Button</Button>

~~~
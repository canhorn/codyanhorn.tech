---
layout: post
author: Cody Merritt Anhorn
title: Blazor - Auto-Reload on Lost Connection
categories: [blog, blazor]
tags: [.NET, C#, Blazor]
---

When building out my bot I wanted to be able to create 

## Where to Start

I use the base Blazor application when starting out layer in the SCSS, is use SCSS because it helps me to structure my files together. The component and the css styles will be next to each other, and if your using Visual Studio 2019, it will group these files. VS2019 is my preferred IDE for Blazor as it currently has better support than other IDEs making it easier to be product with a Blazor application.

~~~
@inherits ZoneStateProviderModel

@if (State.IsLoading)
{
    <div>@Localizer["Loading..."]</div>
}
else if (!State.Zone.IsValid() || State.ZoneInfo == null)
{
    <div>@Localizer["Zone Invalid."]</div>
    <NavLink href="/zone-admin">@Localizer["Back to Zone Admin"]</NavLink>
}
else
{
    <CascadingValue Value="@this.State">
        @ChildContent
    </CascadingValue>
}
~~~
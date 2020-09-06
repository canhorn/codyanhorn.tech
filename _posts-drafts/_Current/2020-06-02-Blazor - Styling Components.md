---
layout: post
author: Cody Merritt Anhorn
title: Blazor - Styling Components
categories: [blog, blazor]
tags: [.NET, C#, Blazor]
---

Even when building an application with Blazor you still need a way to style the components, my personal preference is to use SCSS. It does add an extra build step to my application's build process, but having a clear separation of page structure and the style helps me to develop quickly.

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
---
layout: post
author: Cody Merritt Anhorn
title: Display a Docker Build Version
categories: [blog]
tags: [Docker, .NET, Blazor, C#]
---

In this Article I will show one way, of many out there, to show the Build or Application version passed down through a Docker build.

## Details

While developing my applications for the Game Development Platform I wanted an easy to access the build version. Since all of my service applications use Docker as their main artifact deployment pattern, passing in a build-arg and setting an ENV var for the image. This allows for a variety of ways to access the Application Version, and make all of my Dockerfile's have the same structure for each application build.

I have included a sample application, using Blazor as the main hosting framework, that when built and packaged up will show the version that was passed in during the build.


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



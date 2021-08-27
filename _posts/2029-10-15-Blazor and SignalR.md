---
layout: post
author: Cody Merritt Anhorn
title: Blazor and SignalR
categories: [blog, blazor]
tags: [.NET, Blazor, C#, SignalR]
---

In this Article I go over how to connect out to a SignalR Server from Blazor Wasm, sneak peak its the same as any other SignalR client. 


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



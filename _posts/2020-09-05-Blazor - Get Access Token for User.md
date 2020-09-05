---
layout: post
author: Cody Merritt Anhorn
title: Blazor - Get Access Token for User 
categories: [blog, blazor]
tags: [HTML, CSS, JavaScript]
---

In this article I show off a quick snippet to get the AccessToken for a logged in User. Not much to it just using the <code>IAccessTokenProvider</code>, and if the user is signed in and they have are using an authentication type that provides an access token, like OpenID.

## Get Access Token from IAccessTokenProvider

**Index.razor**
~~~ html
@page "/"

<div>
    @AccessToken
</div>

~~~

**Index.razor.cs**
~~~ csharp

using System.Threading.Tasks;
using Microsoft.AspNetCore.Authorization;
using Microsoft.AspNetCore.Components;
using Microsoft.AspNetCore.Components.WebAssembly.Authentication;

[Authorize]
public partial class Index : ComponentBase
{
    [Inject]
    IAccessTokenProvider TokenProvider { get; set; }

    public string AccessToken { get; set; }

    public override async Task OnInitializedAsync()
    {
        var accessTokenResult = await TokenProvider.RequestAccessToken();
        AccessToken = string.Empty;

        if (accessTokenResult.TryGetToken(out var token))
        {
            AccessToken = token.Value;
        }
    }
}
~~~
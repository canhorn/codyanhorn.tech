---
layout: post
author: Cody Merritt Anhorn
title: Blazor - Custom Blazor Error Display
categories: [blog, blazor]
tags: [.NET, C#, Blazor, CSS, HTML]
---

Blazor has a few built in content areas, mainly for errors or the such, but with most things they should be customized. And this article will go over the little box at the bottom of the screen that shows when the server throws an exception.

## Where to Start

To start we need to include some HTML in the _Host.cshtml or index.html page, this is the content that is displayed when Blazor picks up an error. With the display HTML you will want to include some CSS to hide it, until an error is thrown, and style/brand it to your liking. These two pieces allow for a uniquely customized experience to be supplied by the developer, and give more control over the message that is shown.

The provided code used to get up and running with a Blazor application, both Server and WASM templates, have this provided by the by default. So most of the work for creating a custom error display is down to editing the provided CSS. When using Blazor Server you have access to the <code>&lt;environment /&gt;</code> tag gaining access to create some very unique content.

For more details checkout the documentation <a href="https://docs.microsoft.com/en-us/aspnet/core/blazor/handle-errors" target="_blank">Microsoft - Handle errors</a>.

**_Host.cshtml Example**
~~~ html
@page "/"
@namespace EventHorizon.Blazor.Chat.Website.Pages
@addTagHelper *, Microsoft.AspNetCore.Mvc.TagHelpers
@{
    Layout = null;
}

<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Blazor Website</title>
    <base href="~/" />
    <link rel="stylesheet" href="css/bootstrap/bootstrap.min.css" />
    <link href="css/site.css" rel="stylesheet" />
</head>
<body>
    <app>
        <component type="typeof(App)" render-mode="ServerPrerendered" />
    </app>

    <div id="blazor-error-ui">
        <environment include="Staging,Production">
            An error has occurred. This application may no longer respond until reloaded.
        </environment>
        <environment include="Development">
            An unhandled exception has occurred. See browser dev tools for details.
        </environment>
        <a href="" class="reload">Reload</a>
        <a class="dismiss">🗙</a>
    </div>

    <script src="_framework/blazor.server.js"></script>
</body>
</html>
~~~

**site.css**
~~~ css
#blazor-error-ui {
  background: lightyellow;
  bottom: 0;
  box-shadow: 0 -1px 2px rgba(0, 0, 0, 0.2);
  display: none;
  left: 0;
  padding: 0.6rem 1.25rem 0.7rem 1.25rem;
  position: fixed;
  width: 100%;
  z-index: 1000;
}

#blazor-error-ui .dismiss {
  cursor: pointer;
  position: absolute;
  right: 0.75rem;
  top: 0.5rem;
}
~~~
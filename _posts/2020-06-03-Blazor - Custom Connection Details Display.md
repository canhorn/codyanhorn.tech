---
layout: post
author: Cody Merritt Anhorn
title: Blazor - Custom Connection Details Display
categories: [blog, blazor]
tags: [.NET, C#, Blazor, CSS, HTML]
---

Blazor Server has a standard UX that display the reconnection status of the application, in this article I will show you one way to customize this UX.

# Why Customize the Connection Status Display

Most of the time it is just fine to use the supplied UX for this page, but in my usage I wanted a little more control on where and how the content is displayed. In my application, a Bot build in blazor, has some overlays it supplied my OBS in the form of webpages. With these page supplied by my bot I wanted to display an out of the way message, not completely cover my whole overlay but still get details on when the Client looses the connection to the Bot/Website. So by including the HTML and CSS I was able to customize the messaging and how the UX is displayed when their is a connection loss to my bot.

<a href="/image/Posts/2020-06-03/Reconnecting_Blazor_Message.png" 
    target="_blank"
    title="Customized components-reconnect-modal used by my Bot.">
    ![A picture of the current players Caught Immortals GUI.](/image/Posts/2020-06-03/Reconnecting_Blazor_Message.png)
</a>
**Customized "components-reconnect-modal" used by my Bot. The "Reconnecting to server..." is the area these customizations were made.**

## Where to Start

To start we need to include some HTML in the _Host.cshtml page, this is only supported on the Blazor Server hosting model, since the WASM host does not have a connection to a Server. With the displayed HTML you will want to include some CSS to hide it, the framework will take care of showing the content when a connection state change is picked up. These two pieces allow for a customized UX to be supplied give more control over the where, how and what message is shown.

This UX does not come from the default template so will need to be supplied completely by the developer when customizing. The areas to note is the **id**, it has to be **components-reconnect-modal**, this needs to be supplied so the frameworks knows that customization is provided and should not include its own. 

When the different reconnection states are triggered the framework will change the css class on the **components-reconnect-modal** element to one of the following; **components-reconnect-show**, **components-reconnect-hide**, **components-reconnect-failed**, or **components-reconnect-rejected**, checkout <a href="https://docs.microsoft.com/en-us/aspnet/core/blazor/hosting-model-configuration#blazor-server" target="_blank">Microsoft - Blazor Server</a> for more details on each one.

As with the Error UI, when using Blazor Server you have access to the <code>&lt;environment /&gt;</code> tag gaining access to create some very unique content.

**_Host.cshtml**
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

    <environment include="Staging,Production">
        <div id="components-reconnect-modal" class="my-reconnect-modal components-reconnect-hide">
            <div class="show failed rejected">
                <p>
                    Reconnecting to server...
                </p>
            </div>
        </div>
    </environment>
    <environment include="Development">
        <div id="components-reconnect-modal" class="my-reconnect-modal components-reconnect-hide">
            <div class="show">
                <p>
                    This is the message when attempting to connect to server
                </p>
            </div>
            <div class="failed">
                <p>
                    This is the custom message when failing
                </p>
            </div>
            <div class="rejected">
                <p>
                    This is the custom message when rejected
                </p>
            </div>
        </div>
    </environment>

    <script src="_framework/blazor.server.js"></script>
</body>
</html>

~~~

**site.css**
~~~ css
.my-reconnect-modal > div{
    position: fixed;
    top: 0;
    right: 0;
    bottom: 0;
    left: 0;
    z-index: 1000;
    overflow: hidden;
    background-color: #fff;
    opacity: 0.8;
    text-align: center;
    font-weight: bold;
}

.components-reconnect-hide > div {
    display: none;
}

.components-reconnect-show > div {
    display: none;
}

.components-reconnect-show > .show {
    display: block;
}

.components-reconnect-failed > div {
    display: none;
}

.components-reconnect-failed > .failed {
    display: block;
}

.components-reconnect-rejected > div {
    display: none;
}

.components-reconnect-rejected > .rejected {
    display: block;
}
~~~
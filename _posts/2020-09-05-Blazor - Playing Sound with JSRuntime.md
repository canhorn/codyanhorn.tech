---
layout: post
author: Cody Merritt Anhorn
title: Blazor - Playing Sound with JSRuntime
categories: [blog, blazor]
tags: [.NET Core, C#]
---

With this article is just show off how to play an audio sound from Blazor, it's not much different that using JavaScript, well it is just using JavaScript. But since Blazor does not have a built-in abstraction over the Audio JavaScript API we have to use IJSRuntime to call into it. Their are other libraries that can do this for you, abstracting away the JavaScript, but in some cases you just need an simple example of calling a block of code. Also this article is just to show off more of my Blazor Component Library. 😁

## Example

Checkout a live Demo here <a href="https://lively-mud-0597d4e10.azurestaticapps.net/" target="_blank">EventHorizon.Blazor.UX</a>.

You can also checkout the source of the whole site in the <a href="https://github.com/canhorn/EventHorizon.Blazor.UX" target="_blank">canhorn/EventHorizon.Blazor.UX</a> Github repository. 

**A Blazor Button Component, with sound triggered from JavaScript**
~~~ html
<button @onclick="HandleOnClick">@ChildContent</button>

@code {
    [Inject]
    public IJSRuntime JSRuntime { get; set; }

    private string SoundSrc = "/_content/EventHorizon.Blazor.UX.Controls/sounds/click.mp3";

    private async Task HandleOnClick(MouseEventArgs args)
    {
        await JSRuntime.InvokeVoidAsync(
            "playAudioFile",
            SoundSrc
        );
    }
}
~~~

**JavaScript**<br />
<span style="font-size: 0.8rem">
    Called by JSRuntime call from Blazor button click
</span>
~~~ javascript
window.playAudioFile = (src) => {
    new Audio(src).play();
};
~~~
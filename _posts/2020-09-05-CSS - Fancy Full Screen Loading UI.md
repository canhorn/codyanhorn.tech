---
layout: post
author: Cody Merritt Anhorn
title: CSS - Fancy Full Screen Loading UI
categories: [blog]
tags: [HTML, CSS, JavaScript]
---

This article I am just showing off a simple full screen UI I am playing around with, the main reason to have a Full Screen Loading UI is to hide loading details about your application. In my case I am hosting a Wasm application, and since it requires a hearty load of assets I wanted a Fancy loading screen to show the user when they first hit the site. Below is the HTML, CSS and JavaScript I use to show and manage the UI.

By default it will show full screen, not requiring any external files.

## Example

Checkout a live Demo here <a href="https://lively-mud-0597d4e10.azurestaticapps.net/" target="_blank">EventHorizon.Blazor.UX</a>.

You can also checkout the source of the whole site in the <a href="https://github.com/canhorn/EventHorizon.Blazor.UX" target="_blank">canhorn/EventHorizon.Blazor.UX</a> Github repository. 

**HTML for the Loading UI, place anywhere after your main content.**
~~~ html
<div id="loading-overlay" style="color: #26dafd;
    position: absolute;
    top: 0;
    bottom: 0px;
    width: 100%;
    text-align: center;
    background-color: black;
    background-image: url(/image/background.png);
    z-index: 9999;
">
    <div style="display: grid; height: 100%; text-align: center; vertical-align: middle;">
        <h1 style="display: flex; flex-direction: column; place-content: flex-end;">EventHorizon Blazor.UX</h1>
        <p>The application is loading...</p>
    </div>
</div>

<style>
/* This will make your loading-overlay UI transition out all fancy like */
.hide-loading {
    visibility: hidden;
    opacity: 0;
    transition: visibility 1s, opacity 1s linear;
}
</style>

<script>
// Call window.hideLoading() anytime you want after your content is loaded
window.hideLoading = () => {
    document.getElementById('loading-overlay').classList.add("hide-loading");
};
</script>
~~~